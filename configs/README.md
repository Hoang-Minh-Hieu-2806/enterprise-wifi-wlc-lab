# Configs

Thư mục lưu bản sao cấu hình thiết bị để đối chiếu và chấm bài (đã che secret). Mỗi thiết bị/nhóm thiết bị có thư mục riêng:

- `router/` — liên VLAN, DHCP relay **bắt buộc** cho VLAN 10/20, ACL (Huỳnh Như Hoàng sở hữu; phối hợp Đặng Văn Hoàng về interface/VLAN, Hoàng Minh Hiếu hỗ trợ khi cần).
- `switch/` — VLAN, cổng Switch↔WLC **trunk bắt buộc**, các cổng còn lại chốt theo model thực tế (Huỳnh Như Hoàng sở hữu; phối hợp Đặng Văn Hoàng về uplink WLC/LAP-PT, Hoàng Minh Hiếu hỗ trợ khi cần).
- `wlc-ap/` — WLAN/SSID, WPA2-PSK (làm trước), RADIUS/WPA2-Enterprise (Conditional, thử sau) — Đặng Văn Hoàng sở hữu.

## Hiện trạng

> **Chưa có cấu hình** — chỉ có README định hướng. Không ghi lệnh chưa kiểm chứng thành cấu hình đã chạy.

## Quy định khi bổ sung

1. Chỉ commit cấu hình **đã kiểm chứng** trên đúng model và đúng bản 9.0.1.0858 trong [../docs/implementation-notes.md](../docs/implementation-notes.md).
2. Mỗi file ghi rõ: thiết bị, model, interface thực dùng, ngày xuất, người thực hiện.
3. Ưu tiên `show running-config` lưu `.txt` tiếng Anh (ví dụ `router-r1-running-config.txt`).
4. **Không commit PSK, password, RADIUS secret thật.** Che hoặc thay bằng placeholder như `<STAFF-PSK-REDACTED>`, `<RADIUS-SECRET-REDACTED>`.
5. Mọi nội dung phải đi kèm minh chứng kiểm thử trong `../evidence/` và test tương ứng trong `../tests/test-cases.md`.
