# Ghi chú triển khai (Implementation Notes)

> File này ghi lại **phát hiện đã quan sát** và **quyết định đã kiểm chứng**. Mọi chức năng hoạt động vẫn ở trạng thái chưa kiểm chứng cho tới khi có minh chứng trong `../evidence/`.

## 1. Phiên bản Packet Tracer (đã chốt)

| Hạng mục | Nội dung |
| --- | --- |
| Phiên bản Packet Tracer sử dụng | 9.0.1.0858 (đã chốt) |
| Người kiểm chứng GUI | Nhóm đồ án (chi tiết đóng góp tại [team-contributions.md](team-contributions.md)) |
| Ngày chốt | Giai đoạn finalize tài liệu (ngày kiểm thử thực tế ghi kèm từng evidence) |

Mọi file `.pkt`, cấu hình và screenshot phải thực hiện trên đúng bản này.

## 2. Model thiết bị đã quan sát trong Packet Tracer 9.0.1.0858

| Vai trò | Thiết bị quan sát được | Chi tiết GUI đã thấy (chưa phải bằng chứng chạy được) |
| --- | --- | --- |
| WLC | Wireless LAN Controller | Management IPv4 + mask; Gigabit Ethernet; trang `Wireless LANs` với WLAN Name, SSID, VLAN field; Authentication gồm WPA2-PSK và WPA2; RADIUS Server Settings (IP + shared secret); Encryption Type; central switching / central authentication |
| AP chính | LAP-PT (`Light Weight Access Point`) | GigabitEthernet0; Dot11Radio0; trang `WLC` với trường `Primary Controller` |
| AP dự phòng | AP-PT-A | Port 1 wireless: SSID, WPA-PSK, WPA2-PSK, WEP, Encryption Type; Port 0 Ethernet link settings |
| Palette tham khảo | WLC, 3504, 2504, LAP-PT, 3702i, AP-PT, AP-PT-A, AP-PT-AC, AP-PT-N, Meraki, home router, wireless khác | Khả năng khác nhau, không giả định tương đương |
| Router | Sẽ chốt model cụ thể khi dựng `.pkt` | Cần subinterface 802.1Q (hoặc thiết kế model hỗ trợ) + ACL |
| Switch | Sẽ chốt model cụ thể khi dựng `.pkt` | Cần VLAN + trunk/access |
| Server | Server-PT | DHCP (ưu tiên xác minh trước); RADIUS kiểm tra sau |
| Client | Laptop/Smartphone PT (≥ 4) | 2 Staff + 2 Guest |

> Quy tắc: AP chính là LAP-PT, bắt đầu 01 AP. AP-PT-A chỉ là fallback standalone khi chứng minh WLC tập trung bất khả thi và scope được sửa đổi.

## 3. Ma trận quan sát vs kiểm chứng

| Hạng mục | Đã quan sát trong GUI | Đã kiểm chứng hoạt động |
| --- | --- | --- |
| Tồn tại WLC + trang WLAN | Có | Chưa (TC-01, TC-04, TC-05) |
| LAP-PT + `Primary Controller` | Có | Chưa (TC-02, TC-03) |
| Trường VLAN trên WLAN | Có | Chưa (TC-08, TC-12) |
| Tùy chọn WPA2-PSK | Có (WLC và AP-PT-A) | Chưa (TC-06, TC-07, TC-14) |
| Trường WPA2 / RADIUS IP+secret | Có | Chưa (TC-13, Conditional) |
| Trunk/access cho model đã chọn | Chưa | Chưa |
| DHCP Server-PT mọi VLAN | Chưa | Chưa (TC-08) |
| ACL cô lập Guest | Chưa | Chưa (TC-10, TC-11) |
| WPA3 / EAP / roaming / RF / throughput | Chưa thấy hỗ trợ | Research-only |

## 4. Quyết định cấu hình

_Chưa có quyết định đã kiểm chứng. Khi triển khai, ghi mỗi dòng theo mẫu:_

- `ngày — người làm: chọn model/interface/VLAN ... vì ...; khác đề xuất ở ...; evidence: ...`

Không ghi lệnh CLI vào đây như "đã chạy thành công" khi chưa test trên đúng model. Mọi lệnh phải đi kèm minh chứng trong `../evidence/` và bản redacted trong `../configs/`.

## 5. Vấn đề và cách khắc phục (Issues & Workarounds)

_Chưa phát sinh (chưa triển khai). Ghi theo mẫu khi gặp:_

| Ngày | Người ghi | Vấn đề | Cách khắc phục | TC liên quan | Trạng thái |
| --- | --- | --- | --- | --- | --- |
| — | — | — | — | — | — |

## 6. Hạn chế đã biết (chưa kiểm chứng)

- LAP-PT join WLC, phát WLAN, multi-WLAN đồng thời, map SSID→VLAN: chưa kiểm chứng.
- WPA2-PSK pass/fail, DHCP mọi VLAN, trunk đúng model, Guest isolation end-to-end: chưa kiểm chứng.
- RADIUS + WPA2-Enterprise: Conditional, thử sau cùng; `Blocked` được chấp nhận kèm lý do.
- WPA3 / EAP cụ thể / roaming / RF / throughput: Research-only.
