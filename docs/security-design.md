# Thiết kế bảo mật (WPA2-PSK trước, RADIUS có điều kiện)

> Tài liệu này trình bày **ý định thiết kế**, không phải kết quả đã kiểm chứng. Không khẳng định cơ chế mã hóa hay kiểm soát truy cập có hiệu quả khi chưa có kết quả kiểm thử tại [testing-plan.md](testing-plan.md) và [../tests/test-cases.md](../tests/test-cases.md). Sự tồn tại của các trường WPA2/RADIUS trong GUI Packet Tracer không chứng minh giao thức đã chạy được.

## 1. Nguyên tắc phân tách

- Mỗi nhóm đối tượng dùng SSID và VLAN riêng:
  - `STAFF-WIFI` → VLAN 10 (`192.168.10.0/24`).
  - `GUEST-WIFI` → VLAN 20 (`192.168.20.0/24`).
  - Hạ tầng quản trị (WLC, switch, router, LAP-PT, Server-PT) → VLAN 99 (`192.168.99.0/24`).
- Mục tiêu: Guest không truy cập được tài nguyên Staff và hệ thống quản trị; Staff chỉ giữ đúng truy cập nội bộ cho phép (TC-10, TC-11, TC-12).

Bảng địa chỉ chuẩn xem tại [ip-vlan-plan.md](ip-vlan-plan.md); topology tại [topology.md](topology.md).

## 2. Thứ tự xác thực vô tuyến (bắt buộc)

1. **WPA2-PSK trước** cho cả `STAFF-WIFI` và `GUEST-WIFI` (luồng Core: TC-06, TC-07). Đây là phương thức đầu tiên phải triển khai và kiểm thử thành công/thất bại (TC-14 cho PSK là Core).
2. **WPA2-Enterprise + RADIUS chỉ sau** khi luồng Core (WLC + LAP-PT + WLAN + WPA2-PSK + IP/gateway + VLAN/ACL) đã ổn định, và chỉ cho Staff (Conditional: TC-13).

- Mật khẩu / PSK / shared secret thật **không** commit lên repository (xem `.gitignore` và [../configs/README.md](../configs/README.md)); minh chứng che secret.
- Mạng Guest dùng WPA2-PSK. Chỉ dùng Open cho mục đích kiểm thử và phải ghi rõ rủi ro không mã hóa.

## 3. RADIUS trên Server-PT (Conditional)

- Vai trò dự kiến (nếu thử): Server-PT xác thực người dùng Staff cho `STAFF-WIFI` qua RADIUS Server Settings (IP + shared secret) — **đã quan sát thấy** trong trang `Wireless LANs` của `WLC-PT` (minh chứng `../evidence/audit-wlc-pt-wireless-lans.png`), cùng lựa chọn Authentication `WPA2` và `WPA` (Enterprise).
- `IpcAPI` của bản 9.0.1.0858 có `EapolWirelessServerProcess` / `EapolWirelessServerMainProcess` (xử lý EAPOL) ⇒ hướng WPA2-Enterprise hợp lý về mặt mô phỏng.
- Điều kiện thử: luồng Core đã `Passed`; Server-PT xác nhận hỗ trợ RADIUS; đã tạo được WLAN trên `WLC-PT` (TC-04 `Passed` 2026-10-19; TC-05 chưa).
- Nếu không hỗ trợ: ghi `Blocked` kèm lý do tại TC-13 và [implementation-notes.md](implementation-notes.md). Kết quả này **không** làm mất trạng thái hoàn thành của luồng Core.
- Không ngụy tạo log RADIUS thành công; **không coi việc quan sát thấy trường IP/secret trong GUI là bằng chứng xác thực chạy được**.

## 4. Cô lập Guest bằng ACL trên router (Core)

- Ý định: Guest (VLAN 20) bị chặn truy cập Staff (VLAN 10) và các thiết bị quản trị trong VLAN 99; Guest vẫn tới được gateway của chính mình (`192.168.20.1`).
- **Ngoại lệ bắt buộc — DHCP:** Server-PT nằm ở VLAN 99 (xem [ip-vlan-plan.md](ip-vlan-plan.md) §2) và phải cấp DHCP cho Guest, nên ACL **phải permit** DHCP từ Guest tới Server-PT:

  ```text
  permit udp 192.168.20.0 0.0.0.255 host 192.168.99.20 eq 67
  ```

  Vì ngoại lệ này, phát biểu đúng là "Guest **không tới được các thiết bị quản trị** (WLC, switch, router, LAP-PT)", **không phải** "Guest không tới được VLAN 99". Nếu chặn cả subnet `192.168.99.0/24` thì Guest không nhận được IP ⇒ TC-08 thất bại.
