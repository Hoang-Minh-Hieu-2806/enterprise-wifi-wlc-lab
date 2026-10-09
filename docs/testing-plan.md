# Kế hoạch kiểm thử

> Mọi ca kiểm thử hiện ở trạng thái **Not tested**. Cột kết quả thực tế (`Actual result`) phải để trống cho tới khi có kiểm thử thật. Chi tiết từng bước xem tại [../tests/test-cases.md](../tests/test-cases.md).

## 1. Mục tiêu kiểm thử

1. Xác nhận AP được WLC quản lý (REQ-01).
2. Xác nhận SSID khả dụng đúng (`STAFF-WIFI`, `GUEST-WIFI`) (REQ-02).
3. Xác nhận client nhận đúng IP theo VLAN/SSID (REQ-04).
4. Xác nhận kết nối tới gateway (REQ-05).
5. Xác nhận xác thực thành công và thất bại (nơi được hỗ trợ) (REQ-06, REQ-07).
6. Xác nhận Guest bị hạn chế truy cập mạng nội bộ (REQ-08).
7. Xác nhận VLAN và ACL đúng (REQ-03, REQ-08, REQ-09).

## 2. Ma trận truy vết (test coverage)

| Test ID | Yêu cầu liên quan | Nội dung kiểm thử |
| --- | --- | --- |
| TC-01 | REQ-01 | AP đăng ký / được WLC quản lý |
| TC-02 | REQ-02 | SSID khả dụng |
| TC-03 | REQ-03, REQ-04 | Client nhận IP đúng subnet |
| TC-04 | REQ-05 | Kết nối tới gateway |
| TC-05a / TC-05b | REQ-06, REQ-07 | Xác thực thành công / thất bại |
| TC-06 | REQ-08 | Guest bị hạn chế truy cập Staff |
| TC-07 | REQ-03 | Kiểm chứng VLAN |
| TC-08 | REQ-08, REQ-09 | Kiểm chứng ACL và bảo vệ Management |

## 3. Điều kiện chung

- File `.pkt` mở được trên đúng phiên bản Packet Tracer đã chốt (xem [implementation-notes.md](implementation-notes.md)).
- Topology đúng như [topology.md](topology.md); địa chỉ đúng như [ip-vlan-plan.md](ip-vlan-plan.md) (hoặc bản cập nhật mới nhất).
- Biết SSID, mật khẩu kiểm thử, tài khoản RADIUS kiểm thử (nếu có). **Không commit mật khẩu thật** — dùng tài khoản/mật khẩu kiểm thử riêng và che khi chụp minh chứng.
- Công cụ: `ipconfig`, `ping`, `show` trên thiết bị, quét wireless trên client, GUI WLC.

## 4. Quy định ghi kết quả và minh chứng

- Mỗi test case gồm: ID, preconditions, steps, expected results, actual results, evidence paths, status.
- `Actual result` để trống hoặc ghi `Not tested` cho tới khi test thật.
- `Status` chỉ dùng: `Not tested` / `Pass` / `Fail` / `Blocked` (ghi rõ lý do nếu Blocked, ví dụ thiết bị không hỗ trợ).
- Minh chứng lưu tại `../evidence/` theo quy định ở [../evidence/README.md](../evidence/README.md) (ví dụ `evidence/tc-01-ap-registration.png`).
- Không chỉnh sửa `Expected result` để khớp với lỗi thực tế; nếu kỳ vọng sai, cập nhật tài liệu thiết kế và nêu lý do.

## 5. Tiêu chí đạt (pass criteria)

- TC-01 – TC-04, TC-07: phải Pass mới coi là mạng cơ bản hoạt động.
- TC-05a: ít nhất một phương thức xác thực phải Pass.
- TC-05b: phải ghi được trường hợp thất bại (sai mật khẩu / sai tài khoản) bị từ chối.
- TC-06, TC-08: Guest → Staff/Management phải bị chặn sau ACL; Guest → gateway của chính Guest vẫn thông.
- Mọi Fail/Blocked phải có ghi chú nguyên nhân và hướng xử lý, không giấu lỗi.
