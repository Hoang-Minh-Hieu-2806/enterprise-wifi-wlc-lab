# Thiết kế, Mô phỏng và Đánh giá Bảo mật Mạng Wi-Fi Doanh nghiệp sử dụng Cisco Packet Tracer

> **Tên repository (tiếng Anh):** `enterprise-wifi-wlc-lab`
> **Tiêu đề dự án (tiếng Anh):** Design, Simulation, and Security Evaluation of an Enterprise Wi-Fi Network Using Cisco Packet Tracer

Đồ án môn Quản trị mạng: mô phỏng mạng Wi-Fi doanh nghiệp trên Cisco Packet Tracer, với quản lý tập trung bằng Wireless LAN Controller (WLC), nhiều SSID, phân đoạn VLAN, xác thực truy cập vô tuyến và kiểm soát truy cập giữa các phân đoạn mạng.

> ## ✅ TRẠNG THÁI CUỐI CÙNG (FINAL — 2026-11-07)
>
> Deliverable chính thức: **`packet-tracer/final.pkt`** (18 thiết bị / 21 link; SHA-256 `5AA9ADCA…B5CEF`, 100.297 bytes). Kiến trúc enterprise đầy đủ: **ISP → R1/R2 (2901, PAT overload, 2 path /30) → CORE-SW (3560-24PS L3, SVI VLAN 10/20/99) → ACCESS-SW (2960, trunk) → WLC1 (WLC-3504, mgmt `.99.10`) + LAP-STAFF / LAP-GUEST**, kèm **RADIUS-SRV (`.99.22`, AAA 1645)**, **DHCP-SRV (`.99.20`, 2 pool + relay)**, Web-SRV, ADMIN-PC; **6 wireless client** (3 Staff qua `STAFF-WIFI` WPA2-Enterprise/802.1X; 3 Guest qua `GUEST-WIFI` WPA2-PSK).
>
> **Đã kiểm chứng bằng ping thật (qua bridge, 2026-11-05 → 2026-11-07):**
> - Gateway: cả 6 wireless client + ADMIN-PC `ping` **4/4**.
> - Internet (PAT): Staff → `8.8.8.8` **4/4**; Guest G-LAP1 → `8.8.8.8` **4/4**.
> - Cách ly Guest: G-LAP1 → `192.168.10.1` **0/4** (bị chặn); → `192.168.99.22` **0/4** (bị chặn); DHCP vẫn permit qua `bootps` (lease `.20.142` đạt).
> - Staff leases: `.104/.103/.106`; Guest: `.142/.144/.141`.
>
> **Chưa kiểm chứng end-to-end (ghi rõ trong limitations):** RADIUS 802.1X exchange client↔server (đã cấu hình chuẩn nhưng không chụp được EAP exchange trong PT), failover R1→R2 (đã cấu hình AD 254 nhưng chưa test power-off), sai PSK (TC-14).
>
> Chi tiết nguyên trạng: **[docs/final-implementation.md](docs/final-implementation.md)** (tài liệu chính thức của bản final). Các đoạn "kết quả v1/TC-01→TC-08, `WLC-PT`, `v1.pkt`" bên dưới là **lịch sử giai đoạn P1** (file `v1.pkt` + ảnh minh chứng cũ đã bị xoá khỏi working tree trong đợt chốt final, nằm trong Git history).

## 1. Mục tiêu dự án

