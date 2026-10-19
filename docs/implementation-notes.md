# Ghi chú triển khai (Implementation Notes)

> File này ghi lại **phát hiện đã quan sát** và **quyết định đã kiểm chứng**. Mọi chức năng hoạt động vẫn ở trạng thái chưa kiểm chứng cho tới khi có minh chứng trong `../evidence/`.

## 1. Phiên bản Packet Tracer (đã chốt)

| Hạng mục | Nội dung |
| --- | --- |
| Phiên bản Packet Tracer sử dụng | 9.0.1.0858 (đã chốt) |
| Người kiểm chứng GUI | Nhóm đồ án (chi tiết đóng góp tại [team-contributions.md](team-contributions.md)) |
| Ngày chốt | Giai đoạn finalize tài liệu (ngày kiểm thử thực tế ghi kèm từng evidence) |

Mọi file `.pkt`, cấu hình và screenshot phải thực hiện trên đúng bản này.

## 2. Model thiết bị đã chốt và cổng thật trong Packet Tracer 9.0.1.0858

Model dưới đây **đã chốt** bằng cách đặt thiết bị thật vào bản 9.0.1.0858 và đọc danh sách cổng **trực tiếp từ mô phỏng** — không suy từ tài liệu bên ngoài.

> **Cập nhật 2026-10-18 (audit lựa chọn thiết bị):** controller đổi từ `WLC-3504` → **`WLC-PT`**, vì `WLC-3504` **không có giao diện cấu hình vô tuyến** trong bản này. Lý do và minh chứng tại [device-selection-decision.md](device-selection-decision.md).

