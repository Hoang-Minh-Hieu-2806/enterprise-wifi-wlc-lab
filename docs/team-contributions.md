# Phân công nhóm

> Nhóm 3 sinh viên. Bảng dưới là **đề xuất phân công**, có thể trùng lặp và phải cập nhật theo đóng góp thực tế. Không điền tên khi chưa được xác nhận; không ghi công việc là “đã xong” khi chưa có minh chứng.

## 1. Phân công đề xuất

| Thành viên | Phạm vi chính | Sản phẩm liên quan |
| --- | --- | --- |
| Thành viên 1 | Topology, VLAN, địa chỉ IP, cấu hình router và switch | [topology.md](topology.md), [ip-vlan-plan.md](ip-vlan-plan.md), `configs/router/`, `configs/switch/` |
| Thành viên 2 | WLC, AP, SSID, xác thực vô tuyến, khảo sát tính khả thi RADIUS | [security-design.md](security-design.md), [implementation-notes.md](implementation-notes.md), `configs/wlc-ap/` |
| Thành viên 3 | Kiểm thử kiểm soát truy cập, thu thập minh chứng, tài liệu và trình bày | [testing-plan.md](testing-plan.md), [../tests/test-cases.md](../tests/test-cases.md), `evidence/`, `report/` |

## 2. Nguyên tắc phối hợp

- Công việc có thể **trùng lặp** (overlap): ví dụ Thành viên 1 và 2 cùng kiểm tra trunk giữa switch và AP; Thành viên 3 tham gia test từ đầu, không đợi cấu hình xong mới test.
- Mọi thay đổi topology, VLAN, SSID phải thống nhất trước khi commit file `.pkt`.
- Commit message dùng tiếng Anh, ngắn gọn (ví dụ `docs: update vlan plan`).
- Mật khẩu / key kiểm thử không commit; khi chụp minh chứng cần che thông tin nhạy cảm.
- Cập nhật file này khi phân công thay đổi hoặc khi ghi nhận đóng góp thực tế.

## 3. Bảng đóng góp thực tế (cập nhật sau)

| Thành viên (bổ sung tên sau) | Công việc đã làm | Minh chứng / commit | Ghi chú |
| --- | --- | --- | --- |
| — | — | — | Chưa có (Pending) |
| — | — | — | Chưa có (Pending) |
| — | — | — | Chưa có (Pending) |

> Chỉ điền bảng trên khi có công việc thật và minh chứng tương ứng. Không bịa tên thành viên, không ghi công việc chưa làm.
