# Thiết kế, Mô phỏng và Đánh giá Bảo mật Mạng Wi-Fi Doanh nghiệp sử dụng Cisco Packet Tracer

> **Tên repository (tiếng Anh):** `enterprise-wifi-wlc-lab`
> **Tiêu đề dự án (tiếng Anh):** Design, Simulation, and Security Evaluation of an Enterprise Wi-Fi Network Using Cisco Packet Tracer

Đồ án môn Quản trị mạng: mô phỏng mạng Wi-Fi doanh nghiệp trên Cisco Packet Tracer, với quản lý tập trung, nhiều SSID, phân đoạn VLAN, xác thực vô tuyến 802.1X/RADIUS và kiểm soát truy cập giữa các phân đoạn.

> ## ✅ TRẠNG THÁI FINAL (2026-11-07)
>
> Deliverable chính thức: **`packet-tracer/final.pkt`** (18 thiết bị / 21 link; SHA-256 `5AA9ADCA…B5CEF`, 100.297 bytes). Kiến trúc: **ISP (Router-PT, Lo0 `8.8.8.8`) → R1/R2 (2901, PAT overload, 2 path /30) → CORE-SW (3560-24PS L3, SVI VLAN 10/20/99) → ACCESS-SW (2960, trunk) → WLC1 (WLC-3504, mgmt `192.168.99.10`) + LAP-STAFF / LAP-GUEST**, kèm **RADIUS-SRV (`.99.22`, AAA 1645)**, **DHCP-SRV (`.99.20`)**, **Web-SRV (`.99.21`)**, **ADMIN-PC (`.99.50`, tĩnh)**, **6 wireless client** (3 Staff qua `STAFF-WIFI` WPA2-Enterprise/802.1X; 3 Guest qua `GUEST-WIFI` WPA2-PSK).
>
> **Đã kiểm chứng bằng ping thật (bridge, 2026-11-05 → 2026-11-07):**
>
> - Gateway: cả 6 wireless client + ADMIN-PC `ping` **4/4** (0% loss).
> - Internet (PAT): Staff → `8.8.8.8` **4/4**; Guest G-LAP1 → `8.8.8.8` **4/4** (đạt sau một lần retry — bất ổn sim wireless).
> - Cách ly Guest: G-LAP1 → `192.168.10.1` **0/4** (bị chặn); → `192.168.99.22` **0/4** (bị chặn); DHCP vẫn đạt qua permit `bootps` (lease `.20.142`).
> - Staff leases `.103/.104/.106`, Guest leases `.141/.142/.144` — đúng design.
>
> **Chưa kiểm chứng end-to-end** (ghi trong limitations): RADIUS 802.1X exchange client↔server (đã cấu hình chuẩn hai đầu nhưng PT không log được EAP), failover R1→R2 (đã cấu hình AD 254, chưa test power-off), sai PSK bị từ chối (TC-14).
>
> Chi tiết nguyên trạng: **[docs/final-implementation.md](docs/final-implementation.md)**. Lịch sử giai đoạn v1 (khung P1 9 thiết bị, `WLC-PT`, TC-01→TC-08): **[docs/v1-history.md](docs/v1-history.md)** — được tách khỏi nội dung hướng dẫn này.

## 1. Mục tiêu (đã hoàn thành trên bản final)

1. Xây dựng mạng WLAN doanh nghiệp mô phỏng: **WLC (`WLC-3504`) quản lý tập trung 2 AP (`LAP-PT`) và 2 SSID**; client associate thật và nhận IP đúng subnet.
2. Phân đoạn VLAN 10 (STAFF) / 20 (GUEST) / 99 (MANAGEMENT) với gateway SVI trên **CORE-SW (3560-24PS, `ip routing`)**.
3. Hai SSID phát sóng đồng thời: **`STAFF-WIFI` (WPA2-Enterprise/802.1X + RADIUS)** cho nhân viên, **`GUEST-WIFI` (WPA2-PSK)** cho khách.
4. Biên Internet bằng **2 edge router (2901) song song, PAT overload**, static routes; default route của CORE-SW có **backup AD 254 qua R2** (theo design).
5. Kiểm soát truy cập bằng **ACL `GUEST-ACL` trên SVI VLAN 20**: chặn Guest → Staff và Guest → dải quản trị, permit DHCP.
6. Kiểm thử chức năng thật (ping qua bridge), thu minh chứng, viết tài liệu kỹ thuật.

> Đây là **dự án mô phỏng và tài liệu mạng** — không có web app/API/CI-CD.

