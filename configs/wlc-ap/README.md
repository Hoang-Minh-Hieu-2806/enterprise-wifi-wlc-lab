# WLC & AP Configs

Lưu bản sao cấu hình/ghi chú cấu hình của WLC và AP (screenshot GUI WLC, export cấu hình nếu có, danh sách SSID/WLAN).

## Hiện trạng

> Chưa có cấu hình (Pending).

## Nội dung dự kiến

- Danh sách WLAN/SSID: `STAFF-WIFI` → VLAN 10, `GUEST-WIFI` → VLAN 20.
- Trạng thái AP join WLC.
- Phương thức xác thực đã chọn (WPA2-PSK và/hoặc WPA2-Enterprise + RADIUS nếu khả thi).
- Địa chỉ quản trị WLC/AP thuộc VLAN 99.

## Quy định

- Ghi rõ model WLC/AP, phiên bản Packet Tracer, ngày ghi nhận (xem [../../docs/implementation-notes.md](../../docs/implementation-notes.md)).
- Che pre-shared key và credential RADIUS; dùng placeholder như `<PSK-REDACTED>`.
- Không khẳng định RADIUS/WPA2-Enterprise hoạt động khi chưa có minh chứng tại `../../evidence/`.
- Nếu Packet Tracer không hỗ trợ export text, lưu screenshot GUI vào `../../evidence/` và tóm tắt bằng text tại đây.
