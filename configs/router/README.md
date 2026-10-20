# Router Configs

Lưu bản sao cấu hình của router (ví dụ `show running-config`, `show access-lists`). Người sở hữu: Huỳnh Như Hoàng; phối hợp Đặng Văn Hoàng về interface/VLAN, Hoàng Minh Hiếu hỗ trợ khi cần.

## Hiện trạng

> **Đã có cấu hình khung có dây + TC-08.** `router-r1-running-config.txt` — model `2911`, xuất lại ngày 2026-10-21 từ official `packet-tracer/enterprise-wifi-wlc-lab-v1.pkt`, trên bản 9.0.1.0858.
>
> Đã kiểm chứng: `Gi0/0` = `192.168.99.1/24` (VLAN 99 native), `Gi0/0.10`/`Gi0/0.20` dot1Q với `ip helper-address 192.168.99.20`; client VLAN 10/20 nhận DHCP đúng subnet (TC-08).
>
> **Chưa có:** ACL. **TC-01 `Passed`** — minh chứng [../../evidence/tc-01-wlc-reachability.png](../../evidence/tc-01-wlc-reachability.png). Minh chứng bring-up P1 tại [../../evidence/p1-wired-skeleton-topology.png](../../evidence/p1-wired-skeleton-topology.png).

## Nội dung dự kiến (theo [../../docs/ip-vlan-plan.md](../../docs/ip-vlan-plan.md))

- Liên VLAN cho 10/20/99 với gateway `192.168.10.1`, `192.168.20.1`, `192.168.99.1` (thiết kế subinterface 802.1Q hoặc theo model thực tế).
- DHCP relay **bắt buộc**: `ip helper-address 192.168.99.20` trên subinterface VLAN 10 và VLAN 20 (xem [../../docs/ip-vlan-plan.md](../../docs/ip-vlan-plan.md) §3.1).
- ACL explicit deny Guest→Staff/Management + permit tối thiểu, trong đó **phải permit DHCP** tới Server-PT (kiểm chứng ở TC-10, TC-11, TC-12).

## Quy định

- Tên file tiếng Anh, ví dụ `router-r1-running-config.txt`.
- Ghi rõ model, interface thực dùng, bản 9.0.1.0858, ngày xuất, người thực hiện.
- Che secret; chỉ commit cấu hình đã test, có minh chứng tại `../../evidence/`.
