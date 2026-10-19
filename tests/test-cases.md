# Danh sách ca kiểm thử (Test Cases)

> Cột `Actual result` để trống cho tới khi có kiểm thử thật trên Packet Tracer 9.0.1.0858. Hiện **TC-01, TC-02, TC-03, TC-04, TC-05 và TC-06 đã `Passed`**; các test case còn lại vẫn **Not Tested**. Kế hoạch tổng thể và phân loại Core/Conditional xem tại [../docs/testing-plan.md](../docs/testing-plan.md). RADIUS (TC-13) là Conditional: `Blocked` ở đó không làm mất trạng thái hoàn thành của luồng Core.
>
> **Lưu ý sau audit lựa chọn thiết bị (2026-10-18):** quyết định đổi controller từ `WLC-3504` sang **`WLC-PT`**. **TC-01 đã chạy lại và `Passed` trên `WLC-PT`** (bản test copy) — Management IPv4 `192.168.99.10/24`, gw `192.168.99.1`, đặt trên cổng GUI hiển thị là `Management` (API/mô phỏng đặt tên `Vlan1`), **đặt được bằng API, không cần raw JS**. **TC-03 đã kiểm chứng và `Passed`**: `WLC2 → Config → AP Groups` hiển thị `LAP1` (`0001.968E.6B01`) `Status: Online` trong nhóm `default-group`; `CAPWAP.PRIMARY_AC` của cả `LAP-PT` và `WLC-PT` là `0.0.0.0` (PT dùng auto-discovery, không có trường đặt IP controller trong GUI). **`.pkt` deliverable đã được thay sang `WLC-PT` (2026-10-19)** — hết chờ. Xem [../docs/device-selection-decision.md](../docs/device-selection-decision.md). `Expected result` **không** sửa theo kết quả thực tế.
>
> **Cập nhật 2026-10-19 — TC-04 `Passed`:** WLAN `STAFF-WIFI` (WLAN ID `1`, VLAN `10`, WPA2-PSK/AES) đã tạo trên `WLC2` và **phát sóng được xác nhận bằng quét thật từ client `STAFF1`**; chưa associate (TC-06) và chưa kiểm chứng map VLAN (TC-08). Minh chứng: `../evidence/tc-04-staff-wlan.png`, `../evidence/tc-04-wlc-staff-wlan-config.png` (PSK đã che).
>
> **Cập nhật 2026-10-19 — TC-06 `Passed`:** cả hai Staff client `STAFF1` và `STAFF2` (client thứ hai được bổ sung vào deliverable vì Procedure yêu cầu 2 client) đã **associate thật** vào `STAFF-WIFI` bằng WPA2-PSK qua GUI `PC Wireless`. Minh chứng chính: `../evidence/tc-06-staff-associate.png` — **một ảnh chụp màn hình thật**, hai cửa sổ `STAFF1`/`STAFF2` đặt cạnh nhau, cùng mở trang live `Wireless Network Status`: cả hai hiển thị `Wireless Network Name: STAFF-WIFI`, `Security: WPA2-Personal`, BSSID `0001.968E.6B02`, channel `1 - 2.412GHz` (PSK không xuất hiện trên trang này). Đối chứng trước khi nối của `STAFF2`: `../evidence/tc-06-staff2-noassoc-control.png` (`no association with access point`). IPv4 của client hiện là APIPA `169.254.x.x` — DHCP/map VLAN đúng subnet là phạm vi TC-08, **chưa kết luận**.

