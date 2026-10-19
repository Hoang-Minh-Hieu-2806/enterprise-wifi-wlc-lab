# Evidence

Thư mục lưu minh chứng kiểm thử trên bản 9.0.1.0858: ảnh chụp màn hình, kết quả lệnh, export cấu hình đã che secret. Huỳnh Như Hoàng duy trì; Đặng Văn Hoàng bổ sung phần WLC/AP; Hoàng Minh Hiếu ghi evidence cho phần mình kiểm thử.

## Hiện trạng

> **Đã có minh chứng bring-up P1:** [`p1-wired-skeleton-topology.png`](p1-wired-skeleton-topology.png) — ảnh canvas Packet Tracer chụp ngày 2026-10-18, thể hiện 4 thiết bị (`WLC1` WLC-3504, `R1` 2911, `SW1` 2960-24TT, `SRV1` Server-PT) và 3 liên kết đều up. Đây **không phải kết quả test case** — tên file cố ý không theo mã TC.
>
> **Đã có minh chứng test case:** [`tc-01-wlc-reachability.png`](tc-01-wlc-reachability.png) — cửa sổ Command Prompt của `SRV1` ngày 2026-10-18, thấy rõ chuỗi `ping 192.168.99.10` đi từ `100% loss` (khi uplink WLC còn là access VLAN 1) → `25% loss` (ARP lần đầu) → **`0% loss`** sau khi sửa. TC-01 `Passed` (trên model `WLC-3504`, lịch sử P1).
> **Chạy lại TC-01 trên `WLC-PT` (bản test copy, 2026-10-18):** [`tc-01-wlc-pt-reachability-canvas.png`](tc-01-wlc-pt-reachability-canvas.png) — management `192.168.99.10/24`: `SRV1 → .10` `0% loss` (4/4) và `R1 → .10` `5/5`. (Bộ ảnh giao diện từng được viện dẫn dưới tên `wlc-pt-feasibility-*.png` **hiện chưa có trong repo** — cần chụp bổ sung.)
>
> [`tc-02-lap-ip.png`](tc-02-lap-ip.png), [`tc-02-lap-ping-gateway.png`](tc-02-lap-ping-gateway.png), [`tc-02-lap-arp-r1.png`](tc-02-lap-arp-r1.png), [`tc-02-sw1-mac-fa03.png`](tc-02-sw1-mac-fa03.png) — minh chứng TC-02 `Passed`: cấu hình IP tĩnh của `LAP1`, kết quả ping tới gateway, bảng ARP trên `R1` và MAC-table `SW1 Fa0/3` (VLAN 99, MAC `0001.968e.6b01`).
>
> [`tc-03-lap-wlc-join.png`](tc-03-lap-wlc-join.png) — minh chứng TC-03 `Passed` (2026-10-19, chụp lại từ file deliverable `enterprise-wifi-wlc-lab-v1.pkt` sau khi đặt lại IP tĩnh `192.168.99.30` cho `LAP1`): `WLC2 → Config → AP Groups` liệt kê `LAP1` (MAC `0001.968E.6B01`) với `Status: Online` trong nhóm `default-group` — LAP-PT đã đăng ký/chịu quản lý bởi `WLC-PT`. **Giới hạn:** trạng thái `Online` là **runtime** (không lưu trong `.pkt`; mở lại file AP hiện `Offline` cho tới khi đặt IP cho `LAP1`); không có minh chứng bắt tay CAPWAP; `CAPWAP.PRIMARY_AC = 0.0.0.0` ở cả hai model (PT auto-discovery) và **không** được đọc là bằng chứng theo hướng nào.
>
> **Minh chứng audit lựa chọn thiết bị (2026-10-18) — cảnh báo: các file dưới đây được viện dẫn trong [../docs/device-selection-decision.md](../docs/device-selection-decision.md) nhưng hiện KHÔNG có trong repo; cần chụp bổ sung:**
> `audit-wlc-3504-config-menu.png` (model cũ **không** có trang WLAN) · `audit-wlc-pt-wireless-lans.png` (trang `Wireless LANs` có SSID + VLAN + WPA2) · `audit-wlc-pt-management-ip.png` (địa chỉ quản trị trên cổng GUI `Management` / API port `Vlan1`) · `audit-lap-pt-radio.png` (radio của LAP không có trường xác thực) · `audit-ap-pt-port1-security.png` (AP standalone có SSID/xác thực nhưng không có trường VLAN).
>
> **TC-01, TC-02 và TC-03 đã `Passed`** kèm minh chứng ở trên. Các test case còn lại (TC-04 → TC-14) vẫn `Not Tested`. Không tạo screenshot giả.

## Quy định

1. Tên file tiếng Anh, gắn với test ID:
   `tc-01-wlc-reachability.png`, `tc-02-lap-ip.png`, `tc-03-lap-wlc-join.png`, `tc-04-staff-wlan.png`, `tc-05-guest-wlan.png`, `tc-06-staff-associate.png`, `tc-07-guest-associate.png`, `tc-08-staff-ipconfig.png` / `tc-08-guest-ipconfig.png`, `tc-09-ping-gateway.png`, `tc-10-guest-to-staff.png`, `tc-11-guest-to-mgmt.png`, `tc-12-show-vlan.png`, `tc-13-radius.png` (chỉ khi thử RADIUS), `tc-14-invalid-psk.png` / `tc-14-invalid-radius.png`.
2. Ảnh đọc được: thấy rõ lệnh, kết quả, SSID/IP liên quan; che PSK/password/tài khoản thật.
3. Chỉ commit minh chứng của lần test thật trên đúng topology và đúng bản 9.0.1.0858.
4. Không dùng ảnh minh họa ngoài đồ án, không chỉnh sửa kết quả.
5. Mỗi file phải được tham chiếu từ [../tests/test-cases.md](../tests/test-cases.md) hoặc [../docs/testing-plan.md](../docs/testing-plan.md); `Passed` nào cũng phải có file tương ứng tồn tại.
6. Ảnh **quan sát giao diện thiết bị** (không phải kết quả test) dùng tiền tố `audit-`, ví dụ `audit-wlc-pt-wireless-lans.png`; ảnh này **không** được dùng để chuyển TC nào sang `Passed` và phải được tham chiếu từ [../docs/testing-plan.md](../docs/testing-plan.md) §6.1 hoặc [../docs/device-selection-decision.md](../docs/device-selection-decision.md).
