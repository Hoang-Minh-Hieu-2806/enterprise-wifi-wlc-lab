# Final Implementation (2026-11-07)

> **Tài liệu chính thức của bản deliverable cuối cùng: `packet-tracer/final.pkt`**
> (SHA-256 `5AA9ADCA…B5CEF`, 100.297 bytes; mở và kiểm thử trên Cisco Packet Tracer **9.0.1.0858**).
> Toàn bộ trạng thái dưới đây được đọc trực tiếp từ mô phỏng PT (API + CLI + ping thật) trong chuỗi audit
> 2026-11-05 → 2026-11-07, không suy diễn từ kế hoạch. Các tài liệu v1 cũ bên dưới repo được đánh dấu là
> đã được thay thế bởi tài liệu này.
>
> **Lịch sử ngắn:** khung v1 (9 thiết bị, `WLC-PT`, `v1.pkt`, SW1/SRV1 router-on-a-stick) được mô tả tại
> [topology.md](topology.md), [ip-vlan-plan.md](ip-vlan-plan.md)… — những đoạn "v1" trong đó là **lịch sử giai đoạn P1**.
> Từ 2026-11-01 nhóm chuyển sang kiến trúc enterprise (ghi chú trong
> [implementation-notes.md](implementation-notes.md)) và chốt bản `final.pkt` ngày 2026-11-07.

## 0. Tổng quan kiến trúc

Thiết kế tổng thể:

- **Cấp ISP:** Router-PT `ISP` mô hình hoá một ISP thật: hai path /30 riêng cho hai edge router, phát default route cho R1/R2, một Loopback `8.8.8.8/32` làm "Internet" đo được.
- **Edge (HA theo thiết kế):** `R1` (2901) và `R2` (2901) nối song song ISP↔CORE-SW, mỗi router PAT overload, static route vế về LAN. R1 là đường dùng chính (default của CORE-SW AD 1), R2 dự phòng qua static AD 254 trên CORE-SW.
- **Aggregation/L3:** `CORE-SW` (3560-24PS) `ip routing` + SVI gateway 10/20/99 — inter-VLAN không cần router-on-a-stick.
- **Access:** `ACCESS-SW` (2960) trunk Fa0/1 → CORE-SW; WLC1 và 2 AP nối vào trunk, client traffic được
  map về các VLAN tương ứng qua WLC.
- **Wireless:** `WLC1` (WLC-3504) quản trị qua VLAN 99, tạo VLAN dynamic interface cho STAFF/GUEST và 2 WLAN; `LAP-STAFF` / `LAP-GUEST` (LAP-PT) phát `STAFF-WIFI` (WPA2-Enterprise/RADIUS) và `GUEST-WIFI` (WPA2-PSK).
- **Server zone (VLAN 99):** `DHCP-SRV` (`.20`, 2 pool + relay từ CORE-SW), `RADIUS-SRV` (`.22`, AAA 1645), `Web-SRV` (`.21`), `ADMIN-PC` (`.10`).

## 1. Topology & inventory (18 thiết bị, 21 link)

| Nhóm | Thiết bị | Model | Vai trò |
| --- | --- | --- | --- |
| WAN | ISP | Router-PT | 2 path /30 tới R1/R2; Lo0 `8.8.8.8/32` |
| WAN | R1, R2 | 2901 | Edge + PAT overload; static routes về 10/20/99 |
| L2/L3 | CORE-SW | 3560-24PS | L3 core: SVI 10/20/99, DHCP relay, ACL, default route |
| L2 | ACCESS-SW | 2960-24TT | Access cho WLC/AP |
| Wireless | WLC1 | WLC-3504 | Controller; mgmt `.99.10`; 2 dynamically tạo interface |
| Wireless | LAP-STAFF, LAP-GUEST | LAP-PT | AP bằng CAPWAP join WLC1 |
| Server | DHCP-SRV | Server-PT | DHCP 2 pool (`.20`) + NAT helper đích; RADIUS AAA (`.22`); HTTP (`.21`) |
| Client | S-LAP1/2/3 | Laptop-PT | Staff, wireless 802.1X |
| Client | G-LAP1/2, G-PHONE | Laptop-PT / Smartphone -PT | Guest, WPA2-PSK |
| Mgmt | ADMIN-PC | PC-PT | Quản trị VLAN 99 |

