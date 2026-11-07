# Router Configs

Lưu bản sao cấu hình của router (startup-config thật xuất từ live PT; nếu cần bổ sung thêm `show running-config`, `show access-lists`). Người sở hữu: Huỳnh Như Hoàng; phối hợp Đặng Văn Hoàng về interface/VLAN, Hoàng Minh Hiếu hỗ trợ khi cần.

## Hiện trạng (final build 2026-11-07)

> **Đã xuất:**
>
> - `router-r1-running-config.txt` — `R1` (**2901**, thuộc bản final `packet-tracer/final.pkt`) — xuất 2026-11-07, PT 9.0.1.0858.
> - `router-r2-running-config.txt` — `R2` (**2901**) — cấu hình dự phòng/cân bằng tương tự R1.

> Đã kiểm chứng (final): PAT overload `list 1` ra `Gi0/0`, `ip nat inside` trên `Gi0/1`; static routes `192.168.10/20/99` → next-hop CORE-SW (`10.0.0.2` trên R1, `10.0.1.2` trên R2); default route → ISP (`203.0.113.1` trên R1, `203.0.113.5` trên R2); Staff/Guest ping `8.8.8.8` 4/4 (PAT hoạt động).
>
> Topology + ý nghĩa cấu hình: [../../docs/final-implementation.md](../../docs/final-implementation.md) §2–§3 và §2.3.

## Nội dung cấu hình mỗi file

- PAT overload: `ip nat inside source list 1 interface Gi0/0 overload`; ACL NAT permit 3 subnet nội bộ (10/20/99), dùng chung cho hai path.
- Static route về LAN: 3 route cho `192.168.10.0/24`, `192.168.20.0/24`, `192.168.99.0/24` — next-hop là CORE-SW theo path `/30` (`10.0.0.2` trên R1, `10.0.1.2` trên R2).
- Default route → ISP (`203.0.113.1` trên R1 / `203.0.113.5` trên R2).

## Lịch sử giai đoạn P1 (v1, trước 2026-11-01)

- `router-r1-running-config.txt` trước đợt final mô tả khung `2911` router-on-a-stick (subinterface `Gi0/0.10/.20`, `ip helper-address` trên subinterface); minh chứng bring-up P1 tại [../../evidence/p1-wired-skeleton-topology.png](../../evidence/p1-wired-skeleton-topology.png) — nay file chứa bản final; lịch sử phiên bản v1 xem `git log -- configs/router/`.

## Quy định

- Tên file tiếng Anh, ví dụ `router-r1-running-config.txt`.
- Ghi rõ model, interface thực dùng, bản 9.0.1.0858, ngày xuất, người thực hiện.
- Che secret; chỉ commit cấu hình đã test, có minh chứng tại `../../evidence/`.
