# Quyết định lựa chọn thiết bị vô tuyến (Device Selection Decision)

> Bản ghi quyết định (decision record) cho lựa chọn model **WLC** và **AP**. Mọi khẳng định trong tài liệu này đều dựa trên quan sát trực tiếp trong **Cisco Packet Tracer 9.0.1.0858** (GUI + danh mục thiết bị + tài liệu `IpcAPI` đi kèm), minh chứng tại `../evidence/`. Những gì **chưa** kiểm chứng hoạt động vẫn ghi rõ là chưa kiểm chứng.
>
> **Cảnh báo minh chứng:** các ảnh `audit-*.png` (và `wlc-pt-feasibility-*.png`) được viện dẫn trong tài liệu này **hiện chưa có trong `../evidence/`** (xem [../evidence/README.md](../evidence/README.md)) — cần chụp bổ sung. Các quan sát ở §3 dựa trên ghi nhận trực tiếp cùng ngày (2026-10-18).

## 1. Bối cảnh

Bản `enterprise-wifi-wlc-lab-v1.pkt` được dựng với controller `WLC-3504` và 01 `LAP-PT`. Khi đó nhóm ghi nhận "WLC có trang GUI `Wireless LANs` với WLAN Name/SSID/VLAN, WPA2-PSK, WPA2 + RADIUS, central switching". **Kiểm chứng lại ngày 2026-10-18 cho thấy nhận định này gắn nhầm model** — các trường đó thuộc về `WLC-PT`, không thuộc `WLC-3504`. Vì TC-04/TC-05 (tạo và phát WLAN) là Core, sai lệch này chặn tiến độ, nên cần quyết định lại model controller.

## 2. Các phương án đã cân nhắc

| Phương án | Mô tả | Kết quả khả thi trên 9.0.1.0858 |
| --- | --- | --- |
| A. Giữ `WLC-3504` | Dùng nguyên controller hiện tại | **Không khả thi cho WLAN.** GUI chỉ có `GLOBAL/Settings` + `INTERFACE` (GigabitEthernet1–5, Management). Không có trang tạo WLAN/SSID/VLAN/WPA2 ⇒ TC-04/TC-05 không thể đạt |
| B. Đổi sang `WLC-PT` | Dùng `WLC-PT` làm controller | **Khả thi.** Có `Wireless LANs` (Name/SSID/VLAN), `AP Groups`, `DHCP`, chọn Authentication gồm WPA-PSK/WPA2-PSK/WPA/WPA2 (RADIUS) |
| C. AP độc lập, không WLC | Bỏ controller, dùng AP-PT standalone | **Khả thi nhưng sai phạm vi.** Mỗi AP tự cấu hình SSID, **không có trường VLAN** ⇒ không đáp ứng REQ-07 (phân đoạn VLAN theo SSID) và nội dung "quản lý tập trung bằng WLC" của đề tài |
| D. Hybrid WLC + AP độc lập | WLC cho Staff, AP độc lập làm dự phòng | Có thể giữ như **dự phòng**, không phải luồng chính. Tốn thêm 1 thiết bị và 1 SSID không map VLAN |

## 3. Bằng chứng quan sát (2026-10-18)

### 3.1 `WLC-3504` — không có giao diện cấu hình vô tuyến

Thiết bị đã có sẵn trong dự án (tên `WLC1`), mở tab **Config** trên đúng bản 9.0.1.0858:

| Hạng mục | Quan sát được |
| --- | --- |
| Cây menu Config | `GLOBAL` → `Settings`; `INTERFACE` → `GigabitEthernet1`, `GigabitEthernet2`, `GigabitEthernet3`, `GigabitEthernet4`, `GigabitEthernet5`, `Management` |
| `Settings` | chỉ có `Display Name` và `Device Clock` |
| Trang WLAN/SSID/WPA2/RADIUS | **không tồn tại** |
| Cổng thật | `Vlan1`, `GigabitEthernet1`–`GigabitEthernet5`, `management` (ảo, type 48), `virtual` ⇒ **5 cổng GigE**, không phải 8 |
| Device type | 41 |

Minh chứng: [`../evidence/audit-wlc-3504-config-menu.png`](../evidence/audit-wlc-3504-config-menu.png).

### 3.2 `WLC-PT` — có đầy đủ trang cấu hình WLAN

Đặt tạm `WLC-PT` vào bản 9.0.1.0858 để khảo sát, sau đó gỡ bỏ (số thiết bị trả về nguyên trạng):

