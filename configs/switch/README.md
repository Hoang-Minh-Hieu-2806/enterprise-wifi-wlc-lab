# Switch Configs

Lưu bản sao cấu hình của switch (`show running-config`, `show vlan brief` khi cần). Người sở hữu: Huỳnh Như Hoàng; phối hợp Đặng Văn Hoàng về uplink WLC/AP, Hoàng Minh Hiếu hỗ trợ khi cần.

## Hiện trạng (final build 2026-11-07)

> **Đã xuất:** `core-sw-running-config.txt` — `CORE-SW` (**3560-24PS**, L3; thuộc bản final `packet-tracer/final.pkt`) — startup-config thật từ live PT, xuất **2026-11-07**, bản 9.0.1.0858.
>
> Đã kiểm chứng (final):
>
> - `ip routing` + SVI `Vlan10` `.10.1` / `Vlan20` `.20.1` / `Vlan99` `.99.1` (gateway 3 VLAN).
> - `Gi0/1` (→ R1) và `Gi0/2` (→ R2) `no switchport` với IP `/30` (`10.0.0.2`, `10.0.1.2`).
> - `Fa0/1` trunk (→ ACCESS-SW) mang VLAN 10/20/99 cho WLC/AP.
> - `ip helper-address 192.168.99.20` trên `Vlan10`/`Vlan20` (giữ cho request không đi qua WLC).
> - ACL `GUEST-ACL` binding `ip access-group GUEST-ACL in` trên `Vlan20` — chặn Guest→Staff/Mgmt, permit DHCP (TC-10/TC-11).
> - Định tuyến: `ip route 0.0.0.0 0.0.0.0 10.0.0.1` (AD 1) + backup `10.0.1.1 254` (failover R1→R2, design).
>
> Topology + ý nghĩa cấu hình: [../../docs/final-implementation.md](../../docs/final-implementation.md) §1–§2, §6.

## Lịch sử giai đoạn P1 (v1, trước 2026-11-01)

- `switch-sw1-running-config.txt` — `SW1` (2960-24TT) của khung v1: `Fa0/3`/`Fa0/4` trunk native 99, `Gi0/1` trunk (→ R1 2911), SVI `Vlan99` `.99.2`. **SW1 không còn trong topology final** — giữ lại chỉ để đối chiếu giai đoạn P1; lịch sử phiên bản xem `git log -- configs/switch/` (minh chứng cũ `tc-01-*`, `tc-08-*` đã xoá trong đợt final).

## Quy định

- Tên file tiếng Anh, ví dụ `core-sw-running-config.txt`.
- Ghi rõ model, cổng thực tế, bản 9.0.1.0858, ngày xuất, người thực hiện.
- Chỉ commit cấu hình đã test, có minh chứng tại `../../evidence/`.