Minh chứng canvas: [`../evidence/Final.jpg`](../evidence/Final.jpg), [`../evidence/Final 2.png`](../evidence/Final%202.png).

## 2. Addressing plan (đóng băng)

### 2.1 Điểm nối routed (p2p)

| Link | Subnet /30 | Cổng |
| --- | --- | --- |
| ISP ↔ R1 | `203.0.113.0/30` (`203.0.113.1` ↔ `.2`) | ISP Gi0/0 ↔ R1 Gi0/0 |
| ISP ↔ R2 | `203.0.113.4/30` (`203.0.113.5` ↔ `.6`) | ISP Gi0/0 ↔ R2 Gi0/0 |
| R1 ↔ CORE-SW | `10.0.0.0/30` (`10.0.0.1` ↔ `.2`) | R1 Gi0/1 ↔ CORE-SW Gi0/1 (no switchport) |
| R2 ↔ CORE-SW | `10.0.1.0/30` (`10.0.1.1` ↔ `.2`) | R2 Gi0/1 ↔ CORE-SW Gi0/2 (no switchport) |

`ISP` Loopback0 `8.8.8.8/32`; R1/R2 default route chỉ về ISP (`203.0.113.1` / `.5`).

### 2.2 VLAN + SVI

| VLAN | Tên | Subnet | Gateway (CORE-SW SVI) | Ghi chú |
| --- | --- | --- | --- | --- |
| 10 | STAFF | `192.168.10.0/24` | `.1` (+ `ip helper-address 192.168.99.20`) | `Vlan10` SVI MAC `00d0.5877.bd01` |
| 20 | GUEST | `192.168.20.0/24` | `.1` (+ helper + **ACL `GUEST-ACL` in**) | SVI MAC `…bd02` |
| 99 | MGMT | `192.168.99.0/24` | `.1` | WLC `.10`, DHCP `.20`, Web `.21`, RADIUS `.22`, ADMIN-PC `.10` |

Định tuyến trên `CORE-SW`: `ip route 0.0.0.0 0.0.0.0 10.0.0.1` (AD 1) + `ip route 0.0.0.0 0.0.0.0 10.0.1.1 254` (backup qua R2). Trên R1/R2: static route ve LAN `192.168.10/20/99` → next-hop CORE-SW; default → ISP.

### 2.3 NAT/PAT

`R1`/`R2` dùng chung cơ chế PAT overload ra interface WAN:

```
ip nat inside source list 1 interface GigabitEthernet0/0 overload
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
access-list 1 permit 192.168.99.0 0.0.0.255
(Gi0/0 = ip nat outside, Gi0/1 = ip nat inside)
```

Kiểm chứng: cờ NAT từng bị nghi ngờ rớt sau một chuỗi thao tác — đã đối chiếu lại Running config + startup-config (2026-11-07) — **cấu hình NAT giữ nguyên vẹn** (xem `configs/router/router-r1-running-config.txt` và `router-r2-running-config.txt`).

## 3. Wireless (WLC1 3504 + 2 AP)

- **WLC1 (WLC-3504):** Management `192.168.99.10` (interface `management`); 2 dynamic interface: **staff** (`192.168.10.2`) và **guest** (`192.168.20.2`); **Primary DHCP Server = 192.168.99.10 (chính management của nó)** cho cả hai interface — các pool nội bộ của WLC: STAFF-LOCAL bắt đầu `.10.101`, GUEST-LOCAL bắt đầu `.20.141`.
- **WLAN:**
  - `STAFF-WIFI` VLAN 10, WPA2 + **Auth 802.1X (RADIUS)**, AAA server `.99.22:1645` (`key <RADIUS-SECRET-REDACTED>`, client name `WLC1`).
  - `GUEST-WIFI` VLAN 20, WPA2-PSK.
