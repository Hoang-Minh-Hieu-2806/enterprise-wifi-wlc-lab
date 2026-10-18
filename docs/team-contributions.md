# Phân công nhóm (chia đều 33 / 33 / 34)

> Ba thành viên chia khối lượng tương đương: Hoàng Minh Hiếu 33%, Đặng Văn Hoàng 33%, Huỳnh Như Hoàng 34%. Hiếu là nhóm trưởng, vừa điều phối vừa trực tiếp tham gia kỹ thuật: quản lý Git/tiến độ, rà soát thiết kế, hỗ trợ linh hoạt các mảng theo tiến độ, tham gia kiểm thử, tổng hợp báo cáo. Không đánh giá công việc chỉ bằng commit count — theo dõi cả triển khai, tài liệu, evidence và kiểm thử.

## 1. Hoàng Minh Hiếu — Nhóm trưởng / Điều phối / Hỗ trợ kỹ thuật linh hoạt (33%)

- Quản lý Git, milestone và theo dõi task; giữ README và project overview.
- Cùng cả hai thành viên chốt topology và IP/VLAN plan; phê duyệt mọi đổi VLAN/IP/interface/SSID trước khi cấu hình.
- Rà soát thiết kế và duyệt thay đổi tài liệu của cả nhóm.
- Trực tiếp phụ trách một phần kỹ thuật theo tiến độ (hỗ trợ linh hoạt router/switch/WLC, không chốt mảng cố định để lấp chỗ nghẽn).
- Tham gia kiểm thử (kết hợp acceptance chọn lọc) và ghi evidence phần mình làm.
- Tổng hợp báo cáo cuối từ các nháp thành phần, chuẩn bị dàn ý thuyết trình.
- Review file `.pkt`, evidence kiểm thử và báo cáo trước khi nộp.

## 2. Đặng Văn Hoàng — Topology / WLC / AP / Xác thực (33%)

- Dựng và giữ topology chính trong Packet Tracer 9.0.1.0858 (bắt đầu 01 LAP-PT).
- Verify tương thích WLC/AP và LAP registration (TC-01→TC-03).
- Cấu hình WLAN/SSID trên WLC (`STAFF-WIFI`, `GUEST-WIFI`) và xác thực WPA2-PSK (TC-04→TC-07, TC-14 PSK).
- Khảo sát tính khả thi RADIUS **sau khi Core ổn định** (TC-13, Conditional).
- Lưu file `.pkt` làm việc, thu thập screenshot/evidence phần vô tuyến, ghi chú cấu hình (redacted) và limitation phát hiện khi test.

## 3. Huỳnh Như Hoàng — Định tuyến / VLAN / DHCP / ACL / Kiểm thử / Kết quả (34%)

- Cấu hình VLAN switch, cổng trunk Switch↔WLC và SVI quản trị; phối hợp với Đặng Văn Hoàng về cổng uplink WLC/LAP-PT.
- Cấu hình liên VLAN trên router (router-on-a-stick hoặc thiết kế model hỗ trợ); DHCP scope trên Server-PT và `ip helper-address` **bắt buộc** trên router cho VLAN 10/20 (TC-08, TC-09).
- Triển khai ACL chặn Guest→Staff/Management với explicit deny + permit tối thiểu (TC-10, TC-11, TC-12).
- Chạy kiểm thử IP, gateway, cô lập liên VLAN, sai credential (TC-08→TC-12, TC-14) và hỗ trợ TC-13 nếu tới RADIUS.
- Thu thập evidence, duy trì kết quả test ở [../tests/test-cases.md](../tests/test-cases.md).
- Tổng hợp kết quả kiểm thử (nháp mục kết quả/limitation cho báo cáo trong `report/`).

## 4. Luật phối hợp

- Hiếu điều phối nhưng cũng làm kỹ thuật trực tiếp; khi Hiếu nhận mảng nào thì ghi rõ vào bảng đóng góp để tránh chồng chéo với Đặng Văn Hoàng / Huỳnh Như Hoàng.
- Mọi đổi VLAN, IP, interface, SSID phải thống nhất trước khi sửa file `.pkt` hay tài liệu chuẩn ([ip-vlan-plan.md](ip-vlan-plan.md)).
- Không sửa cùng một file kỹ thuật đồng thời nếu chưa phối hợp.
- Secret/PSK kiểm thử không commit; screenshot che thông tin nhạy cảm.
- Commit message tiếng Anh, ngắn gọn (ví dụ `docs: update vlan plan`).

### Quy tắc cân bằng công việc và rà soát chéo

- Khi Hiếu nhận thêm một hạng mục kỹ thuật trong tuần, nhóm chủ động phân bổ lại công việc review, kiểm thử hoặc thu thập bằng chứng tương ứng cho Đặng Văn Hoàng hoặc Huỳnh Như Hoàng để tránh tập trung quá nhiều trách nhiệm vào trưởng nhóm.
- Mọi thay đổi về người phụ trách, người phối hợp và đóng góp thực tế phải được cập nhật trong bảng phân công (§5).
- Việc phân bổ lại không làm thay đổi tỷ lệ đóng góp dự kiến 33% – 33% – 34%; tỷ lệ chỉ được xem xét lại khi có căn cứ rõ ràng về mức độ đóng góp thực tế.

## 5. Bảng đóng góp thực tế (cập nhật trong quá trình làm)

| Thành viên | Công việc đã làm | Minh chứng / commit | Ghi chú |
| --- | --- | --- | --- |
| Hoàng Minh Hiếu | — | — | Planned (điều phối + hỗ trợ linh hoạt + tổng hợp báo cáo) |
| Đặng Văn Hoàng | — | — | Planned (topology + WLC/AP + auth) |
| Huỳnh Như Hoàng | — | — | Planned (routing + VLAN + DHCP + ACL + test) |

> Chỉ điền khi có công việc thật kèm minh chứng. Không ghi `Passed` khi chưa có evidence.
