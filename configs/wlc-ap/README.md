# WLC & AP Configs

Lưu bản sao cấu hình/ghi chú cấu hình của WLC và LAP-PT (screenshot GUI WLC, export nếu có, danh sách WLAN/SSID). Người sở hữu: Đặng Văn Hoàng.

## Hiện trạng

> **Hạ tầng WLC + LAP-PT đã cấu hình và kiểm chứng — TC-01, TC-02 `Passed` ngày 2026-10-18, TC-03 và TC-04 `Passed` ngày 2026-10-19.** Trong đó **TC-01 đã chạy lại và `Passed` trên `WLC-PT`** (bản test copy `..._TESTCOPY_2026-10-18.pkt`): management `192.168.99.10/24` ping `0% loss`; **TC-03 xác nhận `LAP-PT` join `WLC-PT` (`Status: Online`)**; **TC-04 `Passed`: WLAN `STAFF-WIFI` đã tạo và phát sóng — quét thấy từ client `STAFF1`** (thao tác trên GUI `Wireless LANs` — không có API cho WLAN). **`.pkt` deliverable đã được thay sang `WLC-PT`** (2026-10-19).
>
> **Đã chuyển controller (2026-10-18 → hoàn tất 2026-10-19):** audit lựa chọn thiết bị phát hiện `WLC-3504` **không có giao diện cấu hình vô tuyến** trong bản 9.0.1.0858 (chỉ `GLOBAL/Settings` + `INTERFACE`), nên không thể tạo WLAN/SSID/WPA2. Quyết định đổi sang **`WLC-PT`** — xem [../../docs/device-selection-decision.md](../../docs/device-selection-decision.md). **Mục `WLC-3504` bên dưới giữ lại làm lịch sử của giai đoạn P1** (cấu hình `WLC-PT` đã thay thế trong `.pkt` deliverable).

### WLC-PT — controller hiện tại (đã thay trong `.pkt` deliverable)

| Hạng mục | Giá trị | Ghi chú |
| --- | --- | --- |
| Model | `WLC-PT` | Đặt làm `WLC2` trong `.pkt` deliverable (đã thay `WLC1`/`WLC-3504` ngày 2026-10-19) |
| Cổng thật | `Vlan1` (mang IP quản trị; GUI hiển thị `Management`) + `GigabitEthernet0` (uplink) | đọc trực tiếp từ API/mô phỏng; **khác `WLC-3504` ở chỗ không có cổng API `management`/`virtual` riêng** |
| Management IPv4 | `192.168.99.10/24`, gw `192.168.99.1`, DNS `192.168.99.20` | **đã đặt** trên cổng GUI hiển thị `Management` (API port `Vlan1` — **giống** `WLC-3504`), bằng API `pt_host_ip_config` — **không cần raw JS**; TC-01 chạy lại `Passed` |
| Uplink | `GigabitEthernet0` → SW1 `Fa0/4` | **đã chốt cho management**: SW1 `Fa0/4` **trunk** native VLAN 99, allowed `10,20,99`; phía WLC không cần set trunk (GUI không có tuỳ chọn VLAN/trunk); management ping được (TC-01 `Passed`). **Lưu lượng client VLAN 10/20 chưa chốt** (TC-08) |
| WLAN | **`STAFF-WIFI` đã tạo (2026-10-19)** | Name=SSID `STAFF-WIFI`, `VLAN 10`, WLAN ID `1` (XML), Authentication `WPA2-PSK`, Encryption `AES`, passphrase kiểm thử 15 ký tự (**không ghi giá trị** — PT lưu dạng rõ trong XML `.pkt`), `Enabled`, thuộc `default-group`; TC-04 `Passed`. `GUEST-WIFI` chưa tạo. Trang `Wireless LANs` có `Name`, `SSID`, trường `VLAN`, Authentication (WEP/WPA-PSK/WPA2-PSK/WPA/WPA2 + RADIUS), `Encryption Type`, `Central Control` |
| AP Groups / DHCP | Có trang nhưng **chưa dùng** | nhóm vẫn dùng Server-PT làm nguồn DHCP; built-in DHCP của `WLC-PT` mặc định **TẮT** |

