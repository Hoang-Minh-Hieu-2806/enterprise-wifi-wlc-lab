# Kế hoạch kiểm thử (Core / Conditional / Research-only)

> Cột kết quả thực tế (`Actual result`) chỉ điền khi có kiểm thử thật. Hiện **TC-01 → TC-08 đã `Passed`** (TC-01, TC-02 `2026-10-18`; TC-03 → TC-07 `2026-10-19`; TC-08 `2026-10-20` trên bản test copy, cấu hình đã lưu vào official `.pkt` `2026-10-21`, kèm giới hạn); các test case còn lại (TC-09 → TC-14) vẫn **Not Tested**. Chi tiết từng bước xem tại [../tests/test-cases.md](../tests/test-cases.md).
>
> **Sau audit lựa chọn thiết bị (2026-10-18):** controller đổi từ `WLC-3504` sang **`WLC-PT`**; **TC-01 đã chạy lại và `Passed` trên `WLC-PT`** (bản test copy). Xem [device-selection-decision.md](device-selection-decision.md).

## 1. Phân loại test

- **Core (bắt buộc, TC-01→TC-12, TC-14 cho WPA2-PSK):** quyết định đạt/không đạt của đồ án. Thứ tự ưu tiên: WLC + LAP-PT + WLAN + WPA2-PSK + IP/gateway + VLAN/ACL trước.
- **Conditional (TC-13 RADIUS, phần RADIUS của TC-14):** chỉ thử sau khi Core ổn định và khi Server-PT + model WLC hỗ trợ. `Blocked` kèm lý do được chấp nhận và không làm mất trạng thái hoàn thành của Core.
- **Research-only:** WPA3, EAP-TLS/PEAP, roaming, nhiễu RF, throughput — không có test case, chỉ làm nền báo cáo.

## 2. Ma trận truy vết (test coverage)

| Test ID | Loại | Yêu cầu liên quan | Nội dung kiểm thử |
| --- | --- | --- | --- |
| TC-01 | Core | REQ-01 | WLC management reachability |
| TC-02 | Core | REQ-02 | LAP-PT nhận IP quản trị hợp lệ |
| TC-03 | Core | REQ-03 | LAP-PT join WLC |
| TC-04 | Core | REQ-04 | WLAN `STAFF-WIFI` được tạo và phát sóng |
| TC-05 | Core | REQ-05 | WLAN `GUEST-WIFI` được tạo và phát sóng |
| TC-06 | Core | REQ-06 | Staff associate bằng WPA2-PSK |
| TC-07 | Core | REQ-06 | Guest associate bằng WPA2-PSK |
| TC-08 | Core | REQ-07, REQ-08 | Client nhận IP đúng subnet (DHCP Server-PT qua `ip helper-address`) |
| TC-09 | Core | REQ-09 | Client tới được gateway tương ứng |
| TC-10 | Core | REQ-11 | Guest không tới được Staff |
| TC-11 | Core | REQ-11 | Guest không tới được thiết bị quản trị (DHCP là ngoại lệ có chủ đích) |
| TC-12 | Core | REQ-07, REQ-11 | VLAN tồn tại đúng + Staff chỉ giữ truy cập cho phép + ACL hiển thị |
| TC-13 | Conditional | REQ-12 | RADIUS/WPA2-Enterprise cho Staff (thử sau cùng) |
| TC-14 | Core + Conditional | REQ-10 | Sai credential bị từ chối (PSK là Core; RADIUS là Conditional) |
| TC-01→TC-14 | Core + Conditional | REQ-13 | Bằng chứng: mọi TC Core có `Actual result` thực tế + file tồn tại trong `../evidence/`; báo cáo trung thực tại `../report/` |

## 3. Điều kiện chung và thứ tự chạy

