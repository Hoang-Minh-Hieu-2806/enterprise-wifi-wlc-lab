# Evidence

Thư mục lưu minh chứng kiểm thử: ảnh chụp màn hình, kết quả lệnh, export cấu hình đã che thông tin nhạy cảm.

## Hiện trạng

> **Chưa có minh chứng** — mọi test case đang ở trạng thái `Not tested`.

## Quy định

1. Tên file tiếng Anh, gắn với test ID, ví dụ: `tc-01-ap-registration.png`, `tc-04-ping-gateway.png`.
2. Ảnh phải đọc được: thấy rõ lệnh, kết quả, SSID/IP liên quan; che PSK/password/tài khoản thật.
3. Chỉ commit minh chứng của lần test thật trên đúng topology và phiên bản Packet Tracer đã chốt.
4. Không dùng ảnh minh họa trên mạng, không chỉnh sửa kết quả.
5. Mỗi file evidence phải được tham chiếu từ [../tests/test-cases.md](../tests/test-cases.md) hoặc [../docs/testing-plan.md](../docs/testing-plan.md).