Minh chứng quan sát giao diện: [../../evidence/audit-wlc-pt-wireless-lans.png](../../evidence/audit-wlc-pt-wireless-lans.png), [../../evidence/audit-wlc-pt-management-ip.png](../../evidence/audit-wlc-pt-management-ip.png). Minh chứng chạy lại TC-01: [../../evidence/tc-01-wlc-pt-reachability-canvas.png](../../evidence/tc-01-wlc-pt-reachability-canvas.png).

### WLC-3504 — model cũ, giữ làm lịch sử P1 (TC-01 `Passed`)

| Hạng mục | Giá trị | Ghi chú |
| --- | --- | --- |
| Model | `WLC-3504` | device type thật đọc từ mô phỏng = **41**. **Không dùng làm controller** (không có giao diện vô tuyến) |
| Management IPv4 | `192.168.99.10/24` | đặt trên interface **`Vlan1`**, không phải `management` |
| Default gateway | `192.168.99.1` | R1 `Gi0/0` |
| Uplink | `GigabitEthernet1` | **trunk**, native VLAN **1**; nối SW1 `Fa0/1` (native VLAN 99, allowed 10,20,99) |
| MAC `Vlan1` | `00D0.BAD0.7DA9` | chính là MAC SW1 học được trên `Fa0/1` thuộc VLAN 99 |
| MAC `management` | `0003.E479.4739` | interface ảo type 48, **không** mang lưu lượng |
| MAC `virtual` | `0009.7C0C.2770` | IP `0.0.0.0`, mặc định |

**Vì sao native VLAN hai phía khác nhau (1 và 99):** SVI `Vlan1` của WLC chỉ hoạt động trong VLAN 1 nội bộ. Giữ native VLAN 1 phía WLC để frame của `Vlan1` đi **untagged**; SW1 nhận untagged và gán vào native VLAN 99 — Management IPv4 vẫn nằm đúng VLAN 99 theo thiết kế. Đặt native 99 ở cả hai phía đã thử và **thất bại 100%** (WLC không trả lời ARP).

**Đã sửa so với mặc định:** `GigabitEthernet1` mặc định là **access VLAN 1**; nếu để nguyên, frame đi vào VLAN 1 và bị trunk SW1 (allowed 10,20,99) drop — `show mac address-table` không học được MAC nào.

**Hạn chế khiến model này bị loại khỏi vai trò controller:** GUI Config chỉ có `GLOBAL/Settings` + `INTERFACE` (`GigabitEthernet1`–`5`, `Management`) — **không có trang tạo WLAN/SSID/VLAN/WPA2/RADIUS**. Minh chứng: [../../evidence/audit-wlc-3504-config-menu.png](../../evidence/audit-wlc-3504-config-menu.png).

**Đã xử lý (một phần):** `addTrunkVlans('10,20,99')` trên `WLC-3504` báo `Invalid arguments for IPC call` — không còn áp dụng cho `WLC-PT` (cổng không có tuỳ chọn trunk trong GUI). Trên `WLC-PT` chỉ cần đặt trunk ở phía `SW1 Fa0/4`; management (VLAN 99) thông (TC-01 `Passed`). **Còn mở:** lưu lượng client VLAN 10/20 (TC-08).

Minh chứng: [../../evidence/tc-01-wlc-reachability.png](../../evidence/tc-01-wlc-reachability.png).

### LAP-PT — AP quản trị (đã cấu hình, TC-02 `Passed`)

| Hạng mục | Giá trị | Ghi chú |
| --- | --- | --- |
| Model | `LAP-PT` | device type dùng khi thêm vào: `7` (accesspoint) |
| Cổng thật | `GigabitEthernet0` + `Dot11Radio0` | catalog ghi `Port 0` là **sai** |
| Uplink | `GigabitEthernet0` → SW1 `Fa0/3` | **access VLAN 99** (không phải trunk) |
| IPv4 | `192.168.99.30/24` | **tĩnh**, đặt qua API (`setIpSubnetMask`) |
| Gateway / DNS | `192.168.99.1` / `192.168.99.20` | tab Config > Settings chọn **Static** |
| MAC `GigabitEthernet0` | `0001.968E.6B01` | chính là MAC trong `show ip arp` của R1 và MAC table của SW1 (VLAN 99, `Fa0/3`) |

