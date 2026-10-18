# Kế hoạch địa chỉ IP và VLAN (tài liệu chuẩn)

> Đây là tài liệu **authoritative cho mọi địa chỉ** trong dự án: README, topology, security, test case và configs đều tham chiếu tới đây. Địa chỉ hạ tầng của **khung có dây P1 đã được cấu hình** (§2); các giá trị còn lại vẫn là **đề xuất, chưa kiểm chứng**. Kế hoạch cuối cùng phụ thuộc topology được chốt và model thiết bị trong Packet Tracer 9.0.1.0858.

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

## 2. Địa chỉ hạ tầng (đã cấu hình trong khung có dây P1)

| Thiết bị | VLAN | Địa chỉ | Ghi chú |
| --- | --- | --- | --- |
| Router sub-interface / gateway VLAN 10 | 10 | 192.168.10.1 | Gateway Staff — **đã cấu hình** (`Gi0/0.10`) |
| Router sub-interface / gateway VLAN 20 | 20 | 192.168.20.1 | Gateway Guest — **đã cấu hình** (`Gi0/0.20`) |
| Router interface / gateway VLAN 99 | 99 | 192.168.99.1 | Gateway Management — **đã cấu hình** trên `Gi0/0` (VLAN 99 native, không tạo subinterface) |
| WLC management | 99 | 192.168.99.10 (đề xuất) | **Chưa cấu hình** — điền Management IPv4 qua GUI, chốt sau TC-01 |
| Switch (SVI quản trị) | 99 | 192.168.99.2 | **đã cấu hình** (`interface Vlan99`) + `ip default-gateway 192.168.99.1` |
| Server-PT (DHCP; RADIUS kiểm tra sau) | 99 | 192.168.99.20 | **đã cấu hình** tĩnh; 2 pool DHCP đã tạo |
| LAP-PT-1 (AP đầu tiên) | 99 (quản trị) | DHCP hoặc tĩnh trong 192.168.99.0/24 | **Chưa triển khai** — tùy cơ chế join WLC (TC-02) |

> Địa chỉ trong khung có dây P1 (`192.168.10.1`, `192.168.20.1`, `192.168.99.1`, `.2`, `.20`) **đã được cấu hình** và đối chiếu tại `../configs/`. Địa chỉ WLC `.10` vẫn là **đề xuất**, chờ TC-01.
>
> File này vẫn là nguồn chuẩn: muốn đổi địa chỉ thì sửa ở đây trước, rồi mới cập nhật `../configs/` và file `.pkt`.

## 3. Dải DHCP trên Server-PT và DHCP relay

- VLAN 10: pool `STAFF` — bắt đầu `192.168.10.100`, tối đa 100 người dùng (dải `.100`–`.199`). Gateway `.1` nằm ngoài dải. **Đã tạo và bật Service On.**
- VLAN 20: pool `GUEST` — bắt đầu `192.168.20.100`, tối đa 100 người dùng (dải `.100`–`.199`). Gateway `.1` nằm ngoài dải. **Đã tạo và bật Service On.**
- VLAN 99: ưu tiên địa chỉ tĩnh cho thiết bị hạ tầng; chỉ bật DHCP nếu cần cho mục đích kiểm thử (ví dụ cấp IP cho LAP).
- Ưu tiên xác minh DHCP trước; RADIUS là mục Conditional, kiểm tra sau khi luồng Core ổn định.

> Hai pool đã tạo nhưng **chưa có client** nên việc cấp phát thật chưa được kiểm chứng (TC-08). Pool `serverPool` mặc định của PT vẫn còn và trỏ về `192.168.99.0/24` — cần rà soát để tránh cấp nhầm khi thêm client.

### 3.1 DHCP relay — BẮT BUỘC, không phải tùy chọn

Server-PT nằm ở VLAN 99 (§2) nhưng phải cấp IP cho client ở VLAN 10 và VLAN 20. Đây là ba subnet khác nhau, nên router **bắt buộc** phải chuyển tiếp DHCP từ hai subinterface client tới server:

```text
interface <subinterface VLAN 10>
 ip helper-address 192.168.99.20

interface <subinterface VLAN 20>
 ip helper-address 192.168.99.20
```

- Tên interface và subinterface cụ thể chốt theo model router thực tế (xem [topology.md](topology.md)).
- `192.168.99.20` là địa chỉ Server-PT theo §2. Nếu địa chỉ server thay đổi, phải cập nhật cả hai `ip helper-address`.
- VLAN 99 không cần relay vì client và server cùng subnet.
- **Thiếu relay ⇒ client Staff/Guest không nhận được IP ⇒ TC-08 thất bại, kéo theo TC-09, TC-10, TC-11, TC-12 và TC-14 không kiểm thử được.**

## 4. Những điểm cần chốt trước khi cấu hình (cả 3 thành viên thống nhất)

- [x] Model router/switch cụ thể và thiết kế liên VLAN — `2911` + router-on-a-stick trên `Gi0/0`.
- [x] VLAN đặt Server-PT và địa chỉ IP cuối cùng của server — VLAN 99, `192.168.99.20`.
- [x] Địa chỉ đưa vào `ip helper-address` (§3.1) khớp địa chỉ Server-PT cuối cùng — `192.168.99.20`.
- [ ] LAP nhận IP quản trị bằng DHCP hay tĩnh.
- [x] Dải DHCP loại trừ cụ thể cho từng VLAN — chưa cần loại trừ vì dải động bắt đầu ở `.100`; **còn phải rà soát pool `serverPool` mặc định của PT**.
- [x] Chế độ cổng từng đoạn — Switch↔WLC **trunk** (`Fa0/1`, native VLAN 99 + tagged 10,20), Switch↔Server-PT **access VLAN 99** (`Fa0/2`), Switch↔Router **trunk** (`Gi0/1`). **Còn lại: Switch↔LAP-PT.**