## TC-01 — WLC management reachability (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-01 |
| **Preconditions** | File `.pkt` mở được trên 9.0.1.0858; WLC đã đấu nối có dây và đặt Management IPv4 theo [../docs/ip-vlan-plan.md](../docs/ip-vlan-plan.md) |
| **Procedure** | 1. Từ một thiết bị cùng/thông được VLAN 99 `ping` tới Management IPv4 của WLC (ví dụ `192.168.99.10`). 2. Ghi model WLC và địa chỉ đã dùng. |
| **Expected result** | Ping tới WLC thành công (reply) |
| **Actual result** | **Đã chạy lại trên `WLC-PT` — Passed.** Model `WLC-PT` (device type 41), cổng thật đọc từ mô phỏng: **`Vlan1`** (mang IP quản trị; GUI hiển thị tên `Management`) + **`GigabitEthernet0`** (uplink). Management IPv4 `192.168.99.10/24`, gateway `192.168.99.1`, đặt bằng API đặt IP (`pt_host_ip_config`) — **không cần raw JS**. Uplink `WLC-PT GigabitEthernet0` → `SW1 FastEthernet0/4` **trunk** native VLAN 99, allowed 10,20,99 (frame untagged của `Vlan1` rơi vào native 99 của switch). Nguồn ping: `SRV1` (`192.168.99.20`) `Sent = 4, Received = 4, Lost = 0 (0% loss)`; `R1` (`192.168.99.1`) `Success rate is 100 percent (5/5)`. Bản `WLC-3504` cũ giữ kết quả `Passed` làm lịch sử P1. **Lưu ý model:** khác `WLC-3504`, `WLC-PT` **không có** cổng API tên `management` riêng; GUI gọi cổng quản trị là `Management`, API gọi là `Vlan1`. Xem [../docs/device-selection-decision.md](../docs/device-selection-decision.md) và [../docs/implementation-notes.md](../docs/implementation-notes.md) §5. |
| **Evidence filename** | `evidence/tc-01-wlc-pt-reachability-canvas.png` (chạy lại trên `WLC-PT`); `evidence/tc-01-wlc-reachability.png` (bản `WLC-3504`, lịch sử) |
| **Status** | Passed (chạy lại trên `WLC-PT`; nay là controller trong `.pkt` deliverable) |

