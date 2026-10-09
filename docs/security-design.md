# Thiết kế bảo mật dự kiến

> Tài liệu này trình bày **ý định thiết kế**, không phải kết quả đã kiểm chứng. Không khẳng định cơ chế mã hóa hay kiểm soát truy cập có hiệu quả khi chưa có kết quả kiểm thử tại [testing-plan.md](testing-plan.md) và [../tests/test-cases.md](../tests/test-cases.md).

## 1. Nguyên tắc phân tách

- Mỗi nhóm đối tượng dùng SSID và VLAN riêng:
  - `STAFF-WIFI` → VLAN 10 (`192.168.10.0/24`).
  - `GUEST-WIFI` → VLAN 20 (`192.168.20.0/24`).
  - Hạ tầng quản trị (WLC, switch, router, AP) → VLAN 99 (`192.168.99.0/24`).
- Mục tiêu: Guest không truy cập được tài nguyên nội bộ của Staff và hệ thống quản trị; Staff chỉ truy cập trong phạm vi cho phép.

Chi tiết subnet xem tại [ip-vlan-plan.md](ip-vlan-plan.md); topology xem tại [topology.md](topology.md).

## 2. Xác thực vô tuyến (dự kiến)

- **Ít nhất một phương thức xác thực khả thi** sẽ được triển khai và kiểm thử (yêu cầu REQ-06).
- **Mạng Staff:** ưu tiên khảo sát WPA2-Enterprise trên nền RADIUS nếu thiết bị Packet Tracer hỗ trợ. Nếu không khả thi, chuyển sang WPA2-PSK và ghi rõ lý do trong [implementation-notes.md](implementation-notes.md).
- **Mạng Guest:** dự kiến WPA2-PSK. Chỉ dùng Open khi phục vụ mục đích kiểm thử và phải ghi rõ rủi ro không mã hóa.
- Mật khẩu / shared secret **không** commit lên repository (xem `.gitignore` và quy định tại [../configs/README.md](../configs/README.md)).

## 3. RADIUS (nếu được hỗ trợ)

- Vai trò dự kiến: server RADIUS xác thực người dùng Staff khi kết nối `STAFF-WIFI`.
- Nội dung cần kiểm chứng:
  - [ ] Server trong Packet Tracer có hỗ trợ RADIUS không.
  - [ ] WLC/AP đã chọn có cấu hình được WPA2-Enterprise trỏ về RADIUS không.
  - [ ] Tài khoản kiểm thử thành công và thất bại được ghi nhận thế nào.
- Nếu không hỗ trợ, ghi rõ “không khả thi trên thiết bị/mô phỏng đã chọn” và dùng phương án dự phòng. Không ngụy tạo log RADIUS thành công.

## 4. Hạn chế Guest → mạng nội bộ (dự kiến dùng ACL)

- Ý định: Guest (VLAN 20) chỉ được ra mạng ngoài mô phỏng / Internet, bị chặn khi truy cập VLAN 10 và VLAN 99.
- Biện pháp dự kiến: ACL trên router (inbound/outbound trên sub-interface tương ứng).
- Nội dung kiểm chứng: `ping` từ Guest tới Staff/Management phải thất bại sau khi áp ACL; `ping` tới gateway của chính Guest vẫn thành công (xem TC-06, TC-08).
- Không sao chép câu lệnh ACL mẫu vào đây như “cấu hình đã đúng” — mọi câu lệnh chỉ có giá trị sau khi kiểm thử trên đúng model thiết bị.

## 5. Bảo vệ mạng quản trị (VLAN 99)

- Thiết bị hạ tầng đặt ở VLAN 99 tách biệt khỏi client.
- Hạn chế thiết bị/user được phép truy cập quản trị WLC, switch, router.
- Ưu tiên địa chỉ tĩnh cho thiết bị hạ tầng; không cấp DHCP quản trị cho client.
- Mọi truy cập quản trị từ Guest/Staff phải được kiểm thử và chặn (TC-08).

## 6. So sánh WPA2-PSK và WPA2-Enterprise

| Tiêu chí | WPA2-PSK | WPA2-Enterprise (RADIUS) |
| --- | --- | --- |
| Cơ chế | Một pre-shared key chung cho mọi client của SSID | Mỗi user một tài khoản, xác thực qua RADIUS |
| Quản lý | Đơn giản, phù hợp quy mô nhỏ / Guest | Phù hợp Staff, thu hồi từng tài khoản, ghi log tập trung |
| Rủi ro lộ key | Lộ key là phải đổi cho tất cả client | Giảm rủi ro dùng chung key, nhưng phụ thuộc RADIUS khả dụng |
| Khả thi trong Packet Tracer | Thường khả thi, sẽ kiểm chứng | **Chưa chắc chắn**, cần kiểm chứng model thiết bị |
| Phù hợp đồ án | Phương án dự phòng chắc chắn cho ít nhất 1 SSID | Mục tiêu khảo sát cho `STAFF-WIFI` nếu khả thi |

## 7. Chủ đề nghiên cứu (chưa khẳng định)

Các nội dung sau chỉ là **chủ đề nghiên cứu**, không ghi là đã triển khai:

- WPA3 và các chế độ chuyển tiếp.
- Phương thức EAP cụ thể (EAP-TLS, PEAP, EAP-FAST, ...).
- Roaming giữa các AP.
- Nhiễu RF và tối ưu vùng phủ.
- Đo throughput / hiệu năng vô tuyến.

Chỉ đưa vào báo cáo khi đã xác minh simulator hỗ trợ và có minh chứng kiểm thử.