| Trang Config | Nội dung quan sát được |
| --- | --- |
| Cây menu | `GLOBAL` → `Settings`, **`Wireless LANs`**, **`AP Groups`**, **`DHCP`**; `INTERFACE` → `GigabitEthernet0`, `Management` |
| **`Wireless LANs`** | `Select WLAN`; `Name`; **`SSID`**; **`VLAN`**; Authentication = Disabled / WEP (WEP Key) / WPA-PSK / WPA2-PSK (PSK Pass Phrase) / WPA / WPA2 (RADIUS Server Settings: IP Address + Shared Secret); `Encryption Type`; `Central Control` = Central switching+central authentication / Local switching+central authentication / Local switching+local authentication; nút New / Remove / Save |
| `Management` | IP Configuration: IPv4 Address, Subnet Mask, Default Gateway, DNS Server ⇒ **địa chỉ quản trị đặt trên interface `Management`** |
| `GigabitEthernet0` | chỉ có Port Status / Link Speed / Duplex — cổng L2 access đơn giản, **không có tuỳ chọn VLAN/trunk trong GUI** |
| `DHCP` | pool theo từng interface: Service On/Off, Pool Name, Default Gateway, DNS Server, Start IP, Subnet Mask, Maximum Users, TFTP Server, WLC Address |
| Cổng thật | `GigabitEthernet0` + `Management`; device type **41** (giống `WLC-3504`, phân biệt bằng `PT_MODEL`) |

Minh chứng: [`../evidence/audit-wlc-pt-wireless-lans.png`](../evidence/audit-wlc-pt-wireless-lans.png), [`../evidence/audit-wlc-pt-management-ip.png`](../evidence/audit-wlc-pt-management-ip.png).

### 3.3 `LAP-PT` — không có trường `Primary Controller` trong tab Config

| Hạng mục | Quan sát được |
| --- | --- |
| Cây menu Config | `GLOBAL` → `Settings` (có radio `Ipv4 Static` / `Ipv4 DHCP`, `DNS Server`, `Default Gateway`, `Device Clock`); `INTERFACE` → `GigabitEthernet0`, `Dot11Radio0` |
| Trang `WLC` / `Primary Controller` | **không tìm thấy** trong tab Config |
| `Dot11Radio0` | chỉ `Port Status` + `Coverage Range` (250.00) — không có trường SSID/xác thực (đúng với AP lightweight) |
| CLI | `getCommandLine()` trả `null` ⇒ không có console |

Minh chứng: [`../evidence/audit-lap-pt-radio.png`](../evidence/audit-lap-pt-radio.png).

> **Lưu ý:** nhận định cũ "LAP-PT có trang `WLC / Primary Controller`" **chưa được xác nhận** và hiện **không tìm thấy**. Cơ chế join đã được kiểm chứng ở **TC-03 (`Passed`, 2026-10-19)**: `LAP1` `Status: Online` trong `AP Groups` của `WLC-PT`, qua **auto-discovery** của PT (minh chứng `../evidence/tc-03-lap-wlc-join.png`).

### 3.4 `AccessPoint-PT` (AP độc lập) — có xác thực nhưng không có VLAN

| Hạng mục | Quan sát được |
| --- | --- |
| Cây menu Config | `GLOBAL` → `Settings`; `INTERFACE` → `Port 0`, `Port 1` |
| `Port 1` (radio) | `Port Status`; **`SSID`**; 2.4 GHz Channel (6); Coverage Range (140.00); Authentication = Disabled / WEP (WEP Key) / WPA-PSK (PSK Pass Phrase) / WPA2-PSK (PSK Pass Phrase, User ID, Password); `Encryption Type` |
| Trường VLAN / nhiều SSID | **không có** ⇒ không map được SSID → VLAN |

Minh chứng: [`../evidence/audit-ap-pt-port1-security.png`](../evidence/audit-ap-pt-port1-security.png).

### 3.5 Danh mục thiết bị vô tuyến trong PT 9.0.1.0858

Đọc trực tiếp từ palette **Network Devices → Wireless Devices** (15 model):
`AccessPoint-PT`, `AccessPoint-PT-A`, `AccessPoint-PT-AC`, `AccessPoint-PT-N`, `Linksys-WRT300N`, `HomeRouter-PT-AC`, `DLC100`, `Cell-Tower`, `Central-Office-Server`, `WLC-2504`, `WLC-3504`, **`WLC-PT`**, `3702i`, `LAP-PT`, `Meraki-MX65W`.