1. Xây dựng mạng WLAN doanh nghiệp mô phỏng với quản lý tập trung bằng WLC (`WLC-PT` — model đã xác nhận có trang cấu hình WLAN trong Packet Tracer 9.0.1.0858; **join AP→WLC đã kiểm chứng ở TC-03**; **cả hai WLAN `STAFF-WIFI` + `GUEST-WIFI` đã tạo và phát sóng đồng thời — TC-04 + TC-05 `Passed`**; **associate Staff + Guest bằng WPA2-PSK đã kiểm chứng — TC-06 + TC-07 `Passed`**; **DHCP/VLAN mapping đã `Passed` cho tiêu chí IP đúng subnet (TC-08, kèm giới hạn; cấu hình đã lưu vào official `.pkt` 2026-10-21; local switching qua uplink AP, central switching chưa kiểm chứng); ACL cho client chưa kiểm chứng**).
2. Bắt đầu với 01 Lightweight AP (LAP-PT); chỉ mở rộng lên 2–3 AP sau khi AP đầu tiên join WLC và client kết nối thành công.
3. Triển khai hai SSID dự kiến: `STAFF-WIFI` và `GUEST-WIFI`, mỗi SSID phát sóng và kiểm thử độc lập.
4. Phân đoạn mạng bằng VLAN cho nhân viên (staff), khách (guest) và quản trị (management).
5. Ưu tiên triển khai và kiểm thử **WPA2-PSK** trước như phương thức xác thực đầu tiên.
6. Chỉ thử WPA2-Enterprise trên nền RADIUS cho mạng Staff sau khi luồng cơ bản (WLC + LAP-PT + WLAN + WPA2-PSK + IP/gateway + VLAN/ACL) đã hoạt động ổn định, và chỉ khi Server-PT cùng model WLC đã chọn thực sự hỗ trợ.
7. Kiểm soát truy cập giữa các phân đoạn mạng bằng ACL trên router (explicit deny + permit tối thiểu); Guest không được truy cập Staff và Management.
8. Kiểm thử chức năng, đánh giá bảo mật, thu thập minh chứng và viết tài liệu kỹ thuật.

> Lưu ý: đây là **dự án mô phỏng và tài liệu mạng**, không phải dự án phát triển phần mềm. Không đưa web app, API, database, CI/CD hay hạ tầng không liên quan vào dự án.

## 2. Công nghệ và môi trường mô phỏng

| Hạng mục | Nội dung |
| --- | --- |
| Môi trường mô phỏng | Cisco Packet Tracer 9.0.1.0858 (đã chốt) |
| Thiết bị mạng (đã chốt model) | Router `2911` (subinterface 802.1Q, DHCP relay, ACL), Switch Layer 2 `2960-24TT` (VLAN), WLC **`WLC-PT`** (trang `Wireless LANs`: SSID, VLAN, WPA2-PSK, WPA2 + RADIUS), LAP-PT (AP chính), Server-PT (DHCP, kiểm tra thêm RADIUS), Wireless Clients (≥ 4: 2 Staff + 2 Guest) |
| Thiết bị dự phòng | AP standalone (`AccessPoint-PT`), chỉ dùng khi chứng minh quản lý tập trung bằng WLC là bất khả thi và phạm vi dự án được sửa đổi rõ ràng. Lưu ý AP standalone **không có trường VLAN** nên không thay WLC trong luồng chính |
| Dịch vụ mạng | DHCP trên Server-PT (ưu tiên xác minh trước); RADIUS là mục Conditional, thử sau |
| Ngôn ngữ tài liệu | Tiếng Việt (báo cáo, tài liệu, test case, hướng dẫn nhóm) |
| Ngôn ngữ kỹ thuật | Tiếng Anh (tên thư mục, SSID, VLAN, IP, lệnh, commit message) |

Phiên bản Packet Tracer, model thiết bị quan sát được và các chức năng chưa kiểm chứng được ghi tại [docs/implementation-notes.md](docs/implementation-notes.md). Lựa chọn controller và lý do đổi từ `WLC-3504` sang `WLC-PT` ghi tại [docs/device-selection-decision.md](docs/device-selection-decision.md) (**lịch sử giai đoạn v1** — bản final dùng lại `WLC-3504`, WLAN được cấu hình qua web GUI; xem [docs/final-implementation.md](docs/final-implementation.md) mục 3).

