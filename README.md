# Thiết kế, Mô phỏng và Đánh giá Bảo mật Mạng Wi-Fi Doanh nghiệp sử dụng Cisco Packet Tracer

> **Tên repository (tiếng Anh):** `enterprise-wifi-wlc-lab`
> **Tiêu đề dự án (tiếng Anh):** Design, Simulation, and Security Evaluation of an Enterprise Wi-Fi Network Using Cisco Packet Tracer

Đồ án môn Quản trị mạng: mô phỏng mạng Wi-Fi doanh nghiệp trên Cisco Packet Tracer, với quản lý tập trung bằng Wireless LAN Controller (WLC), nhiều SSID, phân đoạn VLAN, xác thực truy cập vô tuyến và kiểm soát truy cập giữa các phân đoạn mạng.

## 1. Mục tiêu dự án

1. Xây dựng mạng WLAN doanh nghiệp mô phỏng với quản lý tập trung bằng WLC (`WLC-PT` — model đã xác nhận có trang cấu hình WLAN trong Packet Tracer 9.0.1.0858; **join AP→WLC đã kiểm chứng ở TC-03**; **cả hai WLAN `STAFF-WIFI` + `GUEST-WIFI` đã tạo và phát sóng đồng thời — TC-04 + TC-05 `Passed`**; **associate Staff bằng WPA2-PSK đã kiểm chứng — TC-06 `Passed`**; phần DHCP/VLAN mapping/ACL cho client còn chưa kiểm chứng).
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

Phiên bản Packet Tracer, model thiết bị quan sát được và các chức năng chưa kiểm chứng được ghi tại [docs/implementation-notes.md](docs/implementation-notes.md). Lựa chọn controller và lý do đổi từ `WLC-3504` sang `WLC-PT` ghi tại [docs/device-selection-decision.md](docs/device-selection-decision.md).

## 3. Thành phần mạng dự kiến

- 01 Router (định tuyến liên VLAN, DHCP relay **bắt buộc** cho VLAN 10/20 vì Server-PT ở VLAN 99, ACL).
- 01 Switch Layer 2 (VLAN; cổng Switch↔WLC chốt theo model thực tế — xem [docs/topology.md](docs/topology.md) §3).
- 01 WLC (`WLC-PT`) quản lý tập trung AP và WLAN/SSID — trang `Wireless LANs` đã quan sát trực tiếp trong GUI và **đã tạo thành công hai WLAN `STAFF-WIFI` + `GUEST-WIFI`** (TC-04 + TC-05 `Passed`, 2026-10-19).
- 01 LAP-PT ở giai đoạn đầu (mở rộng lên 2–3 AP sau khi AP đầu join thành công); AP standalone chỉ là dự phòng.
- 01 Server-PT cung cấp DHCP; khả năng RADIUS sẽ kiểm tra riêng.
- Ít nhất 4 wireless client: 2 Staff + 2 Guest (hiện có `STAFF1`, `STAFF2` (Staff) và `GUEST1` (Guest) — `Laptop-PT`; `STAFF2` dùng ở TC-06).

Xem chi tiết tại [docs/topology.md](docs/topology.md). Phần vô tuyến đã bắt đầu kiểm chứng — **WLAN `STAFF-WIFI` phát sóng (TC-04 `Passed`)**, **cơ chế join AP→WLC đã kiểm chứng (TC-03 `Passed`)** và **hai Staff client associate bằng WPA2-PSK (TC-06 `Passed`)**; ánh xạ SSID→VLAN cho client chưa được kiểm chứng.

## 4. Phân tách SSID và VLAN (đề xuất)

| SSID (dự kiến) | VLAN (dự kiến) | Tên VLAN | Subnet dự kiến | Đối tượng |
| --- | --- | --- | --- | --- |
| `STAFF-WIFI` | 10 | STAFF | 192.168.10.0/24 | Nhân viên nội bộ |
| `GUEST-WIFI` | 20 | GUEST | 192.168.20.0/24 | Khách |
| — (quản trị, có dây) | 99 | MANAGEMENT | 192.168.99.0/24 | Quản trị thiết bị mạng |

