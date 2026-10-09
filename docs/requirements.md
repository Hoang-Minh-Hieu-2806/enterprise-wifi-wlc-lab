# Yêu cầu dự án (Requirements)

> Tất cả yêu cầu dưới đây ở trạng thái **Proposed / Pending** — chưa triển khai, chưa kiểm chứng.

## Bảng yêu cầu

| ID | Mô tả | Ưu tiên | Phương pháp kiểm chứng | Trạng thái |
| --- | --- | --- | --- | --- |
| REQ-01 | Quản lý tập trung AP bằng WLC (AP đăng ký và chịu quản lý bởi WLC) | Cao | Kiểm thử TC-01: kiểm tra trạng thái AP trên WLC + minh chứng screenshot | Proposed |
| REQ-02 | Triển khai nhiều SSID (`STAFF-WIFI`, `GUEST-WIFI`) | Cao | Kiểm thử TC-02: quét SSID từ client + screenshot | Proposed |
| REQ-03 | Phân đoạn VLAN cho Staff (10), Guest (20), Management (99) | Cao | Kiểm thử TC-07: kiểm tra VLAN trên switch + minh chứng | Proposed |
| REQ-04 | Client nhận đúng IP theo subnet của SSID/VLAN tương ứng | Cao | Kiểm thử TC-03: `ipconfig` / DHCP + screenshot | Proposed |
| REQ-05 | Kết nối tới gateway tương ứng của từng VLAN | Cao | Kiểm thử TC-04: `ping` gateway + screenshot | Proposed |
| REQ-06 | Ít nhất một phương thức xác thực vô tuyến khả thi hoạt động | Cao | Kiểm thử TC-05: kết nối thành công/thất bại + screenshot | Proposed |
| REQ-07 | Khảo sát tính khả thi của RADIUS + WPA2-Enterprise (nếu được hỗ trợ) | Trung bình | Ghi nhận trong [implementation-notes.md](implementation-notes.md) + TC-05 nếu khả thi | Proposed |
| REQ-08 | Kiểm soát truy cập giữa các phân đoạn (dự kiến ACL trên router, chặn Guest → nội bộ) | Cao | Kiểm thử TC-06, TC-08: `ping` liên VLAN + screenshot | Proposed |
| REQ-09 | Bảo vệ mạng quản trị (VLAN 99 tách biệt, hạn chế truy cập quản trị) | Trung bình | Kiểm thử TC-08 + rà soát cấu hình | Proposed |
| REQ-10 | Kiểm thử kết nối đầy đủ và lưu minh chứng | Cao | Toàn bộ test case có `Actual result` và file trong `../evidence/` | Not tested |
| REQ-11 | Tài liệu kỹ thuật đầy đủ (thiết kế, triển khai, kiểm thử, báo cáo) | Cao | Rà soát `docs/`, `tests/`, `report/` | In progress (skeleton) |

## Ghi chú

- Mức ưu tiên: **Cao** = bắt buộc để đạt; **Trung bình** = nên có, chấp nhận phương án dự phòng.
- Cột trạng thái chỉ chuyển sang `Implemented` / `Verified` khi có cấu hình và minh chứng thực tế.
- Các chủ đề WPA3, EAP cụ thể, roaming, nhiễu RF, đo throughput **không** nằm trong bảng yêu cầu chính; chúng là chủ đề nghiên cứu nêu tại [security-design.md](security-design.md).
- Chi tiết ca kiểm thử xem tại [testing-plan.md](testing-plan.md) và [../tests/test-cases.md](../tests/test-cases.md).
- Kế hoạch địa chỉ tham chiếu tại [ip-vlan-plan.md](ip-vlan-plan.md); topology tham chiếu tại [topology.md](topology.md).
