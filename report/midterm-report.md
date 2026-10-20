# Báo cáo giữa kỳ

**Đề tài:** Thiết kế, Mô phỏng và Đánh giá Bảo mật Mạng Wi-Fi Doanh nghiệp sử dụng Cisco Packet Tracer

| Hạng mục | Nội dung |
| --- | --- |
| Người tổng hợp | Hoàng Minh Hiếu (nhóm trưởng) |
| Thành viên | Đặng Văn Hoàng, Huỳnh Như Hoàng |
| Ngày báo cáo | Thứ Năm, 22/10/2026 |
| Môi trường mô phỏng | Cisco Packet Tracer 9.0.1.0858 |

> **Tính trung thực của báo cáo:** đây là báo cáo **giữa kỳ**, trình bày **ý tưởng, công việc đã làm và công việc dự định**. Model thiết bị đã **đối chiếu trực tiếp** trong Packet Tracer 9.0.1.0858 và khung có dây P1 đã dựng được (§3.2). **TC-01 → TC-08 đã `Passed`** kèm minh chứng; các test case còn lại vẫn `Not Tested`. Phần tạo + phát sóng WLAN đã kiểm chứng cho cả hai SSID (TC-04, TC-05); associate WPA2-PSK của nhóm Staff (`STAFF1`, `STAFF2`) và Guest (`GUEST1`, `GUEST2`) đã được kiểm chứng ở mức trạng thái client (TC-06 + TC-07 — chưa có minh chứng bắt tay ở mức gói tin); DHCP/VLAN mapping đã `Passed` cho tiêu chí IP đúng subnet (TC-08, 2026-10-20, kèm giới hạn); cấu hình đã được lưu vào official `.pkt` ngày 2026-10-21 và kiểm chứng lại sau khi mở file. ACL vẫn là **thiết kế đề xuất, chưa kiểm chứng**. Không có kết quả nào được báo cáo là đã chạy thành công khi chưa có minh chứng.
>
> **Đính chính sau audit lựa chọn thiết bị (2026-10-18):** bản soạn trước đây ghi WLC là `WLC-3504` và cho rằng model này có trang `Wireless LANs`. Kiểm chứng lại giao diện cho thấy `WLC-3504` **không có** trang cấu hình vô tuyến; các trường đó thuộc về `WLC-PT`. Nhóm đã đổi controller sang `WLC-PT` và **đã chạy lại TC-01 (`Passed`)**. Chi tiết: [../docs/device-selection-decision.md](../docs/device-selection-decision.md).
>
> **Cập nhật 2026-10-19 — TC-03 `Passed`:** `LAP-PT` join `WLC-PT` đã được kiểm chứng — `WLC2 → Config → AP Groups` hiển thị `LAP1` (MAC `0001.968E.6B01`) `Status: Online` trong nhóm `default-group` (cơ chế auto-discovery của PT; minh chứng `../evidence/tc-03-lap-wlc-join.png`). Đây là **cập nhật tiến độ sau ngày soạn thảo**; phần thân báo cáo bên dưới vẫn giữ nguyên nội dung giữa kỳ, chỉ các ô tiến độ được chú thích thêm. Bước kế tiếp: TC-04 (tạo/phát WLAN `STAFF-WIFI`). **Bổ sung 2026-10-19: TC-04 đã `Passed` — WLAN `STAFF-WIFI` (VLAN 10, WPA2-PSK) tạo + phát sóng, quét thấy từ client `STAFF1` (minh chứng `../evidence/tc-04-staff-wlan.png`, `../evidence/tc-04-wlc-staff-wlan-config.png`). Bổ sung 2026-10-19: TC-05 đã `Passed` — WLAN `GUEST-WIFI` (VLAN 20, WPA2-PSK) tạo độc lập + phát sóng **đồng thời** hai SSID, quét thấy từ client `GUEST1` (minh chứng `../evidence/tc-05-guest-wlan.png`, `../evidence/tc-05-wlc-guest-wlan-config.png`); bước kế tiếp là TC-06 (associate Staff). Bổ sung 2026-10-19: TC-06 đã `Passed` — hai Staff client `STAFF1` + `STAFF2` associate thành công vào `STAFF-WIFI` bằng WPA2-PSK (minh chứng `../evidence/tc-06-staff-associate.png`); bước kế tiếp là TC-07 (associate Guest). Bổ sung 2026-10-19: TC-07 đã `Passed` — hai Guest client `GUEST1` + `GUEST2` đang associate vào `GUEST-WIFI` bằng WPA2-PSK (minh chứng `../evidence/tc-07-guest1-associate-live.png`, `../evidence/tc-07-guest2-associate-live.png`); bước kế tiếp là TC-08 (DHCP/map VLAN). Bổ sung 2026-10-20: TC-08 đã `Passed` cho **tiêu chí IP đúng subnet, với giới hạn** — `STAFF1` `192.168.10.102/24`, `GUEST1` `192.168.20.102/24`, MAC-table đúng VLAN 10/20 trên `SW1 Fa0/3`; chạy trên **bản test copy** `enterprise-wifi-wlc-lab-v1_TC08_TEST_20261020.pkt`; cấu hình đã được lưu vào official `.pkt` ngày 2026-10-21, mở lại và `/renew` cho client (minh chứng `../evidence/tc-08-*.png`); bước kế tiếp là TC-09 (gateway).**

