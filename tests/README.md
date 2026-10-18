# Tests

Thư mục kiểm thử của dự án (bản 9.0.1.0858).

- [test-cases.md](test-cases.md) — 14 ca kiểm thử: TC-01→TC-12 và TC-14 (PSK) là Core; TC-13 và phần RADIUS của TC-14 là Conditional.
- Kế hoạch tổng thể và phân loại Core/Conditional/Research-only xem tại [../docs/testing-plan.md](../docs/testing-plan.md).

## Hiện trạng

> **TC-01 đã `Passed`** (2026-10-18, minh chứng `../evidence/tc-01-wlc-reachability.png`); các test case còn lại ở trạng thái **Not Tested**. Huỳnh Như Hoàng sở hữu chạy test Core và giữ evidence; Đặng Văn Hoàng sở hữu phần WLC/AP; Hoàng Minh Hiếu tham gia kiểm thử và chạy acceptance chọn lọc.

## Quy định

1. Chạy theo thứ tự TC-01→TC-12, TC-14 (PSK) trước; TC-13 sau cùng.
2. Không điền `Actual result` khi chưa test thật; `Blocked` kèm lý do được chấp nhận cho TC-13 mà không ảnh hưởng Core.
3. Mỗi `Passed` phải có file evidence tồn tại trong `../evidence/`.
4. Mọi đổi kỳ vọng (`Expected result`) phải có lý do và đồng bộ với `docs/`.