## 2. Công nghệ và môi trường

| Hạng mục | Nội dung |
| --- | --- |
| Môi trường | Cisco Packet Tracer **9.0.1.0858** |
| Định tuyến | `ISP` + `R1`/`R2` (Router 2901): static routes + PAT overload, 2 đường p2p `/30` |
| Chuyển mạch | `CORE-SW` — Switch **L3 3560-24PS** (`ip routing`, SVI 3 VLAN, DHCP relay, ACL, default ×2 AD); `ACCESS-SW` — Switch L2 2960-24TT (trunk) |
| Wireless | `WLC1` (`WLC-3504`, web GUI qua `ADMIN-PC`) + `LAP-STAFF` / `LAP-GUEST` (`LAP-PT`) |
| Server | Server-PT ×3: `DHCP-SRV` (`.99.20`), `RADIUS-SRV` (`.99.22`, AAA 1645), `Web-SRV` (`.99.21`); DNS khách `8.8.8.8` sử dụng được qua PAT |
| Client | `ADMIN-PC` (PC-PT, `.99.50` tĩnh), 3 Laptop-PT Staff, 2 Laptop-PT + 1 Smartphone-PT Guest |
| Ngôn ngữ tài liệu | Tiếng Việt (tài liệu, test case); định danh kỹ thuật tiếng Anh |

Ghi chú model/limitation chi tiết: [docs/implementation-notes.md](docs/implementation-notes.md); quyết định chọn thiết bị: [docs/device-selection-decision.md](docs/device-selection-decision.md) (decision record, có banner SUPERSEDED cho final).

## 3. Thành phần mạng (final)

| Nhóm | Thiết bị | Model | Vai trò |
| --- | --- | --- | --- |
| WAN | ISP | Router-PT | 2 path `/30` tới R1/R2; Lo0 `8.8.8.8/32` |
| WAN | R1, R2 | 2901 | Edge + PAT overload + static route vế về LAN |
| L2/L3 | CORE-SW | 3560-24PS | L3 core: SVI 10/20/99, DHCP relay, ACL Guest, default AD 1 + backup AD 254 |
| L2 | ACCESS-SW | 2960-24TT | Access/trunk cho WLC + AP |
| Wireless | WLC1 | WLC-3504 | Controller; mgmt `.99.10`; dynamic interfaces staff/guest; WLANs |
| Wireless | LAP-STAFF, LAP-GUEST | LAP-PT | AP join WLC1 |
| Server | DHCP-SRV / RADIUS-SRV / Web-SRV | Server-PT | DHCP `.20` / AAA 1645 `.22` / HTTP `.21` |
| Mgmt | ADMIN-PC | PC-PT | Quản trị VLAN 99 (`.99.50`, tĩnh; web GUI WLC từ đây) |
| Client | S-LAP1/2/3 | Laptop-PT | Staff — `STAFF-WIFI` (802.1X) |
| Client | G-LAP1/2, G-PHONE | Laptop-PT / Smartphone-PT | Guest — `GUEST-WIFI` (PSK) |

Canvas minh chứng: [evidence/Final.jpg](evidence/Final.jpg), [evidence/Final 2.png](evidence/Final%202.png). Chi tiết link/port: [docs/final-implementation.md](docs/final-implementation.md) §1–§2 ([docs/topology.md](docs/topology.md) đã đánh dấu LỊCH SỬ v1).

## 4. Phân tách SSID → VLAN (đã triển khai)

| SSID | VLAN | Subnet | Gateway (SVI) | Xác thực | DHCP |
| --- | --- | --- | --- | --- | --- |
| `STAFF-WIFI` | 10 | `192.168.10.0/24` | `.1` | WPA2-Enterprise + 802.1X + RADIUS | WLC pool STAFF-LOCAL (từ `.101`) |
| `GUEST-WIFI` | 20 | `192.168.20.0/24` | `.1` + ACL `GUEST-ACL` in | WPA2-PSK | WLC pool GUEST-LOCAL (từ `.141`) |
| — (có dây quản trị) | 99 | `192.168.99.0/24` | `.1` | — | Địa chỉ tĩnh cho hạ tầng |

> **DHCP thực tế:** client vô tuyến được WLC1 cấp từ các pool nội bộ (Primary DHCP Server = management `.99.10` của chính WLC — lease quan sát khớp dải `.101+`/`.141+`). `DHCP-SRV` `.99.20` bật Service DHCP nhưng là **dự phòng**, không quan sát được hành động cấp phát cho client trong audit final; `ip helper-address` trên SVI giữ lại cho các DHCP request không đi qua WLC. Chi tiết: [docs/ip-vlan-plan.md](docs/ip-vlan-plan.md) §4.