---

## 1. Tóm tắt ý tưởng và công việc dự định thực hiện

### 1.1 Ý tưởng

Doanh nghiệp vừa và nhỏ cần một mạng Wi-Fi **quản lý tập trung**, **tách biệt nhân viên và khách**, và **kiểm soát truy cập** giữa các vùng mạng. Thử nghiệm trên thiết bị thật tốn kém và rủi ro.

Đồ án mô phỏng một mạng WLAN doanh nghiệp thu nhỏ trên **Cisco Packet Tracer 9.0.1.0858**, trong đó:

- Nhiều Access Point được quản lý tập trung bởi một **Wireless LAN Controller (WLC)** thay vì cấu hình từng AP.
- Mỗi nhóm người dùng có **SSID và VLAN riêng**, đi kèm subnet và gateway riêng.
- Truy cập vô tuyến được **xác thực bằng WPA2-PSK** trước; RADIUS/WPA2-Enterprise là mục có điều kiện.
- Truy cập giữa các VLAN bị **kiểm soát bằng ACL** trên router.
- Mọi kết luận về bảo mật chỉ được đưa ra **sau khi có minh chứng kiểm thử**, không suy đoán từ giao diện.

### 1.2 Phạm vi dự án

| Nhóm | Nội dung | Vai trò |
| --- | --- | --- |
| **Core** | WLC + LAP-PT + 2 WLAN + WPA2-PSK + VLAN/SSID + IP/gateway + DHCP + ACL cô lập Guest | Bắt buộc, quyết định đạt/không đạt |
| **Conditional** | WPA2-Enterprise + RADIUS cho mạng Staff | Cộng thêm; `Blocked` được chấp nhận kèm lý do |
| **Research-only** | WPA3, EAP-TLS/PEAP, roaming, nhiễu RF, đo throughput | Chỉ làm nền báo cáo, không tính vào tiêu chí hoàn thành |

### 1.3 Công việc dự định thực hiện

| Giai đoạn | Nội dung công việc | Phụ trách | Đầu ra |
| --- | --- | --- | --- |
| P1 | Chốt model thiết bị; dựng topology có dây (Router–Switch–Server-PT–WLC) | Đặng Văn Hoàng dựng; Huỳnh Như Hoàng cấu hình VLAN/trunk; Hiếu review | `.pkt` khung + `show vlan brief` |
| P2 | WLC management reachability; LAP-PT nhận IP và join WLC | Đặng Văn Hoàng | **TC-01 `Passed`, TC-02 `Passed`, TC-03 `Passed`** (TC-03 bổ sung 2026-10-19) |
| P2b | **Đổi controller sang `WLC-PT` và chạy lại TC-01** (sau audit 2026-10-18) | Đặng Văn Hoàng, Hiếu hỗ trợ | `.pkt` mới + minh chứng TC-01 mới |
| P3 | Tạo và phát 2 WLAN; client associate bằng WPA2-PSK | Đặng Văn Hoàng | TC-04 → TC-07 |
| P4 | Cấu hình DHCP scope trên Server-PT + `ip helper-address` trên router | Huỳnh Như Hoàng | TC-08, TC-09 |
| P5 | ACL cô lập Guest khỏi Staff và thiết bị quản trị | Huỳnh Như Hoàng | TC-10, TC-11, TC-12 |
| P6 | Kiểm thử sai credential bị từ chối | Huỳnh Như Hoàng + Đặng Văn Hoàng | TC-14 (phần PSK) |
| P7 | Khảo sát RADIUS/WPA2-Enterprise (chỉ sau khi Core ổn định) | Đặng Văn Hoàng | TC-13 (Conditional) |
| P8 | Thu thập minh chứng, viết báo cáo cuối, chuẩn bị thuyết trình | Hiếu điều phối, cả nhóm thực hiện | `evidence/`, `report/` |