## TC-02 — LAP-PT nhận IP quản trị hợp lệ (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-02 |
| **Preconditions** | TC-01 đạt hoặc hạ tầng VLAN 99 thông; 01 LAP-PT đã nối dây tới switch |
| **Procedure** | 1. Kiểm tra địa chỉ IP của LAP-PT-1 (DHCP hoặc tĩnh trong `192.168.99.0/24`). 2. Ghi model LAP-PT và cách cấp IP. |
| **Expected result** | LAP-PT có IP hợp lệ thuộc `192.168.99.0/24` và ping được gateway `192.168.99.1` |
| **Actual result** | **Chạy lại & kiểm chứng ngày 2026-10-19 trên bản làm việc riêng `packet-tracer/enterprise-wifi-wlc-lab-v1_TC02_TEST.pkt` (không sửa file gốc) — Passed.** Model `LAP-PT` (tên `LAP1`); cổng uplink thật `GigabitEthernet0` (catalog ghi `Port 0` — sai) nối `SW1 FastEthernet0/3`; `show interfaces FastEthernet0/3 switchport` = `Administrative Mode: static access`, `Access Mode VLAN: 99 (MANAGEMENT)`; VLAN 99 `active` (`Fa0/2`,`Fa0/3`). **Trạng thái ban đầu khi mở lại:** sau khi PT nạp lại `.pkt`, `LAP1` tự về chế độ DHCP và nhận `192.168.99.3` từ `SRV1` (cùng MAC `0001.968E.6B01`) — ghi nhận `ping R1 → 192.168.99.30 = 0/5` (chưa đúng). Cấu hình mong đợi: IP tĩnh `192.168.99.30/24`, gateway `192.168.99.1`, DNS `192.168.99.20` — đọc từ XML `serializeToXml()` vì **API `LAP-PT` không có getter** `getDefaultGateway`/`getDnsServerIp` (chỉ có setter) và GUI tab `GigabitEthernet0` ghi `Configurations are not available`. Sau khi ghi nhận trạng thái ban đầu, **đã đặt lại IP tĩnh** ở bản làm việc bằng `pt_host_ip_config` (mask `255.255.255.0`, gw `.1`, DNS `.20`); API xác nhận `ip=192.168.99.30`, `dhcp=false`. **Giới hạn của model:** GUI `Global Settings` của `LAP-PT` **không hiển thị** địa chỉ IPv4 và radio vẫn ở **DHCP** kể cả sau `setDhcpClientFlag(false)` (bất nhất nội bộ của PT) ⇒ `evidence/tc-02-lap-ip.png` chỉ thể hiện Display Name + gateway/DNS, còn địa chỉ `.30` được chứng minh gián tiếp qua ARP/ping phía gateway. LAP-PT **không có console/CLI** nên không `ping` trực tiếp được; thay thế bằng Simple PDU `LAP1 → R1` (`ipc.appWindow().getUserCreatedPDU().addSimplePdu()`) và **packet trace ở Simulation mode**: LAP1 `Gi0` → SW1 `Fa0/3` → R1 `Gi0/0` (ICMP Echo Request, R1 `ICMP process replies to the Echo Request`) → SW1 → LAP1 **accepted** ⇒ round-trip tới gateway `192.168.99.1` thành công. Kiểm chứng thật sau khi đặt tĩnh: `R1 → 192.168.99.30` = `Success rate is 100 percent (5/5)`; `show ip arp` R1 có `192.168.99.30 → 0001.968E.6B01`; MAC-table SW1 VLAN 99 `0001.968e.6b01` trên `Fa0/3`; `show interfaces trunk` SW1: `Gig0/1` + `Fa0/4` native 99, allowed `10,20,99`; đường VLAN 99 tới R1 `Gi0/0 192.168.99.1 up/up`. Xem [../docs/implementation-notes.md](../docs/implementation-notes.md) §5. |
| **Evidence filename** | `evidence/tc-02-lap-ip.png` (Display Name + gateway/DNS; IP không hiển thị trên GUI LAP-PT), `evidence/tc-02-lap-ping-gateway.png`, `evidence/tc-02-lap-arp-r1.png`, `evidence/tc-02-sw1-mac-fa03.png` |
| **Status** | Passed (chạy lại & kiểm chứng 2026-10-19, bản làm việc riêng `enterprise-wifi-wlc-lab-v1_TC02_TEST.pkt`) |

