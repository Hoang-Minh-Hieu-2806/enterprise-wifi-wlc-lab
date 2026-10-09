# Packet Tracer

Thư mục chứa file mô phỏng Cisco Packet Tracer (`.pkt`) của dự án.

## Hiện trạng

> **Chưa có file `.pkt`** — thư mục hiện chỉ có README này.

## Quy định khi bổ sung file `.pkt`

1. Chỉ commit file `.pkt` cuối cùng đã được kiểm tra mở được trên đúng phiên bản Packet Tracer đã chốt trong [../docs/implementation-notes.md](../docs/implementation-notes.md).
2. Tên file nên rõ ràng, dùng tiếng Anh, ví dụ: `enterprise-wifi-wlc-lab-v1.pkt`.
3. Hạn chế commit nhiều bản nháp; nếu cần giữ bản cũ, ghi rõ số phiên bản và lý do.
4. Mọi thay đổi topology phải đồng bộ với [../docs/topology.md](../docs/topology.md) và [../docs/ip-vlan-plan.md](../docs/ip-vlan-plan.md).
5. Không commit file tạm (`*.pkttmp`, `*.bak.pkt`) — các định dạng này đã bị loại trừ trong `.gitignore`.

## Cách mở

Xem hướng dẫn tại [../README.md](../README.md#7-cách-mở-dự-án-packet-tracer-khi-đã-có-file-pkt).
