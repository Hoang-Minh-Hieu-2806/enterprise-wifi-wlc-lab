# Switch Configs

Lưu bản sao cấu hình của switch Layer 2 (ví dụ `show running-config`, `show vlan brief`). Người sở hữu: Huỳnh Như Hoàng; phối hợp Đặng Văn Hoàng về uplink WLC/LAP-PT, Hoàng Minh Hiếu hỗ trợ khi cần; chốt mode theo model thực tế.

## Hiện trạng

> **Đã có cấu hình khung có dây.** `switch-sw1-running-config.txt` — model `2960-24TT`, xuất ngày 2026-10-18 gồm `show running-config` và `show vlan brief` trên bản 9.0.1.0858.
>
> Đã kiểm chứng trong khung P1: VLAN 10 (STAFF), 20 (GUEST), 99 (MANAGEMENT) active; `Fa0/1` trunk (→ WLC), `Gi0/1` trunk (→ router) native VLAN 99 + allowed 10,20,99; `Fa0/2` access VLAN 99 (→ Server-PT); SVI `Vlan99` = `192.168.99.2/24`. Model là **dot1q-only** nên không dùng `switchport trunk encapsulation dot1q`.
>
> **Chưa có:** cổng tới LAP-PT, và chưa test case nào `Passed`. Minh chứng bring-up P1 tại [../../evidence/p1-wired-skeleton-topology.png](../../evidence/p1-wired-skeleton-topology.png).

## Nội dung dự kiến

- VLAN 10 (STAFF), 20 (GUEST), 99 (MANAGEMENT).
- Cổng tới WLC: **trunk** mang VLAN 99 untagged/native + VLAN 10, 20 tagged (lý do tại [../../docs/topology.md](../../docs/topology.md) §3.1).
- Cổng tới router, LAP-PT, Server-PT: trunk hay access **chốt độc lập từng đoạn** theo model.
- Bảng VLAN cuối cùng phải khớp [../../docs/ip-vlan-plan.md](../../docs/ip-vlan-plan.md) (kiểm chứng ở TC-12).

## Quy định

- Tên file tiếng Anh, ví dụ `switch-s1-running-config.txt`.
- Ghi rõ model, cổng thực tế, bản 9.0.1.0858, ngày xuất, người thực hiện.
- Chỉ commit cấu hình đã test, có minh chứng tại `../../evidence/`.
