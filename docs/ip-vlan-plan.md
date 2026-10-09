# Kế hoạch địa chỉ IP và VLAN (đề xuất)

> Toàn bộ giá trị trong tài liệu này là **đề xuất, chưa kiểm chứng**. Kế hoạch cuối cùng phụ thuộc vào topology được chốt và khả năng của thiết bị trong Packet Tracer 9.x.

## 1. Bảng VLAN và subnet đề xuất

| VLAN | Name | Subnet | Gateway | Purpose |
| ---- | ---------- | --------------- | ------------ | ------------------ |
| 10 | STAFF | 192.168.10.0/24 | 192.168.10.1 | Staff clients |
| 20 | GUEST | 192.168.20.0/24 | 192.168.20.1 | Guest clients |
| 99 | MANAGEMENT | 192.168.99.0/24 | 192.168.99.1 | Network management |

- Mỗi VLAN dùng `/24` để đơn giản cho đồ án năm 2, đủ địa chỉ cho client mô phỏng.
- Gateway dự kiến đặt tại router (`.1` của mỗi subnet).
- Tên VLAN và SSID dùng tiếng Anh, không dấu, theo quy ước trong [../README.md](../README.md).

## 2. Địa chỉ ví dụ cho thiết bị hạ tầng (tham khảo, chưa chốt)

| Thiết bị | VLAN | Địa chỉ ví dụ | Ghi chú |
| --- | --- | --- | --- |
| Router sub-interface / gateway VLAN 10 | 10 | 192.168.10.1 | Gateway Staff |
| Router sub-interface / gateway VLAN 20 | 20 | 192.168.20.1 | Gateway Guest |
| Router interface / gateway VLAN 99 | 99 | 192.168.99.1 | Gateway Management |
| WLC management | 99 | 192.168.99.10 (ví dụ) | Chỉ ví dụ, sẽ chốt theo model WLC |
| Server (DHCP/RADIUS) | 99 hoặc 10 (sẽ chốt) | 192.168.99.20 (ví dụ) | Chỉ ví dụ, tùy VLAN đặt server |
| Switch (SVI quản trị) | 99 | 192.168.99.2 (ví dụ) | Tùy chọn, nếu cần quản trị qua IP |
| AP-1 / AP-2 / AP-3 | 99 (quản trị) | DHCP hoặc tĩnh trong 192.168.99.0/24 | Tùy cơ chế join WLC |

> Các địa chỉ `.10`, `.20`, `.2` ở trên chỉ là **ví dụ minh họa**, không phải cấu hình đã xác nhận. Không dùng các địa chỉ này làm bằng chứng cấu hình thành công.

## 3. Dải DHCP dự kiến (sẽ cấu hình sau)

- VLAN 10: dải động trong `192.168.10.0/24`, loại trừ gateway và địa chỉ tĩnh.
- VLAN 20: dải động trong `192.168.20.0/24`, loại trừ gateway.
- VLAN 99: nên dùng địa chỉ tĩnh cho thiết bị hạ tầng; chỉ bật DHCP nếu cần cho mục đích kiểm thử.

Vị trí DHCP server (trên router hay server riêng) sẽ được chốt trong [implementation-notes.md](implementation-notes.md) sau khi kiểm tra thiết bị hỗ trợ.

## 4. Những điểm cần chốt sau

- [ ] VLAN đặt server (99 hay 10) và địa chỉ IP cuối cùng của server.
- [ ] AP nhận IP quản trị bằng DHCP hay tĩnh.
- [ ] Router dùng router-on-a-stick hay interface riêng cho từng VLAN.
- [ ] Dải DHCP loại trừ cụ thể cho từng VLAN.

Mọi thay đổi so với bảng đề xuất phải được cập nhật đồng bộ tại [topology.md](topology.md), [requirements.md](requirements.md) và [../tests/test-cases.md](../tests/test-cases.md).