1. File `.pkt` đúng bản 9.0.1.0858; topology đúng [topology.md](topology.md); địa chỉ đúng [ip-vlan-plan.md](ip-vlan-plan.md).
2. Chạy theo thứ tự TC-01→TC-12, TC-14 (PSK) trước; TC-13 và phần RADIUS của TC-14 sau cùng.
3. Bắt đầu với 01 LAP-PT; chỉ thêm AP khi TC-03 + TC-06 đã `Passed`.
4. Biết SSID, PSK kiểm thử, tài khoản RADIUS kiểm thử (nếu tới TC-13). **Không commit secret thật** — che khi chụp minh chứng.
5. Công cụ: `ping`, `ipconfig`, `show vlan brief`, `show access-lists` / `show running-config` (đúng model), quét wireless trên client, GUI WLC.

## 4. Quy định ghi kết quả và minh chứng

- Mỗi test case gồm: ID, loại, requirement, preconditions, procedure, expected, actual, evidence filename, status.
- `Actual result` để trống hoặc ghi `Not Tested` cho tới khi test thật.
- `Status` chỉ dùng: `Not Tested` / `Passed` / `Failed` / `Blocked` (ghi rõ lý do nếu `Blocked`, ví dụ model không hỗ trợ).
- Minh chứng đặt tại `../evidence/` theo tên trong test case (ví dụ `evidence/tc-01-wlc-reachability.png`); file phải tồn tại khi claim `Passed`.
- Không chỉnh `Expected result` để khớp lỗi thực tế; nếu kỳ vọng sai, cập nhật tài liệu thiết kế và nêu lý do.

## 5. Tiêu chí đạt (pass criteria)

- Core TC-01→TC-12 và TC-14 (PSK) phải `Passed` mới công nhận luồng cơ bản hoàn thành.
- TC-13 (RADIUS): `Passed` là cộng thêm; `Failed`/`Blocked` kèm lý do và evidence được chấp nhận mà không ảnh hưởng Core.
- TC-10/TC-11: Guest→Staff/Management phải không tới được sau ACL; TC-09 (gateway của chính Guest) vẫn thông.
- Mọi `Failed`/`Blocked` phải có nguyên nhân và hướng xử lý, không giấu lỗi.

## 6. Minh chứng bring-up P1 (không phải kết quả test case)

Ngoài minh chứng theo mã TC, `../evidence/` hiện có một ảnh chụp từ giai đoạn dựng khung:

- `p1-wired-skeleton-topology.png` — ảnh canvas Packet Tracer (2026-10-18) thể hiện 4 thiết bị của khung có dây và 3 liên kết đều up.

File này **không** đại diện cho test case nào và **không** được dùng để chuyển bất kỳ TC nào sang `Passed`. Cấu hình tương ứng lưu tại `../configs/`.

## 6.1 Minh chứng audit lựa chọn thiết bị (không phải kết quả test case)

Đợt audit 2026-10-18 khảo sát trực tiếp giao diện thiết bị trên đúng bản 9.0.1.0858 để chốt lại model controller và AP. Các ảnh sau **là minh chứng quan sát giao diện**, không phải kết quả test case, và **không** dùng để chuyển TC nào sang `Passed`. **Cảnh báo: các file dưới đây hiện KHÔNG có trong `../evidence/` — cần chụp bổ sung trước khi nghiệm thu.**

| File | Nội dung quan sát |
| --- | --- |
| `../evidence/audit-wlc-3504-config-menu.png` | `WLC-3504` tab Config chỉ có `GLOBAL/Settings` + `INTERFACE` (GigabitEthernet1–5, Management) — **không có trang WLAN** |
| `../evidence/audit-wlc-pt-wireless-lans.png` | `WLC-PT` trang `Wireless LANs`: Name, SSID, **VLAN**, Authentication (WEP/WPA-PSK/WPA2-PSK/WPA/WPA2 + RADIUS), Encryption Type, Central Control |
| `../evidence/audit-wlc-pt-management-ip.png` | `WLC-PT` trang `Management`: IPv4 Address, Subnet Mask, Default Gateway, DNS Server |
| `../evidence/audit-lap-pt-radio.png` | `LAP-PT` `Dot11Radio0` chỉ có Port Status + Coverage Range; **không thấy** trang `WLC`/`Primary Controller` |
| `../evidence/audit-ap-pt-port1-security.png` | AP standalone `Port 1`: SSID + WEP/WPA-PSK/WPA2-PSK, **không có trường VLAN** |