**Nguyên tắc thứ tự:** đi tuần tự P1 → P8. Chỉ mở rộng lên AP thứ 2–3 **sau khi** AP đầu join thành công và client kết nối được. Không nhảy sang RADIUS khi luồng cơ bản còn lỗi.

### 1.4 Rủi ro đã nhận diện

| Rủi ro | Ứng phó |
| --- | --- |
| **Cơ chế join LAP-PT → WLC chưa rõ** (chưa tìm thấy trường `Primary Controller`; `WLC-PT` không có trang danh sách AP) **(đã giải quyết 2026-10-19 — TC-03 `Passed` qua auto-discovery)** | Chạy TC-03 ngay sau khi đổi controller; nếu không đạt thì ghi `Blocked`, dừng trước TC-04, chuyển AP standalone kèm sửa phạm vi rõ ràng |
| `WLC-PT` không có tuỳ chọn VLAN/trunk trên `GigabitEthernet0`; phải cấu hình trunk phía switch | **Management đã kiểm chứng (TC-01)**. **TC-08:** client VLAN 10/20 đi qua trunk **uplink AP `Fa0/3`** (local switching); `Fa0/4` (WLC) chỉ management; **central switching chưa kiểm chứng**; cấu hình đã lưu vào official `.pkt` 2026-10-21 |
| Server-PT không hỗ trợ RADIUS | Ghi `Blocked` cho TC-13, luồng Core vẫn nguyên vẹn |
| WLAN tạo được nhưng không phát / không map đúng VLAN | Kiểm tra lại cơ chế join và chế độ cổng, không giả định |
| **Ghi nhầm giao diện giữa các model WLC** (đã xảy ra: `WLC-3504` vs `WLC-PT`) | Luôn kiểm chứng trực tiếp trên từng model; lưu minh chứng chụp màn hình kèm quyết định |

---

## 2. Các tính năng chính của sản phẩm

| # | Tính năng | Mô tả | Trạng thái |
| --- | --- | --- | --- |
| 1 | Quản lý tập trung bằng WLC | WLC quản lý LAP-PT và phân phối cấu hình WLAN xuất ra AP thay vì cấu hình từng AP | **Đã kiểm chứng:** join AP→WLC (TC-03), tạo + phát sóng hai WLAN (TC-04/TC-05) |
| 2 | Nhiều SSID đồng thời | `STAFF-WIFI` và `GUEST-WIFI` phát sóng song song, phục vụ hai nhóm đối tượng | **Đã kiểm chứng (TC-05):** quét thấy đồng thời cả hai SSID |
| 3 | Phân đoạn VLAN theo SSID | `STAFF-WIFI` → VLAN 10; `GUEST-WIFI` → VLAN 20; hạ tầng → VLAN 99 | **Đã kiểm chứng (TC-08):** client nhận IP đúng subnet, MAC-table đúng VLAN trên uplink AP `Fa0/3` (local switching); **central switching của WLC chưa kiểm chứng**; cấu hình đã lưu vào official `.pkt` 2026-10-21 |
| 4 | Xác thực WPA2-PSK | Cả hai SSID dùng WPA2-PSK; sai key bị từ chối | **Associate đúng PSK đã kiểm chứng (TC-06/TC-07); PSK sai (TC-14) chưa kiểm chứng** |
| 5 | Xác thực WPA2-Enterprise + RADIUS | Mạng Staff xác thực theo từng tài khoản qua RADIUS | Tuỳ chọn **đã quan sát**; Conditional, chưa kiểm chứng |
| 6 | DHCP tập trung cho mọi VLAN | Server-PT cấp IP cho client Staff/Guest qua DHCP relay | **TC-08 `Passed` cho tiêu chí IP đúng subnet** (2026-10-20, kèm giới hạn); cấu hình đã lưu vào official `.pkt` 2026-10-21; lease cần `/renew` sau khi mở lại |
| 7 | Cô lập Guest bằng ACL | Guest không tới được Staff và các thiết bị quản trị, nhưng vẫn nhận được DHCP | Dự định |
| 8 | Bảo vệ mạng quản trị | Thiết bị hạ tầng nằm riêng ở VLAN 99, ưu tiên địa chỉ tĩnh | Hạ tầng **đã cấu hình** (TC-01/TC-02, join TC-03); chưa kiểm thử truy cập chéo |
| 9 | Kiểm thử và minh chứng đầy đủ | 14 test case Core/Conditional, mỗi kết quả gắn với file minh chứng | **TC-01 → TC-08 `Passed`**; 6 case còn lại `Not Tested` (cập nhật 2026-10-21; TC-08 kèm giới hạn, cấu hình đã lưu vào official `.pkt`) |

