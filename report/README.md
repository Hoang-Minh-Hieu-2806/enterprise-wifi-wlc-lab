# Report

Thư mục quản lý dàn ý và nội dung báo cáo tổng hợp trong giai đoạn hiện tại. **Chưa cần tạo DOCX/PDF.** Huỳnh Như Hoàng viết nháp kết quả/limitation kỹ thuật; Hoàng Minh Hiếu tổng hợp báo cáo cuối và chuẩn bị dàn ý thuyết trình.

> **⚠ Cập nhật final (2026-11-07):** deliverable cuối là `packet-tracer/final.pkt` (topology enterprise 18 thiết bị). Trạng thái và kết quả kiểm thử cuối kỳ tại **[../docs/final-implementation.md](../docs/final-implementation.md)** — đây là nguồn chính thức cho báo cáo cuối kỳ; các đoạn dưới đây giữ làm trạng thái giai đoạn giữa kỳ (v1).

## Hiện trạng

> **Đã có báo cáo giữa kỳ:** [midterm-report.md](midterm-report.md) — theo đúng 4 mục yêu cầu của buổi báo cáo giữa kỳ (tóm tắt ý tưởng + công việc dự định, tính năng chính, mô hình triển khai, kịch bản demo). Nội dung là **ý tưởng và dự định**, chưa có kết quả kiểm thử.
>
> **Báo cáo cuối kỳ đã hoàn thành** (file DOCX `Bao-cao-cuoi-ky-Nhom-11-ban-hoan-chinh.docx` lưu tại thư mục `Đồ án`, không nằm trong repo). Kết quả cuối kỳ tại **[../docs/final-implementation.md](../docs/final-implementation.md)**: TC-01→TC-08 là lịch sử giai đoạn v1 (`WLC-PT`); TC-09→TC-12 Passed trên bản final, TC-13 "đã cấu hình, chưa chứng minh exchange", TC-14 Not Tested. Đoạn mô tả v1 dưới đây chỉ giữ làm đối chiếu giai đoạn giữa kỳ.

## Dàn ý báo cáo cuối kỳ (dự kiến)

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