> Phần vô tuyến đã bắt đầu kiểm chứng: `STAFF-WIFI` (VLAN 10) đã tạo và phát sóng (**TC-04 `Passed`**, 2026-10-19); ánh xạ SSID→VLAN cho client và các phần WLAN còn lại vẫn **chưa kiểm chứng**. Riêng **hạ tầng có dây và địa chỉ quản trị đã cấu hình và kiểm chứng**: gateway `.1` mỗi VLAN, WLC `.10`, switch SVI `.2`, server `.20` — xem [docs/ip-vlan-plan.md](docs/ip-vlan-plan.md) (tài liệu chuẩn cho mọi địa chỉ) và minh chứng TC-01.

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
│   ├── topology.md
│   ├── ip-vlan-plan.md
│   ├── security-design.md
│   ├── testing-plan.md
│   ├── implementation-notes.md
│   ├── device-selection-decision.md
│   └── team-contributions.md
├── packet-tracer/
│   ├── README.md
│   └── enterprise-wifi-wlc-lab-v1.pkt
├── configs/
│   ├── README.md
│   ├── router/
│   │   ├── README.md
│   │   └── router-r1-running-config.txt
│   ├── switch/
│   │   ├── README.md
│   │   └── switch-sw1-running-config.txt
│   └── wlc-ap/
│       └── README.md
├── tests/
│   ├── README.md
│   └── test-cases.md
├── evidence/
│   ├── README.md
│   ├── p1-wired-skeleton-topology.png
│   ├── tc-01-wlc-reachability.png
│   ├── tc-02-lap-ip.png
│   ├── tc-02-lap-ping-gateway.png
│   ├── tc-02-lap-arp-r1.png
│   └── audit-*.png
└── report/
    ├── README.md
    └── midterm-report.md
