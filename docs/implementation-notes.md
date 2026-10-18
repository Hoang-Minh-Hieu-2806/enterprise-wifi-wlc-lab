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

| Vai trò | Model đã chốt | Cổng thật đọc được từ mô phỏng | Ghi chú đã quan sát |
| --- | --- | --- | --- |
| WLC | `WLC-3504` | `Vlan1`, `GigabitEthernet1`–`GigabitEthernet8`, `management`, `virtual` | Danh mục tham khảo chỉ ghi 4 cổng GigE, **thực tế có 8**. Device type thật = **41**. `GigabitEthernet1` là **switchport** (mặc định access VLAN 1) — đổi được sang trunk bằng API. Interface mang IP quản trị là **`Vlan1`**, không phải `management` (`management` là interface ảo type 48, không nối vào cổng vật lý). IP/mask/gateway đặt được bằng API. GUI có `Wireless LANs` (WLAN Name, SSID, VLAN), Authentication WPA2-PSK và WPA2, RADIUS Server Settings (IP + shared secret), central switching / central authentication |
| Router | `2911` | `Vlan1`, `GigabitEthernet0/0`–`GigabitEthernet0/2` | Đã dùng `GigabitEthernet0/0` + subinterface 802.1Q; `ip helper-address` cấu hình và chạy được |
| Switch Layer 2 | `2960-24TT` | `Vlan1`, `FastEthernet0/1`–`FastEthernet0/24`, `GigabitEthernet0/1`, `GigabitEthernet0/2` (27 cổng) | Model **dot1q-only**: KHÔNG dùng `switchport trunk encapsulation dot1q`. Trunk + access VLAN chạy được |
| Server | `Server-PT` | `FastEthernet0` | Services > DHCP tạo được nhiều pool và bật Service On |
| AP chính | `LAP-PT` (`Light Weight Access Point`) | `GigabitEthernet0`, `Dot11Radio0`; trang `WLC` với `Primary Controller` | **Chưa** đọc lại cổng trực tiếp từ mô phỏng — xem cảnh báo bên dưới |
| AP dự phòng | `AP-PT-A` standalone | `Port 0` Ethernet, Port 1 wireless (SSID, WPA-PSK, WPA2-PSK, WEP, Encryption Type) | Chỉ dùng khi chứng minh WLC tập trung bất khả thi và scope được sửa đổi |
| Client | Chưa chốt | — | 2 Staff + 2 Guest (≥ 4) |

> Quy tắc: AP chính là LAP-PT, bắt đầu 01 AP. AP-PT-A chỉ là fallback standalone khi chứng minh WLC tập trung bất khả thi và scope được sửa đổi.

> **Cảnh báo tên cổng (chưa giải quyết):** danh mục thiết bị của công cụ MCP ghi cổng của `LAP-PT` là `Port 0` (FastEthernet), trong khi GUI ghi `GigabitEthernet0`. Phải đọc tên cổng trực tiếp từ mô phỏng ngay trước khi đấu cáp LAP-PT. Cắm sai tên cổng có thể làm link không lên mà không báo lỗi rõ ràng.

## 3. Ma trận quan sát vs kiểm chứng

