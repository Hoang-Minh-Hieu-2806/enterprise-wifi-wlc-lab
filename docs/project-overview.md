# Tổng quan dự án

## 1. Phát biểu vấn đề

Các doanh nghiệp vừa và nhỏ cần mạng Wi-Fi tập trung, dễ quản lý, phân tách được lưu lượng nhân viên và khách, đồng thời kiểm soát truy cập cơ bản giữa các vùng mạng. Tuy nhiên, việc thử nghiệm trực tiếp trên thiết bị thật tốn kém và rủi ro. Dự án này sử dụng **Cisco Packet Tracer 9.0.1.0858** để mô phỏng một mạng WLAN doanh nghiệp thu nhỏ, giúp sinh viên luyện tập thiết kế, cấu hình và đánh giá bảo mật trong môi trường an toàn.

Quan sát thực tế trong Packet Tracer 9.0.1.0858 cho thấy `WLC-PT` có trang cấu hình `Wireless LANs` (SSID, trường VLAN, WPA2-PSK, WPA2 + RADIUS) và `LAP-PT` làm AP chính. Đây mới là **phát hiện giao diện, chưa phải bằng chứng các chức năng đã chạy được** — chi tiết tại [implementation-notes.md](implementation-notes.md).

> **Lưu ý sau audit 2026-10-18:** controller ban đầu chọn là `WLC-3504`, nhưng kiểm chứng lại cho thấy model này **không có giao diện cấu hình vô tuyến** trong bản 9.0.1.0858; các trường WLAN/SSID/VLAN/WPA2 từng được ghi nhận thuộc về `WLC-PT`. Nhóm đã đổi controller sang `WLC-PT` — xem [device-selection-decision.md](device-selection-decision.md).

## 2. Phạm vi dự án

**Trong phạm vi (luồng Core, phải hoàn thành):**

- Thiết kế topology logic gồm router, switch Layer 2, 01 WLC, bắt đầu 01 LAP-PT, Server-PT và ≥ 4 wireless client (2 Staff + 2 Guest).
- Phân đoạn VLAN cho staff / guest / management (kế hoạch chuẩn tại [ip-vlan-plan.md](ip-vlan-plan.md)).
- Tạo và phát hai WLAN/SSID (`STAFF-WIFI`, `GUEST-WIFI`), kiểm thử độc lập từng SSID.
- Triển khai và kiểm thử **WPA2-PSK** trước cho cả hai SSID.
- Kiểm soát truy cập liên VLAN bằng ACL trên router (explicit deny + permit tối thiểu).
- Kiểm thử kết nối (IP, gateway), kiểm thử bảo mật cơ bản (cô lập Guest, sai credential bị từ chối), thu thập minh chứng, viết báo cáo.

**Có điều kiện (Conditional, thử sau khi luồng Core ổn định):**

- WPA2-Enterprise + RADIUS cho Staff, chỉ khi Server-PT và model WLC đã chọn thực sự hỗ trợ. Tính năng này không quyết định trạng thái hoàn thành của toàn bộ đồ án.

**Ngoài phạm vi / chỉ nghiên cứu (Research-only):**

- WPA3, EAP cụ thể (EAP-TLS, PEAP, ...), roaming liền mạch, phân tích nhiễu RF, đo throughput — chỉ làm nền báo cáo.
- Triển khai thực tế trên thiết bị vật lý.
- Phát triển phần mềm, API, web app, CI/CD hay hạ tầng không liên quan.

AP standalone được giữ làm dự phòng, chỉ dùng khi chứng minh quản lý tập trung bằng WLC là bất khả thi và phạm vi được sửa đổi rõ ràng. Lưu ý AP standalone trong Packet Tracer **không có trường VLAN**, nên không thể thay WLC trong luồng chính (xem [device-selection-decision.md](device-selection-decision.md) §2).

## 3. Mục tiêu

