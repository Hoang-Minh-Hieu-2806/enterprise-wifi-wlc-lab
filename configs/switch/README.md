# Switch Configs

Lưu bản sao cấu hình của switch Layer 2 (ví dụ `show running-config`, `show vlan brief`). Người sở hữu: Huỳnh Như Hoàng; phối hợp Đặng Văn Hoàng về uplink WLC/LAP-PT, Hoàng Minh Hiếu hỗ trợ khi cần; chốt mode theo model thực tế.

## Hiện trạng

> **Đã có cấu hình khung có dây + TC-08.** `switch-sw1-running-config.txt` — model `2960-24TT`, xuất lại ngày 2026-10-21 từ official `packet-tracer/enterprise-wifi-wlc-lab-v1.pkt`, gồm `show running-config` và `show vlan brief` trên bản 9.0.1.0858.
>
> Đã kiểm chứng: VLAN 10 (STAFF), 20 (GUEST), 99 (MANAGEMENT) active; `Fa0/3` trunk (→ LAP-PT) và `Fa0/4` trunk (→ WLC-PT) native VLAN 99 + allowed 10,20,99; `Gi0/1` trunk (→ router); `Fa0/2` access VLAN 99 (→ Server-PT); SVI `Vlan99` = `192.168.99.2/24`. `Fa0/1` giữ trunk legacy nhưng **không có link**, không dùng trong topology. Model là **dot1q-only** nên không dùng `switchport trunk encapsulation dot1q`.
>
> **`Fa0/3` (LAP-PT)** từ access VLAN 99 ở TC-02 đã chuyển thành **trunk native 99, allowed 10,20,99** cho TC-08; client VLAN 10/20 học MAC trên `Fa0/3` (local switching). Minh chứng: [../../evidence/tc-08-trunk.png](../../evidence/tc-08-trunk.png), [../../evidence/tc-08-mac-vlan.png](../../evidence/tc-08-mac-vlan.png).
>
> **TC-01 đã `Passed`** — minh chứng [../../evidence/tc-01-wlc-reachability.png](../../evidence/tc-01-wlc-reachability.png); lưu ý phía WLC giữ trunk native VLAN 1 (xem [../wlc-ap/README.md](../wlc-ap/README.md)). Minh chứng bring-up P1 tại [../../evidence/p1-wired-skeleton-topology.png](../../evidence/p1-wired-skeleton-topology.png).

## Nội dung dự kiến

- VLAN 10 (STAFF), 20 (GUEST), 99 (MANAGEMENT).
- Cổng tới WLC: **trunk** mang VLAN 99 untagged/native + VLAN 10, 20 tagged (lý do tại [../../docs/topology.md](../../docs/topology.md) §3.1); đường này đã kiểm chứng cho management, **central switching chưa kiểm chứng**.
- Cổng tới LAP-PT: **trunk native 99, allowed 10,20,99** trên `Fa0/3` (TC-08, local switching).
- Cổng tới router, Server-PT: trunk hay access **chốt độc lập từng đoạn** theo model.
- Bảng VLAN cuối cùng phải khớp [../../docs/ip-vlan-plan.md](../../docs/ip-vlan-plan.md) (kiểm chứng ở TC-12).

## Quy định

- Tên file tiếng Anh, ví dụ `switch-s1-running-config.txt`.
- Ghi rõ model, cổng thực tế, bản 9.0.1.0858, ngày xuất, người thực hiện.
- Chỉ commit cấu hình đã test, có minh chứng tại `../../evidence/`.