| Hạng mục | Đã quan sát trong GUI | Đã kiểm chứng hoạt động |
| --- | --- | --- |
| Tồn tại WLC + trang WLAN | Có | Một phần — TC-01 **`Passed`** (2026-10-18); TC-04, TC-05 chưa |
| WLC management IP + reachability | Có | **Có** — TC-01 `Passed`: `192.168.99.10` trên `Vlan1`, ping 4/4 `0% loss` từ `SRV1` |
| LAP-PT + `Primary Controller` | Có | Chưa (TC-02, TC-03) |
| Trường VLAN trên WLAN | Có | Chưa (TC-08, TC-12) |
| Tùy chọn WPA2-PSK | Có (WLC và AP-PT-A) | Chưa (TC-06, TC-07, TC-14) |
| Trường WPA2 / RADIUS IP+secret | Có | Chưa (TC-13, Conditional) |
| Trunk/access cho model đã chốt | Có | **Có** (khung có dây) — trunk `FastEthernet0/1` và `GigabitEthernet0/1` native VLAN 99 + tagged 10,20; access `FastEthernet0/2` VLAN 99 |
| VLAN 10/20/99 trên switch | Có | **Có** (khung có dây) — đọc lại từ VlanManager của switch |
| Liên VLAN trên router | Có | **Có** (khung có dây) — `Gi0/0.10`/`Gi0/0.20` + `ip helper-address 192.168.99.20` |
| DHCP Server-PT mọi VLAN | Có | Chưa — 2 pool đã tạo và Service On, nhưng **chưa có client** để kiểm chứng cấp phát (TC-08) |
| ACL cô lập Guest | Chưa | Chưa (TC-10, TC-11) |
| WPA3 / EAP / roaming / RF / throughput | Chưa thấy hỗ trợ | Research-only |

> Các mục `Có` ở cột kiểm chứng mới chỉ thuộc **khung có dây P1 + TC-01**, chưa có client vô tuyến nên **chỉ TC-01 đã chuyển sang `Passed`**. Bản cấu hình đối chiếu lưu tại `../configs/`.

## 4. Quyết định cấu hình

- `2026-10-18 — Hoàng Minh Hiếu: chốt Router 2911, Switch 2960-24TT, WLC-3504, Server-PT sau khi đặt thử vào đúng bản 9.0.1.0858 và đọc danh sách cổng thật từ mô phỏng; khác đề xuất ban đầu ở chỗ WLC-3504 có 8 cổng GigE chứ không phải 4; config: ../configs/`
- `2026-10-18 — Hoàng Minh Hiếu: router-on-a-stick trên 2911 — Gi0/0 = 192.168.99.1/24 (VLAN 99 native) + Gi0/0.10 (dot1Q 10) + Gi0/0.20 (dot1Q 20) kèm ip helper-address 192.168.99.20 trên cả hai subinterface; khác đề xuất ở chỗ KHÔNG tạo subinterface riêng cho VLAN 99 vì VLAN 99 đi untagged trên cổng vật lý; config: ../configs/router/`
- `2026-10-18 — Hoàng Minh Hiếu: 2960-24TT là dot1q-only nên KHÔNG cấu hình switchport trunk encapsulation dot1q; trunk Fa0/1 (WLC) và Gi0/1 (router) đặt native VLAN 99 + allowed 10,20,99; Fa0/2 là access VLAN 99 cho Server-PT; SVI Vlan99 = 192.168.99.2 + ip default-gateway 192.168.99.1; config: ../configs/switch/`
- `2026-10-18 — Hoàng Minh Hiếu: Server-PT dùng địa chỉ tĩnh 192.168.99.20/24 (gw .1) và 2 pool DHCP STAFF (192.168.10.100–199) + GUEST (192.168.20.100–199), Service On; config: ../configs/`
- `2026-10-18 — Hoàng Minh Hiếu: đặt Management IPv4 của WLC-3504 trên interface Vlan1 (192.168.99.10/24, gw 192.168.99.1) chứ không phải interface management — management là interface ảo (type 48, MAC 0003.E479.4739) không nối vào cổng vật lý, còn frame thật ra uplink mang MAC của Vlan1 (00D0.BAD0.7DA9); khác giả định ban đầu rằng "management" là interface quản trị; evidence: ../evidence/tc-01-wlc-reachability.png`
- `2026-10-18 — Hoàng Minh Hiếu: chuyển uplink WLC1 GigabitEthernet1 từ access VLAN 1 (mặc định) sang trunk nhưng giữ native VLAN 1, trong khi SW1 Fa0/1 native VLAN 99 — frame untagged của WLC rơi vào VLAN 99 của switch nên Management IPv4 vẫn nằm đúng VLAN 99; khác đề xuất ban đầu là đặt native 99 ở cả hai phía (làm vậy WLC không trả lời ARP vì SVI Vlan1 thuộc VLAN 1 nội bộ); evidence: ../evidence/tc-01-wlc-reachability.png`

