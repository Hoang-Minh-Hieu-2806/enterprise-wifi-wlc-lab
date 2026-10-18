# Evidence

Thư mục lưu minh chứng kiểm thử trên bản 9.0.1.0858: ảnh chụp màn hình, kết quả lệnh, export cấu hình đã che secret. Huỳnh Như Hoàng duy trì; Đặng Văn Hoàng bổ sung phần WLC/AP; Hoàng Minh Hiếu ghi evidence cho phần mình kiểm thử.

## Hiện trạng

> **Đã có minh chứng bring-up P1:** [`p1-wired-skeleton-topology.png`](p1-wired-skeleton-topology.png) — ảnh canvas Packet Tracer chụp ngày 2026-10-18, thể hiện 4 thiết bị (`WLC1` WLC-3504, `R1` 2911, `SW1` 2960-24TT, `SRV1` Server-PT) và 3 liên kết đều up.
>
> Đây **không phải kết quả test case** — tên file cố ý không theo mã TC. Mọi test case vẫn đang ở trạng thái `Not Tested`. Không tạo screenshot giả.

## Quy định

1. Tên file tiếng Anh, gắn với test ID:
   `tc-01-wlc-reachability.png`, `tc-02-lap-ip.png`, `tc-03-lap-join.png`, `tc-04-staff-wlan.png`, `tc-05-guest-wlan.png`, `tc-06-staff-associate.png`, `tc-07-guest-associate.png`, `tc-08-staff-ipconfig.png` / `tc-08-guest-ipconfig.png`, `tc-09-ping-gateway.png`, `tc-10-guest-to-staff.png`, `tc-11-guest-to-mgmt.png`, `tc-12-show-vlan.png`, `tc-13-radius.png` (chỉ khi thử RADIUS), `tc-14-invalid-psk.png` / `tc-14-invalid-radius.png`.
2. Ảnh đọc được: thấy rõ lệnh, kết quả, SSID/IP liên quan; che PSK/password/tài khoản thật.
3. Chỉ commit minh chứng của lần test thật trên đúng topology và đúng bản 9.0.1.0858.
4. Không dùng ảnh minh họa ngoài đồ án, không chỉnh sửa kết quả.
5. Mỗi file phải được tham chiếu từ [../tests/test-cases.md](../tests/test-cases.md) hoặc [../docs/testing-plan.md](../docs/testing-plan.md); `Passed` nào cũng phải có file tương ứng tồn tại.