> Nhận định cũ "AP-PT-A / AP-PT-AC không còn trong danh mục" là **sai** — cả hai đều có trong palette.

### 3.6 Khả năng bảo mật theo tài liệu `IpcAPI` đi kèm 9.0.1.0858

| Hạng mục | Giá trị | Nghĩa |
| --- | --- | --- |
| `WirelessAuthenType` | `eAuthenNull`, `eAuthenWep`, `eAuthenWPA1_PSK`, `eAuthenWPA1_EAP`, `eAuthenWPA2_PSK`, `eAuthenWPA2_EAP`, `eAuthenOpen` | Có WEP/WPA/WPA2 (PSK + EAP). **Không có WPA3** |
| `WirelessEncryptType` | Null, WEP 64/128 bit, **TKIP**, **AES** | Chọn được mã hóa |
| `struct_wireless_profile` | `ssid`, `key`, `vlan` | Mô hình SSID→VLAN có tồn tại trong simulator |
| `EapolWirelessServerProcess` / `EapolWirelessServerMainProcess` | có | Có xử lý EAPOL ⇒ hợp lý cho hướng WPA2-Enterprise/RADIUS |
| struct `capwap_control_message`, `lwapp_udp_packet` | có | Có mô hình CAPWAP/LWAPP giữa WLC và LAP |
| WPA3 / SAE / 802.11ac / 802.11ax | **không có tham chiếu nào** | Không thể triển khai trên 9.0.1.0858 ⇒ Research-only |

