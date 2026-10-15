# Kế hoạch địa chỉ IP và VLAN (tài liệu chuẩn)

> Đây là tài liệu **authoritative cho mọi địa chỉ** trong dự án: README, topology, security, test case và configs đều tham chiếu tới đây. Toàn bộ giá trị là **đề xuất, chưa kiểm chứng**. Kế hoạch cuối cùng phụ thuộc topology được chốt và model thiết bị trong Packet Tracer 9.0.1.0858.

## 1. Bảng VLAN và subnet đề xuất

| VLAN | Name | Subnet | Gateway | Purpose |
| ---- | ---------- | --------------- | ------------ | ------------------ |
| 10 | STAFF | 192.168.10.0/24 | 192.168.10.1 | Staff clients |
| 20 | GUEST | 192.168.20.0/24 | 192.168.20.1 | Guest clients |
| 99 | MANAGEMENT | 192.168.99.0/24 | 192.168.99.1 | Network management |

- Mỗi VLAN dùng `/24` để đơn giản cho đồ án, đủ địa chỉ cho client mô phỏng.
- Gateway đặt tại router (`.1` của mỗi subnet).
- Tên VLAN và SSID dùng tiếng Anh, không dấu.
- Mọi thay đổi bảng này phải cập nhật đồng bộ tại [topology.md](topology.md), [requirements.md](requirements.md), [security-design.md](security-design.md) và [../tests/test-cases.md](../tests/test-cases.md).

## 2. Địa chỉ hạ tầng (ví dụ, chưa chốt — không phải cấu hình đã xác nhận)

| Thiết bị | VLAN | Địa chỉ ví dụ | Ghi chú |
| --- | --- | --- | --- |
| Router sub-interface / gateway VLAN 10 | 10 | 192.168.10.1 | Gateway Staff |
| Router sub-interface / gateway VLAN 20 | 20 | 192.168.20.1 | Gateway Guest |
| Router interface / gateway VLAN 99 | 99 | 192.168.99.1 | Gateway Management |
| WLC management | 99 | 192.168.99.10 (ví dụ) | Điền Management IPv4 đã thấy trong GUI; chốt sau kiểm tra reachability (TC-01) |
| Switch (SVI quản trị) | 99 | 192.168.99.2 (ví dụ) | Tùy chọn, nếu cần quản trị qua IP |
| Server-PT (DHCP; RADIUS kiểm tra sau) | 99 (dự kiến) | 192.168.99.20 (ví dụ) | VLAN đặt server chốt khi triển khai |
| LAP-PT-1 (AP đầu tiên) | 99 (quản trị) | DHCP hoặc tĩnh trong 192.168.99.0/24 | Tùy cơ chế join WLC (TC-02) |

> Các địa chỉ `.10`, `.20`, `.2` chỉ là **ví dụ minh họa**. Không dùng làm bằng chứng cấu hình thành công. Interface WLC (Gigabit Ethernet), WLAN VLAN mapping và AP uplink có thể yêu cầu thiết kế cổng khác nhau — chốt độc lập theo model.

## 3. Dải DHCP trên Server-PT (sẽ cấu hình sau)

- VLAN 10: dải động trong `192.168.10.0/24`, loại trừ gateway và địa chỉ tĩnh.
- VLAN 20: dải động trong `192.168.20.0/24`, loại trừ gateway.
- VLAN 99: ưu tiên địa chỉ tĩnh cho thiết bị hạ tầng; chỉ bật DHCP nếu cần cho mục đích kiểm thử (ví dụ cấp IP cho LAP).
- Ưu tiên xác minh DHCP trước; RADIUS là mục Conditional, kiểm tra sau khi luồng Core ổn định.

## 4. Những điểm cần chốt trước khi cấu hình (cả 3 thành viên thống nhất)

- [ ] Model router/switch cụ thể và thiết kế liên VLAN (subinterface 802.1Q hay phương án model hỗ trợ).
- [ ] VLAN đặt Server-PT và địa chỉ IP cuối cùng của server.
- [ ] LAP nhận IP quản trị bằng DHCP hay tĩnh.
- [ ] Dải DHCP loại trừ cụ thể cho từng VLAN.
- [ ] Chế độ cổng cho 3 đoạn độc lập: Switch↔WLC, Switch↔LAP-PT, Switch↔Server-PT.