> **Bảng thiết bị dưới đây là giai đoạn v1 (lịch sử).** Danh mục thiết bị của bản final: `ISP` (Router-PT), `R1`/`R2` (2901), `CORE-SW` (3560-24PS), `ACCESS-SW` (2960-24TT), `WLC1` (`WLC-3504`), `LAP-STAFF`/`LAP-GUEST` (LAP-PT), `DHCP-SRV`/`RADIUS-SRV`/`Web-SRV` (Server-PT), `ADMIN-PC`, `S-LAP1/2/3` (Laptop-PT), `G-LAP1/2` (Laptop-PT), `G-PHONE` (Smartphone-PT) — chi tiết tại [docs/final-implementation.md](docs/final-implementation.md).

## 3. Thành phần mạng dự kiến

- 01 Router (định tuyến liên VLAN, DHCP relay **bắt buộc** cho VLAN 10/20 vì Server-PT ở VLAN 99, ACL).
- 01 Switch Layer 2 (VLAN; cổng Switch↔WLC chốt theo model thực tế — xem [docs/topology.md](docs/topology.md) §3).
- 01 WLC (`WLC-PT`) quản lý tập trung AP và WLAN/SSID — trang `Wireless LANs` đã quan sát trực tiếp trong GUI và **đã tạo thành công hai WLAN `STAFF-WIFI` + `GUEST-WIFI`** (TC-04 + TC-05 `Passed`, 2026-10-19).
- 01 LAP-PT ở giai đoạn đầu (mở rộng lên 2–3 AP sau khi AP đầu join thành công); AP standalone chỉ là dự phòng.
- 01 Server-PT cung cấp DHCP; khả năng RADIUS sẽ kiểm tra riêng.
- Ít nhất 4 wireless client: 2 Staff + 2 Guest (hiện có `STAFF1`, `STAFF2` (Staff) và `GUEST1`, `GUEST2` (Guest) — `Laptop-PT`; `STAFF2` dùng ở TC-06, `GUEST2` dùng ở TC-07 và hiện chỉ có trong phiên WIP, chưa lưu vào `.pkt` deliverable).

Xem chi tiết tại [docs/topology.md](docs/topology.md). Phần vô tuyến đã bắt đầu kiểm chứng — **WLAN `STAFF-WIFI` phát sóng (TC-04 `Passed`)**, **cơ chế join AP→WLC đã kiểm chứng (TC-03 `Passed`)** và **hai Staff client associate bằng WPA2-PSK (TC-06 `Passed`)**, **hai Guest client associate bằng WPA2-PSK (TC-07 `Passed`)**; **ánh xạ SSID→VLAN cho client đã `Passed` cho tiêu chí IP đúng subnet (TC-08, kèm giới hạn)** — đường dữ liệu qua uplink AP (local switching), **central switching của WLC chưa kiểm chứng**. Cấu hình TC-08 đã được lưu vào official `.pkt` ngày 2026-10-21; lease DHCP phải `/renew` sau khi mở lại.

## 4. Phân tách SSID và VLAN (đề xuất)

| SSID (dự kiến) | VLAN (dự kiến) | Tên VLAN | Subnet dự kiến | Đối tượng |
| --- | --- | --- | --- | --- |
| `STAFF-WIFI` | 10 | STAFF | 192.168.10.0/24 | Nhân viên nội bộ |
| `GUEST-WIFI` | 20 | GUEST | 192.168.20.0/24 | Khách |
| — (quản trị, có dây) | 99 | MANAGEMENT | 192.168.99.0/24 | Quản trị thiết bị mạng |

> Phần vô tuyến đã bắt đầu kiểm chứng: `STAFF-WIFI` (VLAN 10) đã tạo và phát sóng (**TC-04 `Passed`**, 2026-10-19); **ánh xạ SSID→VLAN cho client đã `Passed` cho tiêu chí IP đúng subnet (TC-08, kèm giới hạn; cấu hình đã lưu vào official `.pkt` 2026-10-21)** — qua uplink AP, local switching; **central switching chưa kiểm chứng**; các phần WLAN còn lại (gateway/ACL) vẫn **chưa kiểm chứng**. Riêng **hạ tầng có dây và địa chỉ quản trị đã cấu hình và kiểm chứng**: gateway `.1` mỗi VLAN, WLC `.10`, switch SVI `.2`, server `.20` — xem [docs/ip-vlan-plan.md](docs/ip-vlan-plan.md) (tài liệu chuẩn cho mọi địa chỉ) và minh chứng TC-01.