**Điểm nhấn kỹ thuật:** tính năng số 7 là điểm thú vị nhất — Guest bị chặn khỏi VLAN 99 nhưng **vẫn phải nhận được IP** từ server nằm trong VLAN 99. Điều này buộc ACL phải có ngoại lệ DHCP có chủ đích, chứ không chặn cả subnet.

---

## 3. Mô hình triển khai

### 3.1 Sơ đồ logic

```mermaid
flowchart TB
    subgraph Clients["Wireless Clients"]
        S1["Staff 1<br/>STAFF-WIFI / VLAN 10"]
        S2["Staff 2<br/>STAFF-WIFI / VLAN 10"]
        G1["Guest 1<br/>GUEST-WIFI / VLAN 20"]
        G2["Guest 2<br/>GUEST-WIFI / VLAN 20"]
    end

    LAP["LAP-PT<br/>Lightweight AP"]

    subgraph Wired["Hạ tầng có dây"]
        WLC["WLC<br/>Quản lý LAP + WLAN<br/>VLAN 99"]
        SW["Switch L2<br/>VLAN 10, 20, 99"]
        RTR["Router<br/>Gateway + DHCP relay + ACL"]
        SRV["Server-PT<br/>DHCP + RADIUS (thử sau)<br/>VLAN 99"]
    end

    S1 -. WPA2-PSK .-> LAP
    S2 -. WPA2-PSK .-> LAP
    G1 -. WPA2-PSK .-> LAP
    G2 -. WPA2-PSK .-> LAP

    LAP --- SW
    SW ---|trunk VLAN 99 + 10 + 20| WLC
    SW --- SRV
    SW --- RTR
```

### 3.2 Thiết bị đã chốt

| Vai trò | Số lượng | Model | Trạng thái chốt |
| --- | --- | --- | --- |
| Router | 01 | `2911` | **Đã chốt** |
| Switch Layer 2 | 01 | `2960-24TT` | **Đã chốt** |
| WLC | 01 | **`WLC-PT`** (đổi từ `WLC-3504` sau audit 2026-10-18) | **Đã chốt** — có trang `Wireless LANs` (SSID, VLAN, WPA2-PSK, WPA2 + RADIUS) đã quan sát trực tiếp |
| AP chính | 01 | `LAP-PT` (mở rộng +1–2 sau) | **Đã chốt** |
| AP dự phòng | — | AP standalone (không có trường VLAN, chỉ dùng khi WLC bất khả thi) | Dự phòng |
| Server | 01 | `Server-PT` (DHCP; kiểm tra RADIUS sau) | **Đã chốt** |
| Wireless client | ≥ 4 | 2 Staff + 2 Guest | **Đã chốt:** `STAFF1`, `STAFF2`, `GUEST1`, `GUEST2`; associate đã kiểm chứng (TC-06/TC-07) |

> Model đã **đối chiếu trực tiếp** trong Packet Tracer 9.0.1.0858 và dựng khung có dây — không giả định trước. Đợt audit 2026-10-18 đính chính một nhận định sai: trang `Wireless LANs` thuộc `WLC-PT`, **không** thuộc `WLC-3504`. Chi tiết danh mục, tên cổng thật, hạn chế đã quan sát và minh chứng tại [../docs/implementation-notes.md](../docs/implementation-notes.md) và [../docs/device-selection-decision.md](../docs/device-selection-decision.md).

### 3.3 Kế hoạch VLAN và địa chỉ IP

| VLAN | Name | Subnet | Gateway | Phục vụ |
| --- | --- | --- | --- | --- |
| 10 | STAFF | `192.168.10.0/24` | `192.168.10.1` | Nhân viên (`STAFF-WIFI`) |
| 20 | GUEST | `192.168.20.0/24` | `192.168.20.1` | Khách (`GUEST-WIFI`) |
| 99 | MANAGEMENT | `192.168.99.0/24` | `192.168.99.1` | WLC, switch, router, Server-PT, LAP-PT |