- **Cổng AP chỉ lên khi có cấu hình IP.** Lúc mới đấu cáp: `LAP1 Gi0` `up=false`, đèn link tắt, SW1 `Fa0/3` báo `line protocol down (disabled)` và không học MAC. `setDhcpClientFlag(true)` **không** làm cổng lên; đặt IP tĩnh thì cổng lên `up/up` ngay.
- **DHCP không dùng được cho LAP** vì lý do trên (cổng down ⇒ không gửi được DHCP Discover). Địa chỉ chốt: tĩnh `.30`.
- **LAP-PT không có CLI/console** (`getCommandLine()` trả `null`) ⇒ không chạy được lệnh `ping`. Đã kiểm chứng bằng **Simple PDU** của PT: `ipc.appWindow().getUserCreatedPDU().addSimplePdu('LAP1','R1')` → PDU List Window báo **Successful** (ICMP), và ping ngược `R1 → 192.168.99.30` = `5/5`.
- GUI của LAP **không có trường IP** — tab Config chỉ có radio `Ipv4 Static`/`Ipv4 DHCP` cho Gateway/DNS. Chọn radio `Static` trên GUI sẽ **xoá IP** đã đặt qua API (đã gặp và đặt lại).
- **Cơ chế join đã kiểm chứng (TC-03 `Passed`, 2026-10-19):** `LAP1` hiển thị `Status: Online` trong `AP Groups` của `WLC-PT` (nhóm `default-group`). Trước đó audit **không tìm thấy** trang `WLC` / trường `Primary Controller` trong tab Config của `LAP-PT`; cơ chế thực tế là **auto-discovery** của PT (không có trường đặt IP controller, `CAPWAP.PRIMARY_AC = 0.0.0.0` ở cả hai model — **không** đọc là bằng chứng). Chưa có minh chứng bắt tay CAPWAP. Minh chứng: [../../evidence/tc-03-lap-wlc-join.png](../../evidence/tc-03-lap-wlc-join.png).
- Uplink AP để **access VLAN 99** là đủ ở giai đoạn này: AP ở chế độ lightweight đóng gói lưu lượng client vào CAPWAP gửi về WLC, còn VLAN 10/20 đi ra uplink WLC. Phải kiểm chứng lại khi có client thật (TC-08).

Minh chứng: [../../evidence/tc-02-lap-ip.png](../../evidence/tc-02-lap-ip.png), [../../evidence/tc-02-lap-ping-gateway.png](../../evidence/tc-02-lap-ping-gateway.png), [../../evidence/tc-02-lap-arp-r1.png](../../evidence/tc-02-lap-arp-r1.png).

## Nội dung dự kiến (làm theo thứ tự)

1. **(xong)** Thay controller `WLC-3504` → `WLC-PT`; đặt Management IPv4 `192.168.99.10/24` trên cổng GUI `Management` (API `Vlan1`) bằng API; nối `WLC2 Gi0` → `SW1 Fa0/4` (trunk native 99, allowed 10,20,99); **TC-01 chạy lại `Passed`**. **Đã thay `WLC-3504` bằng `WLC-PT` trong `.pkt` deliverable** (2026-10-19).
2. **(xong)** LAP-PT join `WLC-PT` (TC-03) — **`Passed` 2026-10-19**: `LAP1` `Status: Online` trong `AP Groups`; PT auto-discovery. Minh chứng: [../../evidence/tc-03-lap-wlc-join.png](../../evidence/tc-03-lap-wlc-join.png).
3. **(xong một phần)** WLAN `STAFF-WIFI` → VLAN 10 với **WPA2-PSK** — **TC-04 `Passed` 2026-10-19** (tạo + phát sóng, quét thấy từ `STAFF1`); **còn** `GUEST-WIFI` → VLAN 20 (TC-05→TC-07).
4. RADIUS/WPA2-Enterprise (TC-13, Conditional) chỉ sau khi Core ổn định và khi được hỗ trợ.

AP standalone chỉ là dự phòng, không thuộc luồng chính (không có trường VLAN nên không thay WLC).

## Quy định

- Ghi rõ model WLC/LAP-PT, bản 9.0.1.0858, ngày ghi nhận (xem [../../docs/implementation-notes.md](../../docs/implementation-notes.md)).
- Che PSK và RADIUS secret (`<PSK-REDACTED>`, `<RADIUS-SECRET-REDACTED>`).
- Không khẳng định WLAN phát sóng, map VLAN hay RADIUS chạy được khi chưa có minh chứng tại `../../evidence/`.
- Nếu không export được text, lưu screenshot GUI vào `../../evidence/` và tóm tắt text tại đây.