| Vai trò | Model đã chốt | Cổng thật đọc được từ mô phỏng | Ghi chú đã quan sát |
| --- | --- | --- | --- |
| WLC (controller) | **`WLC-PT`** | `GigabitEthernet0` (Ethernet), `Management` (quản trị) | Device type thật = **41** (giống `WLC-3504`, phân biệt bằng `PT_MODEL`). Có trang **`Wireless LANs`** (Name/SSID/**VLAN**, Authentication gồm WPA-PSK/WPA2-PSK/WPA2 + RADIUS IP & shared secret, Encryption Type, Central switching/authentication), trang `AP Groups`, trang `DHCP`. Management IP đặt trên interface **`Management`**. `GigabitEthernet0` chỉ có Port Status/Link/Duplex — **không có tuỳ chọn VLAN/trunk trong GUI**. Evidence: `../evidence/audit-wlc-pt-wireless-lans.png` |
| WLC (model cũ, giữ làm lịch sử P1) | `WLC-3504` | `Vlan1`, `GigabitEthernet1`–`GigabitEthernet5`, `management`, `virtual` | Cổng thật: **5 GigE** (`GigabitEthernet1`–`5`), **không phải 8**. Interface mang IP quản trị là `Vlan1`; `management` là interface ảo type 48 không nối cổng vật lý. **KHÔNG có trang `Wireless LANs`** — chỉ có `GLOBAL/Settings` + `INTERFACE` (Gi1–5, Management). Vì vậy không dùng làm controller. Evidence: `../evidence/audit-wlc-3504-config-menu.png` |
| Router | `2911` | `Vlan1`, `GigabitEthernet0/0`–`GigabitEthernet0/2` | Đã dùng `GigabitEthernet0/0` + subinterface 802.1Q; `ip helper-address` cấu hình và chạy được |
| Switch Layer 2 | `2960-24TT` | `Vlan1`, `FastEthernet0/1`–`FastEthernet0/24`, `GigabitEthernet0/1`, `GigabitEthernet0/2` (27 cổng) | Model **dot1q-only**: KHÔNG dùng `switchport trunk encapsulation dot1q`. Trunk + access VLAN chạy được |
| Server | `Server-PT` | `FastEthernet0` | Services > DHCP tạo được nhiều pool và bật Service On |
| AP chính | `LAP-PT` (`Light Weight Access Point`) | `GigabitEthernet0` (Ethernet), `Dot11Radio0` (radio); tab Config có radio `Ipv4 Static`/`Ipv4 DHCP` + `Default Gateway`/`DNS Server` | Device type khi thêm vào = `7`. Danh mục thiết bị của công cụ ghi `Port 0` — **sai**, tên thật là `GigabitEthernet0`. **Không có CLI/console** (`getCommandLine()` trả `null`) ⇒ không chạy được `ping` bằng lệnh. Cổng Ethernet chỉ lên khi có cấu hình IP (xem §5). **Không tìm thấy** trang `WLC`/`Primary Controller` trong tab Config, và `Dot11Radio0` chỉ có Port Status + Coverage Range — phần bảo mật do WLC đảm nhiệm. **Cơ chế join đã kiểm chứng ở TC-03 (`Passed`, 2026-10-19):** `LAP1` xuất hiện `Status: Online` trong `AP Groups` của `WLC-PT` (PT auto-discovery; không có trường đặt IP controller, `CAPWAP.PRIMARY_AC = 0.0.0.0` ở cả hai model) |
| AP dự phòng | `AccessPoint-PT` standalone | `Port 0` Ethernet, `Port 1` wireless (SSID, WPA-PSK, WPA2-PSK, WEP, Encryption Type) | Có SSID + xác thực nhưng **không có trường VLAN** ⇒ không thay WLC trong luồng chính. Chỉ dùng khi chứng minh WLC tập trung bất khả thi và scope được sửa đổi. Evidence: `../evidence/audit-ap-pt-port1-security.png` |
| Client | Chưa chốt | — | 2 Staff + 2 Guest (≥ 4) |

> Quy tắc: AP chính là LAP-PT, bắt đầu 01 AP. AP standalone chỉ là fallback khi chứng minh WLC tập trung bất khả thi và scope được sửa đổi.

> **Cảnh báo tên cổng:** danh mục thiết bị của công cụ MCP ghi cổng của `LAP-PT` là `Port 0` (FastEthernet), trong khi GUI ghi `GigabitEthernet0`. Phải đọc tên cổng trực tiếp từ mô phỏng ngay trước khi đấu cáp LAP-PT. Cắm sai tên cổng có thể làm link không lên mà không báo lỗi rõ ràng.

> **Danh mục vô tuyến trong palette 9.0.1.0858** (đọc trực tiếp, 15 model): `AccessPoint-PT`, `AccessPoint-PT-A`, `AccessPoint-PT-AC`, `AccessPoint-PT-N`, `Linksys-WRT300N`, `HomeRouter-PT-AC`, `DLC100`, `Cell-Tower`, `Central-Office-Server`, `WLC-2504`, `WLC-3504`, `WLC-PT`, `3702i`, `LAP-PT`, `Meraki-MX65W`. Nhận định cũ "AP-PT-A/AP-PT-AC không còn trong danh mục" là **sai**.

## 3. Ma trận quan sát vs kiểm chứng

| Hạng mục | Đã quan sát trong GUI | Đã kiểm chứng hoạt động |
| --- | --- | --- |
| `WLC-PT` có trang `Wireless LANs` (SSID + VLAN) | **Có** — quan sát trực tiếp 2026-10-18, `../evidence/audit-wlc-pt-wireless-lans.png` | Chưa (TC-04, TC-05) |
| `WLC-3504` có trang cấu hình vô tuyến | **Không** — chỉ `GLOBAL/Settings` + `INTERFACE`, `../evidence/audit-wlc-3504-config-menu.png` | Không áp dụng (loại khỏi vai trò controller) |
| Tuỳ chọn WPA2-PSK / WPA2 + RADIUS | **Có** trên `WLC-PT`; có trên AP standalone dạng PSK | Chưa (TC-06, TC-07, TC-13, TC-14) |
| Management IP + reachability (model cũ `WLC-3504`) | Có | **Có** — TC-01 `Passed` trên `WLC-3504` (2026-10-18, lịch sử) |
| Management IP + reachability (model mới `WLC-PT`) | Có — trang GUI `Management` (API port `Vlan1`), `../evidence/audit-wlc-pt-management-ip.png` | **Có** — TC-01 chạy lại `Passed` trên `WLC-PT` (bản test copy, 2026-10-18); `../evidence/tc-01-wlc-pt-reachability-canvas.png` |
| LAP-PT + cơ chế join WLC | **Không thấy** trường `Primary Controller` trong tab Config | **Có** — TC-03 `Passed` (2026-10-19): `LAP1` `Status: Online` trong `AP Groups` của `WLC-PT`; `../evidence/tc-03-lap-wlc-join.png` |
| Trường VLAN trên WLAN | **Có** (trường `VLAN` trong `Wireless LANs` của `WLC-PT`) | Chưa (TC-08, TC-12) |
| Trunk/access cho model đã chốt (phía switch/router) | Có | **Có** (khung có dây) — trunk `FastEthernet0/1` và `GigabitEthernet0/1` native VLAN 99 + tagged 10,20; access `FastEthernet0/2` VLAN 99 |
| VLAN 10/20/99 trên switch | Có | **Có** (khung có dây) — đọc lại từ VlanManager của switch |
| Liên VLAN trên router | Có | **Có** (khung có dây) — `Gi0/0.10`/`Gi0/0.20` + `ip helper-address 192.168.99.20` |
| DHCP Server-PT mọi VLAN | Có | Chưa — 2 pool đã tạo và Service On, nhưng **chưa có client** để kiểm chứng cấp phát (TC-08) |
| ACL cô lập Guest | Chưa | Chưa (TC-10, TC-11) |
| WPA3 / EAP cụ thể / roaming / RF / throughput | **Không hỗ trợ** — `WirelessAuthenType` trong `IpcAPI` không có WPA3/SAE | Research-only |
| WPA2-Enterprise / EAPOL | Có class `EapolWirelessServerProcess` + struct `capwap_control_message`/`lwapp_udp_packet` trong `IpcAPI` | Chưa (TC-13, Conditional) |

> Các mục `Có` ở cột kiểm chứng mới thuộc **khung có dây P1**, cộng **TC-01 (đã `Passed` trên cả `WLC-3504` lẫn `WLC-PT`)**, **TC-02 trên `LAP-PT`** và **TC-03 (LAP-PT join `WLC-PT`, `Status: Online`)**; chưa có client vô tuyến nên các mục WLAN/VLAN client vẫn `Chưa`. Lưu ý kiểm chứng `WLC-PT` thực hiện trên **bản test copy** (`..._TESTCOPY_2026-10-18.pkt`); **`.pkt` deliverable đã được thay sang `WLC-PT` (2026-10-19)**. Bản cấu hình đối chiếu lưu tại `../configs/`.

## 4. Quyết định cấu hình

- `2026-10-18 — Hoàng Minh Hiếu: chốt Router 2911, Switch 2960-24TT, WLC-3504, Server-PT sau khi đặt thử vào đúng bản 9.0.1.0858 và đọc danh sách cổng thật từ mô phỏng; cổng thật đối chiếu lại: WLC-3504 có **5 cổng GigE** (GigabitEthernet1–5) + Vlan1 + management + virtual, **không phải 8 cổng GigE** như ghi ở nhận định ban đầu; config: ../configs/`
- `2026-10-18 — Hoàng Minh Hiếu: audit lựa chọn thiết bị phát hiện WLC-3504 KHÔNG có giao diện cấu hình vô tuyến trong bản 9.0.1.0858 (chỉ GLOBAL/Settings + INTERFACE GigabitEthernet1–5 và Management); các trường WLAN Name/SSID/VLAN/WPA2/RADIUS từng được ghi nhận thuộc về model khác là WLC-PT. Quyết định đổi controller sang WLC-PT; giữ LAP-PT làm AP chính; WLC-3504 chỉ giữ làm thiết bị quản trị lịch sử, không làm controller; evidence: ../evidence/audit-wlc-3504-config-menu.png, ../evidence/audit-wlc-pt-wireless-lans.png; quyết định đầy đủ: device-selection-decision.md`
- `2026-10-18 — Hoàng Minh Hiếu: sau khi đổi controller, đã chạy lại TC-01 trên WLC-PT (bản test copy ..._TESTCOPY_2026-10-18.pkt, đã xoá WLC1): management IP 192.168.99.10/24, gw 192.168.99.1 đặt trên cổng GUI hiển thị là "Management" mà API/mô phỏng đặt tên port là "Vlan1" — GIỐNG WLC-3504; WLC-PT KHÔNG có cổng API "management" riêng; đặt được BẰNG API (pt_host_ip_config), KHÔNG cần raw JS; ping SRV1 (192.168.99.20) 0% loss (4/4) và R1 5/5; kết quả Passed cũ của WLC-3504 giữ làm lịch sử: ../evidence/tc-01-wlc-reachability.png; evidence mới: ../evidence/tc-01-wlc-pt-reachability-canvas.png`
- `2026-10-18 — Hoàng Minh Hiếu: quan sát WLC-PT: cổng thật = Vlan1 (mang IP quản trị) + GigabitEthernet0 (uplink); built-in DHCP mặc định TẮT (DhcpServerMain false, 0 pool); LAP-PT tĩnh 192.168.99.30/24 đặt được và giữ qua API (không bị xoá); cấu hình WLAN/SSID/VLAN/WPA2 chỉ có qua GUI (không có API); config: ../configs/wlc-ap/`
- `2026-10-18 — Hoàng Minh Hiếu: uplink WLC-PT GigabitEthernet0 → SW1 Fa0/4 đặt trunk native VLAN 99 + allowed 10,20,99; frame untagged của Vlan1 rơi vào native 99 nên management ping được (TC-01 Passed); sau khi đổi port sang trunk phải chờ STP hội tụ (~30s) mới ping được; VLAN 10/20 cho client CHƯA chứng minh (cần WLAN + client ở TC-08); evidence: ../evidence/tc-01-wlc-pt-reachability-canvas.png`
- `2026-10-18 — Hoàng Minh Hiếu: LAP-PT không tìm thấy trường Primary Controller trong tab Config; không kết luận join được/không được cho tới khi chạy TC-03; coi đây là cổng chặn tiếp theo sau khi đổi controller`
- `2026-10-18 — Hoàng Minh Hiếu: WLC-PT có trang DHCP cấu hình pool theo từng interface (Service On/Off, Pool Name, Default Gateway, DNS Server, Start IP, Subnet Mask, Maximum Users, TFTP Server, WLC Address) nhưng nhóm vẫn dùng Server-PT làm nguồn DHCP để giữ nguyên thiết kế đã kiểm chứng; WLC DHCP chỉ dùng nếu Server-PT không đáp ứng`
- `2026-10-18 — Hoàng Minh Hiếu: router-on-a-stick trên 2911 — Gi0/0 = 192.168.99.1/24 (VLAN 99 native) + Gi0/0.10 (dot1Q 10) + Gi0/0.20 (dot1Q 20) kèm ip helper-address 192.168.99.20 trên cả hai subinterface; khác đề xuất ở chỗ KHÔNG tạo subinterface riêng cho VLAN 99 vì VLAN 99 đi untagged trên cổng vật lý; config: ../configs/router/`
- `2026-10-18 — Hoàng Minh Hiếu: 2960-24TT là dot1q-only nên KHÔNG cấu hình switchport trunk encapsulation dot1q; trunk Fa0/1 (WLC) và Gi0/1 (router) đặt native VLAN 99 + allowed 10,20,99; Fa0/2 là access VLAN 99 cho Server-PT; SVI Vlan99 = 192.168.99.2 + ip default-gateway 192.168.99.1; config: ../configs/switch/`
- `2026-10-18 — Hoàng Minh Hiếu: Server-PT dùng địa chỉ tĩnh 192.168.99.20/24 (gw .1) và 2 pool DHCP STAFF (192.168.10.100–199) + GUEST (192.168.20.100–199), Service On; config: ../configs/`
- `2026-10-18 — Hoàng Minh Hiếu (model cũ WLC-3504, giữ làm lịch sử): đặt Management IPv4 của WLC-3504 trên interface Vlan1 (192.168.99.10/24, gw 192.168.99.1) chứ không phải interface management — management là interface ảo (type 48, MAC 0003.E479.4739) không nối vào cổng vật lý, còn frame thật ra uplink mang MAC của Vlan1 (00D0.BAD0.7DA9); evidence: ../evidence/tc-01-wlc-reachability.png`
- `2026-10-18 — Hoàng Minh Hiếu (model cũ WLC-3504, giữ làm lịch sử): chuyển uplink WLC1 GigabitEthernet1 từ access VLAN 1 (mặc định) sang trunk nhưng giữ native VLAN 1, trong khi SW1 Fa0/1 native VLAN 99 — frame untagged của WLC rơi vào VLAN 99 của switch nên Management IPv4 vẫn nằm đúng VLAN 99; khác đề xuất ban đầu là đặt native 99 ở cả hai phía (làm vậy WLC không trả lời ARP vì SVI Vlan1 thuộc VLAN 1 nội bộ); evidence: ../evidence/tc-01-wlc-reachability.png`

- `2026-10-19 — Hoàng Minh Hiếu: TC-03 Passed — LAP1 (LAP-PT) join WLC2 (WLC-PT); quan sát WLC2 → Config → AP Groups: AP LAP1, MAC 0001.968E.6B01, Status Online, nhóm default-group; khớp XML CAPWAP_AC/APS của WLC. Cơ chế là auto-discovery của PT (không có trường Primary Controller; CAPWAP.PRIMARY_AC = 0.0.0.0 ở cả hai model, KHÔNG đọc là bằng chứng); không có minh chứng bắt tay CAPWAP; không cấu hình WLAN/DHCP/ACL trong TC-03; evidence: ../evidence/tc-03-lap-wlc-join.png`
- `2026-10-19 — Hoàng Minh Hiếu: promote bản `enterprise-wifi-wlc-lab-v1_TC03_TEST.pkt` thành deliverable `enterprise-wifi-wlc-lab-v1.pkt` (2 file cùng nội dung, SHA-256 A531B654…) và re-audit TC-03 trên chính file deliverable: `WLC2 Vlan1 192.168.99.10/24` gw `192.168.99.1` (lưu trong XML); `R1 → .10`=5/5, `R1 → .30`=5/5; SW1 VLAN 99 học MAC LAP1 (Fa0/3). Phát hiện: **IP tĩnh `LAP1` và trạng thái join KHÔNG lưu vào `.pkt`** (mở lại thấy `PORT_DHCP_ENABLE=true`, `IP` trống, AP `Offline`); phải đặt lại tĩnh `192.168.99.30` thì AP rejoin `Online`; evidence làm mới: ../evidence/tc-03-lap-wlc-join.png`

Khi bổ sung quyết định mới, ghi theo mẫu: `ngày — người làm: chọn model/interface/VLAN ... vì ...; khác đề xuất ở ...; config/evidence: ...`

Không ghi lệnh CLI vào đây như "đã chạy thành công" khi chưa test trên đúng model. Mọi lệnh phải đi kèm minh chứng trong `../evidence/` và bản redacted trong `../configs/`.

## 5. Vấn đề và cách khắc phục (Issues & Workarounds)

| Ngày | Người ghi | Vấn đề | Cách khắc phục | TC liên quan | Trạng thái |
| --- | --- | --- | --- | --- | --- |
| 2026-10-18 | Hoàng Minh Hiếu | Cầu nối điều khiển báo DOWN, không đặt được thiết bị vào PT | Mở `Extensions` → `MCP BUILDER` trong Packet Tracer để extension nối vào cầu ở cổng 54321 | — | Đã xử lý |
| 2026-10-18 | Hoàng Minh Hiếu | Danh mục công cụ ghi `WLC-3504` có 4 cổng GigE, mô phỏng thực tế có **5** (`GigabitEthernet1`–`5`, cộng `Vlan1`/`management`/`virtual`) | Luôn đọc danh sách cổng **trực tiếp từ mô phỏng** trước khi đấu cáp, không suy từ danh mục | TC-01, TC-12 | Đã xử lý |
| 2026-10-18 | Hoàng Minh Hiếu | **Nhận định cũ gán nhầm GUI của `WLC-PT` cho `WLC-3504`**: bản ghi trước đây nói WLC-3504 có `Wireless LANs`/SSID/VLAN/WPA2/RADIUS, nhưng kiểm chứng lại cho thấy `WLC-3504` không có trang vô tuyến nào | Đổi controller sang `WLC-PT`; cập nhật §2, §3, §4 và `device-selection-decision.md`; evidence `../evidence/audit-wlc-3504-config-menu.png` | TC-01→TC-05 | Đã xử lý |
| 2026-10-19 | Hoàng Minh Hiếu | `LAP-PT` không tìm thấy trang `WLC` / trường `Primary Controller` trong tab Config (bản ghi trước đây nói là có) | **Đã giải quyết** — TC-03 `Passed`: `LAP1` `Status: Online` trong `AP Groups` của `WLC-PT`; PT tự auto-discovery, không cần trường `Primary Controller` (`CAPWAP.PRIMARY_AC = 0.0.0.0` ở cả hai model) | TC-03 | Đã xử lý |
| 2026-10-18 | Hoàng Minh Hiếu | `WLC-PT` không có tuỳ chọn VLAN/trunk trên `GigabitEthernet0` trong GUI | **Đã xử lý cho management** — chỉ cần đặt trunk ở phía `SW1 Fa0/4` (native 99, allowed 10,20,99); management thông (TC-01 `Passed`). **Còn mở** cho lưu lượng client VLAN 10/20 (TC-08) | TC-08 | Đang mở (một phần) |
| 2026-10-18 | Hoàng Minh Hiếu | Cổng của `LAP-PT` được danh mục ghi là `Port 0`, GUI ghi `GigabitEthernet0` | **Đã giải quyết** — đọc trực tiếp từ mô phỏng: `GigabitEthernet0`; đã đấu `SW1 Fa0/3` và TC-02 `Passed` | TC-02 | Đã xử lý |
| 2026-10-18 | Hoàng Minh Hiếu | Công cụ đặt thiết bị không có mã loại riêng cho WLC | Đặt WLC bằng mã loại mặc định rồi kiểm tra `PT_MODEL` để xác nhận đúng model (`WLC-3504` hay `WLC-PT`) — cả hai đều là device type 41 | — | Đã xử lý |
| 2026-10-18 | Hoàng Minh Hiếu | Danh sách thiết bị có thêm một thiết bị không nằm trong topology (`Power Distribution Device0`) | Không hiện trên canvas nên không ảnh hưởng sơ đồ; theo dõi, xoá nếu gây nhiễu khi kiểm tra | — | Đang theo dõi |
| 2026-10-19 | Hoàng Minh Hiếu | Công cụ kiểm tra kết nối chỉ hỗ trợ host và thiết bị IOS, không hỗ trợ LAP/WLC | **Đã xử lý** — kiểm chứng gián tiếp: Simple PDU `LAP1 → R1` + `ping` từ `R1`/`SRV1` tới LAP/WLC + đọc ARP/MAC-table; TC-02 và TC-03 đều `Passed` | TC-02, TC-03 | Đã xử lý |
| 2026-10-18 | Hoàng Minh Hiếu | Ping tới WLC 100% loss dù link up và trunk phía switch đúng; `show mac address-table` **không học được MAC nào** trên `Fa0/1` | Uplink `WLC1 GigabitEthernet1` mặc định là **access VLAN 1** → frame vào VLAN 1 bị trunk SW1 (allowed 10,20,99) drop. Khắc phục: `setAccessPort(false)` để chuyển trunk | TC-01 | Đã xử lý |
| 2026-10-18 | Hoàng Minh Hiếu | Đặt IP trên interface `management` của WLC xong vẫn không ping được | Interface thật mang lưu lượng là `Vlan1`; `management` là interface ảo type 48 không nối vào cổng vật lý. Chuyển IP sang `Vlan1`, trả `management` về mặc định `192.168.1.1/28` | TC-01 | Đã xử lý |
| 2026-10-18 | Hoàng Minh Hiếu | Trunk hai phía cùng native VLAN 99 thì WLC không trả lời ARP | SVI `Vlan1` chỉ hoạt động trong VLAN 1 nội bộ. Giữ native VLAN 1 phía WLC, SW1 native 99: frame untagged đi qua được, mỗi bên tự gán VLAN | TC-01 | Đã xử lý |
| 2026-10-18 | Hoàng Minh Hiếu | `addTrunkVlans('10,20,99')` trên switchport WLC báo `Invalid arguments for IPC call` | **Một phần:** trên `WLC-PT` không cần set trunk phía WLC (GUI không có tuỳ chọn); đặt trunk ở phía `SW1 Fa0/4` (native 99, allowed 10,20,99) là đủ để management hoạt động (TC-01 `Passed`). Việc mang VLAN 10/20 cho client vẫn cần kiểm chứng (TC-08) | TC-08 | Đang mở |
| 2026-10-18 | Hoàng Minh Hiếu | Bảng mã loại thiết bị của công cụ không có WLC; device type thật đọc được là **41**, không phải mã mặc định 21 đã dùng để đặt | Ghi nhận để tra cứu; thiết bị vẫn vào đúng `WLC-3504` và cấu hình được. Lưu ý `WLC-PT` cũng là type 41 nên phải phân biệt qua `PT_MODEL` | — | Đã xử lý |

## 6. Hạn chế đã biết

- **TC-03 (LAP-PT join WLC) — ĐÃ KIỂM CHỨNG `Passed` (2026-10-19):** `LAP1` xuất hiện `Status: Online` trong `AP Groups` của `WLC-PT` (nhóm `default-group`), khớp XML `CAPWAP_AC/APS` của WLC. Trước đó nhóm không tìm thấy trường `Primary Controller` trên `LAP-PT`; cơ chế thực tế là **auto-discovery** của PT, không có trường đặt IP controller. **Giới hạn:** không có minh chứng bắt tay CAPWAP và PT không có API trả trạng thái AP; `CAPWAP.PRIMARY_AC = 0.0.0.0` ở cả hai model **không** được đọc là bằng chứng theo hướng nào. `WLC-PT` vẫn **không có trang danh sách AP riêng** — trạng thái join chỉ thấy qua `AP Groups` (hoặc XML).
- **`LAP-PT` không lưu IP tĩnh và trạng thái join vào `.pkt` (re-audit 2026-10-19):** mở lại `packet-tracer/enterprise-wifi-wlc-lab-v1.pkt` thấy `LAP1 GigabitEthernet0` trở về **DHCP** (`serializeToXml()` cho `<PORT_DHCP_ENABLE>true</PORT_DHCP_ENABLE>`, `<IP/>`, `<SUBNET/>`, `<PORT_GATEWAY/>`) ⇒ AP hiện `Offline` và không có IP. Đặt lại tĩnh `192.168.99.30/24` (gw `.1`, DNS `.20`) bằng API thì gateway/DNS được ghi vào XML nhưng `PORT_DHCP_ENABLE` vẫn `true` và `IP` vẫn trống ⇒ **IP tĩnh + join chỉ tồn tại trong phiên, không persist**. Hệ quả: sau khi mở file **phải đặt lại IP cho `LAP1`** thì AP mới rejoin (`Status: Online`). (Ngược lại, `WLC-PT` `Vlan1` `192.168.99.10/24` + gateway/DNS **có** lưu trong `.pkt`.)
- `WLC-PT` tạo/phát WLAN, multi-WLAN đồng thời, map SSID→VLAN: **chưa kiểm chứng** (TC-04, TC-05).
- WPA2-PSK pass/fail, DHCP cấp phát cho client, Guest isolation end-to-end: **chưa kiểm chứng**.
- Chế độ cổng Switch↔`WLC-PT`: **đã chạy được cho management** — `SW1 Fa0/4` trunk native VLAN 99 + allowed 10,20,99, `WLC-PT Gi0` uplink (GUI không có tuỳ chọn trunk/VLAN); management (VLAN 99) ping được (TC-01 `Passed`). **Chưa chốt** cho **lưu lượng client VLAN 10/20** — phải kiểm chứng central switching ở TC-08.
- RADIUS + WPA2-Enterprise: Conditional, thử sau cùng; `Blocked` được chấp nhận kèm lý do.
- **WPA3/SAE: `IpcAPI` của bản 9.0.1.0858 không có** trong `WirelessAuthenType` (chỉ có WEP/WPA1/WPA2 PSK+EAP/Open) ⇒ không thể triển khai trên phiên bản đã chốt. Tương tự, không có tham chiếu 802.11ac/802.11ax. Các mục này Research-only.
- **Ranh giới tự động hoá (kiểm chứng lại 2026-10-18):** phần **hạ tầng** làm được bằng API — đặt Management IPv4/mask/gateway (`setIpSubnetMask`, `setDefaultGateway`), cấu hình switchport uplink (`setAccessPort`, `setNativeVlanId`, `setAccessVlan`), thêm/xoá thiết bị, đấu cáp, cấu hình IOS, ping thật, đọc cổng/VLAN thật, lưu `.pkt`, chụp ảnh canvas. **Đặt IP cho `WLC-PT` và `LAP-PT` cũng làm được bằng API** (`pt_host_ip_config` chạy được trên thiết bị không phải host), **không cần raw JS**. Phần **không có API** và vẫn phải thao tác GUI: tạo WLAN/SSID/VLAN/WPA2 trên `WLC-PT`, kiểm tra AP join (qua `AP Groups`).
- **Không thể kiểm tra nội dung `.pkt` bằng công cụ dòng lệnh** — file `.pkt` được mã hóa; muốn kiểm tra phải mở bằng Packet Tracer.
- **Chụp minh chứng GUI:** cần gói `comtypes` (extra `[ui]` của công cụ MCP). Đã cài `comtypes 1.4.17` trên máy dựng bài; thiếu gói này thì `pt_ui_open`/`pt_ui_capture` không chạy và phải chụp màn hình bằng tay.

## 7. Công cụ hỗ trợ (MCP) và cách ghi nhận

Nhóm dùng công cụ `packet-tracer-mcp` làm **công cụ tăng tốc** cho các bước dựng topology, cấu hình IOS và thu thập kết quả — không thay thế việc kiểm chứng thủ công.

| Hạng mục | Nội dung |
| --- | --- |
| Nguồn | GitHub `Mats2208/MCP-Packet-Tracer`, cài từ mã nguồn mới nhất (bản phát hành trên PyPI thiếu một số lệnh cần dùng) |
| Phần chạy trong PT | Extension `V5.2.pts` cài thủ công qua `Extensions` → `Scripting` → `Configure PT Script Modules`, sau đó mở `Extensions` → `MCP BUILDER` |
| Việc công cụ làm được | Đặt thiết bị, đấu cáp, cấu hình IOS, đọc VLAN/cổng thật, cấu hình service của Server-PT, ping thật, đặt IP cho LAP/WLC, thêm/xoá thiết bị (kể cả thiết bị tạm để khảo sát rồi gỡ bỏ), lưu `.pkt`, chụp ảnh canvas, mở cửa sổ Config của thiết bị và chụp minh chứng GUI |
| Việc phải làm tay | Tạo WLAN/SSID/VLAN/WPA2/RADIUS trên `WLC-PT`, kiểm tra AP join (qua `AP Groups`) |

Quy định: kết quả do công cụ tạo ra vẫn phải **đối chiếu lại bằng `show`/đọc trạng thái thật** trước khi ghi vào tài liệu. Không ghi `Passed` chỉ vì công cụ báo thành công.