Địa chỉ hạ tầng: switch SVI `192.168.99.2` và Server-PT `192.168.99.20` đã cấu hình trong khung có dây P1; WLC `192.168.99.10` đã đặt và **kiểm chứng ping được** (TC-01, chạy lại trên `WLC-PT`) — trên cổng GUI `Management` (API `Vlan1`).

### 3.4 Hai điểm kỹ thuật quyết định sự thành bại

1. **Phải xác định đúng đường dữ liệu client (central switching hay local switching).** `WLC-PT` có tuỳ chọn central switching trong GUI, nhưng TC-08 trên bản test copy cho thấy **MAC client VLAN 10/20 học trên `SW1 Fa0/3` (uplink AP)**, còn uplink WLC `Fa0/4` chỉ có MAC management VLAN 99 ⇒ đường đang chạy là **local switching tại AP**, không phải central switching. Để phân đoạn hoạt động, `Fa0/3` được đặt **trunk native 99, allowed 10,20,99**; **central switching của WLC vẫn chưa kiểm chứng**. Trunk management Switch↔WLC (`Fa0/4`) đã kiểm chứng ở TC-01; cấu hình TC-08 đã lưu vào official `.pkt` ngày 2026-10-21.
2. **DHCP relay là bắt buộc, không phải tùy chọn.** Server-PT nằm ở VLAN 99 nhưng cấp IP cho VLAN 10 và 20, nên router phải có `ip helper-address 192.168.99.20` trên cả hai subinterface client. Thiếu relay ⇒ client không có IP ⇒ toàn bộ các bước kiểm thử sau không chạy được.
3. **Phải kiểm chứng giao diện trên đúng model trước khi dựa vào nó.** Audit 2026-10-18 đính chính một nhận định sai về `WLC-3504`; từ đó nhóm giữ quy tắc chụp minh chứng màn hình kèm mọi quyết định thiết bị.

---

## 4. Các kịch bản demo (ý tưởng)

### 4.1 Kịch bản chính — "Một ngày tại doanh nghiệp"

**Demo 1 — Nhân viên vào mạng và làm việc**
Kết nối Staff client vào `STAFF-WIFI` bằng WPA2-PSK → nhận IP `192.168.10.x` → `ping` gateway `192.168.10.1` thành công.
*Chứng minh:* WLC + LAP-PT + WLAN + WPA2-PSK + VLAN 10 + DHCP hoạt động.

**Demo 2 — Khách vào mạng**
Kết nối Guest client vào `GUEST-WIFI` → nhận IP `192.168.20.x` → phân biệt rõ với dải IP của Staff.
*Chứng minh:* hai SSID hoạt động đồng thời và ánh xạ đúng VLAN.

**Demo 3 — Khách bị cô lập (điểm nhấn)**
Từ Guest `ping` tới Staff và tới WLC → **thất bại**. Sau đó chạy `show access-lists` trên router để cho thấy luật chặn.
*Chứng minh:* kiểm soát truy cập liên VLAN có hiệu lực thực tế, không chỉ là cấu hình.

**Demo 4 — Ngoại lệ DHCP có chủ đích**
Trong khi Demo 3 cho thấy Guest bị chặn khỏi VLAN 99, Guest **vẫn nhận được IP** từ Server-PT ở VLAN 99. Giải thích: ACL có permit DHCP riêng, chặn cả subnet sẽ làm hỏng chính nó.
*Chứng minh:* hiểu sâu về thiết kế ACL theo dịch vụ, không chặn thô theo subnet.

**Demo 5 — Sai mật khẩu bị từ chối**
Nhập PSK sai → client không associate được, không nhận IP nội bộ.
*Chứng minh:* xác thực WPA2-PSK thực sự hoạt động theo cả hai chiều.

### 4.2 Kịch bản mở rộng (nếu kịp)

| Demo | Nội dung | Điều kiện |
| --- | --- | --- |
| Demo 6 | WPA2-Enterprise: mỗi nhân viên một tài khoản qua RADIUS | Conditional, chỉ khi Core đã ổn định |
| Demo 7 | Mở rộng lên 2–3 AP, cùng SSID phủ rộng hơn | Sau khi AP đầu + client chạy ổn định |
| Demo 8 | So sánh trước/sau ACL: cùng một lệnh `ping`, một lần thông một lần chặn | Luôn làm được, rất trực quan |

### 4.3 Phương án dự phòng khi demo trực tiếp gặp sự cố

Nếu file `.pkt` hoặc Packet Tracer gặp lỗi ngay tại buổi báo cáo:

