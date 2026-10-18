# Thiết kế, Mô phỏng và Đánh giá Bảo mật Mạng Wi-Fi Doanh nghiệp sử dụng Cisco Packet Tracer

> **Tên repository (tiếng Anh):** `enterprise-wifi-wlc-lab`
> **Tiêu đề dự án (tiếng Anh):** Design, Simulation, and Security Evaluation of an Enterprise Wi-Fi Network Using Cisco Packet Tracer

Đồ án môn Quản trị mạng: mô phỏng mạng Wi-Fi doanh nghiệp trên Cisco Packet Tracer, với quản lý tập trung bằng Wireless LAN Controller (WLC), nhiều SSID, phân đoạn VLAN, xác thực truy cập vô tuyến và kiểm soát truy cập giữa các phân đoạn mạng.

## 1. Mục tiêu dự án

1. Xây dựng mạng WLAN doanh nghiệp mô phỏng với quản lý tập trung bằng WLC (thiết bị WLC đã xác nhận có trong Packet Tracer 9.0.1.0858, nhưng việc join AP và phát WLAN vẫn chưa được kiểm chứng).
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
| Thiết bị mạng (đã chốt model) | Router `2911` (subinterface 802.1Q, DHCP relay, ACL), Switch Layer 2 `2960-24TT` (VLAN, trunk tới WLC), WLC `WLC-3504`, LAP-PT (AP chính), Server-PT (DHCP, kiểm tra thêm RADIUS), Wireless Clients (≥ 4: 2 Staff + 2 Guest) |
| Thiết bị dự phòng | AP-PT-A ở chế độ standalone, chỉ dùng khi chứng minh quản lý tập trung bằng WLC là bất khả thi và phạm vi dự án được sửa đổi rõ ràng |
| Dịch vụ mạng | DHCP trên Server-PT (ưu tiên xác minh trước); RADIUS là mục Conditional, thử sau |
| Ngôn ngữ tài liệu | Tiếng Việt (báo cáo, tài liệu, test case, hướng dẫn nhóm) |
| Ngôn ngữ kỹ thuật | Tiếng Anh (tên thư mục, SSID, VLAN, IP, lệnh, commit message) |

Phiên bản Packet Tracer, model thiết bị quan sát được và các chức năng chưa kiểm chứng được ghi tại [docs/implementation-notes.md](docs/implementation-notes.md).

## 3. Thành phần mạng dự kiến

- 01 Router (định tuyến liên VLAN, DHCP relay **bắt buộc** cho VLAN 10/20 vì Server-PT ở VLAN 99, ACL).
- 01 Switch Layer 2 (VLAN; cổng Switch↔WLC **bắt buộc trunk** để mang VLAN 99+10+20, các cổng còn lại chốt theo model thực tế).
- 01 WLC quản lý tập trung AP và WLAN/SSID (giao diện `Wireless LANs` đã được nhìn thấy trong GUI, chưa kiểm chứng tạo WLAN thành công).
- 01 LAP-PT ở giai đoạn đầu (mở rộng lên 2–3 AP sau khi AP đầu join thành công); AP-PT-A chỉ là dự phòng standalone.
- 01 Server-PT cung cấp DHCP; khả năng RADIUS sẽ kiểm tra riêng.
- Ít nhất 4 wireless client: 2 Staff + 2 Guest.

Xem chi tiết tại [docs/topology.md](docs/topology.md). Ánh xạ WLAN→VLAN và cơ chế join AP→WLC chưa được kiểm chứng. Cổng Switch↔WLC **bắt buộc trunk** (nếu Packet Tracer không hỗ trợ thì phải sửa scope, xem `docs/topology.md` §3); các đoạn còn lại chốt theo model thực tế.

## 4. Phân tách SSID và VLAN (đề xuất)

| SSID (dự kiến) | VLAN (dự kiến) | Tên VLAN | Subnet dự kiến | Đối tượng |
| --- | --- | --- | --- | --- |
| `STAFF-WIFI` | 10 | STAFF | 192.168.10.0/24 | Nhân viên nội bộ |
| `GUEST-WIFI` | 20 | GUEST | 192.168.20.0/24 | Khách |
| — (quản trị, có dây) | 99 | MANAGEMENT | 192.168.99.0/24 | Quản trị thiết bị mạng |

> Ánh xạ SSID→VLAN và phần vô tuyến vẫn là **đề xuất, chưa kiểm chứng**. Riêng **hạ tầng có dây và địa chỉ quản trị đã cấu hình và kiểm chứng**: gateway `.1` mỗi VLAN, WLC `.10`, switch SVI `.2`, server `.20` — xem [docs/ip-vlan-plan.md](docs/ip-vlan-plan.md) (tài liệu chuẩn cho mọi địa chỉ) và minh chứng TC-01.

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
│   └── tc-01-wlc-reachability.png
└── report/
    ├── README.md
    └── midterm-report.md
