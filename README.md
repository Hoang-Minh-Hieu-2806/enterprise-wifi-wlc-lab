# Thiết kế, Mô phỏng và Đánh giá Bảo mật Mạng Wi-Fi Doanh nghiệp sử dụng Cisco Packet Tracer

> **Tên repository (tiếng Anh):** `enterprise-wifi-wlc-lab`
> **Tiêu đề dự án (tiếng Anh):** Design, Simulation, and Security Evaluation of an Enterprise Wi-Fi Network Using Cisco Packet Tracer

Đồ án môn Quản trị mạng: mô phỏng mạng Wi-Fi doanh nghiệp trên Cisco Packet Tracer, với quản lý tập trung bằng Wireless LAN Controller (WLC), nhiều SSID, phân đoạn VLAN, xác thực truy cập vô tuyến và kiểm soát truy cập giữa các phân đoạn mạng.

## 1. Mục tiêu dự án

1. Xây dựng mạng WLAN doanh nghiệp mô phỏng với quản lý tập trung bằng WLC.
2. Triển khai 2–3 Access Point (số lượng cuối cùng tùy thuộc khả năng hỗ trợ của Packet Tracer).
3. Triển khai nhiều SSID, dự kiến gồm `STAFF-WIFI` và `GUEST-WIFI`.
4. Phân đoạn mạng bằng VLAN cho nhân viên (staff), khách (guest) và quản trị (management).
5. Áp dụng ít nhất một phương thức xác thực vô tuyến khả thi trong môi trường mô phỏng.
6. Khảo sát khả năng xác thực WPA2-Enterprise trên nền RADIUS nếu thiết bị Packet Tracer hỗ trợ.
7. Kiểm soát truy cập giữa các phân đoạn mạng (dự kiến dùng ACL trên router nếu phù hợp).
8. Kiểm thử chức năng, đánh giá bảo mật, thu thập minh chứng và viết tài liệu kỹ thuật.

> Lưu ý: đây là **dự án mô phỏng và tài liệu mạng**, không phải dự án phát triển phần mềm.

## 2. Công nghệ và môi trường mô phỏng

| Hạng mục | Nội dung |
| --- | --- |
| Môi trường mô phỏng chính | Cisco Packet Tracer 9.x |
| Thiết bị mạng (dự kiến) | Router, Switch Layer 2, WLC, AP, Server, Wireless Clients |
| Dịch vụ mạng (dự kiến) | DHCP và/hoặc RADIUS trên server, tùy khả năng hỗ trợ |
| Ngôn ngữ tài liệu | Tiếng Việt (báo cáo, tài liệu, test case, hướng dẫn nhóm) |
| Ngôn ngữ kỹ thuật | Tiếng Anh (tên thư mục, định danh kỹ thuật, commit message) |

Phiên bản Packet Tracer cụ thể và model thiết bị sẽ được chốt trong [docs/implementation-notes.md](docs/implementation-notes.md).

## 3. Thành phần mạng dự kiến

- 01 Router (định tuyến giữa các VLAN, DHCP relay nếu cần, ACL).
- 01 Switch Layer 2 (trunk/access VLAN).
- 01 Wireless LAN Controller (WLC) quản lý tập trung AP và SSID.
- 02–03 Access Point (số lượng cuối cùng phụ thuộc thiết bị Packet Tracer hỗ trợ).
- 01 Server cung cấp DHCP và/hoặc RADIUS (nếu được hỗ trợ).
- Các wireless client đại diện cho nhân viên (staff) và khách (guest).

Xem chi tiết tại [docs/topology.md](docs/topology.md).

## 4. Phân tách SSID và VLAN (dự kiến)

| SSID (dự kiến) | VLAN (dự kiến) | Tên VLAN | Subnet dự kiến | Đối tượng |
| --- | --- | --- | --- | --- |
| `STAFF-WIFI` | 10 | STAFF | 192.168.10.0/24 | Nhân viên nội bộ |
| `GUEST-WIFI` | 20 | GUEST | 192.168.20.0/24 | Khách |
| — (quản trị, có dây) | 99 | MANAGEMENT | 192.168.99.0/24 | Quản trị thiết bị mạng |

> Tất cả giá trị trên là **đề xuất, chưa kiểm chứng**. Kế hoạch địa chỉ chi tiết xem tại [docs/ip-vlan-plan.md](docs/ip-vlan-plan.md).

## 5. Hướng tiếp cận xác thực và kiểm soát truy cập (dự kiến)

- **Mạng Staff:** ưu tiên khảo sát WPA2-Enterprise trên nền RADIUS nếu Packet Tracer hỗ trợ; nếu không khả thi, dùng WPA2-PSK như phương án dự phòng khả thi.
- **Mạng Guest:** dự kiến WPA2-PSK (hoặc Open nếu cần phục vụ mục đích kiểm thử, sẽ ghi rõ rủi ro).
- **Phân tách mạng:** VLAN riêng cho staff / guest / management.
- **Kiểm soát liên VLAN:** dự kiến dùng ACL trên router để chặn Guest truy cập vùng nội bộ, chỉ cho phép ra Internet/mạng ngoài mô phỏng.
- **Bảo vệ mạng quản trị:** giới hạn thiết bị được truy cập WLC/switch/router, ưu tiên dải VLAN 99.

Phân tích chi tiết xem tại [docs/security-design.md](docs/security-design.md).

So sánh WPA2-PSK và WPA2-Enterprise, cùng các chủ đề nghiên cứu (WPA3, EAP, roaming, nhiễu RF, đo throughput) được trình bày trong tài liệu thiết kế bảo mật. **Không khẳng định các tính năng này đã hoạt động khi chưa kiểm chứng.**

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
│   └── README.md
├── configs/
│   ├── README.md
│   ├── router/
│   │   └── README.md
│   ├── switch/
│   │   └── README.md
│   └── wlc-ap/
│       └── README.md
├── tests/
│   ├── README.md
│   └── test-cases.md
├── evidence/
│   └── README.md
└── report/
    └── README.md