## 5. Xác thực và kiểm soát truy cập (đã triển khai)

- **Staff (`STAFF-WIFI`):** WPA2 + Auth 802.1X, AAA `192.168.99.22:1645`; RADIUS-SRV AAA On port 1645, client entry `WLC1`, users `staff1/2/3`. Trạng thái trung thực: **đã cấu hình quy chuẩn — chưa có minh chứng EAP exchange** (PT không cho quan sát) — xem [docs/final-implementation.md](docs/final-implementation.md) §5.
- **Guest (`GUEST-WIFI`):** WPA2-PSK.
- **ACL** trên `Vlan20` (CORE-SW): `permit udp any any eq bootps` → `deny 192.168.10.0` → `deny 192.168.99.0` → `permit any`; **đã kiểm chứng** 0/4 tới Staff và RADIUS-SRV, Internet vẫn thông. Xem [docs/security-design.md](docs/security-design.md).
- **Internet (PAT):** Staff/Guest → `8.8.8.8` đạt; cấu hình trên cả R1 và R2.
- **Hạ tầng quản trị:** VLAN 99, tất cả địa chỉ tĩnh; Web GUI WLC truy cập từ ADMIN-PC (bên trong VLAN 99).
- **Nghiên cứu (Research-only):** WPA3/EAP-TLS/PEAP, roaming, nhiễu RF, throughput — nền báo cáo, không tính vào tiêu chí.

## 6. Cấu trúc repository

```text
enterprise-wifi-wlc-lab/
├── README.md
├── .gitignore
├── LICENSE
├── docs/
│   ├── project-overview.md
│   ├── requirements.md
│   ├── final-implementation.md      ⭐ tài liệu chính thức của bản final
│   ├── topology.md
│   ├── ip-vlan-plan.md              ⭐ chuẩn địa chỉ (final)
│   ├── security-design.md           ⭐ thiết bị bảo mật (final)
│   ├── testing-plan.md
│   ├── implementation-notes.md
│   ├── device-selection-decision.md (decision record; banner SUPERSEDED)
│   ├── v1-history.md                📦 toàn bộ lịch sử giai đoạn v1
│   └── team-contributions.md
├── packet-tracer/
│   ├── README.md
│   └── final.pkt                    ⭐ deliverable chính thức (18 thiết bị)
├── configs/
│   ├── README.md
│   ├── router/     (R1 + R2, 2901 — xuất startup 2026-11-07)
│   ├── switch/     (CORE-SW 3560-24PS — final; sw1 = history v1)
│   └── wlc-ap/     (WLAN qua web GUI — không có CLI)
├── tests/
│   ├── README.md
│   └── test-cases.md                (kết quả final TC-09→TC-14 + lịch sử TC-01→TC-08)
├── evidence/                        (Final.jpg / Final 2.png / Final but error.png + README)
└── report/                          (README + midterm-report.md — báo cáo cuối kỳ cập nhật theo docs/final-implementation.md)
```

Quy ước đặt tên `.pkt` và quy định khi bổ sung file: [packet-tracer/README.md](packet-tracer/README.md).

## 7. Cách mở `final.pkt`

1. Cài Cisco Packet Tracer **9.0.1.0858** (bản chốt; ghi chú tại [docs/implementation-notes.md](docs/implementation-notes.md)).
2. Clone repository và mở `packet-tracer/final.pkt` (double-click hoặc `File → Open`).
3. Trên từng client: `Desktop → Command Prompt` → `ipconfig /renew` (lease DHCP **không giữ** sau reopen — đặc tính PT).
4. Nếu association wireless rớt sau khi mở file: power-cycle AP rồi client, đặt lại DHCP, đợi 10–20 s, `/renew`. Xem mục "Lưu ý khi mở" tại [packet-tracer/README.md](packet-tracer/README.md).
5. Demo nhanh: `ping <gateway VLAN>` (4/4) → `ping 8.8.8.8` (4/4 Staff/Guest); từ Guest → `ping 192.168.10.1` (bị chặn); từ `ADMIN-PC` mở trình duyệt `https://192.168.99.10` (web GUI WLC).
6. Không chỉnh sửa topology khi chưa thống nhất với nhóm; mọi thay đổi cấu hình cần lưu sang `configs/` (che secret) và ghi vào docs.

## 8. Trạng thái dự án

