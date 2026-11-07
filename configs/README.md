# Configs

Thư mục lưu bản sao cấu hình thiết bị để đối chiếu và chấm bài (đã che secret). Mỗi thiết bị/nhóm thiết bị có thư mục riêng:

- `router/` — **R1 + R2 (2901)**: PAT overload, static routes, access NAT ACL — cấu hình **final build** xuất 2026-11-07.
- `switch/` — **CORE-SW (3560-24PS, L3)**: SVI 10/20/99, DHCP relay, ACL Guest, default route (AD 1 + backup 254) — cấu hình **final build** xuất 2026-11-07.
- `wlc-ap/` — WLAN/SSID, WPA2 (PSK + 802.1X/RADIUS) trên WLC-3504 + 2 AP (Đặng Văn Hoàng sở hữu) — cấu hình WLC không có CLI, lưu dạng ghi chú GUI.

> ⚠ **Không commit RADIUS secret / PSK thật vào các file này** (placeholder `<RADIUS-SECRET-REDACTED>`). Giá trị thật chỉ tồn tại trong `final.pkt`.
>
> *Các file `switch-sw1-*`, `router-r1…` v1 cũ trước 2026-11-01 là **lịch sử giai đoạn P1** (SW1 không còn trong topology final).*

## Hiện trạng

> **Final build (2026-11-07):** `router/router-r1-running-config.txt` (R1, 2901), `router/router-r2-running-config.txt` (R2, 2901) và `switch/core-sw-running-config.txt` (CORE-SW, 3560-24PS) — xuất **startup-config thật từ live PT** thuộc `packet-tracer/final.pkt`. Không có secret (lab minh bạch, không đặt enable secret). Topology + ý nghĩa cấu hình: [../docs/final-implementation.md](../docs/final-implementation.md).
>
> **Kiểm chứng kèm theo:** ACL `GUEST-ACL` binding trên `Vlan20 in` nằm trong startup-config; PAT + static route khớp trên R1/R2; `ip helper-address` trên Vlan10/Vlan20.
>
> **Kiểm chứng kèm theo (giới hạn):** WLC1/RADIUS-SRV/DHCP-SRV cấu hình qua GUI (không có CLI) — mô tả tại `configs/wlc-ap/README.md` và final doc mục 3–5.
>
> **Lịch sử giai đoạn P1 (v1, trước 2026-11-01):** `router/`, `switch/` có các file cấu hình cũ của giai đoạn khung có dây 9-thiết bị (SW1 + R1 2911 router-on-a-stick). File `switch-sw1-running-config.txt` sẽ được giữ lại cho đối chiếu; minh chứng cũ `tc-08-*` đã bị xoá trong đợt final (xem `git log -- configs/ evidence/`).

## Quy định khi bổ sung

1. Chỉ commit cấu hình **đã kiểm chứng** trên đúng model và đúng bản 9.0.1.0858 trong [../docs/implementation-notes.md](../docs/implementation-notes.md).
2. Mỗi file ghi rõ: thiết bị, model, interface thực dùng, ngày xuất, người thực hiện.
3. Ưu tiên `show running-config` lưu `.txt` tiếng Anh (ví dụ `router-r1-running-config.txt`).
4. **Không commit PSK, password, RADIUS secret thật.** Che hoặc thay bằng placeholder như `<STAFF-PSK-REDACTED>`, `<RADIUS-SECRET-REDACTED>`.
5. Mọi nội dung phải đi kèm minh chứng kiểm thử trong `../evidence/` và test tương ứng trong `../tests/test-cases.md`.
