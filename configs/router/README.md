# Router Configs

Lưu bản sao cấu hình của router (ví dụ `show running-config`).

## Hiện trạng

> Chưa có cấu hình (Pending).

## Nội dung dự kiến

- Sub-interface hoặc interface cho VLAN 10, 20, 99 với gateway `192.168.10.1`, `192.168.20.1`, `192.168.99.1` (xem [../../docs/ip-vlan-plan.md](../../docs/ip-vlan-plan.md)).
- DHCP hoặc DHCP relay (nếu dùng).
- ACL kiểm soát Guest → Staff/Management (nếu triển khai).

## Quy định

- Tên file tiếng Anh, ví dụ `router-r1-running-config.txt`.
- Ghi rõ model thiết bị, phiên bản Packet Tracer, ngày xuất.
- Che mật khẩu/secret; không commit thông tin nhạy cảm.
- Chỉ commit cấu hình đã test, có minh chứng tại `../../evidence/`.