## 5. Hướng tiếp cận xác thực và kiểm soát truy cập

- **Thứ tự triển khai:** WPA2-PSK trước cho cả hai SSID; RADIUS/WPA2-Enterprise (nếu có) chỉ thử sau khi luồng cơ bản ổn định. Sự tồn tại của các trường WPA2/RADIUS trong GUI **không** chứng minh giao thức đã chạy được.
- **Mạng Staff (`STAFF-WIFI`):** WPA2-PSK trước; WPA2-Enterprise + RADIUS là mục Conditional, chỉ cho Staff và chỉ khi được hỗ trợ.
- **Mạng Guest (`GUEST-WIFI`):** WPA2-PSK. Chỉ dùng Open cho mục đích kiểm thử và phải ghi rõ rủi ro không mã hóa.
- **Phân tách mạng:** VLAN riêng cho staff / guest / management (chi tiết tại [docs/security-design.md](docs/security-design.md)).
- **Kiểm soát liên VLAN:** ACL trên router với explicit deny + permit tối thiểu; Guest bị chặn truy cập Staff và Management.
- **Truy cập Internet:** không khẳng định có Internet trừ khi có đích ISP/Internet được mô phỏng và kiểm thử thực tế.
- **Bảo vệ mạng quản trị:** thiết bị hạ tầng ở VLAN 99, ưu tiên địa chỉ tĩnh, hạn chế truy cập quản trị.
- **Chủ đề nghiên cứu (Research-only):** WPA3, EAP-TLS/PEAP, roaming, nhiễu RF — chỉ làm nền báo cáo, không tính vào tiêu chí hoàn thành.

## 6. Cấu trúc repository

```text
enterprise-wifi-wlc-lab/
├── README.md
├── .gitignore
├── LICENSE
├── docs/
│   ├── project-overview.md
│   ├── requirements.md
│   ├── topology.md                  (chứa mục lịch sử v1)
│   ├── ip-vlan-plan.md              (chứa mục lịch sử v1)
│   ├── security-design.md
│   ├── testing-plan.md
│   ├── final-implementation.md      ⭐ tài liệu chính thức của bản final
│   ├── implementation-notes.md
│   ├── device-selection-decision.md
│   └── team-contributions.md
├── packet-tracer/
│   ├── README.md
│   └── final.pkt                    ⭐ deliverable chính thức (18 thiết bị)
├── configs/
│   ├── README.md
│   ├── router/
│   │   ├── README.md
│   │   ├── router-r1-running-config.txt    (final: R1 2901)
│   │   └── router-r2-running-config.txt    (final: R2 2901)
│   ├── switch/
│   │   ├── README.md
│   │   ├── core-sw-running-config.txt      (final: CORE-SW 3560-24PS)
│   │   └── switch-sw1-running-config.txt   (lịch sử v1)
│   └── wlc-ap/
│       └── README.md (ghi chú cấu hình WLAN qua web GUI — WLC/server không có CLI)
├── tests/
│   ├── README.md
│   └── test-cases.md
├── evidence/
│   ├── README.md
│   ├── Final.jpg
│   ├── Final 2.png
│   └── Final but error.png
└── report/
    ├── README.md
    └── midterm-report.md
```