Quyết định dựa trên các minh chứng này: [device-selection-decision.md](device-selection-decision.md) (kèm checklist 15 mục ở §5).

## 7. Kết quả kiểm thử đã chạy

| Test ID | Ngày | Kết quả | Minh chứng | Ghi chú |
| --- | --- | --- | --- | --- |
| TC-01 | 2026-10-18 | `Passed` *(chạy lại trên `WLC-PT`, bản test copy)* | `../evidence/tc-01-wlc-pt-reachability-canvas.png` | Management IPv4 `192.168.99.10/24` trên cổng GUI `Management` (API `Vlan1`), đặt bằng API; ping từ `SRV1` `192.168.99.20`: 4/4 `0% loss`, và `R1` `5/5`. Uplink `WLC-PT Gi0` → `SW1 Fa0/4` trunk native 99 (không cần trunk phía WLC). Bản `WLC-3504` cũ: `../evidence/tc-01-wlc-reachability.png` — chi tiết tại [implementation-notes.md](implementation-notes.md) §5 |
| TC-02 | 2026-10-18 | `Passed` | `../evidence/tc-02-lap-ip.png`, `../evidence/tc-02-lap-ping-gateway.png`, `../evidence/tc-02-lap-arp-r1.png`, `../evidence/tc-02-sw1-mac-fa03.png` | `LAP-PT` trên `SW1 Fa0/3` (access VLAN 99), IP tĩnh `192.168.99.30/24`, gateway `192.168.99.1`. Simple PDU `LAP1 → R1` = Successful; ping ngược `R1 → .30` = 5/5. Cổng AP chỉ lên khi có cấu hình IP; LAP không có CLI — chi tiết tại [implementation-notes.md](implementation-notes.md) §5 |
| TC-03 | 2026-10-19 | `Passed` | `../evidence/tc-03-lap-wlc-join.png` | `WLC2 → Config → AP Groups` hiển thị `LAP1` (MAC `0001.968E.6B01`) `Status: Online` trong nhóm `default-group` — LAP-PT đã đăng ký/join `WLC-PT` qua auto-discovery của PT. Hỗ trợ: XML `CAPWAP_AC/APS` của WLC chứa MAC LAP; `R1 → .10`/`.30` = 5/5. **Giới hạn:** không có minh chứng bắt tay CAPWAP; `CAPWAP.PRIMARY_AC = 0.0.0.0` ở cả hai model không đọc là bằng chứng |
| TC-04 | 2026-10-19 | `Passed` | `../evidence/tc-04-staff-wlan.png`, `../evidence/tc-04-wlc-staff-wlan-config.png` | WLAN `STAFF-WIFI` tạo trên `WLC2` qua GUI (không có API cho WLAN): WLAN ID `1`, VLAN `10`, WPA2-PSK/AES, `Enabled`, thuộc `default-group` (đối chiếu XML `serializeToXml()`). Phát sóng xác nhận bằng quét từ `STAFF1` (`Laptop-PT`, `Wireless0`): thấy `STAFF-WIFI`, channel 1, 59%, `WPA2-PSK`. **Chưa** associate tại thời điểm TC-04 (TC-06 sau đó `Passed`); map VLAN cho client được kiểm chứng sau ở TC-08 (bản test copy). PSK lưu dạng rõ trong `.pkt` — đã che khi chụp, không ghi giá trị |
| TC-05 | 2026-10-19 | `Passed` | `../evidence/tc-05-guest-wlan.png`, `../evidence/tc-05-wlc-guest-wlan-config.png` | WLAN `GUEST-WIFI` tạo **độc lập** trên `WLC2` qua GUI: WLAN ID `2`, VLAN `20`, WPA2-PSK/AES, `Enabled`, thuộc `default-group` (đối chiếu XML `serializeToXml()` — 2 `<WLAN_CONFIG>`, `STAFF-WIFI` nguyên vẹn). Phát sóng **đồng thời** xác nhận bằng quét từ client mới `GUEST1` (`Laptop-PT`, `Wireless0`): thấy đồng thời `STAFF-WIFI` và `GUEST-WIFI`, channel 1, 63%, `WPA2-PSK`. Lưu + mở lại `.pkt`: cả hai WLAN còn nguyên, quét lại vẫn thấy hai SSID. **Chưa** associate tại thời điểm TC-05 (TC-06/TC-07 sau đó `Passed`); map VLAN cho client được kiểm chứng sau ở TC-08 (bản test copy). PSK lưu dạng rõ trong `.pkt` — đã che khi chụp, không ghi giá trị; `LAP1` cần đặt lại IP tĩnh runtime sau khi mở file |
| TC-06 | 2026-10-19 | `Passed` | `../evidence/tc-06-staff-associate.png`, `../evidence/tc-06-staff2-noassoc-control.png` | Hai Staff client `STAFF1` + `STAFF2` associate thật vào `STAFF-WIFI` (`WPA2-Personal`, BSSID `0001.968E.6B02`); IPv4 APIPA — DHCP/map VLAN ở TC-08. Chi tiết tại [../tests/test-cases.md](../tests/test-cases.md) TC-06 |
| TC-07 | 2026-10-19 | `Passed` | `../evidence/tc-07-guest1-associate-live.png`, `../evidence/tc-07-guest2-associate-live.png` | Hai Guest client `GUEST1` + `GUEST2` đang associate vào `GUEST-WIFI` (`WPA2-Personal`, BSSID `0001.968E.6B02` = radio `LAP1`); adapter `WPC300N` active; IPv4 APIPA tại thời điểm TC-07 — DHCP/map VLAN sau đó kiểm chứng ở TC-08 (bản test copy); gateway/Guest isolation vẫn **chưa kết luận**. Chạy trên phiên WIP TC-07 (có thêm `GUEST2`); file `.pkt` deliverable không đổi. Chi tiết tại [../tests/test-cases.md](../tests/test-cases.md) TC-07 |
| TC-08 | 2026-10-20 | **Passed for its stated IP-subnet acceptance criterion, with explicit limitations** | `../evidence/tc-08-staff-ipconfig.png`, `../evidence/tc-08-guest-ipconfig.png`, `../evidence/tc-08-trunk.png`, `../evidence/tc-08-mac-vlan.png`, `../evidence/tc-08-dhcp-pools.png`, `../evidence/tc-08-ap-online.png`, `../evidence/tc-08-guest-associate.png` | **Kiểm chứng trên bản test copy** `enterprise-wifi-wlc-lab-v1_TC08_TEST_20261020.pkt` (SHA-256 `9D787E30…`); cấu hình đã **lưu vào official** `.pkt` ngày 2026-10-21 (SHA-256 `EEA60292…`, 68.796 bytes) và mở lại kiểm chứng. `STAFF1` `192.168.10.102/24` và `GUEST1` `192.168.20.102/24` trên test copy; sau khi mở official và `/renew`: `.10.101/24`, `.20.100/24`, `STAFF2` `.10.100/24`; gateway `.10.1`/`.20.1`, DHCP `.99.20`. `SW1 Fa0/3` (LAP), `Fa0/4` (WLC) và `Gi0/1` (R1) đều trunk 802.1Q native `99`, allowed `10,20,99`; MAC VLAN 10/20 chỉ trên `Fa0/3` (local switching tại AP), **không** trên `Fa0/4` (WLC chỉ management); `LAP1` online `.99.30/24`. **Giới hạn:** central switching của WLC **chưa kiểm chứng**; `serverPool` không xoá vĩnh viễn được (PT tạo lại), chỉ giới hạn `.99.30–.30`, max 1; `LAP-PT` không có static config, `.30` qua DHCP chưa xác nhận lưu bền; lease DHCP không giữ sau khi mở lại, phải `/renew`. Chi tiết tại [../tests/test-cases.md](../tests/test-cases.md) TC-08 |