Khi bổ sung quyết định mới, ghi theo mẫu: `ngày — người làm: chọn model/interface/VLAN ... vì ...; khác đề xuất ở ...; config/evidence: ...`

Không ghi lệnh CLI vào đây như "đã chạy thành công" khi chưa test trên đúng model. Mọi lệnh phải đi kèm minh chứng trong `../evidence/` và bản redacted trong `../configs/`.

## 5. Vấn đề và cách khắc phục (Issues & Workarounds)

| Ngày | Người ghi | Vấn đề | Cách khắc phục | TC liên quan | Trạng thái |
| --- | --- | --- | --- | --- | --- |
| 2026-10-18 | Hoàng Minh Hiếu | Cầu nối điều khiển báo DOWN, không đặt được thiết bị vào PT | Mở `Extensions` → `MCP BUILDER` trong Packet Tracer để extension nối vào cầu ở cổng 54321 | — | Đã xử lý |
| 2026-10-18 | Hoàng Minh Hiếu | Danh mục công cụ ghi `WLC-3504` có 4 cổng GigE, mô phỏng thực tế có 8 | Luôn đọc danh sách cổng **trực tiếp từ mô phỏng** trước khi đấu cáp, không suy từ danh mục | TC-01, TC-12 | Đã xử lý |
| 2026-10-18 | Hoàng Minh Hiếu | Công cụ đặt thiết bị không có mã loại riêng cho WLC | Đặt WLC bằng mã loại mặc định; thiết bị vào đúng là `WLC-3504` | — | Đã xử lý |
| 2026-10-18 | Hoàng Minh Hiếu | Tên cổng `LAP-PT` khác nhau giữa danh mục (`Port 0`) và GUI (`GigabitEthernet0`) | **Chưa giải quyết** — phải đọc cổng trực tiếp từ mô phỏng ngay trước khi đấu cáp LAP-PT | TC-02, TC-03 | Đang mở |
| 2026-10-18 | Hoàng Minh Hiếu | Danh sách thiết bị có thêm một thiết bị không nằm trong topology (`Power Distribution Device0`) | Không hiện trên canvas nên không ảnh hưởng sơ đồ; theo dõi, xoá nếu gây nhiễu khi kiểm tra | — | Đang theo dõi |
| 2026-10-18 | Hoàng Minh Hiếu | Công cụ kiểm tra kết nối chỉ hỗ trợ host và thiết bị IOS, không hỗ trợ LAP/WLC | TC-02 phải kiểm bằng cách khác (ping từ LAP không tự động hoá được), hoặc ghi `Blocked` kèm lý do | TC-02 | Đang mở |
| 2026-10-18 | Hoàng Minh Hiếu | Ping tới WLC 100% loss dù link up và trunk phía switch đúng; `show mac address-table` **không học được MAC nào** trên `Fa0/1` | Uplink `WLC1 GigabitEthernet1` mặc định là **access VLAN 1** → frame vào VLAN 1 bị trunk SW1 (allowed 10,20,99) drop. Khắc phục: `setAccessPort(false)` để chuyển trunk | TC-01 | Đã xử lý |
| 2026-10-18 | Hoàng Minh Hiếu | Đặt IP trên interface `management` của WLC xong vẫn không ping được | Interface thật mang lưu lượng là `Vlan1`; `management` là interface ảo type 48 không nối vào cổng vật lý. Chuyển IP sang `Vlan1`, trả `management` về mặc định `192.168.1.1/28` | TC-01 | Đã xử lý |
| 2026-10-18 | Hoàng Minh Hiếu | Trunk hai phía cùng native VLAN 99 thì WLC không trả lời ARP | SVI `Vlan1` chỉ hoạt động trong VLAN 1 nội bộ. Giữ native VLAN 1 phía WLC, SW1 native 99: frame untagged đi qua được, mỗi bên tự gán VLAN | TC-01 | Đã xử lý |
| 2026-10-18 | Hoàng Minh Hiếu | `addTrunkVlans('10,20,99')` trên switchport WLC báo `Invalid arguments for IPC call` | **Chưa giải quyết** — chưa biết cú pháp đúng; hiện trunk của WLC chưa khai báo danh sách VLAN được phép. Cần thử khi làm TC-03/TC-08 (lưu lượng WLAN→VLAN 10/20) | TC-03, TC-08 | Đang mở |
| 2026-10-18 | Hoàng Minh Hiếu | Bảng mã loại thiết bị của công cụ không có WLC; device type thật đọc được là **41**, không phải mã mặc định 21 đã dùng để đặt | Ghi nhận để tra cứu; thiết bị vẫn vào đúng `WLC-3504` và cấu hình được | — | Đã xử lý |