- `docs/` — tài liệu thiết kế, kế hoạch và ghi chú triển khai (tiếng Việt); **`final-implementation.md` là tài liệu chính thức của bản deliverable cuối**.
- `packet-tracer/` — `final.pkt` (topology enterprise final, 18 thiết bị). File v1 cũ (`enterprise-wifi-wlc-lab-v1.pkt`, 9 thiết bị, `WLC-PT`) đã bị xoá trong đợt chốt final (xem Git history).
- `configs/` — bản sao cấu hình thiết bị (không có secret); `router/` (R1 + R2, final 2026-11-07), `switch/` (CORE-SW final + sw1 lịch sử v1), `wlc-ap/` ghi cấu hình WLC/SSID qua GUI (không ghi giá trị PSK/secret).
- `tests/` — kế hoạch và ca kiểm thử (Core / Conditional / Research-only), cập nhật kết quả final tại mục "Minh chứng final" của `test-cases.md`.
- `evidence/` — minh chứng final: canvas topology (`Final.jpg`, `Final 2.png`) và ghi nhận bất ổn mô phỏng (`Final but error.png`); minh chứng v1 cũ (tc-01→tc-08, audit-*) đã bị xoá, nằm trong Git history. Ảnh chụp audit trung gian của công cụ tự động (`projects/`, `screenshots/`) là scratch cục bộ, không track trong Git.
- `report/` — dàn ý và nội dung báo cáo trong giai đoạn hiện tại (chưa cần DOCX/PDF).

## 7. Cách mở dự án Packet Tracer

> File `.pkt` hiện tại: **`packet-tracer/final.pkt`** — topology enterprise final (ISP + R1/R2 + CORE-SW + ACCESS-SW + WLC-3504 + 2 AP + DHCP/RADIUS/Web server + 6 wireless client + ADMIN-PC). Sau khi mở lại file: chạy `ipconfig /renew` trên từng client (lease DHCP không giữ sau reopen); nếu AP/client mất association, đọc mục "Lưu ý khi mở" tại [packet-tracer/README.md](packet-tracer/README.md). File v1 cũ (`enterprise-wifi-wlc-lab-v1.pkt`) đã bị xoá khỏi repo ngày 2026-11-07 (history: `git log -- packet-tracer/`).

1. Cài đặt đúng Cisco Packet Tracer **9.0.1.0858** (bản đã chốt, xem tại [docs/implementation-notes.md](docs/implementation-notes.md)).
2. Clone repository:
   ```bash
   git clone <repository-url>
   cd enterprise-wifi-wlc-lab
   ```
3. Mở thư mục `packet-tracer/` và double-click file `.pkt`.
4. Nếu Packet Tracer báo thiếu model thiết bị, đối chiếu với danh sách thiết bị trong [docs/implementation-notes.md](docs/implementation-notes.md) và [docs/topology.md](docs/topology.md).
5. Không chỉnh sửa topology khi chưa thống nhất với nhóm; mọi thay đổi cấu hình cần được lưu sang `configs/` (đã che secret) và ghi chú lại.

Quy định đặt file `.pkt` xem tại [packet-tracer/README.md](packet-tracer/README.md).

## 8. Trạng thái dự án hiện tại

**⚠ Mục này chính thức cập nhật cho bản FINAL (2026-11-07).** Repository hiện ở giai đoạn **deliverable hoàn chỉnh**: topology enterprise 18 thiết bị được dựng xong, cấu hình đầy đủ và đã chạy kiểm thử chức năng bằng ping thật (chi tiết tại [docs/final-implementation.md](docs/final-implementation.md)).

**Đã kiểm chứng trên final (2026-11-05 → 2026-11-07):**

| Nhóm trạng thái | Ý nghĩa | Trạng thái hiện tại |
| --- | --- | --- |
| Passed | Đã test trên final, có kết quả thật | **Gateway 6/6 client 4/4**; **Staff/Guest → Internet `8.8.8.8` 4/4** (PAT); **Guest bị chặn Staff `.10.1` 0/4 và Mgmt `.99.22` 0/4** (ACL `GUEST-ACL`); **DHCP lease đúng subnet cho 6 client**; Staff/RADIUS/DHCP config khớp 2 đầu |
| Configured / chưa chứng minh exchange | Cấu hình đúng chuẩn nhưng thiếu minh chứng quá trình | **RADIUS 802.1X** (WLC + server khớp port 1645, users đã tạo; EAP exchange không quan sát được trong PT) |
| Not Tested | Chưa kiểm thử, chưa có minh chứng | **Sai PSK bị từ chối (TC-14)**; **failover R1→R2** (AD 254 cấu hình xong); screenshot `show vlan brief` riêng |
| Lịch sử v1 (kết quả trước 2026-11-01) | Giai đoạn P1 trên `v1.pkt` + `WLC-PT` | TC-01→TC-08 `Passed` kèm minh chứng (và giới hạn đã ghi); file `v1.pkt` + ảnh tc-*-old đã xoá khỏi working tree |