```

- `docs/` — tài liệu thiết kế, kế hoạch và ghi chú triển khai (tiếng Việt).
- `packet-tracer/` — file mô phỏng `.pkt`; `enterprise-wifi-wlc-lab-v1.pkt` gồm khung có dây P1 + hai WLAN `STAFF-WIFI`/`GUEST-WIFI` + client `STAFF1`, `STAFF2`, `GUEST1`.
- `configs/` — bản sao cấu hình thiết bị đã che secret; `router/` và `switch/` có bản khung P1, `wlc-ap/` ghi cấu hình hạ tầng WLC + WLAN `STAFF-WIFI` (không ghi giá trị PSK).
- `tests/` — kế hoạch và ca kiểm thử (Core / Conditional / Research-only).
- `evidence/` — ảnh chụp màn hình, kết quả kiểm thử; có minh chứng bring-up P1, minh chứng TC-01 → TC-06 và minh chứng audit lựa chọn thiết bị (`audit-*.png`).
- `report/` — dàn ý và nội dung báo cáo trong giai đoạn hiện tại (chưa cần DOCX/PDF).

## 7. Cách mở dự án Packet Tracer

> File `.pkt` đã có: `packet-tracer/enterprise-wifi-wlc-lab-v1.pkt` — khung có dây P1 + `WLC-PT` + `LAP-PT` + hai WLAN `STAFF-WIFI`/`GUEST-WIFI` + các client `STAFF1`, `STAFF2`, `GUEST1` (`Laptop-PT`); **cần đặt lại IP tĩnh `192.168.99.30` cho `LAP1` sau khi mở file** (trạng thái join/broadcast là runtime); client Staff đã associate trước đó tự kết nối lại khi AP online — kiểm chứng khi mở lại file 2026-10-19, không cần bấm `Connect`; client chưa từng associate (Guest, TC-07) bấm `Connect` để quét/associate SSID.

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

Repository hiện ở giai đoạn **đã dựng khung có dây (P1), controller + LAP-PT hoạt động, cả hai WLAN đã phát sóng và hai Staff client associate bằng WPA2-PSK**: **TC-01 `Passed`**, **TC-02 `Passed`**, **TC-03 `Passed`**, **TC-04 `Passed`**, **TC-05 `Passed`** và **TC-06 `Passed`** (2026-10-19); DHCP/VLAN mapping/ACL cho client và associate Guest vẫn chưa kiểm chứng.

> **Đã hoàn tất — đổi controller sang `WLC-PT` (2026-10-19).** Audit lựa chọn thiết bị phát hiện `WLC-3504` **không có giao diện cấu hình vô tuyến** trong bản 9.0.1.0858. Đã thay `WLC-3504` → `WLC-PT` **ngay trong `.pkt` deliverable** và **chạy lại TC-01** (địa chỉ quản trị trên interface GUI `Management`, API `Vlan1`). Xem [docs/device-selection-decision.md](docs/device-selection-decision.md).

| Nhóm trạng thái | Ý nghĩa | Trạng thái hiện tại |
| --- | --- | --- |
| Planned | Đã đề xuất trong tài liệu, chưa cấu hình | Cấu hình ACL, DHCP/VLAN mapping cho client và associate Guest vào `GUEST-WIFI` (TC-07) chưa triển khai |
| In Progress | Đang thực hiện | Hạ tầng có dây + TC-01→TC-06 xong; phần vô tuyến đang kiểm chứng (client `STAFF1`, `STAFF2`, `GUEST1`; hai WLAN `STAFF-WIFI`/`GUEST-WIFI` đã phát sóng đồng thời; associate Staff đã kiểm chứng — TC-06 `Passed`) |
| Not Tested | Chưa kiểm thử, chưa có minh chứng | TC-07 → TC-14 trong [tests/test-cases.md](tests/test-cases.md) |
| Passed / Failed / Blocked | Chỉ dùng khi có evidence thực tế | **TC-01 `Passed`** (chạy lại trên `WLC-PT`), **TC-02 `Passed`** (trên `LAP-PT`), **TC-03 `Passed`** (LAP-PT join `WLC-PT`), **TC-04 `Passed`** (WLAN `STAFF-WIFI` phát sóng), **TC-05 `Passed`** (WLAN `GUEST-WIFI` phát sóng đồng thời), **TC-06 `Passed`** (hai Staff client associate vào `STAFF-WIFI` bằng WPA2-PSK); chưa có `Failed`/`Blocked` |

Nhãn `Passed` chỉ được dùng khi có minh chứng hỗ trợ. Một tính năng Conditional (RADIUS) chưa được hỗ trợ **không** làm mất trạng thái hoàn thành của luồng Core.

## 9. Hạn chế đã biết và nội dung cần kiểm chứng

Đã kiểm chứng trong **khung có dây P1**:

- [x] Model Router `2911`, Switch `2960-24TT`, Server-PT — chốt bằng cách đặt thật vào bản 9.0.1.0858.
- [x] VLAN 10/20/99 trên switch; trunk `Fa0/4` (WLC — thay cho `Fa0/1` thời `WLC-3504`) và `Gi0/1` (router) native VLAN 99 + tagged 10,20; access `Fa0/2` (Server-PT) và `Fa0/3` (LAP-PT) VLAN 99.
- [x] Liên VLAN trên router (`Gi0/0.10`, `Gi0/0.20`) kèm `ip helper-address 192.168.99.20`.
- [x] Server-PT tĩnh `192.168.99.20` + 2 pool DHCP (STAFF, GUEST), Service On.
- [x] **WLC management reachability (TC-01 `Passed`, chạy lại trên `WLC-PT` 2026-10-19):** Management IPv4 `192.168.99.10/24` trên cổng GUI `Management` (API port `Vlan1`), ping từ `SRV1` đạt `0% loss` — minh chứng `evidence/tc-01-wlc-pt-reachability-canvas.png`. Bản `WLC-3504` cũ (`evidence/tc-01-wlc-reachability.png`) giữ làm lịch sử P1.
- [x] **LAP-PT nhận IP quản trị (TC-02 `Passed`, 2026-10-18):** `192.168.99.30/24` tĩnh trên `GigabitEthernet0`, uplink `SW1 Fa0/3` access VLAN 99 — minh chứng `evidence/tc-02-*.png`.
- [x] **LAP-PT join `WLC-PT` (TC-03 `Passed`, 2026-10-19):** `WLC2 → Config → AP Groups` hiển thị `LAP1` (`0001.968E.6B01`) `Status: Online` trong nhóm `default-group` (auto-discovery của PT) — minh chứng `evidence/tc-03-lap-wlc-join.png`. Không có minh chứng bắt tay CAPWAP.
- [x] **Giao diện của `WLC-PT`** đã quan sát trực tiếp: có `Wireless LANs` (SSID, trường VLAN, WPA2-PSK, WPA2 + RADIUS), `AP Groups`, `DHCP` — minh chứng `evidence/audit-wlc-pt-wireless-lans.png`.
- [x] **WLAN `STAFF-WIFI` tạo và phát sóng (TC-04 `Passed`, 2026-10-19):** WLAN ID 1, VLAN 10, WPA2-PSK/AES trên `WLC2`; quét từ `STAFF1` (`Laptop-PT`) thấy `STAFF-WIFI` — minh chứng `evidence/tc-04-staff-wlan.png`, `evidence/tc-04-wlc-staff-wlan-config.png` (PSK đã che).
- [x] **WLAN `GUEST-WIFI` tạo độc lập và phát sóng đồng thời (TC-05 `Passed`, 2026-10-19):** WLAN ID 2, VLAN 20, WPA2-PSK/AES trên `WLC2`; quét từ `GUEST1` (`Laptop-PT`, thêm mới) thấy **đồng thời** `STAFF-WIFI` + `GUEST-WIFI`; lưu + mở lại `.pkt` vẫn còn — minh chứng `evidence/tc-05-guest-wlan.png`, `evidence/tc-05-wlc-guest-wlan-config.png` (PSK đã che).

Đã quan sát trong GUI nhưng **chưa kiểm chứng hoạt động**:

- [x] **Thay controller `WLC-3504` → `WLC-PT` trong `.pkt` và chạy lại TC-01** (hoàn tất 2026-10-19).
- [x] Tạo và phát `GUEST-WIFI` trên `WLC-PT` (`STAFF-WIFI` đã xong — TC-04 `Passed`; `GUEST-WIFI` xong — TC-05 `Passed`).
- [x] Nhiều WLAN phát đồng thời (hai SSID cùng phát sóng — TC-05 `Passed`; associate Staff đã kiểm chứng — TC-06 `Passed`; associate Guest còn chưa — TC-07).
- [ ] Ánh xạ mỗi SSID tới đúng VLAN (trường VLAN đã quan sát trong GUI nhưng chưa kiểm chứng hoạt động).
- [ ] WPA2-PSK: associate thành công cho Staff đã kiểm chứng (TC-06 `Passed`); associate Guest (TC-07) và sai PSK bị từ chối (TC-14) chưa kiểm chứng.
- [ ] WPA2-Enterprise/RADIUS vận hành (Conditional, thử sau cùng).
- [ ] Chế độ cổng Switch↔`WLC-PT` mang được lưu lượng client VLAN 10/20 (cổng không có tuỳ chọn trunk trong GUI).
- [ ] Guest isolation end-to-end sau ACL.
- [ ] DHCP cấp phát thật cho client mọi VLAN (pool đã tạo, chưa có client).

Chủ đề Research-only: WPA3, EAP-TLS/PEAP, roaming, nhiễu RF, đo throughput — **không khẳng định**, chỉ làm nền báo cáo. Riêng **WPA3/SAE không có trong `WirelessAuthenType` của `IpcAPI` bản 9.0.1.0858** nên không thể triển khai trên phiên bản đã chốt.

Mọi khẳng định về hiệu quả mã hóa hay kiểm soát truy cập chỉ được ghi nhận sau khi kiểm thử, xem [docs/testing-plan.md](docs/testing-plan.md).

## 10. Thành viên nhóm

| Thành viên | Vai trò | Tỉ trọng mục tiêu |
| --- | --- | --- |
| Hoàng Minh Hiếu (nhóm trưởng) | Điều phối, quản lý Git/tiến độ, rà soát thiết kế, hỗ trợ kỹ thuật linh hoạt, tham gia kiểm thử, tổng hợp báo cáo | ~33% |
| Đặng Văn Hoàng | Topology, WLC/LAP, WLAN, xác thực, evidence vô tuyến | ~33% |
| Huỳnh Như Hoàng | Định tuyến, VLAN, DHCP, ACL, kiểm thử phân tách, tổng hợp kết quả | ~34% |

Phân công chi tiết xem tại [docs/team-contributions.md](docs/team-contributions.md). Ba thành viên chia khối lượng tương đương; nhóm trưởng vừa điều phối vừa trực tiếp làm kỹ thuật.

## 11. Tài liệu liên quan

- [docs/project-overview.md](docs/project-overview.md) — tổng quan, phạm vi, deliverables.
- [docs/requirements.md](docs/requirements.md) — bảng yêu cầu và trạng thái.
- [docs/topology.md](docs/topology.md) — topology logic (mức triển khai được).
- [docs/ip-vlan-plan.md](docs/ip-vlan-plan.md) — kế hoạch địa chỉ chuẩn.
- [docs/security-design.md](docs/security-design.md) — thiết kế bảo mật (PSK-first).
- [docs/testing-plan.md](docs/testing-plan.md) — kế hoạch kiểm thử Core/Conditional.
- [docs/implementation-notes.md](docs/implementation-notes.md) — phát hiện GUI và hạn chế.
- [docs/device-selection-decision.md](docs/device-selection-decision.md) — quyết định lựa chọn controller/AP và lý do đổi `WLC-3504` → `WLC-PT`.
- [docs/team-contributions.md](docs/team-contributions.md) — phân công nhóm.
- [tests/test-cases.md](tests/test-cases.md) — danh sách ca kiểm thử.