1. Dùng **ảnh minh chứng đã chụp trước** trong `evidence/` và trình bày theo từng bước.
2. Trình bày sơ đồ topology + bảng VLAN/IP trên slide.
3. Nêu rõ **phần nào đã kiểm chứng, phần nào chưa** — báo cáo lỗi trung thực thay vì che.

### 4.4 Phân công trình bày

| Phần | Người trình bày |
| --- | --- |
| Mở đầu, ý tưởng, phạm vi, kết luận | Hoàng Minh Hiếu |
| Mô hình triển khai, WLC/AP, xác thực vô tuyến | Đặng Văn Hoàng |
| VLAN, DHCP, ACL, kịch bản cô lập Guest | Huỳnh Như Hoàng |

---

## 5. Kết luận giữa kỳ

- Đồ án đã hoàn thành phần **thiết kế và kế hoạch**: topology logic, kế hoạch VLAN/IP, thiết kế bảo mật và bộ 14 test case có truy vết tới yêu cầu.
- **Model thiết bị đã chốt** và **khung có dây P1 đã dựng được**: Router `2911` – Switch `2960-24TT` – Server-PT – WLC, gồm VLAN 10/20/99, trunk Switch↔WLC và DHCP relay `ip helper-address` trên router.
- **Đã có 8 kết quả kiểm thử thật kèm minh chứng**: TC-01 `Passed` (management reachability của WLC, `0% loss`), TC-02 `Passed` (LAP-PT có IP quản trị hợp lệ và ping được gateway), **TC-03 `Passed` (LAP-PT join `WLC-PT`, `Status: Online`; cập nhật 2026-10-19)**, **TC-04 `Passed` (WLAN `STAFF-WIFI` tạo + phát sóng; cập nhật 2026-10-19)**, **TC-05 `Passed` (WLAN `GUEST-WIFI` tạo độc lập + phát sóng đồng thời; cập nhật 2026-10-19)**, **TC-06 `Passed` (hai Staff client associate vào `STAFF-WIFI` bằng WPA2-PSK; cập nhật 2026-10-19)**, **TC-07 `Passed` (hai Guest client associate vào `GUEST-WIFI` bằng WPA2-PSK; cập nhật 2026-10-19)** và **TC-08 `Passed` cho tiêu chí IP đúng subnet, kèm giới hạn (cập nhật 2026-10-20; cấu hình đã lưu vào official `.pkt` 2026-10-21)**. 6 test case còn lại vẫn `Not Tested`.
- **Phát hiện quan trọng của đợt audit 2026-10-18:** controller ban đầu `WLC-3504` **không có giao diện cấu hình vô tuyến** trong Packet Tracer 9.0.1.0858; các trường WLAN/SSID/VLAN/WPA2 từng được ghi nhận thực ra thuộc `WLC-PT`. Nhóm đã đổi controller sang `WLC-PT` và **đã chạy lại TC-01 (`Passed`)**. Đây là ví dụ điển hình cho việc không được suy luận khả năng thiết bị từ tài liệu bên ngoài.
- Phần vô tuyến (WLAN/SSID/WPA2, AP join) đã có minh chứng qua TC-03→TC-07; TC-08 xác nhận client nhận IP đúng subnet `10/20` (đường dữ liệu qua uplink AP — local switching), kèm giới hạn; cấu hình đã lưu vào official `.pkt` 2026-10-21; **central switching của WLC chưa kiểm chứng**. ACL và các kiểm thử end-to-end vẫn ở giai đoạn tiếp theo, bắt đầu từ **TC-09 (gateway)**.
- Cam kết của nhóm: mọi kết luận về hiệu quả mã hóa và kiểm soát truy cập chỉ được đưa ra khi có minh chứng kiểm thử đi kèm; **không tính việc thấy trường trong GUI là bằng chứng chức năng chạy được**.

---

## Tài liệu tham chiếu

- [../README.md](../README.md) — tổng quan dự án
- [../docs/topology.md](../docs/topology.md) — topology logic và quyết định cổng
- [../docs/device-selection-decision.md](../docs/device-selection-decision.md) — quyết định lựa chọn controller/AP
- [../docs/ip-vlan-plan.md](../docs/ip-vlan-plan.md) — kế hoạch địa chỉ chuẩn
- [../docs/security-design.md](../docs/security-design.md) — thiết kế bảo mật
- [../docs/testing-plan.md](../docs/testing-plan.md) — kế hoạch kiểm thử
- [../tests/test-cases.md](../tests/test-cases.md) — 14 ca kiểm thử
- [../docs/team-contributions.md](../docs/team-contributions.md) — phân công nhóm
