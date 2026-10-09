# Tests

Thư mục kiểm thử của dự án.

- [test-cases.md](test-cases.md) — danh sách ca kiểm thử chi tiết (ID, preconditions, steps, expected/actual results, evidence, status).
- Kế hoạch tổng thể xem tại [../docs/testing-plan.md](../docs/testing-plan.md).

## Hiện trạng

> Tất cả test case ở trạng thái **Not tested**. Chưa có kết quả thực tế.

## Quy định

1. Không điền `Actual result` khi chưa test thật.
2. Mỗi test Pass/Fail phải có minh chứng trong `../evidence/` và đường dẫn evidence phải tồn tại.
3. Nếu thiết bị không hỗ trợ, ghi `Blocked` kèm lý do, không ghi Pass.
4. Mọi thay đổi kỳ vọng (`Expected result`) phải có lý do và đồng bộ với tài liệu trong `docs/`.
