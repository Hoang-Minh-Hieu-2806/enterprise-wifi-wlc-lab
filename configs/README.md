# Configs

Thư mục lưu bản sao cấu hình thiết bị để đối chiếu và chấm bài. Mỗi thiết bị/nhóm thiết bị có thư mục riêng:

- `router/` — cấu hình router (định tuyến VLAN, DHCP relay nếu có, ACL).
- `switch/` — cấu hình switch Layer 2 (VLAN, trunk/access).
- `wlc-ap/` — cấu hình WLC và AP (SSID, ánh xạ VLAN, xác thực vô tuyến, RADIUS nếu có).

## Hiện trạng

> **Chưa có cấu hình** — chỉ có README định hướng.

## Quy định khi bổ sung

1. Chỉ commit cấu hình **đã kiểm chứng** trên đúng model thiết bị trong [../docs/implementation-notes.md](../docs/implementation-notes.md).
2. Mỗi file ghi rõ: thiết bị, model, phiên bản Packet Tracer, ngày xuất cấu hình, người thực hiện.
3. Ưu tiên xuất bằng `show running-config` và lưu dưới dạng `.txt`, tên file tiếng Anh (ví dụ `router-r1-running-config.txt`).
4. **Không commit mật khẩu, pre-shared key, secret thật.** Che hoặc thay bằng placeholder như `<STAFF-PSK-REDACTED>`.
5. Không viết câu lệnh CLI “mẫu” như thể đã chạy thành công — mọi nội dung ở đây phải đi kèm minh chứng kiểm thử trong `../evidence/`.
