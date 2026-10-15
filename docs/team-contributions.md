# Phân công nhóm (đã chốt tên và tỉ trọng)

> Nhóm trưởng Hoàng Minh Hiếu giữ khối lượng hands-on nhẹ hơn (~25%), tập trung điều phối, kiến trúc và nghiệm thu. Đặng Văn Hoàng (~37.5%) và Huỳnh Như Hoàng (~37.5%) đảm nhận phần lớn triển khai trực tiếp. Tỉ trọng là mục tiêu; có thể điều chỉnh nếu testing phát hiện blocker, nhưng Hiếu vẫn nhẹ hơn tổng thể. Không đánh giá công việc chỉ bằng commit count — theo dõi cả triển khai, tài liệu, evidence và kiểm thử.

## 1. Hoàng Minh Hiếu — Nhóm trưởng / Kiến trúc / Nghiệm thu

- Giữ scope, milestone và theo dõi task.
- Cùng cả hai thành viên chốt topology và IP/VLAN plan; phê duyệt mọi đổi VLAN/IP/interface trước khi cấu hình.
- Duyệt thay đổi tài liệu; giữ README và project overview.
- Review file `.pkt`, evidence kiểm thử và báo cáo cuối.
- Tổng hợp bài nộp và chuẩn bị dàn ý thuyết trình.
- Chạy một số acceptance test chọn lọc; không ôm toàn bộ triển khai kỹ thuật hay viết toàn bộ báo cáo.

Không giao Hiếu: full cấu hình router, full cấu hình switch, full WLC/AP, chạy toàn bộ test, viết toàn bộ báo cáo một mình.

## 2. Đặng Văn Hoàng — Hạ tầng mạng / WLC / AP

- Dựng và giữ topology chính trong Packet Tracer 9.0.1.0858 (bắt đầu 01 LAP-PT).
- Verify tương thích WLC/AP và LAP registration (TC-01→TC-03).
- Cấu hình WLAN/SSID trên WLC (`STAFF-WIFI`, `GUEST-WIFI`) và xác thực WPA2-PSK (TC-04→TC-07, TC-14 PSK).
- Khảo sát tính khả thi RADIUS **sau khi Core ổn định** (TC-13, Conditional).
- Phối hợp với Huỳnh Như Hoàng cấu hình VLAN switch, trunk/access và các nối hạ tầng.
- Lưu file `.pkt` làm việc, chụp screenshot, ghi chú cấu hình (redacted) và limitation phát hiện khi test.

## 3. Huỳnh Như Hoàng — Định tuyến / Bảo mật / Kiểm thử / Báo cáo

- Cấu hình liên VLAN trên router (router-on-a-stick hoặc thiết kế model hỗ trợ) và DHCP scope/relay trên Server-PT (TC-08, TC-09).
- Triển khai ACL chặn Guest→Staff/Management với explicit deny + permit tối thiểu (TC-10, TC-11, TC-12).
- Chạy kiểm thử IP, gateway, cô lập liên VLAN, sai credential (TC-08→TC-12, TC-14) và hỗ trợ TC-13 nếu tới RADIUS.
- Thu thập evidence, duy trì kết quả test ở [../tests/test-cases.md](../tests/test-cases.md).
- Viết nháp đánh giá kỹ thuật, limitation và kết quả cho báo cáo trong `report/`.

## 4. Luật phối hợp

- Hiếu điều phối và phê duyệt, không làm thay phần việc của hai bạn.
- Mọi đổi VLAN, IP, interface, SSID phải thống nhất trước khi sửa file `.pkt` hay tài liệu chuẩn ([ip-vlan-plan.md](ip-vlan-plan.md)).
- Không sửa cùng một file kỹ thuật đồng thời nếu chưa phối hợp.
- Secret/PSK kiểm thử không commit; screenshot che thông tin nhạy cảm.
- Commit message tiếng Anh, ngắn gọn (ví dụ `docs: update vlan plan`).

## 5. Bảng đóng góp thực tế (cập nhật trong quá trình làm)

| Thành viên | Công việc đã làm | Minh chứng / commit | Ghi chú |
| --- | --- | --- | --- |
| Hoàng Minh Hiếu | — | — | Planned (điều phối + review) |
| Đặng Văn Hoàng | — | — | Planned (topology + WLC/AP) |
| Huỳnh Như Hoàng | — | — | Planned (routing + ACL + test + report) |

> Chỉ điền khi có công việc thật kèm minh chứng. Không ghi `Passed` khi chưa có evidence.