| Nhóm trạng thái | Ý nghĩa | Hiện tại |
| --- | --- | --- |
| **Passed** (có kết quả thật) | Gateway 6/6 client 4/4; Staff/Guest → Internet 4/4; Guest bị chặn Staff + Mgmt 0/4; DHCP lease đúng subnet; PAT và static routes khớp | ✅ |
| **Configured / chưa chứng minh exchange** | RADIUS 802.1X (cấu hình hai đầu khớp chuẩn port 1645 + users; EAP không quan sát được trong PT) | 🟡 |
| **Not Tested** | Sai PSK (TC-14); failover R1→R2 power-off; screenshot `show vlan brief` riêng | ⬜ |
| **Lịch sử v1** | TC-01→TC-08 `Passed` trên `v1.pkt` + `WLC-PT` (file + minh chứng đã xoá khỏi working tree, trong Git history) | 📦 [docs/v1-history.md](docs/v1-history.md) |

Chi tiết từng test case: [tests/test-cases.md](tests/test-cases.md); kết quả + limitations đầy đủ: [docs/final-implementation.md](docs/final-implementation.md) §7–§8.

## 9. Hạn chế đã biết (final)

1. **RADIUS 802.1X chưa có minh chứng exchange end-to-end** — cấu hình hai đầu chuẩn, PT không log được EAP.
2. **Failover R1→R2 chưa test power-off** — backup AD 254 đã cấu hình đúng trên CORE-SW.
3. **Bất ổn mô phỏng wireless PT 9.0.1.0858:** association đôi lúc rớt khi thao tác nhiều (`evidence/Final but error.png`); recovery power-cycle + `/renew`.
4. **`show ip interface` của PT không hiển thị ACL named** — xem Running Config / `show access-lists` / file config xuất.
5. **DHCP lease không giữ sau reopen** `final.pkt` — `/renew` từng client; `serverPool` factory không xoá vĩnh viễn.
6. **Central switching của WLC-3504 không kiểm chứng bằng packet-trace** — wireless được xác nhận ở mức chức năng qua associate + lease.
7. PSK/secret lưu dạng rõ trong XML `.pkt` — không chia sẻ file ngoài nhóm.

## 10. Thành viên nhóm

| Thành viên | Vai trò | Tỉ trọng mục tiêu |
| --- | --- | --- |
| Hoàng Minh Hiếu (nhóm trưởng) | Điều phối, quản lý Git/tiến độ, rà soát thiết kế, kỹ thuật linh hoạt, tham gia kiểm thử, tổng hợp báo cáo | ~33% |
| Đặng Văn Hoàng | Topology, WLC/LAP, WLAN, xác thực, evidence vô tuyến | ~33% |
| Huỳnh Như Hoàng | Định tuyến, VLAN, DHCP, ACL, kiểm thử phân tách, tổng hợp kết quả | ~34% |

Phân công chi tiết và ghi nhận góp phần giai đoạn final: [docs/team-contributions.md](docs/team-contributions.md).

## 11. Tài liệu liên quan

**Final (đi kèm deliverable):**

- [docs/final-implementation.md](docs/final-implementation.md) — ⭐ tài liệu chính thức của bản final.
- [docs/ip-vlan-plan.md](docs/ip-vlan-plan.md) — chuẩn địa chỉ/IP-VLAN (final).
- [docs/security-design.md](docs/security-design.md) — thiết kế bảo mật (final).
- [docs/testing-plan.md](docs/testing-plan.md) — kế hoạch kiểm thử (kết quả final tại [tests/test-cases.md](tests/test-cases.md)).
- [docs/implementation-notes.md](docs/implementation-notes.md) — ghi chú GUI + limitation phát hiện được.
- [docs/requirements.md](docs/requirements.md) + [docs/project-overview.md](docs/project-overview.md).
- [docs/team-contributions.md](docs/team-contributions.md) — phân công nhóm.

**Tài liệu lịch sử / v1 (được tách khỏi hướng dẫn chính):**

- [docs/v1-history.md](docs/v1-history.md) — tóm tắt giai đoạn P1/v1 + kết quả TC-01→TC-08.
- [docs/topology.md](docs/topology.md) — banner LỊCH SỬ v1 (nội dung khung có dây + WLC-PT; thay bởi final-implementation).
- [docs/device-selection-decision.md](docs/device-selection-decision.md) — decision record (banner SUPERSEDED bởi final).
- [configs/switch/switch-sw1-running-config.txt](configs/switch/switch-sw1-running-config.txt) — cấu hình v1 giữ làm đối chiếu.