- **AP:** LAP-STAFF và LAP-GUEST join WLC1 bằng cơ chế auto-discovery của PT (`Status: Online`; bắt tay CAPWAP chưa có minh chứng — xem limitations).
- **Client associate thật (leases final):** S-LAP1 `.10.104`, S-LAP2 `.10.103`, S-LAP3 `.10.106`, G-LAP1 `.20.142`, G-LAP2 `.20.144`, G-PHONE `.20.141`. Tất cả gateway ping 4/4 — chứng minh map đúng VLAN + DHCP.
- **Nếu sau khi mở file association rớt (PT runtime):** power-cycle AP rồi client, ở client lại đặt DHCP mode sau khi đã `0.0.0.0`, đợi 10–20 s, chạy `/renew`.

> **Giới hạn:** bắt tay **802.1X RADIUS end-to-end chưa có bằng chứng** quá trình exchange giữa client↔AP↔WLC↔server (PT không log được EAP trên client); cấu hình ở cả hai phía WLC1 + RADIUS-SRV đã được kiểm soát khớp chuẩn (port 1645 khớp cả 2 đầu) nhưng phải xử lý là "đã cấu hình, chưa chứng minh exchange" (TC-13).

## 4. DHCP (Server-PT + relay)

- `DHCP-SRV` (`.99.20`), pool nội bộ cho Mgmt + 2 pool chính: **STAFF-POOL** `192.168.10.101 → n`, gateway `.10.1`; **GUEST-POOL** `192.168.20.141 → n`, gateway `.20.1`. Service DHCP bật On.
- `CORE-SW` đặt `ip helper-address 192.168.99.20` trên Vlan10 và Vlan20 — CORE-SW chỉ làm **DHCP relay**, không giữ pool.
- PT hạn chế: `serverPool` factory vẫn tồn tại (limit 1 user, tối ưu range thấp — ghi nhận limitation).
- Lease **không được giữ sau khi reopen file** — client phải `/renew` (độc tính đã biết của PT).

## 5. RADIUS/AAA (RADIUS-SRV .99.22)

- Service **AAA** On, port **1645** (khớp đúng WLC1 `Authentication Port 1645`).
- Client network entry: name `WLC1`, IP `192.168.99.10`, shared secret `<RADIUS-SECRET-REDACTED>` (**secret nằm dạng rõ trong file .pkt; không commit giá trị thật vào bất kỳ file tài liệu/config nào**)
- User: `staff1`, `staff2`, `staff3` (dùng cho đăng nhập 802.1X của client Staff).
- WPA2-Enterprise: `STAFF-WIFI` — WPA2, WPA2 Policy AES, Auth 802.1X, `.99.22:1645`.

## 6. Security — ACL GUEST

ACL named `GUEST-ACL` (**không dùng số** — giữ tên dễ đọc) trên `CORE-SW Vlan20 in`:

```
ip access-list extended GUEST-ACL
 permit udp any any eq bootps        ! DHCP discover từ Guest phải ra tới relay/server
 deny   ip any 192.168.10.0 0.0.0.255 ! chặn Guest → Staff
 deny   ip any 192.168.99.0 0.0.0.255 ! chặn Guest → Mgmt (bao gồm DHCP-SRV nhưng bootps vẫn permit trước)
 permit ip any any                    ! cho phép Guest đi Internet
```

Kiểm chứng chức năng final audit: G-LAP1 → `192.168.10.1` **0/4 reply**; G-LAP1 → `192.168.99.22` **0/4 reply**; đồng thời DHCP Discover vẫn được permit (client có lease).

⚠ **Lỗi hiển thị PT:** lệnh `show ip interface Vlan20` trong bản 9.0.1.0858 vẫn ghi `access-group … not set` cho ACL named — đây là lỗi hiển thị của PT (binding có thật trong startup-config, mở Running Config/`show access-lists` là thấy rõ `ip access-group GUEST-ACL in`). Hãy xem Running config thay vì `show ip interface`.

## 7. Ma trận kiểm thử final (map các TC legacy → kết quả)

