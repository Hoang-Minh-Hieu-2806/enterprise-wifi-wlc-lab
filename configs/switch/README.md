# Switch Configs

Lưu bản sao cấu hình của switch Layer 2 (ví dụ `show running-config`, `show vlan brief`). Người sở hữu: Huỳnh Như Hoàng; phối hợp Đặng Văn Hoàng về uplink WLC/LAP-PT, Hoàng Minh Hiếu hỗ trợ khi cần; chốt mode theo model thực tế.

## Hiện trạng

> Chưa có cấu hình (Planned). Cổng Switch↔WLC **bắt buộc trunk**; các cổng còn lại không giả định trunk.

## Nội dung dự kiến

- VLAN 10 (STAFF), 20 (GUEST), 99 (MANAGEMENT).
- Cổng tới WLC: **trunk** mang VLAN 99 untagged/native + VLAN 10, 20 tagged (lý do tại [../../docs/topology.md](../../docs/topology.md) §3.1).
- Cổng tới router, LAP-PT, Server-PT: trunk hay access **chốt độc lập từng đoạn** theo model.
- Bảng VLAN cuối cùng phải khớp [../../docs/ip-vlan-plan.md](../../docs/ip-vlan-plan.md) (kiểm chứng ở TC-12).

## Quy định

- Tên file tiếng Anh, ví dụ `switch-s1-running-config.txt`.
- Ghi rõ model, cổng thực tế, bản 9.0.1.0858, ngày xuất, người thực hiện.
- Chỉ commit cấu hình đã test, có minh chứng tại `../../evidence/`.