## 6. Hạn chế đã biết

- LAP-PT join WLC, phát WLAN, multi-WLAN đồng thời, map SSID→VLAN: **chưa kiểm chứng**.
- WPA2-PSK pass/fail, DHCP cấp phát cho client, Guest isolation end-to-end: **chưa kiểm chứng**.
- Trunk trên model đã chốt: **đã dựng được** trong khung có dây, nhưng chưa chứng minh mang được lưu lượng client (còn thiếu AP + client).
- RADIUS + WPA2-Enterprise: Conditional, thử sau cùng; `Blocked` được chấp nhận kèm lý do.
- WPA3 / EAP cụ thể / roaming / RF / throughput: Research-only.
- **Ranh giới tự động hoá (đã kiểm chứng lại 2026-10-18):** phần **hạ tầng** của WLC làm được bằng API — đặt Management IPv4/mask/gateway trên `Vlan1` (`setIpSubnetMask`, `setDefaultGateway`) và cấu hình switchport uplink (`setAccessPort`, `setNativeVlanId`, `setAccessVlan`). Phần **không có API** và vẫn phải thao tác GUI: tạo WLAN/SSID/WPA2, đặt `Primary Controller` cho LAP-PT, kiểm tra AP join.
- **Chụp minh chứng GUI:** cần gói `comtypes` (extra `[ui]` của công cụ MCP). Đã cài `comtypes 1.4.17` trên máy dựng bài; thiếu gói này thì `pt_ui_open`/`pt_ui_capture` không chạy và phải chụp màn hình bằng tay.

## 7. Công cụ hỗ trợ (MCP) và cách ghi nhận

Nhóm dùng công cụ `packet-tracer-mcp` làm **công cụ tăng tốc** cho các bước dựng topology, cấu hình IOS và thu thập kết quả — không thay thế việc kiểm chứng thủ công.

| Hạng mục | Nội dung |
| --- | --- |
| Nguồn | GitHub `Mats2208/MCP-Packet-Tracer`, cài từ mã nguồn mới nhất (bản phát hành trên PyPI thiếu một số lệnh cần dùng) |
| Phần chạy trong PT | Extension `V5.2.pts` cài thủ công qua `Extensions` → `Scripting` → `Configure PT Script Modules`, sau đó mở `Extensions` → `MCP BUILDER` |
| Việc công cụ làm được | Đặt thiết bị, đấu cáp, cấu hình IOS, đọc VLAN/cổng thật, cấu hình service của Server-PT, ping thật, lưu `.pkt`, chụp ảnh canvas |
| Việc phải làm tay | Tạo WLAN/SSID/WPA2 trên WLC, đặt `Primary Controller` cho LAP-PT, kiểm tra AP join |

Quy định: kết quả do công cụ tạo ra vẫn phải **đối chiếu lại bằng `show`/đọc trạng thái thật** trước khi ghi vào tài liệu. Không ghi `Passed` chỉ vì công cụ báo thành công.