| TC | Nội dung hiện trường | Kết quả **final** |
| --- | --- | --- |
| TC-09 | Client tới gateway | **Passed** — 6/6 client ping gateway 4/4 |
| TC-10 | Guest bị chặn tới Staff | **Passed** — G-LAP1 → `.10.1` 0/4; (G-LAP2/G-PHONE đã associate và có lease; ping matrix riêng của G-LAP2/G-PHONE *chưa chạy riêng* — ghi nhận limitation) |
| TC-11 | Guest bị chặn tới Mgmt, DHCP vẫn đạt | **Passed** — G-LAP1 → `.99.22` 0/4; lease `.20.142` cho thấy DHCP permit vẫn hoạt động |
| TC-12 | VLAN/plumbing đúng + `show run` ACL hiển thị đúng | **Passed at config level** — ACL nằm trong startup/running; screenshot `show vlan brief` riêng **chưa có** (nội dung config thực nằm trong `configs/switch/core-sw-running-config.txt`) |
| TC-13 | RADIUS/WPA2-Enterprise Staff | **Configured, end-to-end exchange chưa chụp** — AAA 2 vendor khớp port 1645, user + client entry đầy đủ; client-side EAP exchange chưa có bằng chứng |
| TC-14 | Sai credential bị từ chối | **Not Tested** — không có lần thử PSK sai/tài khoản sai được ghi lại |

Kết quả end-to-end đã chạy thật (audit 2026-11-05 → 2026-11-07, ping thật qua PT bridge): S-LAP1/2/3 → `8.8.8.8` 4/4; G-LAP1 → `8.8.8.8` 4/4 (đạt sau một lần retry); ADMIN-PC → RADIUS-SRV 4/4; tất cả client → gateway 4/4.

## 8. Limitations (ghi nguyên trong báo cáo)

1. **Failover chưa được chứng minh end-to-end:** cấu hình backup route AD 254 đã đúng trên CORE-SW; chưa chạy bài test power-off R1 và confirm traffic chuyển R2.
2. **Central switching của WLC-3504 (CAPWAP) không kiểm chứng bằng packet-trace** — tính năng wireless được xác nhận bằng associate + lease của client (functional proxy).
3. **RADIUS end-to-end 802.1X chưa được chụp bằng chứng** (TC-13 ở trạng thái "configured, chưa xác nhận exchange").
4. **Bất ổn mô phỏng wireless của PT 9.0.1.0858:** association đôi lúc rớt khi thao tác nhiều (chứng kiến ở `evidence/Final but error.png`); bước recovery ghi ở mục 3; trong các lần chụp canvas cuối (Final.jpg / Final 2.png) trạng thái đã hồi phục.
5. `show ip interface` của PT không hiển thị ACL named (lỗi hiển thị; dùng Running Config).
6. PSK/secret/mật khẩu lưu dạng rõ trong XML `.pkt` — không chia sẻ file ra ngoài nhóm.
7. `serverPool` factory DHCP không xoá được vĩnh viễn.

## 9. Artifact & cấu hình tham chiếu

- `packet-tracer/final.pkt` — 100.297 bytes, SHA-256 `5AA9ADCA75402F8D099427BC94C137C7ECF389E102187994D4F220491A8B5CEF`.
- Cấu hình thiết bị xuất ngày 2026-11-07 từ live PT (startup-config truth):
  - [`../configs/router/router-r1-running-config.txt`](../configs/router/router-r1-running-config.txt) (R1, 2901)
  - [`../configs/router/router-r2-running-config.txt`](../configs/router/router-r2-running-config.txt) (R2, 2901)
  - [`../configs/switch/core-sw-running-config.txt`](../configs/switch/core-sw-running-config.txt) (CORE-SW, 3560-24PS)
  - `configs/switch/switch-sw1-running-config.txt` là bản **v1 lịch sử** (SW1 không tồn tại trong final — giữ lại chỉ để đối chiếu giai đoạn P1).
- `configs/wlc-ap/README.md` cập nhật mục WLC-3504 final tại mục "WLC-3504 — controller final".

## 10. Hướng dẫn demo nhanh (mở final.pkt)

1. Mở `packet-tracer/final.pkt` (PT 9.0.1.0858); đợi 20–30 s cho ARP/table hội tụ.
2. Trên mỗi laptop: `Desktop → Command Prompt`: `ipconfig /renew` → nhận lease đúng subnet.
3. `ping gateway của VLAN mình` (4/4) → rồi `ping 8.8.8.8` (4/4 Staff/Guest).
4. Từ G-LAP1: `ping 192.168.10.1` → `Request timed out` (ACL theo design).
5. Từ ADMIN-PC mở trình duyệt `http://192.168.99.10` để xem WLC1 GUI (WLAN/RADIUS) khi demo.