Xem chi tiết tại [../README.md](../README.md#1-mục-tiêu-dự-án) và bảng yêu cầu tại [requirements.md](requirements.md).

Tóm tắt: quản lý tập trung bằng WLC (01 LAP-PT trước), hai SSID, phân đoạn VLAN, WPA2-PSK trước, RADIUS có điều kiện, ACL cô lập Guest, kiểm thử theo nhóm Core/Conditional và tài liệu hóa đầy đủ.

## 4. Sản phẩm bàn giao dự kiến (Deliverables)

| # | Sản phẩm | Vị trí | Trạng thái hiện tại |
| --- | --- | --- | --- |
| 1 | Tài liệu thiết kế (overview, requirements, topology, IP/VLAN, security) | `docs/` | In Progress (finalize, chưa nghiệm thu) |
| 2 | File mô phỏng Packet Tracer (`.pkt`, đúng bản 9.0.1.0858) | `packet-tracer/` | In Progress — `enterprise-wifi-wlc-lab-v1.pkt` là khung có dây P1 + `WLC-PT` (đã thay `WLC-3504`) + LAP-PT + hai WLAN `STAFF-WIFI`/`GUEST-WIFI` + client pilot `STAFF1`, `STAFF2` (Staff) và `GUEST1` (Guest); phiên WIP TC-07 có thêm `GUEST2` nhưng chưa lưu vào file deliverable |
| 3 | Bản sao cấu hình thiết bị (đã che secret) | `configs/` | In Progress — `router/` + `switch/` có bản khung P1; `wlc-ap/` ghi cấu hình hạ tầng `WLC-PT` (đã cập nhật sau khi chạy lại TC-01) |
| 4 | Kế hoạch và ca kiểm thử Core/Conditional | `docs/testing-plan.md`, `tests/test-cases.md` | In Progress — **TC-01 → TC-08 `Passed`**; TC-09→TC-14 `Not Tested`. **TC-01 đã chạy lại và `Passed` trên `WLC-PT`** (bản test copy); **TC-03 xác nhận `LAP-PT` join `WLC-PT` (`Status: Online`)**; **TC-04 xác nhận `STAFF-WIFI` tạo + phát sóng (quét thấy từ `STAFF1`)**; **TC-05 xác nhận `GUEST-WIFI` tạo độc lập + phát sóng đồng thời (quét thấy cả hai SSID từ `GUEST1`)**; **TC-06 xác nhận hai Staff client associate vào `STAFF-WIFI` bằng WPA2-PSK (minh chứng `../evidence/tc-06-staff-associate.png`)**; **TC-07 xác nhận hai Guest client associate vào `GUEST-WIFI` bằng WPA2-PSK (minh chứng `../evidence/tc-07-guest1-associate-live.png`, `../evidence/tc-07-guest2-associate-live.png`)**; **TC-08 `Passed` cho tiêu chí IP đúng subnet, kèm giới hạn (2026-10-20); cấu hình đã lưu vào official `.pkt` 2026-10-21** |
| 5 | Minh chứng (screenshots, kết quả lệnh) | `evidence/` | In Progress — minh chứng bring-up P1 + TC-01 → TC-08 + **minh chứng audit lựa chọn thiết bị (một số file `audit-*.png` chưa có, cần chụp bổ sung)** |
| 6 | Báo cáo (dàn ý + nội dung trong `report/`, chưa cần DOCX/PDF) | `report/` | In Progress — `midterm-report.md` đã soạn |

## 5. Tiêu chí hoàn thành dự án

Luồng Core (bắt buộc):

1. Có file `.pkt` mở được trên Packet Tracer 9.0.1.0858, đúng topology đã chốt (bắt đầu 01 LAP-PT).
2. WLC có thể ping/tới được (management reachability); LAP-PT nhận IP hợp lệ và join WLC.
3. Hai WLAN `STAFF-WIFI`, `GUEST-WIFI` được tạo, phát sóng và client associate được bằng WPA2-PSK.
4. Client nhận đúng IP theo VLAN/SSID và ping được gateway tương ứng.
5. Sai credential bị từ chối ở chế độ WPA2-PSK.
6. Guest không truy cập được Staff và Management sau ACL; Staff chỉ giữ đúng truy cập nội bộ cho phép.
7. Mọi test Core trong [../tests/test-cases.md](../tests/test-cases.md) có kết quả thực tế và minh chứng trong `evidence/`.
8. Báo cáo trong `report/` mô tả trung thực điều làm được và điều chưa kiểm chứng.

Luồng Conditional (không bắt buộc để đạt): RADIUS/WPA2-Enterprise cho Staff, ghi `Passed`/`Failed`/`Blocked` kèm evidence; nếu `Blocked` thì luồng Core vẫn được công nhận hoàn thành.

Không có khẳng định nào về WPA3, EAP, roaming, throughput khi chưa có minh chứng.

## 6. Rủi ro chính

- **Cổng chặn — LAP-PT join WLC:** **đã giải quyết (TC-03 `Passed`, 2026-10-19 — `LAP1` `Status: Online` trong `AP Groups`, qua auto-discovery của PT).** Rủi ro còn lại liên quan: `WLC-PT` không có trang danh sách AP riêng và PT không có API trả trạng thái AP, nên trạng thái join chỉ quan sát được qua `AP Groups`/XML. Không có minh chứng bắt tay CAPWAP.
- **`WLC-PT` không có tuỳ chọn VLAN/trunk trên `GigabitEthernet0`** ⇒ management đã kiểm chứng (TC-01). **TC-08** cho thấy client VLAN 10/20 đi qua **uplink AP `Fa0/3`** (trunk, local switching), còn uplink WLC `Fa0/4` chỉ có MAC management; **central switching chưa kiểm chứng**. Cấu hình TC-08 **đã lưu vào official `.pkt` 2026-10-21** và mở lại kiểm chứng (client phải `/renew` để nhận lease); nếu official không tái hiện được đường **local switching qua uplink AP** ⇒ ghi `Blocked` và chỉ chuyển AP standalone kèm sửa scope rõ ràng. Nếu chỉ hạng mục **central switching** không đạt, ghi `Blocked` riêng cho hạng mục đó, không hạ yêu cầu phân đoạn VLAN (xem [topology.md](topology.md) §3.1).
- WLAN tạo được trong GUI nhưng không phát / không map đúng VLAN → kiểm tra lại cơ chế join và chế độ cổng.
- Server-PT không hỗ trợ RADIUS → ghi `Blocked` cho TC RADIUS, giữ nguyên kết quả Core.
- Cấu hình ACL hoặc DHCP không như kỳ vọng → ghi lại workaround, không ngụy tạo kết quả. Riêng thiếu `ip helper-address` là **lỗi cấu hình**, không phải limitation của simulator.
- **Chênh lệch model nếu đổi Packet Tracer phiên bản khác 9.0.1.0858** → toàn bộ quan sát giao diện và danh sách cổng phải kiểm chứng lại; đặc biệt `WLC-3504` và `WLC-PT` dùng chung device type 41 nên dễ nhầm.