Bằng chứng phụ trợ: các lab mẫu đi kèn của Packet Tracer có `wlc_pt_two_wlans_wpa_radius.pkt`, `wlc_pt_simple_wlan_wpa_radius.pkt`, `wlc_3504_two_wlans.pkt`, `wpa_eap_test.pkt`, `meraki_SA_wireless_wpa2_enterprise.pkt` (đường dẫn: `C:\Program Files\Cisco Packet Tracer 9.0.1\saves\`). Lưu ý file `.pkt` được **mã hóa**, không đọc nội dung ngoài được bằng công cụ dòng lệnh.

## 4. Quyết định

**2026-10-18 — Hoàng Minh Hiếu: đổi controller từ `WLC-3504` sang `WLC-PT`.**

- `WLC-PT` là model **duy nhất** trong palette 9.0.1.0858 được quan sát có trang `Wireless LANs` với đủ SSID, trường VLAN và các lựa chọn WPA2-PSK / WPA2 + RADIUS — tức là đáp ứng REQ-03→REQ-06 và giữ được nội dung "quản lý tập trung bằng WLC" của đề tài.
- Giữ `LAP-PT` làm AP chính (đúng bản chất lightweight AP). Phần bảo mật do WLC đảm nhiệm; việc `Dot11Radio0` của LAP không có trường xác thực là **đúng thiết kế**, không phải thiếu sót.
- `AccessPoint-PT` giữ làm **dự phòng standalone** (phương án C/D), chỉ dùng nếu chứng minh WLC tập trung bất khả thi **và** phạm vi được sửa đổi rõ ràng. Không dùng nó để thay WLC trong luồng chính vì thiếu trường VLAN.
- **Loại bỏ phương án A** (giữ `WLC-3504` làm controller): không có giao diện tạo WLAN ⇒ TC-04/TC-05 không thể đạt, phạm vi đồ án bị thu hẹp ngoài ý muốn.
- **TC-01 chạy lại ĐẠT trên `WLC-PT`** (bản test copy `..._TESTCOPY_2026-10-18.pkt`): Management `192.168.99.10/24` reachable (`SRV1` `0% loss` 4/4; `R1` `5/5`). Địa chỉ đặt được **bằng API, không cần raw JS**. **`.pkt` deliverable đã được migrate sang `WLC-PT` (Phase C, 2026-10-19)** — xem §4.1.

### 4.1 Hệ quả bắt buộc

1. **TC-01 đã chạy lại trên `WLC-PT` — `Passed` (bản test copy).** Management IPv4 `192.168.99.10/24`, gateway `192.168.99.1`, đặt trên cổng GUI hiển thị là **`Management`** (API/mô phỏng đặt tên port là `Vlan1` — **giống `WLC-3504`**, `WLC-PT` **không có** cổng API tên `management` riêng). Đặt địa chỉ **bằng API** (`pt_host_ip_config`), **không cần raw JS**. Nguồn ping `SRV1` (`0% loss`, 4/4) và `R1` (`5/5`). Kết quả `Passed` của `WLC-3504` giữ lại làm lịch sử P1. Lệnh/địa chỉ chi tiết xem [implementation-notes.md](implementation-notes.md) §5 và [../tests/test-cases.md](../tests/test-cases.md) TC-01. **Lưu ý:** kiểm chứng này ban đầu thực hiện trên **bản test copy** (`..._TESTCOPY_2026-10-18.pkt`, đã xoá `WLC1`); **file deliverable sau đó đã được migrate sang `WLC-PT`** (Phase C, 2026-10-19).
2. **Uplink Switch↔WLC đã kiểm chứng trên `WLC-PT` — cho management.** `WLC-PT` chỉ có `GigabitEthernet0` (uplink, **không có tuỳ chọn VLAN/trunk trong GUI**) + `Vlan1` (management). Cách dùng: `WLC2 Gi0` → `SW1 Fa0/4` đặt **trunk** native VLAN 99, allowed `10,20,99`; frame untagged của `Vlan1` rơi vào native 99 ⇒ management ping được (TC-01). **VLAN 10/20 của client đã kiểm chứng trong TC-08** (2026-10-20): test copy `STAFF1` `.10.102/24`, `GUEST1` `.20.102/24`; official `/renew` `.10.101/24`, `.20.100/24`; MAC VLAN 10/20 trên **uplink AP `Fa0/3`**; uplink WLC `Fa0/4` chỉ có MAC management ⇒ **central switching chưa kiểm chứng**; cấu hình đã lưu vào official `.pkt` 2026-10-21.
3. **`addTrunkVlans` trên `WLC-3504`** (`Invalid arguments for IPC call`) trở nên không còn ý nghĩa với controller mới; vấn đề uplink giữa WLC-PT và switch cần đánh giá mới.
4. **Cơ chế join LAP→WLC đã kiểm chứng (TC-03 `Passed`, 2026-10-19):** `LAP1` `Status: Online` trong `AP Groups` của `WLC-PT`; PT dùng **auto-discovery** (không có trường `Primary Controller` để đặt). Chưa có minh chứng bắt tay CAPWAP; `CAPWAP.PRIMARY_AC = 0.0.0.0` ở cả hai model **không** được đọc là bằng chứng. **TC-04 + TC-05 `Passed` (2026-10-19): hai WLAN `STAFF-WIFI` (VLAN 10) + `GUEST-WIFI` (VLAN 20) tạo độc lập và phát sóng đồng thời** (quét thấy từ `STAFF1` và `GUEST1`); **TC-06 + TC-07 `Passed` (2026-10-19): Staff và Guest associate bằng WPA2-PSK**; **TC-08 `Passed` cho tiêu chí IP đúng subnet (2026-10-20, kèm giới hạn), cấu hình đã lưu vào official `.pkt` 2026-10-21**; bước kế tiếp là gateway/ACL (TC-09→TC-12).
5. Cấu hình `WLC-3504` trong `../configs/wlc-ap/` giữ lại làm **tài liệu lịch sử của P1**; cấu hình mới ghi vào mục riêng cho `WLC-PT` sau khi chạy lại TC-01.

### 4.2 Điều kiện xem lại quyết định

Quyết định này cần xem lại nếu xảy ra bất kỳ trường hợp nào sau:

- `WLC-PT` **không** nhận LAP-PT (đóng frame đăng ký) trong TC-03 ⇒ ghi `Blocked`, cân nhắc phương án C kèm sửa phạm vi. **Điều kiện này KHÔNG xảy ra — TC-03 `Passed` ngày 2026-10-19 (`LAP1` `Status: Online`).**
- `WLC-PT` tạo được WLAN nhưng **không phát sóng** hoặc **không map được VLAN** ⇒ dừng ở TC-04/TC-05, ghi `Blocked`, không giả định. **Phần tạo + phát sóng (cả hai WLAN) KHÔNG xảy ra (TC-04 + TC-05 `Passed`, 2026-10-19); map VLAN cho client đã `Passed` cho tiêu chí IP đúng subnet trong TC-08 (2026-10-20, kèm giới hạn)** — đường dữ liệu qua uplink AP `Fa0/3` (local switching), **central switching chưa kiểm chứng**; cấu hình đã lưu vào official `.pkt` 2026-10-21.
- Packet Tracer được nâng cấp phiên bản khác 9.0.1.0858 ⇒ toàn bộ quan sát ở §3 phải kiểm chứng lại.

## 5. Checklist kiểm chứng lựa chọn thiết bị

| # | Kiểm chứng | Trạng thái | Minh chứng |
| --- | --- | --- | --- |
| 1 | `WLC-3504` **không** có trang cấu hình vô tuyến | **Verified** | `../evidence/audit-wlc-3504-config-menu.png` |
| 2 | `WLC-PT` có trang `Wireless LANs` với `SSID` + trường `VLAN` | **Verified** | `../evidence/audit-wlc-pt-wireless-lans.png` |
| 3 | `WLC-PT` có lựa chọn WPA2-PSK và WPA2 + RADIUS (IP + shared secret) | **Verified** | `../evidence/audit-wlc-pt-wireless-lans.png` |
| 4 | Địa chỉ quản trị `WLC-PT` đặt trên cổng GUI `Management` (API/mô phỏng đặt tên port `Vlan1`) | **Verified** | `../evidence/audit-wlc-pt-management-ip.png` |
| 5 | `LAP-PT` có `GigabitEthernet0` + `Dot11Radio0`; radio không có trường xác thực | **Verified** | `../evidence/audit-lap-pt-radio.png` |
| 6 | `LAP-PT` **không** thấy trang `WLC` / `Primary Controller` trong tab Config | **Verified (quan sát)** | `../evidence/audit-lap-pt-radio.png` |
| 7 | `AccessPoint-PT` có SSID + WPA-PSK/WPA2-PSK nhưng **không** có trường VLAN | **Verified** | `../evidence/audit-ap-pt-port1-security.png` |
| 8 | `AP-PT-A` và `AP-PT-AC` vẫn có trong palette 9.0.1.0858 | **Verified** | bảng §3.5 (quan sát palette) |
| 9 | Không có WPA3/SAE trong `WirelessAuthenType` | **Verified** | `IpcAPI/class_wireless_*.html` |
| 10 | LAP-PT join được `WLC-PT` | **Verified** (2026-10-19; TC-03 `Passed`) | `../evidence/tc-03-lap-wlc-join.png` |
| 11 | `WLC-PT` tạo + phát được `STAFF-WIFI` (VLAN 10) | **Verified** (2026-10-19; TC-04 `Passed` — phát sóng xác nhận bằng quét; map VLAN cho client ở TC-08) | `../evidence/tc-04-staff-wlan.png` |
| 12 | `WLC-PT` tạo + phát được `GUEST-WIFI` (VLAN 20) | **Verified** (2026-10-19; TC-05 `Passed` — phát sóng đồng thời hai SSID xác nhận bằng quét từ `GUEST1`; map VLAN cho client ở TC-08) | `../evidence/tc-05-guest-wlan.png` |
| 13 | Client associate `STAFF-WIFI` bằng WPA2-PSK | **Verified** (2026-10-19; TC-06 `Passed`; Guest qua TC-07 `Passed`) | `../evidence/tc-06-staff-associate.png`, `../evidence/tc-07-guest1-associate-live.png`, `../evidence/tc-07-guest2-associate-live.png` |
| 14 | Trunk Switch↔LAP mang được VLAN 10/20 (đường dữ liệu client) | **Verified trong TC-08** (2026-10-20; IP đúng subnet, kèm giới hạn; MAC client trên `Fa0/3`; **central switching/WLC uplink chưa kiểm chứng**; cấu hình đã lưu vào official `.pkt` 2026-10-21) | `../evidence/tc-08-trunk.png`, `../evidence/tc-08-mac-vlan.png` |
| 15 | Management reachability trên `WLC-PT` (địa chỉ trên `Management`/API `Vlan1`) | **Verified** (test copy, 2026-10-18) | TC-01; `../evidence/tc-01-wlc-pt-reachability-canvas.png` |

## 6. Tài liệu liên quan

- [implementation-notes.md](implementation-notes.md) — model đã chốt, danh sách cổng thật, ma trận quan sát/kiểm chứng, vấn đề và workaround.
- [topology.md](topology.md) — topology logic và quyết định chế độ cổng.
- [ip-vlan-plan.md](ip-vlan-plan.md) — địa chỉ chuẩn (authoritative).
- [security-design.md](security-design.md) — WPA2-PSK trước, RADIUS có điều kiện.
- [testing-plan.md](testing-plan.md) — phân loại Core/Conditional, kết quả đã chạy.
- [../tests/test-cases.md](../tests/test-cases.md) — 14 ca kiểm thử.
- [../evidence/README.md](../evidence/README.md) — quy định minh chứng.