- Biện pháp: ACL trên router với **explicit deny** cho Guest→Staff và Guest→dải quản trị (trừ DHCP), kèm **permit tối thiểu** cần thiết (định tuyến, DHCP, gateway). Nội dung ACL chốt theo model router thực tế, không sao chép mẫu chưa kiểm chứng.
- Kiểm chứng: TC-10 (Guest↛Staff), TC-11 (Guest↛thiết bị quản trị) phải `Failed-to-reach` sau ACL; TC-08 (Guest nhận IP qua DHCP) và TC-09 (gateway của chính Guest) vẫn `Passed`.
- Không khẳng định có truy cập Internet trừ khi có đích ISP/Internet được mô phỏng và kiểm thử thực tế.

## 5. Bảo vệ mạng quản trị (VLAN 99)

- Thiết bị hạ tầng đặt ở VLAN 99 tách biệt khỏi client; ưu tiên địa chỉ tĩnh.
- Hạn chế thiết bị/user được phép truy cập quản trị WLC, switch, router.
- Không cấp DHCP quản trị cho client trừ khi cần cho mục đích kiểm thử có ghi rõ.
- Mọi truy cập quản trị từ Guest/Staff được kiểm thử và chặn (TC-11, TC-12).
- Nhóm phải chốt và ghi phạm vi truy cập cho phép của Staff vào [implementation-notes.md](implementation-notes.md) (§4) **trước khi chạy TC-12**; TC-12 chỉ `Passed` khi Staff không vượt phạm vi đã chốt đó.

## 6. So sánh WPA2-PSK và WPA2-Enterprise

| Tiêu chí | WPA2-PSK (Core, làm trước) | WPA2-Enterprise + RADIUS (Conditional) |
| --- | --- | --- |
| Cơ chế | Một pre-shared key chung cho mọi client của SSID | Mỗi user một tài khoản, xác thực qua RADIUS |
| Quản lý | Đơn giản, phù hợp cả Guest và bước đầu Staff | Phù hợp Staff khi cần thu hồi từng tài khoản, log tập trung |
| Rủi ro | Lộ key phải đổi cho tất cả client | Giảm rủi ro key chung, nhưng phụ thuộc RADIUS khả dụng |
| Khả thi trên 9.0.1.0858 | **Đã quan sát** tuỳ chọn WPA2-PSK trên `WLC-PT`, **vẫn phải kiểm chứng** (TC-06/TC-07/TC-14) | **Đã quan sát** tuỳ chọn WPA/WPA2 + RADIUS IP & shared secret, **vẫn phải kiểm chứng** (TC-13) |
| Vai trò trong đồ án | Bắt buộc, quyết định đạt/không đạt luồng Core | Cộng thêm, `Blocked` được chấp nhận kèm lý do |

## 7. Chủ đề Research-only (không tính vào tiêu chí hoàn thành)

Các nội dung sau chỉ làm nền báo cáo, không ghi là đã triển khai:

- **WPA3 và các chế độ chuyển tiếp** — `WirelessAuthenType` trong `IpcAPI` của bản 9.0.1.0858 chỉ có WEP, WPA1-PSK/EAP, WPA2-PSK/EAP, Open; **không có WPA3/SAE** ⇒ không thể triển khai trên phiên bản đã chốt.
- Phương thức EAP cụ thể (EAP-TLS, PEAP, ...) — mô phỏng có xử lý EAPOL nhưng chưa kiểm chứng được một EAP cụ thể nào.
- Roaming giữa các AP (kể cả khi mở rộng 2–3 AP).
- Nhiễu RF và tối ưu vùng phủ.
- Đo throughput / hiệu năng vô tuyến.
- Chuẩn 802.11ac / 802.11ax — không có tham chiếu trong tài liệu `IpcAPI` của bản này.

Chỉ đưa vào kết quả khi đã xác minh simulator hỗ trợ và có minh chứng kiểm thử; ngược lại ghi rõ là nghiên cứu. Chi tiết khả năng của từng model: [device-selection-decision.md](device-selection-decision.md) §3.