## TC-03 — LAP-PT join WLC (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-03 |
| **Preconditions** | TC-02 đạt; controller đã đổi sang `WLC-PT` và TC-01 đã chạy lại; LAP-PT đã đăng ký về controller theo cơ chế join của model (audit chưa tìm thấy trường `Primary Controller` trong tab Config của `LAP-PT`) |
| **Procedure** | 1. Mở GUI/CLI controller, kiểm tra danh sách AP đã join. 2. Ghi model LAP-PT, controller và trạng thái join. Chỉ thêm AP thứ 2 sau khi TC này và TC-06 đã `Passed` (điều kiện mở rộng AP tại [../docs/testing-plan.md](../docs/testing-plan.md)). |
| **Expected result** | LAP-PT-1 hiển thị Registered/Joined trên WLC |
| **Actual result** | **Passed** — kiểm chứng 2026-10-19. Bản test copy đã được **promote** thành `packet-tracer/enterprise-wifi-wlc-lab-v1.pkt` (deliverable) và **re-audit lại trên chính file deliverable**. `WLC2` (`WLC-PT`) `Vlan1`/GUI `Management` = `192.168.99.10/24`, gw `192.168.99.1`, DNS `192.168.99.20` (đọc từ XML `serializeToXml()` — **có lưu trong `.pkt`**); `R1 Gi0/0` = `192.168.99.1/24`; liên kết `LAP1 Gi0 ↔ SW1 Fa0/3` (access VLAN 99), `WLC2 Gi0 ↔ SW1 Fa0/4` (trunk native 99, allowed 10,20,99). **Bằng chứng join/registration tường minh:** `WLC2 → Config → AP Groups` (group `default-group`) liệt kê AP `LAP1`, MAC `0001.968E.6B01`, **`Status: Online`**, cột `In AP Group` được tick. Hỗ trợ thêm: SW1 MAC-table VLAN 99 có `0001.968e.6b01` (Fa0/3) và `00e0.f700.ea6e` — WLC (Fa0/4); R1 ARP `.30 → 0001.968E.6B01`, `.10 → 00E0.F700.EA6E`; `R1 → 192.168.99.30` = `5/5`, `R1 → 192.168.99.10` = `5/5` (lần đầu thấp hơn do ARP delay). **Giới hạn quan trọng (phát hiện khi re-audit):** mở `.pkt` mới, `LAP1 Gi0` trở về **DHCP** (XML `PORT_DHCP_ENABLE=true`, `IP`/`SUBNET`/`PORT_GATEWAY` trống) ⇒ AP hiển thị `Offline` và không có IP; **đặt lại tĩnh `192.168.99.30/24` (gw `.1`, DNS `.20`) qua API (`pt_host_ip_config`) thì `LAP1` rejoin → `Status: Online`**. **IP tĩnh của `LAP-PT` và trạng thái join KHÔNG được lưu vào `.pkt`** (chỉ tồn tại trong phiên làm việc) — **phải đặt lại sau mỗi lần mở file**. Giới hạn khác (không suy diễn): `LAP-PT`/`WLC-PT` không có console/Desktop ⇒ **không ping trực tiếp** LAP1↔WLC/LAP1→gateway (`NOT VERIFIED`), chỉ kiểm chứng gián tiếp qua R1/ARP/MAC; `CAPWAP.PRIMARY_AC = 0.0.0.0` ở cả hai model và không có trường đặt IP controller trong GUI ⇒ PT **auto-discovery**; không có API trả trạng thái AP (chỉ cột `Status` trên GUI); **không có minh chứng bắt tay CAPWAP packet-level**; không cấu hình WLAN/DHCP/ACL trong TC-03. |
| **Evidence filename** | `evidence/tc-03-lap-wlc-join.png` (chụp lại 2026-10-19 từ file deliverable, `LAP1` `Status: Online`) |
| **Status** | Passed (chứng minh bằng trạng thái controller `Online`; IP tĩnh/join là trạng thái runtime, phải đặt lại sau khi mở `.pkt`) |

