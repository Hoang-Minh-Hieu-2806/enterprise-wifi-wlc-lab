# Yêu cầu dự án (Requirements)

> Trạng thái dùng các nhãn `Planned` / `In Progress` / `Not Tested` / `Passed` / `Failed` / `Blocked`. **REQ-01, REQ-02 và REQ-03 đã `Passed`** (TC-01, TC-02 `2026-10-18`; TC-03 `2026-10-19`); **REQ-07 đang `In Progress`** vì hạ tầng VLAN đã dựng và kiểm chứng ở khung có dây P1 (+ LAP ở VLAN 99), nhưng chưa có client để chứng minh phân đoạn thật. Các yêu cầu còn lại vẫn `Planned`/`Not Tested` — chỉ dùng `Passed` khi có minh chứng. RADIUS là Conditional: `Blocked` ở đó không làm mất trạng thái hoàn thành của luồng Core.

## Bảng yêu cầu

| ID | Mô tả | Loại | Ưu tiên | Phương pháp kiểm chứng | Trạng thái |
| --- | --- | --- | --- | --- | --- |
| REQ-01 | WLC có thể tới được qua mạng quản trị (management reachability) | Core | Cao | TC-01: ping/tới WLC + screenshot | **Passed** — `WLC-PT`, `192.168.99.10` (GUI `Management`/API `Vlan1`), ping 4/4 `0% loss` từ `SRV1` + `5/5` từ `R1`; chạy lại trên `WLC-PT` (bản test copy) 2026-10-18 |
| REQ-02 | LAP-PT nhận địa chỉ IP quản trị hợp lệ | Core | Cao | TC-02: kiểm tra IP LAP + screenshot | **Passed** — IP tĩnh `192.168.99.30/24`, gw `192.168.99.1`; Simple PDU `LAP1 → R1` Successful + ping ngược `R1 → .30` 5/5 (2026-10-18) |
| REQ-03 | LAP-PT đăng ký và chịu quản lý bởi WLC | Core | Cao | TC-03: trạng thái join trên WLC + screenshot | **Passed** — `WLC2 → Config → AP Groups` hiển thị `LAP1` (MAC `0001.968E.6B01`) `Status: Online` trong nhóm `default-group` (2026-10-19); minh chứng `../evidence/tc-03-lap-wlc-join.png` |
| REQ-04 | WLAN `STAFF-WIFI` được tạo và phát sóng | Core | Cao | TC-04: quét SSID từ client + screenshot | Planned |
| REQ-05 | WLAN `GUEST-WIFI` được tạo và phát sóng | Core | Cao | TC-05: quét SSID từ client + screenshot | Planned |
| REQ-06 | Client Staff/Guest associate được bằng WPA2-PSK đã cấu hình | Core | Cao | TC-06, TC-07 + screenshot | Planned |
| REQ-07 | Phân đoạn VLAN cho Staff (10), Guest (20), Management (99) | Core | Cao | TC-08 (IP đúng subnet) + TC-12 (VLAN) + screenshot | In Progress — VLAN 10/20/99 đã cấu hình và đọc lại được trên `2960-24TT` (khung có dây P1); chưa có client để chứng minh phân đoạn thật |
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