```

- `docs/` — tài liệu thiết kế, kế hoạch và ghi chú triển khai.
- `packet-tracer/` — file mô phỏng `.pkt` (sẽ bổ sung sau, hiện chưa có).
- `configs/` — bản sao cấu hình thiết bị (sẽ bổ sung sau khi cấu hình được kiểm chứng).
- `tests/` — kế hoạch và ca kiểm thử.
- `evidence/` — ảnh chụp màn hình, kết quả kiểm thử (sẽ bổ sung sau).
- `report/` — báo cáo tổng hợp (sẽ bổ sung sau).

## 7. Cách mở dự án Packet Tracer (khi đã có file `.pkt`)

> Hiện tại **chưa có file `.pkt`** trong repository. Các bước dưới đây chỉ dùng khi file đã được commit.

1. Cài đặt Cisco Packet Tracer 9.x (phiên bản cụ thể xem tại [docs/implementation-notes.md](docs/implementation-notes.md)).
2. Clone repository:
   ```bash
   git clone <repository-url>
   cd enterprise-wifi-wlc-lab
   ```
3. Mở thư mục `packet-tracer/` và double-click file `.pkt`.
4. Nếu Packet Tracer báo thiếu model thiết bị, đối chiếu với danh sách thiết bị trong [docs/implementation-notes.md](docs/implementation-notes.md) và [docs/topology.md](docs/topology.md).
5. Không chỉnh sửa topology khi chưa thống nhất với nhóm; mọi thay đổi cấu hình cần được lưu sang `configs/` và ghi chú lại.

Quy định đặt file `.pkt` xem tại [packet-tracer/README.md](packet-tracer/README.md).

## 8. Trạng thái dự án hiện tại

Repository hiện ở giai đoạn **khung sườn (skeleton)**: chỉ có cấu trúc thư mục và tài liệu định hướng, chưa có triển khai hay kiểm thử.

| Nhóm trạng thái | Ý nghĩa | Trạng thái hiện tại |
| --- | --- | --- |
| Tính năng dự kiến (Planned) | Đã đề xuất trong tài liệu, chưa cấu hình | Xem [docs/requirements.md](docs/requirements.md) |
| Tính năng đã triển khai (Implemented) | Đã cấu hình trong Packet Tracer | **Chưa có — đang chờ (Pending)** |
| Tính năng đã kiểm thử (Tested & Verified) | Đã kiểm thử và có minh chứng | **Chưa có — đang chờ (Pending)** |

Bảng yêu cầu chi tiết và trạng thái từng yêu cầu xem tại [docs/requirements.md](docs/requirements.md). Kết quả kiểm thử chi tiết xem tại [tests/test-cases.md](tests/test-cases.md) (hiện tất cả đều là `Not tested`).

## 9. Hạn chế đã biết và nội dung cần kiểm chứng

- [ ] Model WLC/AP nào khả dụng trong Packet Tracer 9.x và có hỗ trợ quản lý tập trung không.
- [ ] Số lượng AP tối đa có thể triển khai ổn định (2 hay 3).
- [ ] Packet Tracer có hỗ trợ RADIUS + WPA2-Enterprise trên thiết bị đã chọn không.
- [ ] Phương thức EAP cụ thể nào (nếu có) khả thi — hiện chỉ coi là **chủ đề nghiên cứu**.
- [ ] ACL trên router có chặn đúng lưu lượng Guest → Staff/Management không.
- [ ] DHCP cấp đúng subnet theo SSID/VLAN không.
- [ ] WPA3, roaming, nhiễu RF, đo throughput: **chưa khẳng định**, chỉ nghiên cứu khi có thời gian và khi xác minh được simulator hỗ trợ.

Mọi khẳng định về hiệu quả mã hóa hay kiểm soát truy cập chỉ được ghi nhận sau khi kiểm thử, xem [docs/testing-plan.md](docs/testing-plan.md).

## 10. Thành viên nhóm

Nhóm gồm 3 sinh viên. Phân công dự kiến xem tại [docs/team-contributions.md](docs/team-contributions.md).

| Thành viên | Vai trò dự kiến | GitHub |
| --- | --- | --- |
| Thành viên 1 | Topology, VLAN, địa chỉ IP, cấu hình router & switch | — |
| Thành viên 2 | WLC, AP, SSID, xác thực vô tuyến, khảo sát RADIUS | — |
| Thành viên 3 | Kiểm thử kiểm soát truy cập, minh chứng, tài liệu, trình bày | — |

> Tên thành viên để trống và sẽ bổ sung sau. Không tự ý điền tên khi chưa được xác nhận.

## 11. Tài liệu liên quan

- [docs/project-overview.md](docs/project-overview.md) — tổng quan, phạm vi, deliverables.
- [docs/requirements.md](docs/requirements.md) — bảng yêu cầu và trạng thái.
- [docs/topology.md](docs/topology.md) — topology logic dự kiến.
- [docs/ip-vlan-plan.md](docs/ip-vlan-plan.md) — kế hoạch địa chỉ đề xuất.
- [docs/security-design.md](docs/security-design.md) — thiết kế bảo mật dự kiến.
- [docs/testing-plan.md](docs/testing-plan.md) — kế hoạch kiểm thử.
- [docs/implementation-notes.md](docs/implementation-notes.md) — ghi chú triển khai.
- [docs/team-contributions.md](docs/team-contributions.md) — phân công nhóm.
- [tests/test-cases.md](tests/test-cases.md) — danh sách ca kiểm thử.