## TC-04 — WLAN STAFF-WIFI được tạo và phát sóng (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-04 |
| **Preconditions** | TC-03 đạt; WLAN `STAFF-WIFI` đã cấu hình trên trang `Wireless LANs` của WLC |
| **Procedure** | 1. Từ Staff client quét mạng vô tuyến tại vùng phủ LAP-PT-1. 2. Ghi SSID, WLAN ID và VLAN đã cấu hình. |
| **Expected result** | Nhìn thấy `STAFF-WIFI` và associate thử được ở bước TC-06 |
| **Actual result** | **Passed** — kiểm chứng 2026-10-19 trên `.pkt` deliverable `packet-tracer/enterprise-wifi-wlc-lab-v1.pkt` (đã thêm client `STAFF1`). **Tạo WLAN:** trường WLAN không có API trong bản 9.0.1.0858 nên thực hiện trên GUI `WLC2 → Config → Wireless LANs` (điều khiển GUI tự động bằng UI Automation của gói MCP — vẫn là kênh GUI, không phải API): `Name` = `SSID` = `STAFF-WIFI`, `VLAN` = `10`, Authentication = `WPA2-PSK`, `Encryption Type` = `AES` (chọn WPA2-PSK tự đặt AES), passphrase kiểm thử 15 ký tự (**che khi chụp, không ghi giá trị**), trạng thái `Enabled`; WLAN nằm trong nhóm AP `default-group`. **Đối chiếu XML `serializeToXml()`:** `<WLAN_CONFIG>` có `NAME`/`SSID` `STAFF-WIFI`, `AUTHEN_TYPE 4` (WPA2-PSK), `ENCRYPT_TYPE 4`, `VLAN 10`, `WLAN ID 1`, `ENABLED true`; `AP_GROUP default-group` liệt kê WLAN `STAFF-WIFI`. **Phát sóng — xác nhận bằng quét thật:** client `STAFF1` (`Laptop-PT`, cổng vô tuyến `Wireless0`, MAC `0001.968E.6B02`) mở `PC Wireless` → `Connect` → danh sách mạng thấy **`STAFF-WIFI`**, channel `1`, tín hiệu `59%`, `Security Setting: WPA2-PSK`, Wireless Mode `Infrastructure`, Network Type `Mixed B/G/N`. **Chưa làm (ngoài phạm vi TC-04, có chủ đích):** chưa associate vào SSID (TC-06), chưa kiểm chứng map SSID→VLAN 10 cho client (TC-08); `Central Control` của WLAN giữ nguyên mặc định model (`Local switching, local authentication`) — chưa kết luận. **Giới hạn đã ghi nhận:** PT lưu passphrase WPA2-PSK **dạng rõ** trong XML `.pkt` (`<WEP_KEY>`) — đã che khi chụp và không ghi giá trị vào tài liệu; trạng thái `LAP1` (IP tĩnh + join) là runtime — phải đặt lại sau khi mở file thì SSID mới phát. |
| **Evidence filename** | `evidence/tc-04-staff-wlan.png` (quét SSID từ `STAFF1`), `evidence/tc-04-wlc-staff-wlan-config.png` (trang `Wireless LANs` của `WLC2`, PSK đã che) |
| **Status** | Passed (2026-10-19; broadcast xác nhận bằng quét; associate để TC-06; map VLAN để TC-08) |

## TC-05 — WLAN GUEST-WIFI được tạo và phát sóng (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-05 |
| **Preconditions** | TC-03 đạt; WLAN `GUEST-WIFI` đã cấu hình độc lập với `STAFF-WIFI` |
| **Procedure** | 1. Từ Guest client quét mạng vô tuyến tại vùng phủ LAP-PT-1. 2. Xác nhận hai WLAN tồn tại đồng thời (nếu model cho phép). |
| **Expected result** | Nhìn thấy `GUEST-WIFI` đồng thời với `STAFF-WIFI` (hoặc ghi rõ giới hạn model nếu không) |
| **Actual result** | **Passed** — kiểm chứng 2026-10-19 trên `.pkt` deliverable `packet-tracer/enterprise-wifi-wlc-lab-v1.pkt` (đã thêm client `GUEST1`). **Tạo WLAN độc lập thứ hai:** trên GUI `WLC2 → Config → Wireless LANs` (điều khiển GUI tự động bằng UI Automation của gói MCP — WLAN không có API trong bản 9.0.1.0858, cùng cách làm TC-04): `Name` = `SSID` = `GUEST-WIFI`, `VLAN` = `20`, Authentication `WPA2-PSK`, Encryption `AES` (chọn WPA2-PSK tự đặt AES), passphrase kiểm thử riêng 15 ký tự (**khác PSK của `STAFF-WIFI`; che khi chụp, không ghi giá trị**), trạng thái `Enabled`; không thay đổi `STAFF-WIFI`. **Đối chiếu XML `serializeToXml()`:** có **2** `<WLAN_CONFIG>` — `STAFF-WIFI` nguyên vẹn (`ID 1`, `VLAN 10`, `AUTHEN_TYPE 4`, `ENCRYPT_TYPE 4`, `ENABLED true`) và `GUEST-WIFI` (`ID 2`, `VLAN 20`, `AUTHEN_TYPE 4`, `ENCRYPT_TYPE 4`, `ENABLED true`); `AP_GROUP default-group` liệt kê **cả hai** WLAN. **Phát sóng đồng thời — xác nhận bằng quét thật:** client mới `GUEST1` (`Laptop-PT`, cổng vô tuyến `Wireless0`) mở `PC Wireless` → `Connect`: danh sách mạng hiển thị đồng thời **`STAFF-WIFI`** (channel `1`, `63%`) và **`GUEST-WIFI`** (channel `1`, `63%`); `Security Setting: WPA2-PSK`, `Wireless Mode: Infrastructure`. **Kiểm chứng lưu file:** lưu `.pkt` + mở lại — cả hai WLAN còn nguyên (đọc lại XML) và **quét lại từ `GUEST1` vẫn thấy hai SSID đồng thời**. **Chưa làm (ngoài phạm vi TC-05, có chủ đích):** chưa associate client vào SSID (TC-06/TC-07), chưa kiểm chứng map SSID→VLAN cho client (TC-08), không cấu hình RADIUS/guest isolation. **Giới hạn đã ghi nhận:** passphrase các WLAN lưu **dạng rõ** trong XML `.pkt` (`<WEP_KEY>`) — đã che khi chụp và không ghi giá trị; trạng thái `LAP1` (IP tĩnh + join) là runtime — phải đặt lại `192.168.99.30` sau khi mở file thì SSID mới phát (đã đặt lại và quét xác nhận trong lần này). |
| **Evidence filename** | `evidence/tc-05-guest-wlan.png` (quét từ `GUEST1` — thấy đồng thời hai SSID), `evidence/tc-05-wlc-guest-wlan-config.png` (trang `Wireless LANs` của `WLC2` với `GUEST-WIFI`, PSK đã che) |
| **Status** | Passed (2026-10-19; broadcast đồng thời xác nhận bằng quét; associate để TC-06/TC-07; map VLAN để TC-08) |