> Trạng thái TC-01→TC-08 ở **[tests/test-cases.md](tests/test-cases.md)** giữ nguyên làm lịch sử giai đoạn v1; trạng thái tương ứng trên final được mô tả tại [docs/final-implementation.md](docs/final-implementation.md) mục 7. Nhãn `Passed` chỉ dùng khi có kết quả thật; một tính năng Conditional (RADIUS exchange) chưa minh chứng **không** làm mất trạng thái hoàn thành của luồng Core.

## 9. Hạn chế đã biết và nội dung cần kiểm chứng

Đã kiểm chứng trong **khung có dây P1**:

- [x] Model Router `2911`, Switch `2960-24TT`, Server-PT — chốt bằng cách đặt thật vào bản 9.0.1.0858.
- [x] VLAN 10/20/99 trên switch; trunk `Fa0/3` (LAP-PT), `Fa0/4` (WLC) và `Gi0/1` (router) native VLAN 99 + tagged 10,20; access `Fa0/2` (Server-PT) VLAN 99; `Fa0/1` là trunk legacy không có link.
- [x] Liên VLAN trên router (`Gi0/0.10`, `Gi0/0.20`) kèm `ip helper-address 192.168.99.20`.
- [x] Server-PT tĩnh `192.168.99.20` + 2 pool chính (STAFF, GUEST) và `serverPool` giới hạn `.30`, Service On.
- [x] **WLC management reachability (TC-01 `Passed`, chạy lại trên `WLC-PT` 2026-10-19):** Management IPv4 `192.168.99.10/24` trên cổng GUI `Management` (API port `Vlan1`), ping từ `SRV1` đạt `0% loss` — minh chứng `evidence/tc-01-wlc-pt-reachability-canvas.png`. Bản `WLC-3504` cũ (`evidence/tc-01-wlc-reachability.png`) giữ làm lịch sử P1.
- [x] **LAP-PT nhận IP quản trị (TC-02 `Passed`, 2026-10-18):** `192.168.99.30/24` tĩnh trên `GigabitEthernet0`, uplink `SW1 Fa0/3` access VLAN 99 — minh chứng `evidence/tc-02-*.png`.
- [x] **LAP-PT join `WLC-PT` (TC-03 `Passed`, 2026-10-19):** `WLC2 → Config → AP Groups` hiển thị `LAP1` (`0001.968E.6B01`) `Status: Online` trong nhóm `default-group` (auto-discovery của PT) — minh chứng `evidence/tc-03-lap-wlc-join.png`. Không có minh chứng bắt tay CAPWAP.
- [x] **Giao diện của `WLC-PT`** đã quan sát trực tiếp: có `Wireless LANs` (SSID, trường VLAN, WPA2-PSK, WPA2 + RADIUS), `AP Groups`, `DHCP` — minh chứng `evidence/audit-wlc-pt-wireless-lans.png`.
- [x] **WLAN `STAFF-WIFI` tạo và phát sóng (TC-04 `Passed`, 2026-10-19):** WLAN ID 1, VLAN 10, WPA2-PSK/AES trên `WLC2`; quét từ `STAFF1` (`Laptop-PT`) thấy `STAFF-WIFI` — minh chứng `evidence/tc-04-staff-wlan.png`, `evidence/tc-04-wlc-staff-wlan-config.png` (PSK đã che).
- [x] **WLAN `GUEST-WIFI` tạo độc lập và phát sóng đồng thời (TC-05 `Passed`, 2026-10-19):** WLAN ID 2, VLAN 20, WPA2-PSK/AES trên `WLC2`; quét từ `GUEST1` (`Laptop-PT`, thêm mới) thấy **đồng thời** `STAFF-WIFI` + `GUEST-WIFI`; lưu + mở lại `.pkt` vẫn còn — minh chứng `evidence/tc-05-guest-wlan.png`, `evidence/tc-05-wlc-guest-wlan-config.png` (PSK đã che).
- [x] **Client nhận IP đúng subnet (TC-08 `Passed`, 2026-10-20; cấu hình lưu vào official `.pkt` 2026-10-21):** test copy `STAFF1` `192.168.10.102/24`, `GUEST1` `192.168.20.102/24`; official sau khi mở lại và `/renew`: `STAFF1` `.10.101/24`, `GUEST1` `.20.100/24`, `STAFF2` `.10.100/24`; `SW1 Fa0/3` (uplink AP) trunk native 99 mang VLAN 10/20; MAC-table đúng VLAN trên `Fa0/3`, **không** có MAC client trên `Fa0/4` (WLC) — minh chứng `evidence/tc-08-*.png`. **Giới hạn:** đường dữ liệu là local switching tại AP, **central switching của WLC chưa kiểm chứng**; `serverPool` không xoá vĩnh viễn được, chỉ giới hạn `.99.30–.30` max 1; `LAP-PT` nhận `.30` qua DHCP, chưa xác nhận tĩnh lưu bền; lease DHCP cần `/renew` sau khi mở lại.

