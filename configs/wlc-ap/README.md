# WLC & AP Configs

Lưu bản sao cấu hình/ghi chú cấu hình của WLC và LAP-PT (screenshot GUI WLC, export nếu có, danh sách WLAN/SSID). Người sở hữu: Đặng Văn Hoàng.

## Hiện trạng

> Chưa có cấu hình (Planned). GUI `Wireless LANs` và `Primary Controller` mới chỉ được nhìn thấy, chưa kiểm chứng hoạt động.

## Nội dung dự kiến (làm theo thứ tự)

1. WLC management reachability (TC-01) và LAP-PT join (TC-02, TC-03) — bắt đầu 01 LAP-PT.
2. WLAN `STAFF-WIFI` → VLAN 10 và `GUEST-WIFI` → VLAN 20 với **WPA2-PSK trước** (TC-04→TC-07).
3. RADIUS/WPA2-Enterprise (TC-13, Conditional) chỉ sau khi Core ổn định và khi được hỗ trợ.

AP-PT-A chỉ là dự phòng standalone, không thuộc luồng chính.

## Quy định

- Ghi rõ model WLC/LAP-PT, bản 9.0.1.0858, ngày ghi nhận (xem [../../docs/implementation-notes.md](../../docs/implementation-notes.md)).
- Che PSK và RADIUS secret (`<PSK-REDACTED>`, `<RADIUS-SECRET-REDACTED>`).
- Không khẳng định WLAN phát sóng, map VLAN hay RADIUS chạy được khi chưa có minh chứng tại `../../evidence/`.
- Nếu không export được text, lưu screenshot GUI vào `../../evidence/` và tóm tắt text tại đây.
