# Kế hoạch địa chỉ IP và VLAN (tài liệu chuẩn)

> Đây là tài liệu **authoritative cho mọi địa chỉ** trong dự án: README, topology, security, test case và configs đều tham chiếu tới đây. Địa chỉ hạ tầng của **khung có dây P1 đã được cấu hình** (§2); DHCP cho client đã được kiểm chứng trong **TC-08** (2026-10-20), cấu hình đã lưu vào official `.pkt` 2026-10-21, với các giới hạn ghi tại §3. Kế hoạch cuối cùng phụ thuộc topology được chốt và model thiết bị trong Packet Tracer 9.0.1.0858.

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
| WLC management (`WLC-PT`) | 99 | 192.168.99.10 | **đã cấu hình** trên cổng GUI hiển thị là `Management` (API/mô phỏng đặt tên port `Vlan1`; `WLC-PT` không có cổng API `management` riêng); gateway `192.168.99.1`; đặt **bằng API, không cần raw JS**; TC-01 `Passed` (chạy lại trên `WLC-PT`) |
| Switch (SVI quản trị) | 99 | 192.168.99.2 | **đã cấu hình** (`interface Vlan99`) + `ip default-gateway 192.168.99.1` |
| Server-PT (DHCP; RADIUS kiểm tra sau) | 99 | 192.168.99.20 | **đã cấu hình** tĩnh; 2 pool DHCP chính (STAFF, GUEST) + `serverPool` giới hạn `.30` |
| LAP-PT-1 (AP đầu tiên) | 99 (quản trị) | 192.168.99.30 | **TC-02/TC-03: đặt tĩnh runtime** trên cổng `GigabitEthernet0` (gateway `192.168.99.1`, DNS `192.168.99.20`), TC-02 `Passed`; **TC-08 test copy: `LAP1` online và nhận `.30/24` qua DHCP** từ `serverPool` — chưa xác nhận tĩnh lưu bền |

> Toàn bộ địa chỉ hạ tầng trong §2 (kể cả WLC `.10` và LAP `.30`) **đã được cấu hình** và kiểm chứng: TC-01 `Passed` ngày 2026-10-18 — **chạy lại trên model `WLC-PT`**, ping `192.168.99.20 → 192.168.99.10` đạt `0% loss` (4/4) và `R1 → 192.168.99.10` `5/5`; TC-02 `Passed` cùng ngày, Simple PDU `LAP1 → R1` Successful và ping ngược `R1 → 192.168.99.30` đạt `5/5`. **Lưu ý:** kiểm chứng `WLC-PT` ban đầu thực hiện trên bản test copy (`..._TESTCOPY_2026-10-18.pkt`); **`.pkt` deliverable đã được thay sang `WLC-PT` (2026-10-19)** nên địa chỉ trên file gốc đã khớp. Bản cấu hình IOS đối chiếu tại `../configs/`; cấu hình WLC và LAP-PT không có file CLI nên ghi lại tại [../configs/wlc-ap/README.md](../configs/wlc-ap/README.md).
>
> **Cập nhật TC-08 (2026-10-20; cấu hình lưu vào official `.pkt` 2026-10-21):** `LAP1` online và nhận `192.168.99.30/24`; `STAFF1`/`GUEST1` nhận DHCP đúng subnet (test copy `.10.102`/`.20.102`; official `/renew` `.10.101`/`.20.100`). `SW1 Fa0/3` (nối LAP) là trunk native `99`, allowed `10,20,99`; MAC client VLAN 10/20 học trên `Fa0/3`, không có trên `Fa0/4` (WLC) ⇒ **local switching tại AP; central switching chưa kiểm chứng**. Địa chỉ `.30` trong lần chạy này đến từ DHCP, **không** phải bằng chứng tĩnh lưu bền. Lease DHCP phải `/renew` sau khi mở lại official.
>
> File này vẫn là nguồn chuẩn: muốn đổi địa chỉ thì sửa ở đây trước, rồi mới cập nhật `../configs/` và file `.pkt`.

## 3. Dải DHCP trên Server-PT và DHCP relay

- VLAN 10: pool `STAFF` — bắt đầu `192.168.10.100`, tối đa 100 người dùng (dải `.100`–`.199`). Gateway `.1` nằm ngoài dải. **Đã tạo và bật Service On.**
- VLAN 20: pool `GUEST` — bắt đầu `192.168.20.100`, tối đa 100 người dùng (dải `.100`–`.199`). Gateway `.1` nằm ngoài dải. **Đã tạo và bật Service On.**
- VLAN 99: ưu tiên địa chỉ tĩnh cho thiết bị hạ tầng; chỉ bật DHCP nếu cần cho mục đích kiểm thử (ví dụ cấp IP cho LAP).
- Ưu tiên xác minh DHCP trước; RADIUS là mục Conditional, kiểm tra sau khi luồng Core ổn định.

> **TC-08 (2026-10-20) đã kiểm chứng cấp phát thật:** test copy `STAFF1` nhận `192.168.10.102/24` (gw `.10.1`, DHCP `.99.20`) và `GUEST1` nhận `192.168.20.102/24` (gw `.20.1`, DHCP `.99.20`); official sau khi mở lại và `/renew` nhận `.10.101/24` và `.20.100/24`. **Giới hạn:** pool factory `serverPool` của PT **không xoá vĩnh viễn được** (PT tạo lại sau save/reopen); pool được giới hạn còn `192.168.99.30–.30`, gateway `192.168.99.1`, DNS `192.168.99.20`, tối đa 1 user; lease DHCP cần `/renew` sau khi mở lại. Cấu hình đã lưu vào official `.pkt` 2026-10-21.

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
- [x] LAP nhận IP quản trị bằng DHCP hay tĩnh — **TC-02/TC-03 dùng tĩnh runtime** `192.168.99.30`; **TC-08 dùng DHCP** và `LAP1` nhận `.30`, **chưa xác nhận tĩnh lưu bền**. Cổng `GigabitEthernet0` của AP vẫn **down** cho tới khi có cấu hình IP, nên DHCP không thể khởi động nếu chưa có cấu hình ban đầu (đã thử `setDhcpClientFlag(true)`, cổng không lên).
- [x] Dải DHCP loại trừ cụ thể cho từng VLAN — không cần loại trừ vì dải động bắt đầu ở `.100`; **`serverPool` đã được giới hạn** còn `.99.30–.30`, tối đa 1 user (không xoá vĩnh viễn được — PT tạo lại sau save/reopen).
- [x] Chế độ cổng từng đoạn — Switch↔WLC **trunk** (`Fa0/4`, native VLAN 99 + tagged 10,20) **cho management**, Switch↔Server-PT **access VLAN 99** (`Fa0/2`), Switch↔Router **trunk** (`Gi0/1`). Phía WLC-PT (`GigabitEthernet0`) không có tuỳ chọn trunk/VLAN trong GUI; cấu hình trunk ở phía switch là đủ (đã kiểm chứng TC-01). Switch↔LAP-PT: **access VLAN 99** trên `Fa0/3` ở TC-02; **từ TC-08 đặt `Fa0/3` trunk** native `99`, allowed `10,20,99` — MAC client VLAN 10/20 học trên `Fa0/3`, **không** trên `Fa0/4` (WLC) ⇒ **local switching tại AP; central switching chưa kiểm chứng**; cấu hình đã lưu vào official `.pkt` 2026-10-21.
