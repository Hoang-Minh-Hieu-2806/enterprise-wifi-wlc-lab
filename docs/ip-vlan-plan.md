# Kế hoạch địa chỉ IP và VLAN (tài liệu chuẩn — final 2026-11-07)

> Tài liệu **authoritative cho mọi địa chỉ** của bản deliverable `packet-tracer/final.pkt` (topology enterprise 18 thiết bị). Toàn bộ giá trị dưới đây được đọc/đối chiếu từ mô phỏng thật trong audit 2026-11-05 → 2026-11-07; chi tiết đầy đủ tại [final-implementation.md](final-implementation.md). Lịch sử: khung v1 (router-on-a-stick trên `2911` + `SW1` + `SRV1`, cùng các subnet 10/20/99 nhưng khác điểm gateway) xem [v1-history.md](v1-history.md).

## 1. Bảng VLAN và subnet

| VLAN | Name | Subnet | Gateway (SVI trên CORE-SW) | Purpose |
| ---- | ---------- | --------------- | ------------ | ------------------ |
| 10 | STAFF | 192.168.10.0/24 | `192.168.10.1` (`Vlan10`, MAC `00d0.5877.bd01`) | Staff clients (wireless) |
| 20 | GUEST | 192.168.20.0/24 | `192.168.20.1` (`Vlan20`, MAC `…bd02`) | Guest clients (wireless) — có ACL `GUEST-ACL` in |
| 99 | MANAGEMENT | 192.168.99.0/24 | `192.168.99.1` (`Vlan99`, MAC `…bd03`) | Quản trị: WLC, servers, ADMIN-PC |

- Mỗi VLAN `/24`; gateway là **SVI trên CORE-SW (3560-24PS, `ip routing`)** — không còn router-on-a-stick như v1.
- Tên VLAN và SSID dùng tiếng Anh, không dấu.
- Mọi thay đổi bảng này phải cập nhật đồng bộ [final-implementation.md](final-implementation.md), [security-design.md](security-design.md), [../configs/README.md](../configs/README.md) và [../tests/test-cases.md](../tests/test-cases.md).

## 2. Liên kết routed (p2p `/30`)

| Link | Subnet | Bên A | Bên B |
| --- | --- | --- | --- |
| ISP ↔ R1 | `203.0.113.0/30` | ISP `Gi0/0` → `203.0.113.1` | R1 `Gi0/0` → `203.0.113.2` |
| ISP ↔ R2 | `203.0.113.4/30` | ISP `Gi0/0` → `203.0.113.5` | R2 `Gi0/0` → `203.0.113.6` |
| R1 ↔ CORE-SW | `10.0.0.0/30` | R1 `Gi0/1` → `10.0.0.1` | CORE-SW `Gi0/1` (no switchport) → `10.0.0.2` |
| R2 ↔ CORE-SW | `10.0.1.0/30` | R2 `Gi0/1` → `10.0.1.1` | CORE-SW `Gi0/2` (no switchport) → `10.0.1.2` |

- `ISP` Loopback0 `8.8.8.8/32` — đích "Internet" đo được.
- Default route: R1/R2 → ISP; CORE-SW → R1 (AD 1) + backup → R2 (AD 254). R1/R2 có static routes về `192.168.10/20/99` → next-hop là CORE-SW (`10.0.0.2` / `10.0.1.2`).

## 3. Địa chỉ thiết bị (VLAN 99 — tĩnh)

| Thiết bị | Địa chỉ | Ghi chú |
| --- | --- | --- |
| CORE-SW (SVI) | `192.168.99.1` | Gateway quản trị |
| WLC1 (`WLC-3504`) management | `192.168.99.10/24`, gw `.1` | Web GUI quản trị; đồng thời là **Primary DHCP Server** cho wireless (xem §4) |
| DHCP-SRV (Server-PT) | `192.168.99.20` | Service DHCP bật; **không phải nguồn lease chính cho wireless** (xem §4) |
| Web-SRV (Server-PT) | `192.168.99.21` | HTTP |
| RADIUS-SRV (Server-PT) | `192.168.99.22` | AAA, port **1645** |
| **ADMIN-PC (PC-PT)** | **`192.168.99.50/24`** | **Tĩnh** (đặt từ phiên 2026-11-01, truy cập web GUI `https://192.168.99.10`) |

## 4. DHCP thực tế — ai cấp IP cho ai

Đây là điểm dễ nhầm nhất của bản final; phân vai rõ:

| Nguồn DHCP | Địa chỉ | Đối tượng phục vụ | Trạng thái |
| --- | --- | --- | --- |
| **WLC1 (WLC-3504) — nguồn thực tế cho wireless** | Primary DHCP Server = `192.168.99.10` (management của chính nó) trên 2 dynamic interface **staff** (`192.168.10.2`) và **guest** (`192.168.20.2`) | Client vô tuyến `STAFF-WIFI` / `GUEST-WIFI` (FlexConnect local switching) | **Đang hiệu lực** — lease quan sát nằm trong dải pool nội bộ WLC: STAFF-LOCAL bắt đầu `192.168.10.101` (leases `.103/.104/.106`), GUEST-LOCAL bắt đầu `192.168.20.141` (leases `.141/.142/.144`) |
| DHCP-SRV (`192.168.99.20`) | Service DHCP bật, có pools STAFF/GUEST | Không quan sát thấy client nào nhận lease từ nguồn này trong audit final | **Dự phòng** — giữ theo kế hoạch ban đầu |
| `ip helper-address 192.168.99.20` trên `Vlan10`/`Vlan20` (CORE-SW) | — | Đường relay cho client DHCP trong VLAN 10/20 | Cấu hình có thật; thực tế các request của wireless được WLC phục vụ trước (xem lưu ý) |

Lưu ý vận hành:

- **Fact quét thấy:** client Staff nhận `.10.103/.104/.106`, Guest nhận `.20.141/.142/.144` — khớp dải pool nội bộ WLC (start `.101` / `.141`), **không** khớp luồng cấp từ `192.168.99.20` (dải bắt đầu `.100` và sẽ cấp tuần tự .100/.101/.102 trước).
- PT đặc thù: khi **Primary DHCP Server của dynamic interface trỏ lại chính management**, WLC xử lý DHCP cho client bằng pool nội bộ của nó — đây là cấu hình đã chốt trên final (đặt 2026-11-05).
- `serverPool` factory của Server-PT **không xoá vĩnh viễn được** (PT tạo lại sau save/reopen), giới hạn `.99.30–.30`, max 1 user.
- **Lease không giữ sau khi mở lại file `.pkt`** — cần `ipconfig /renew` trên từng client.
- Thiết bị VLAN 99 (WLC, servers, ADMIN-PC) dùng **địa chỉ tĩnh**, không phụ thuộc DHCP.

## 5. Dải cấp phát chốt (trạng thái final)

| Pool | Địa chỉ WLC | Dải bắt đầu | Gateway cấp cho client |
| --- | --- | --- | --- |
| STAFF-LOCAL (trên WLC1) | dynamic interface `staff` `192.168.10.2` | `192.168.10.101` | `192.168.10.1` |
| GUEST-LOCAL (trên WLC1) | dynamic interface `guest` `192.168.20.2` | `192.168.20.141` | `192.168.20.1` |

> Muốn đổi bất kỳ địa chỉ nào: sửa tại đây trước, rồi cập nhật `final.pkt` + `../configs/` + [final-implementation.md](final-implementation.md). Lịch sử thay đổi địa chỉ giai đoạn v1: [v1-history.md](v1-history.md).
