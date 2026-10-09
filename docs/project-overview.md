# Tổng quan dự án

## 1. Phát biểu vấn đề

Các doanh nghiệp vừa và nhỏ cần mạng Wi-Fi tập trung, dễ quản lý, phân tách được lưu lượng nhân viên và khách, đồng thời kiểm soát truy cập cơ bản giữa các vùng mạng. Tuy nhiên, việc thử nghiệm trực tiếp trên thiết bị thật tốn kém và rủi ro. Dự án này sử dụng **Cisco Packet Tracer 9.x** để mô phỏng một mạng WLAN doanh nghiệp thu nhỏ, giúp sinh viên luyện tập thiết kế, cấu hình và đánh giá bảo mật trong môi trường an toàn.

## 2. Phạm vi dự án

**Trong phạm vi:**

- Thiết kế topology logic gồm router, switch Layer 2, WLC, 2–3 AP, server và wireless client.
- Phân đoạn VLAN cho staff / guest / management.
- Triển khai nhiều SSID (dự kiến `STAFF-WIFI`, `GUEST-WIFI`).
- Áp dụng ít nhất một phương thức xác thực vô tuyến khả thi.
- Khảo sát tính khả thi của RADIUS + WPA2-Enterprise nếu thiết bị mô phỏng hỗ trợ.
- Kiểm soát truy cập liên VLAN (dự kiến ACL trên router).
- Kiểm thử kết nối, kiểm thử bảo mật cơ bản, thu thập minh chứng, viết báo cáo.

**Ngoài phạm vi (trừ khi kiểm chứng được tính khả thi):**

- WPA3, EAP cụ thể, roaming liền mạch, phân tích nhiễu RF, đo throughput.
- Triển khai thực tế trên thiết bị vật lý.
- Phát triển phần mềm, API, web app, CI/CD.

## 3. Mục tiêu

Xem chi tiết tại [../README.md](../README.md#1-mục-tiêu-dự-án) và bảng yêu cầu tại [requirements.md](requirements.md).

Tóm tắt: quản lý tập trung bằng WLC, nhiều SSID, phân đoạn VLAN, xác thực vô tuyến, kiểm soát truy cập, kiểm thử và tài liệu hóa đầy đủ.

## 4. Sản phẩm bàn giao dự kiến (Deliverables)

| # | Sản phẩm | Vị trí | Trạng thái hiện tại |
| --- | --- | --- | --- |
| 1 | Tài liệu thiết kế (overview, requirements, topology, IP/VLAN, security) | `docs/` | Khung sườn, nội dung là đề xuất |
| 2 | File mô phỏng Packet Tracer (`.pkt`) | `packet-tracer/` | Chưa có (Pending) |
| 3 | Bản sao cấu hình thiết bị | `configs/` | Chưa có (Pending) |
| 4 | Kế hoạch và ca kiểm thử | `docs/testing-plan.md`, `tests/test-cases.md` | Khung sườn, chưa test |
| 5 | Minh chứng (screenshots, kết quả lệnh) | `evidence/` | Chưa có (Pending) |
| 6 | Báo cáo tổng hợp | `report/` | Chưa có (Pending) |

## 5. Tiêu chí hoàn thành dự án

1. Có file `.pkt` mở được trên Packet Tracer 9.x, đúng topology đã chốt.
2. WLC quản lý được AP; các SSID dự kiến phát sóng đúng.
3. Client nhận đúng IP theo VLAN/SSID dự kiến và ping được gateway tương ứng.
4. Ít nhất một phương thức xác thực vô tuyến hoạt động và được ghi nhận.
5. Có kiểm thử phân tách Guest/Staff và kiểm thử ACL (nếu triển khai ACL).
6. Mọi test case trong [../tests/test-cases.md](../tests/test-cases.md) có kết quả thực tế và minh chứng trong `evidence/`.
7. Báo cáo tổng hợp trong `report/` mô tả trung thực những gì làm được và những gì chưa kiểm chứng được.
8. Không có khẳng định nào về WPA3, EAP, roaming, throughput khi chưa có minh chứng.

## 6. Rủi ro chính

- Model WLC/AP hoặc tính năng RADIUS không khả dụng trong Packet Tracer → cần phương án dự phòng (WPA2-PSK).
- Giới hạn số lượng AP hoặc tính năng WLC → giảm còn 2 AP, ghi rõ trong [implementation-notes.md](implementation-notes.md).
- Cấu hình ACL hoặc DHCP không như kỳ vọng → ghi lại workaround, không ngụy tạo kết quả.