Đã quan sát trong GUI nhưng **chưa kiểm chứng hoạt động**:

- [x] **Thay controller `WLC-3504` → `WLC-PT` trong `.pkt` và chạy lại TC-01** (hoàn tất 2026-10-19).
- [x] Tạo và phát `GUEST-WIFI` trên `WLC-PT` (`STAFF-WIFI` đã xong — TC-04 `Passed`; `GUEST-WIFI` xong — TC-05 `Passed`).
- [x] Nhiều WLAN phát đồng thời (hai SSID cùng phát sóng — TC-05 `Passed`; associate Staff đã kiểm chứng — TC-06 `Passed`; associate Guest đã kiểm chứng — TC-07 `Passed`).
- [x] Ánh xạ mỗi SSID tới đúng VLAN — **kiểm chứng TC-08** (`STAFF1` `.10.102/24`, `GUEST1` `.20.102/24` trên test copy; official `/renew` `.10.101/24`, `.20.100/24`; MAC VLAN 10/20 trên `Fa0/3`), với giới hạn; cấu hình đã lưu vào official `.pkt` 2026-10-21.
- [ ] WPA2-PSK: associate thành công cho Staff + Guest đã kiểm chứng (TC-06 + TC-07 `Passed`); sai PSK bị từ chối (TC-14) chưa kiểm chứng.
- [ ] WPA2-Enterprise/RADIUS vận hành (Conditional, thử sau cùng).
- [x] Chế độ cổng **Switch↔LAP** mang được lưu lượng client VLAN 10/20 — **kiểm chứng TC-08** (`Fa0/3` trunk, client nhận đúng subnet, MAC VLAN 10/20 trên `Fa0/3`); cấu hình đã lưu vào official `.pkt` 2026-10-21.
- [ ] **Central switching của `WLC-PT`** (client VLAN đi qua uplink WLC `Fa0/4`) — chưa kiểm chứng; TC-08 cho thấy đường dữ liệu đang là local switching qua uplink AP.
- [ ] Guest isolation end-to-end sau ACL.
- [x] DHCP cấp phát thật cho client VLAN 10/20 — **kiểm chứng TC-08** (test copy `.10.102/24`, `.20.102/24`; official `/renew` `.10.101/24`, `.20.100/24`); **giới hạn** `serverPool` chỉ `.99.30–.30`, max 1, không xoá vĩnh viễn được; lease DHCP cần `/renew` sau khi mở lại.

