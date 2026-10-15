# Report

Thư mục quản lý dàn ý và nội dung báo cáo tổng hợp trong giai đoạn hiện tại. **Chưa cần tạo DOCX/PDF.** Người viết nháp chính: Huỳnh Như Hoàng (kỹ thuật, limitation, kết quả); Hoàng Minh Hiếu duyệt cuối và chuẩn bị dàn ý thuyết trình.

## Hiện trạng

> **Chưa có báo cáo** — đang ở giai đoạn tài liệu sẵn sàng triển khai, toàn bộ test `Not Tested`.

## Dàn ý báo cáo (dự kiến)

1. Mục tiêu và phạm vi (Core / Conditional / Research-only).
2. Topology cuối cùng (model thực tế, 01 LAP-PT trước) + sơ đồ.
3. Kế hoạch IP/VLAN theo [../docs/ip-vlan-plan.md](../docs/ip-vlan-plan.md).
4. Thiết kế bảo mật: WPA2-PSK trước, RADIUS Conditional, ACL explicit deny + permit.
5. Quá trình triển khai + limitation theo [../docs/implementation-notes.md](../docs/implementation-notes.md).
6. Kết quả kiểm thử TC-01→TC-14 kèm minh chứng trong `../evidence/`.
7. Hạn chế, blocker và hướng phát triển (ghi rõ RADIUS `Blocked` nếu có mà không ảnh hưởng Core).

## Quy định khi bổ sung

1. Báo cáo viết bằng **tiếng Việt**, thuật ngữ kỹ thuật và định danh (SSID, VLAN, IP) giữ **tiếng Anh**.
2. Chỉ ghi nhận điều **đã làm và đã kiểm chứng**; WPA3, EAP cụ thể, roaming, throughput ghi rõ là nghiên cứu.
3. Mọi số liệu, ảnh, kết quả lệnh phải có nguồn tương ứng trong `../packet-tracer/`, `../configs/`, `../evidence/` hoặc `../tests/test-cases.md`.
4. Không bịa kết quả, không dùng ảnh minh họa ngoài đồ án.