```

- `docs/` — tài liệu thiết kế, kế hoạch và ghi chú triển khai (tiếng Việt).
- `packet-tracer/` — file mô phỏng `.pkt`; `enterprise-wifi-wlc-lab-v1.pkt` là khung có dây P1.
- `configs/` — bản sao cấu hình thiết bị đã che secret; `router/` và `switch/` có bản khung P1, `wlc-ap/` ghi cấu hình hạ tầng WLC (chưa có WLAN/SSID).
- `tests/` — kế hoạch và ca kiểm thử (Core / Conditional / Research-only).
- `evidence/` — ảnh chụp màn hình, kết quả kiểm thử; có minh chứng bring-up P1 và minh chứng TC-01 `Passed`.
- `report/` — dàn ý và nội dung báo cáo trong giai đoạn hiện tại (chưa cần DOCX/PDF).

## 7. Cách mở dự án Packet Tracer

> File `.pkt` đã có: `packet-tracer/enterprise-wifi-wlc-lab-v1.pkt` — khung có dây P1, **chưa có AP và client vô tuyến**.

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

Repository hiện ở giai đoạn **đã dựng khung có dây (P1) và kiểm chứng WLC management reachability**: model thiết bị đã chốt, khung Router–Switch–Server-PT–WLC đã dựng và cấu hình xong, **TC-01 đã `Passed`**, nhưng **chưa có phần vô tuyến (LAP-PT, WLAN, client)**.

| Nhóm trạng thái | Ý nghĩa | Trạng thái hiện tại |
| --- | --- | --- |
| Planned | Đã đề xuất trong tài liệu, chưa cấu hình | Phần vô tuyến: LAP-PT, WLAN/SSID, WPA2, ACL |
| In Progress | Đang thực hiện | Hạ tầng có dây + TC-01 đã xong; chờ LAP-PT và client vô tuyến |
| Not Tested | Chưa kiểm thử, chưa có minh chứng | TC-02 → TC-14 trong [tests/test-cases.md](tests/test-cases.md) |
| Passed / Failed / Blocked | Chỉ dùng khi có evidence thực tế | **TC-01 `Passed`** (2026-10-18); chưa có `Failed`/`Blocked` |

Nhãn `Passed` chỉ được dùng khi có minh chứng hỗ trợ. Một tính năng Conditional (RADIUS) chưa được hỗ trợ **không** làm mất trạng thái hoàn thành của luồng Core.

## 9. Hạn chế đã biết và nội dung cần kiểm chứng

Đã kiểm chứng trong **khung có dây P1**:

- [x] Model Router `2911`, Switch `2960-24TT`, WLC `WLC-3504`, Server-PT — chốt bằng cách đặt thật vào bản 9.0.1.0858.
- [x] VLAN 10/20/99 trên switch; trunk `Fa0/1` (WLC) và `Gi0/1` (router) native VLAN 99 + tagged 10,20; access `Fa0/2` VLAN 99.
- [x] Liên VLAN trên router (`Gi0/0.10`, `Gi0/0.20`) kèm `ip helper-address 192.168.99.20`.
- [x] Server-PT tĩnh `192.168.99.20` + 2 pool DHCP (STAFF, GUEST), Service On.
- [x] **WLC management reachability (TC-01 `Passed`, 2026-10-18):** Management IPv4 `192.168.99.10/24` trên `Vlan1`, ping từ `SRV1` đạt `0% loss`; uplink WLC `GigabitEthernet1` chuyển sang trunk (native VLAN 1) — minh chứng `evidence/tc-01-wlc-reachability.png`.

Đã quan sát trong GUI nhưng **chưa kiểm chứng hoạt động**:

- [ ] LAP-PT join/register với WLC qua trường `Primary Controller`.
- [ ] Tạo và phát WLAN (`STAFF-WIFI`, `GUEST-WIFI`) trên WLC.
- [ ] Nhiều WLAN chạy đồng thời.
- [ ] Ánh xạ mỗi SSID tới đúng VLAN (trường VLAN trong GUI mới chỉ được nhìn thấy).
- [ ] WPA2-PSK: client associate thành công/thất bại.
- [ ] WPA2-Enterprise/RADIUS vận hành (Conditional, thử sau cùng).
- [ ] Trunk Switch↔WLC mang được **lưu lượng client**: VLAN 99 đã kiểm chứng (TC-01), nhưng VLAN 10/20 chưa có client thật và chưa khai báo được danh sách VLAN cho phép trên trunk phía WLC.
- [ ] Guest isolation end-to-end sau ACL.
- [ ] DHCP cấp phát thật cho client mọi VLAN (pool đã tạo, chưa có client).
- [ ] LAP nhận IP hợp lệ.

Chủ đề Research-only (WPA3, EAP-TLS/PEAP, roaming, nhiễu RF, đo throughput): **chưa khẳng định**, chỉ làm nền báo cáo.

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
- [docs/team-contributions.md](docs/team-contributions.md) — phân công nhóm.
- [tests/test-cases.md](tests/test-cases.md) — danh sách ca kiểm thử.