## TC-06 — Staff associate bằng WPA2-PSK (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-06 |
| **Preconditions** | TC-04 đạt; `STAFF-WIFI` dùng WPA2-PSK |
| **Procedure** | 1. Kết nối Staff client 1 và 2 vào `STAFF-WIFI` bằng PSK đúng (che secret khi chụp). 2. Ghi trạng thái association. |
| **Expected result** | Cả hai Staff client associate thành công |
| **Actual result** | **Passed** — kiểm chứng 2026-10-19 trên `.pkt` deliverable `packet-tracer/enterprise-wifi-wlc-lab-v1.pkt` (đã **bổ sung client thứ hai `STAFF2`** `Laptop-PT` để đủ "Staff client 1 và 2"; `.pkt` lưu lại lúc 68,402 bytes). **`STAFF1`:** mở `PC Wireless` → `Connect`, quét thấy `STAFF-WIFI` (`WPA2-PSK`, channel `1`); chọn SSID → hộp thoại WPA2-PSK → nhập PSK thật (15 ký tự, đọc trực tiếp từ cấu hình `WLC2` ngay lúc chạy — **không in/không ghi giá trị**) → hộp thoại đóng, **không lỗi** (`m_errorLabel` trống). **`STAFF2`:** lần đầu lắp adapter `PT-LAPTOP-NM-1W` bị applet từ chối bằng hộp thoại *"A WMP300N or WPC300N wireless interface is required to connect."* — sửa bằng cách lắp đúng adapter **`Linksys-WPC300N`** (như `STAFF1`) → applet mở bình thường; **đối chứng trước khi nối:** applet hiển thị `no association with access point` / `Adapter is Inactive`; nhập cùng PSK → hộp thoại đóng, không lỗi. **Association kiểm chứng bằng trang live `Wireless Network Status` (nút `More Info`):** `STAFF1` — `Wireless Network Name: STAFF-WIFI`, `Mac Address 0001.968E.6B02` (BSSID radio của `LAP1`), `Security: WPA2-Personal`, `Standard Channel: 1 - 2.412GHz`, IPv4 `169.254.50.223`; `STAFF2` — cùng SSID/BSSID/security/channel, IPv4 `169.254.58.5`. Adapter chỉ đổi IPv4 từ `0.0.0.0` sang APIPA (`169.254.0.0/16`) **sau khi** nối thành công (tốc độ link adapter đồng thời đổi 11 → 300 Mbps) — phù hợp việc **không** có DHCP trên WLAN; subnet/DHCP đúng là phạm vi TC-08, **chưa kết luận**. **Lưu ý bằng chứng:** trang `Link Information` của applet là minh họa **tĩnh** (giống hệt trước/sau khi nối) — **không dùng** làm bằng chứng; bằng chứng là trang live `Wireless Network Status` + **một ảnh chụp màn hình thật** với hai cửa sổ cạnh nhau. **Giới hạn:** không có minh chứng bắt tay WPA2 (4-way handshake) packet-level — PT không hiển thị; `WLC2` GUI không có trang danh sách client (việc xác nhận AP qua `AP Groups` thuộc TC-03); độ bền associate sau khi mở lại `.pkt` **chưa xác minh**; chưa thử PSK sai (TC-14); không cấu hình thêm WLAN/VLAN/routing/DHCP. |
| **Evidence filename** | `evidence/tc-06-staff-associate.png` (một ảnh chụp màn hình thật, hai cửa sổ `STAFF1` + `STAFF2` cạnh nhau cùng mở trang `Wireless Network Status`; PSK không hiển thị), `evidence/tc-06-staff2-noassoc-control.png` (đối chứng trước khi nối của `STAFF2`: `no association with access point`) |
| **Status** | Passed (2026-10-19; cả hai Staff client associate thật bằng WPA2-PSK; DHCP/map VLAN để TC-08; PSK sai để TC-14) |

