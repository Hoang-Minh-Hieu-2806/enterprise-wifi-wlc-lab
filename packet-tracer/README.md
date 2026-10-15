# Packet Tracer

Thư mục chứa file mô phỏng Cisco Packet Tracer (`.pkt`) của dự án, thực hiện trên đúng bản **9.0.1.0858**.

## Hiện trạng

> **Chưa có file `.pkt`** — thư mục hiện chỉ có README này. Không tạo file `.pkt` giả.

## Quy định khi bổ sung file `.pkt`

1. Chỉ commit file `.pkt` đã mở và kiểm thử được trên đúng bản 9.0.1.0858 (xem [../docs/implementation-notes.md](../docs/implementation-notes.md)).
2. Bắt đầu với **01 LAP-PT**; chỉ thêm AP khi TC-03 và TC-06 đã `Passed` (xem [../tests/test-cases.md](../tests/test-cases.md)).
3. Tên file rõ ràng, tiếng Anh, ví dụ: `enterprise-wifi-wlc-lab-v1.pkt`; hạn chế nhiều bản nháp.
4. Mọi thay đổi topology phải đồng bộ với [../docs/topology.md](../docs/topology.md) và địa chỉ chuẩn tại [../docs/ip-vlan-plan.md](../docs/ip-vlan-plan.md), sau khi cả 3 thành viên thống nhất.
5. Không commit file tạm (`*.pkttmp`, `*.bak.pkt`) — đã loại trừ trong `.gitignore`.

## Cách mở

Xem hướng dẫn tại [../README.md](../README.md#7-cách-mở-dự-án-packet-tracer-khi-đã-có-file-pkt).
