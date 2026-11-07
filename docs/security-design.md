# Thiết kế bảo mật — trạng thái final (2026-11-07)

> Tài liệu trình bày thiết kế bảo mật **đã triển khai trên bản deliverable `packet-tracer/final.pkt`** và kết quả kiểm chứng tương ứng. Các khẳng định dưới đây dựa trên ping thật + đọc cấu hình thật (audit 2026-11-05 → 2026-11-07), chi tiết tại [final-implementation.md](final-implementation.md) §3–§6. Ý định thiết kế giai đoạn v1 (PSK-first trên `WLC-PT`) đã được kế thừa thay thế; lịch sử xem [v1-history.md](v1-history.md).

## 1. Nguyên tắc phân tách (đã triển khai)

Mỗi nhóm đối tượng dùng SSID và VLAN riêng trên WLC1 (`WLC-3504`):

| SSID | WLAN | VLAN | Subnet | Xác thực | Client |
| --- | --- | --- | --- | --- | --- |
| `STAFF-WIFI` | 1 | 10 | `192.168.10.0/24` | **WPA2-Enterprise (802.1X/RADIUS)** | S-LAP1/2/3 (Laptop-PT) |
| `GUEST-WIFI` | 2 | 20 | `192.168.20.0/24` | **WPA2-PSK** | G-LAP1/2 (Laptop-PT), G-PHONE (Smartphone-PT) |
| — (có dây) | — | 99 | `192.168.99.0/24` | — | Hạ tầng + `ADMIN-PC` (`.99.50`, tĩnh) |

Mục tiêu đạt: Guest không truy cập được tài nguyên Staff và hệ thống quản trị (**đã kiểm chứng**, §3); Staff đi được Internet qua PAT (**đã kiểm chứng**, §5).

## 2. Xác thực vô tuyến

**Guest — WPA2-PSK (đã triển khai):** cả hai WLAN của AP LAP-GUEST sử dụng PSK; không dùng Open. (TC-14 "PSK sai bị từ chối" **chưa test**.)

**Staff — WPA2-Enterprise/802.1X + RADIUS (đã cấu hình, chưa có minh chứng exchange end-to-end):**

- Hai đầu khớp chuẩn: `RADIUS-SRV` `192.168.99.22`, service **AAA On**, port **1645**; WLC1 `STAFF-WIFI` = WPA2 + **Auth 802.1X**, AAA server `192.168.99.22:1645`, client network entry `WLC1` (`192.168.99.10`), users `staff1/2/3` trên server.
- Trạng thái trung thực: **Configured — chưa chứng minh exchange**: PT không ghi được giao dịch EAP giữa client↔AP↔WLC↔server; Staff client associate + nhận đúng lease (`.103/.104/.106`) nhưng không thể tách bằng chứng "xác thực qua RADIUS" khỏi "associate vô tuyến".
- Shared secret: **không commit giá trị thật** vào bất kỳ file nào (`<RADIUS-SECRET-REDACTED>`); giá trị thực chỉ tồn tại trong `.pkt`.
- PSK/thiết lập vô tuyến tương tự: không commit, che khi chụp minh chứng.

## 3. Cô lập Guest — ACL `GUEST-ACL` (đã kiểm chứng chức năng)

ACL named đặt trên `Vlan20` inbound của **CORE-SW** (không phải router như ý định v1 — final dùng SVI L3):

```text
ip access-list extended GUEST-ACL
 permit udp any any eq bootps           ! không chặn DHCP Discover của Guest
 deny   ip any 192.168.10.0 0.0.0.255   ! Guest → Staff: chặn
 deny   ip any 192.168.99.0 0.0.0.255   ! Guest → dải quản trị: chặn
 permit ip any any                      ! phần còn lại (Internet qua PAT): cho phép
```

Kết quả kiểm chứng (G-LAP1, `192.168.20.142`):

| Ping | Kết quả |
| --- | --- |
| `192.168.10.1` (gateway Staff) | **0/4 reply — bị chặn** |
| `192.168.99.22` (RADIUS-SRV, dải quản trị) | **0/4 reply — bị chặn** |
| `8.8.8.8` (Internet) | 4/4 ✓ (đạt sau retry sim) |
| `ipconfig /renew` (DHCP tiếp diễn) | vẫn nhận lease ⇒ `bootps` permit hiệu lực |

- Phát biểu đúng: "Guest **không tới được Staff và dải quản trị**"; Guest vẫn tới được Internet nhờ rule permit cuối (mọi đích ngoài hai dải bị chặn).
- **Lỗi hiển thị PT:** `show ip interface Vlan20` in "access-group … not set" cho ACL named — kiểm chứng binding bằng Running config/`show access-lists` hoặc cấu hình thật trong `../configs/switch/core-sw-running-config.txt`.

## 4. Bảo vệ mạng quản trị (VLAN 99)

- Thiết bị hạ tầng + ADMIN-PC tĩnh trong VLAN 99, tách khỏi client vô tuyến (WLC `.10`, Web `.21`, DHCP `.20`, RADIUS `.22`, ADMIN-PC `.50`, gateway SVI `.1`).
- Truy cập quản trị WLC (web GUI `https://192.168.99.10`) chỉ từ trong nội bộ — đối chứng hướng ngược: `ADMIN-PC → RADIUS-SRV` 4/4 (mạng quản trị hoạt động bình thường, chỉ Guest bị chặn vào).
- DHCP dành cho client vô tuyến; VLAN 99 dùng địa chỉ tĩnh (không cấp DHCP cho hạ tầng).

## 5. Internet / PAT qua biên (đã kiểm chứng)

- R1/R2 (2901): `ip nat inside` (Gi0/1 về CORE-SW) / `ip nat outside` (Gi0/0 ra ISP), **PAT overload** `access-list 1` permit cả 3 subnet nội bộ.
- Kết quả: Staff S-LAP1/2/3 → `8.8.8.8` **4/4**; Guest G-LAP1 → `8.8.8.8` **4/4** (đạt sau một lần retry do bất ổn sim wireless).
- Failover R1→R2 (default AD 254 trên CORE-SW): đã cấu hình, **chưa test power-off** — ghi ở limitations.

## 6. So sánh hai phương thức (trạng thái final)

| Tiêu chí | WPA2-PSK (Guest) | WPA2-Enterprise + RADIUS (Staff) |
| --- | --- | --- |
| Trạng thái triển khai | Đang phát sóng, client associate + lease ✓ | Đang phát sóng, client associate + lease ✓ |
| Bằng chứng xác thực thời gian thực | Associate thật quan sát được | **Chưa có minh chứng EAP exchange** (PT limitation) |
| Rủi ro | Lộ key chung | Phụ thuộc RADIUS (`192.168.99.22:1645`) khả dụng |
| Test còn thiếu | TC-14 (PSK sai bị từ chối) | TC-13 (minh chứng exchange) |

## 7. Chủ đề Research-only (không tính vào tiêu chí hoàn thành)

WPA3/SAE (bản 9.0.1.0858 không có `WirelessAuthenType` WPA3), EAP cụ thể (EAP-TLS/PEAP), roaming, nhiễu RF, throughput, 802.11ac/ax — chỉ làm nền báo cáo, không ghi là đã triển khai.
