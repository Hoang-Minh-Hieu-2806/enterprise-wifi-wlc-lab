# Switch Configs

Lưu bản sao cấu hình của switch Layer 2 (ví dụ `show running-config`, `show vlan brief`).

## Hiện trạng

> Chưa có cấu hình (Pending).

## Nội dung dự kiến

- VLAN 10 (STAFF), 20 (GUEST), 99 (MANAGEMENT).
- Cổng trunk tới router, WLC, AP; cổng access tới server theo VLAN đã chốt.
- Bảng VLAN cuối cùng phải khớp [../../docs/ip-vlan-plan.md](../../docs/ip-vlan-plan.md) và [../../docs/topology.md](../../docs/topology.md).

## Quy định

- Tên file tiếng Anh, ví dụ `switch-s1-running-config.txt`.
- Ghi rõ model thiết bị, phiên bản Packet Tracer, ngày xuất.
- Chỉ commit cấu hình đã test, có minh chứng tại `../../evidence/`.
