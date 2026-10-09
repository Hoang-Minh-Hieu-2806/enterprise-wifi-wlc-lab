# Ghi chú triển khai (Implementation Notes)

> File này ghi lại quyết định cấu hình **đã được kiểm chứng**. Hiện chưa có triển khai — mọi mục để ở trạng thái chờ.

## 1. Phiên bản Packet Tracer

| Hạng mục | Nội dung |
| --- | --- |
| Phiên bản Packet Tracer sử dụng | _Chưa chốt (Pending)_ — dự kiến 9.x |
| Người kiểm chứng | _Chưa có_ |
| Ngày chốt | _Chưa có_ |

Khi chốt phiên bản, ghi rõ số bản đầy đủ (ví dụ `9.x.x`) và hệ điều hành kiểm thử.

## 2. Model thiết bị (sẽ chốt sau kiểm chứng)

| Vai trò | Model dự kiến | Trạng thái hỗ trợ |
| --- | --- | --- |
| Router | _Chưa chốt_ | Pending — cần hỗ trợ sub-interface / trunk và ACL |
| Switch L2 | _Chưa chốt_ | Pending — cần hỗ trợ VLAN + trunk |
| WLC | _Chưa chốt_ | Pending — cần hỗ trợ quản lý AP tập trung |
| AP | _Chưa chốt_ (2–3 thiết bị) | Pending — cần hỗ trợ multi-SSID |
| Server | _Chưa chốt_ | Pending — cần DHCP; RADIUS là điểm cần kiểm chứng riêng |
| Wireless client | Laptop/Smartphone PT | Pending |

> Không giả định mọi model WLC/AP hay tính năng xác thực đều có trong Packet Tracer 9.x. Chỉ điền model sau khi mở Packet Tracer và xác nhận tồn tại.

## 3. Tính năng được hỗ trợ / không được hỗ trợ

| Tính năng | Kết quả kiểm chứng |
| --- | --- |
| WLC quản lý tập trung AP | _Chưa kiểm chứng_ |
| Multi-SSID trên AP | _Chưa kiểm chứng_ |
| Ánh xạ SSID → VLAN | _Chưa kiểm chứng_ |
| DHCP theo VLAN | _Chưa kiểm chứng_ |
| WPA2-PSK | _Chưa kiểm chứng (dự kiến khả thi)_ |
| RADIUS + WPA2-Enterprise | _Chưa kiểm chứng — cần xác minh_ |
| ACL trên router | _Chưa kiểm chứng_ |
| WPA3 / EAP cụ thể / roaming / đo throughput | Chủ đề nghiên cứu, chưa kiểm chứng |

## 4. Quyết định cấu hình

_Chưa có. Khi bắt đầu cấu hình, ghi ngắn gọn: chọn gì, vì sao, khác gì so với [ip-vlan-plan.md](ip-vlan-plan.md) và [topology.md](topology.md)._

Ví dụ cách ghi (chưa áp dụng):

- `2026-xx-xx: chọn model ... vì ...; khác đề xuất ở ...`.

## 5. Vấn đề và cách khắc phục (Issues & Workarounds)

_Chưa có. Ghi theo mẫu:_

| Ngày | Vấn đề | Cách khắc phục | Trạng thái |
| --- | --- | --- | --- |
| — | — | — | — |

## 6. Hạn chế đã biết

- Chưa xác nhận model WLC/AP khả dụng và số AP tối đa ổn định.
- Chưa xác nhận RADIUS + WPA2-Enterprise có khả thi không.
- Chưa xác nhận AP có hỗ trợ multi-SSID ánh xạ multi-VLAN không.
- Không đưa câu lệnh CLI chi tiết vào đây như “đã xác nhận đúng” khi chưa test trên đúng model thiết bị. Mọi lệnh phải đi kèm minh chứng trong `../evidence/`.
