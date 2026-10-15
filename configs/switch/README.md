# Switch Configs

Lưu bản sao cấu hình của switch Layer 2 (ví dụ `show running-config`, `show vlan brief`). Đặng Văn Hoàng và Huỳnh Như Hoàng phối hợp; chốt mode theo model thực tế.

## Hiện trạng

> Chưa có cấu hình (Planned). Không giả định trunk cho mọi cổng.

## Nội dung dự kiến

- VLAN 10 (STAFF), 20 (GUEST), 99 (MANAGEMENT).
- Cổng tới router, WLC, LAP-PT, Server-PT: trunk hay access **chốt độc lập từng đoạn** theo model (xem [../../docs/topology.md](../../docs/topology.md)).
- Bảng VLAN cuối cùng phải khớp [../../docs/ip-vlan-plan.md](../../docs/ip-vlan-plan.md) (kiểm chứng ở TC-12).

## Quy định

- Tên file tiếng Anh, ví dụ `switch-s1-running-config.txt`.
- Ghi rõ model, cổng thực tế, bản 9.0.1.0858, ngày xuất, người thực hiện.
- Chỉ commit cấu hình đã test, có minh chứng tại `../../evidence/`.
