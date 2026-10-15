# Yêu cầu dự án (Requirements)

> Trạng thái dùng các nhãn `Planned` / `In Progress` / `Not Tested` / `Passed` / `Failed` / `Blocked`. Hiện mọi yêu cầu là `Planned`, mọi test là `Not Tested`. Chỉ dùng `Passed` khi có minh chứng. RADIUS là Conditional: `Blocked` ở đó không làm mất trạng thái hoàn thành của luồng Core.

## Bảng yêu cầu

| ID | Mô tả | Loại | Ưu tiên | Phương pháp kiểm chứng | Trạng thái |
| --- | --- | --- | --- | --- | --- |
| REQ-01 | WLC có thể tới được qua mạng quản trị (management reachability) | Core | Cao | TC-01: ping/tới WLC + screenshot | Planned |
| REQ-02 | LAP-PT nhận địa chỉ IP quản trị hợp lệ | Core | Cao | TC-02: kiểm tra IP LAP + screenshot | Planned |
| REQ-03 | LAP-PT đăng ký và chịu quản lý bởi WLC | Core | Cao | TC-03: trạng thái join trên WLC + screenshot | Planned |
| REQ-04 | WLAN `STAFF-WIFI` được tạo và phát sóng | Core | Cao | TC-04: quét SSID từ client + screenshot | Planned |
| REQ-05 | WLAN `GUEST-WIFI` được tạo và phát sóng | Core | Cao | TC-05: quét SSID từ client + screenshot | Planned |
| REQ-06 | Client Staff/Guest associate được bằng WPA2-PSK đã cấu hình | Core | Cao | TC-06, TC-07 + screenshot | Planned |
| REQ-07 | Phân đoạn VLAN cho Staff (10), Guest (20), Management (99) | Core | Cao | TC-08 (IP đúng subnet) + TC-12 (VLAN) + screenshot | Planned |
| REQ-08 | Client nhận đúng IP theo subnet của SSID/VLAN tương ứng (DHCP trên Server-PT) | Core | Cao | TC-08: `ipconfig` + screenshot | Planned |
| REQ-09 | Kết nối tới gateway tương ứng của từng VLAN | Core | Cao | TC-09: `ping` gateway + screenshot | Planned |
| REQ-10 | Credential sai bị từ chối ở chế độ đã cấu hình (WPA2-PSK bắt buộc; RADIUS là Conditional) | Core + Conditional | Cao | TC-14 + screenshot | Planned |
| REQ-11 | Kiểm soát truy cập: Guest không tới được Staff và Management; Staff chỉ giữ truy cập nội bộ cho phép (ACL explicit deny + permit tối thiểu) | Core | Cao | TC-10, TC-11, TC-12: `ping` liên VLAN + `show access-lists` | Planned |
| REQ-12 | Khảo sát WPA2-Enterprise + RADIUS cho Staff **sau khi luồng Core ổn định, chỉ khi Server-PT và model WLC hỗ trợ** | Conditional | Trung bình | TC-13 + ghi nhận tại [implementation-notes.md](implementation-notes.md) | Planned |
| REQ-13 | Kiểm thử đầy đủ, lưu minh chứng, báo cáo trung thực | Core | Cao | Toàn bộ TC Core có `Actual result` + file trong `../evidence/`; báo cáo tại `../report/` | Planned |

## Ghi chú

- Mức ưu tiên: **Cao** = bắt buộc để đạt (luồng Core); **Trung bình** = nên có, chấp nhận `Blocked` có lý do (luồng Conditional).
- Cột trạng thái chỉ chuyển sang `Passed` / `Failed` / `Blocked` khi có cấu hình và minh chứng thực tế.
- WPA3, EAP cụ thể, roaming, nhiễu RF, đo throughput là **Research-only**, không nằm trong bảng yêu cầu; chỉ làm nền báo cáo, nêu tại [security-design.md](security-design.md).
- Chi tiết ca kiểm thử xem tại [testing-plan.md](testing-plan.md) và [../tests/test-cases.md](../tests/test-cases.md).
- Kế hoạch địa chỉ chuẩn tại [ip-vlan-plan.md](ip-vlan-plan.md) (tài liệu authoritative cho mọi địa chỉ); topology tại [topology.md](topology.md).