Chủ đề Research-only: WPA3, EAP-TLS/PEAP, roaming, nhiễu RF, đo throughput — **không khẳng định**, chỉ làm nền báo cáo. Riêng **WPA3/SAE không có trong `WirelessAuthenType` của `IpcAPI` bản 9.0.1.0858** nên không thể triển khai trên phiên bản đã chốt.

### Hạn chế của bản FINAL (2026-11-07) — ghi nguyên vào báo cáo

1. **RADIUS 802.1X chưa có minh chứng exchange end-to-end** — cấu hình 2 đầu khớp (port 1645, users, client `WLC1`) nhưng PT không log được EAP giữa client↔AP↔WLC↔server.
2. **Failover R1→R2 chưa test power-off** — backup default route AD 254 đã cấu hình đúng trên CORE-SW.
3. **Bất ổn mô phỏng wireless PT 9.0.1.0858** — association đôi lúc rớt khi thao tác nhiều (chứng kiến `evidence/Final but error.png`); recovery power-cycle + DHCP re-lease.
4. **`show ip interface` của PT không hiển thị ACL named** trên `Vlan20` — dùng Running Config thay thế.
5. **DHCP lease không giữ sau reopen** `final.pkt` — phải `ipconfig /renew`; `serverPool` factory không xoá vĩnh viễn.
6. **Central switching của WLC-3504 không kiểm chứng bằng packet-trace** — chứng nhận chức năng bằng associate + lease.
7. PSK/secret lưu dạng rõ trong XML `.pkt` — không chia sẻ file ngoài nhóm.

Mọi khẳng định về hiệu quả mã hóa hay kiểm soát truy cập chỉ được ghi nhận sau khi kiểm thử, xem [docs/testing-plan.md](docs/testing-plan.md) và [docs/final-implementation.md](docs/final-implementation.md).

## 10. Thành viên nhóm

| Thành viên | Vai trò | Tỉ trọng mục tiêu |
| --- | --- | --- |
| Hoàng Minh Hiếu (nhóm trưởng) | Điều phối, quản lý Git/tiến độ, rà soát thiết kế, hỗ trợ kỹ thuật linh hoạt, tham gia kiểm thử, tổng hợp báo cáo | ~33% |
| Đặng Văn Hoàng | Topology, WLC/LAP, WLAN, xác thực, evidence vô tuyến | ~33% |
| Huỳnh Như Hoàng | Định tuyến, VLAN, DHCP, ACL, kiểm thử phân tách, tổng hợp kết quả | ~34% |

Phân công chi tiết xem tại [docs/team-contributions.md](docs/team-contributions.md). Ba thành viên chia khối lượng tương đương; nhóm trưởng vừa điều phối vừa trực tiếp làm kỹ thuật.

## 11. Tài liệu liên quan

- [docs/project-overview.md](docs/project-overview.md) — tổng quan, phạm vi, deliverables.
- [docs/final-implementation.md](docs/final-implementation.md) — ⭐ **tài liệu chính thức của bản final (2026-11-07)**: kiến trúc, địa chỉ, cấu hình, kết quả kiểm thử, limitations.
- [docs/requirements.md](docs/requirements.md) — bảng yêu cầu và trạng thái.
- [docs/topology.md](docs/topology.md) — topology logic (mức triển khai được).
- [docs/ip-vlan-plan.md](docs/ip-vlan-plan.md) — kế hoạch địa chỉ chuẩn.
- [docs/security-design.md](docs/security-design.md) — thiết kế bảo mật (PSK-first).
- [docs/testing-plan.md](docs/testing-plan.md) — kế hoạch kiểm thử Core/Conditional.
- [docs/implementation-notes.md](docs/implementation-notes.md) — phát hiện GUI và hạn chế.
- [docs/device-selection-decision.md](docs/device-selection-decision.md) — quyết định lựa chọn controller/AP và lý do đổi `WLC-3504` → `WLC-PT`.
- [docs/team-contributions.md](docs/team-contributions.md) — phân công nhóm.
- [tests/test-cases.md](tests/test-cases.md) — danh sách ca kiểm thử.