## TC-07 — Guest associate bằng WPA2-PSK (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-06 |
| **Preconditions** | TC-05 đạt; `GUEST-WIFI` dùng WPA2-PSK |
| **Procedure** | 1. Kết nối Guest client 1 và 2 vào `GUEST-WIFI` bằng PSK đúng (che secret khi chụp). 2. Ghi trạng thái association. |
| **Expected result** | Cả hai Guest client associate thành công |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-07-guest-associate.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-08 — Client nhận IP đúng subnet (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-07, REQ-08 |
| **Preconditions** | TC-06, TC-07 đạt; DHCP trên Server-PT đã cấu hình cho VLAN 10/20 |
| **Procedure** | 1. Chạy `ipconfig` trên 1 Staff và 1 Guest client. 2. Đối chiếu với [../docs/ip-vlan-plan.md](../docs/ip-vlan-plan.md). |
| **Expected result** | Staff nhận IP `192.168.10.0/24`; Guest nhận IP `192.168.20.0/24` |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-08-staff-ipconfig.png`, `evidence/tc-08-guest-ipconfig.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-09 — Client tới được gateway (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-09 |
| **Preconditions** | TC-08 đạt |
| **Procedure** | 1. Từ Staff `ping 192.168.10.1`. 2. Từ Guest `ping 192.168.20.1`. |
| **Expected result** | Ping tới gateway của đúng VLAN thành công (reply) |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-09-ping-gateway.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-10 — Guest không tới được Staff (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-11 |
| **Preconditions** | TC-08 đạt; ACL đã áp trên router (explicit deny Guest→Staff + permit tối thiểu) |
| **Procedure** | 1. Từ Guest (`192.168.20.0/24`) `ping` tới IP Staff client và `192.168.10.1`. 2. Ghi kết quả trước/sau ACL. |
| **Expected result** | Ping từ Guest tới Staff thất bại sau ACL (Request timed out / Destination unreachable) |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-10-guest-to-staff.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-11 — Guest không tới được thiết bị quản trị (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-11 |
| **Preconditions** | TC-08 đạt (Guest đã nhận IP qua DHCP — chứng minh ngoại lệ DHCP đang hiệu lực); ACL đã áp; địa chỉ quản trị chốt theo [../docs/ip-vlan-plan.md](../docs/ip-vlan-plan.md) |
| **Procedure** | 1. Từ Guest `ping` tới địa chỉ quản trị (ví dụ WLC `192.168.99.10`) → phải thất bại. 2. Từ Guest `ping` tới switch/router trong VLAN 99 → phải thất bại. 3. Xác nhận Guest **vẫn nhận được IP** từ Server-PT `192.168.99.20` (`ipconfig`, đối chiếu TC-08). 4. Chạy `show access-lists` / `show running-config` trên router. |
| **Expected result** | Guest không tới được **thiết bị quản trị** (WLC, switch, router, LAP-PT), nhưng **vẫn nhận được IP** từ Server-PT qua DHCP; ACL hiển thị khớp [../docs/security-design.md](../docs/security-design.md). Guest **vẫn tới được** `192.168.99.20` cho DHCP — đây là ngoại lệ có chủ đích, không phải lỗi |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-11-guest-to-mgmt.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-12 — VLAN đúng + Staff chỉ giữ truy cập cho phép + ACL hiển thị (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-07, REQ-11 |
| **Preconditions** | Switch đã cấu hình VLAN; router đã áp ACL |
| **Procedure** | 1. Chạy `show vlan brief` (hoặc lệnh tương đương đúng model). 2. Kiểm tra cổng theo [../docs/topology.md](../docs/topology.md). 3. Từ Staff kiểm tra chỉ còn đúng truy cập nội bộ cho phép theo phạm vi đã chốt trước trong [../docs/implementation-notes.md](../docs/implementation-notes.md) (§4). |
| **Expected result** | VLAN 10/20/99 tồn tại, cổng đúng thiết kế đã chốt; Staff không vượt phạm vi cho phép |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-12-show-vlan.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-13 — RADIUS/WPA2-Enterprise cho Staff (Conditional)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Conditional — chỉ thử sau khi Core ổn định và khi Server-PT + model WLC hỗ trợ |
| **Requirement** | REQ-12 |
| **Preconditions** | TC-06 `Passed`; RADIUS đã cấu hình trên Server-PT và WLC (IP + shared secret, che khi chụp) |
| **Procedure** | 1. Kết nối Staff client bằng tài khoản RADIUS đúng. 2. Ghi kết quả associate và IP nhận được. Nếu Server-PT/model không hỗ trợ, ghi `Blocked` kèm lý do. |
| **Expected result** | Associate thành công qua RADIUS (hoặc `Blocked` có lý do nếu không hỗ trợ) |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-13-radius.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-14 — Sai credential bị từ chối (Core cho PSK, Conditional cho RADIUS)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core (WPA2-PSK) + Conditional (RADIUS) |
| **Requirement** | REQ-10 |
| **Preconditions** | TC-06/TC-07 đạt (PSK); TC-13 đã thử (cho phần RADIUS, nếu có) |
| **Procedure** | 1. Kết nối bằng PSK sai → phải bị từ chối. 2. Nếu đã tới TC-13, kết nối bằng tài khoản RADIUS sai → phải bị từ chối. |
| **Expected result** | Sai PSK bị từ chối, không nhận IP nội bộ (bắt buộc); sai tài khoản RADIUS bị từ chối (nếu đã triển khai RADIUS) |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-14-invalid-psk.png`, `evidence/tc-14-invalid-radius.png` (dự kiến, chưa có; file RADIUS chỉ cần khi TC-13 được thử) |
| **Status** | Not Tested |

> Research-only (không có test case): WPA3, EAP-TLS/PEAP, roaming, nhiễu RF, đo throughput — chỉ làm nền báo cáo.
