# Topology logic (mức triển khai được)

> Topology dưới đây là **thiết kế đã chốt model, đang kiểm chứng dần hoạt động vô tuyến**. Model thiết bị đã chốt, khung có dây và LAP-PT đã dựng trong Packet Tracer 9.0.1.0858 (xem [implementation-notes.md](implementation-notes.md) §1–§2). Cơ chế join AP→WLC **đã kiểm chứng (TC-03 `Passed`, 2026-10-19)** và **cả hai WLAN đã tạo + phát sóng đồng thời (TC-04, TC-05 `Passed`, 2026-10-19: `STAFF-WIFI` + `GUEST-WIFI`)**; ánh xạ WLAN→VLAN cho client đã kiểm chứng trong **TC-08** qua uplink AP (local switching); **central switching của WLC chưa kiểm chứng**; cấu hình TC-08 đã lưu vào official `.pkt` 2026-10-21.
>
> **Cập nhật 2026-10-18 — đổi controller sang `WLC-PT`.** `WLC-3504` không có giao diện cấu hình vô tuyến trong bản 9.0.1.0858 nên không thể làm controller; lý do và minh chứng tại [device-selection-decision.md](device-selection-decision.md). **TC-01 đã chạy lại và `Passed` trên `WLC-PT`** (bản test copy): địa chỉ quản trị `192.168.99.10` đặt trên cổng GUI `Management` (API port `Vlan1`). **Chế độ cổng Switch↔WLC đã chạy được cho management** (`SW1 Fa0/4` trunk native 99); **lưu lượng client VLAN 10/20 đã kiểm chứng trong TC-08 qua uplink AP `Fa0/3`** (local switching; uplink WLC `Fa0/4` chỉ management; **central switching chưa kiểm chứng**; cấu hình đã lưu vào official `.pkt` 2026-10-21). **`.pkt` deliverable đã thay sang `WLC-PT`** (2026-10-19). **Cập nhật 2026-10-19 — TC-03 `Passed`:** `LAP1` `Status: Online` trong `AP Groups` của `WLC-PT` (auto-discovery). **TC-04 `Passed` (2026-10-19):** WLAN `STAFF-WIFI` (WLAN ID 1, VLAN 10, WPA2-PSK/AES) đã tạo trên `WLC2` và **phát sóng — quét thấy từ client `STAFF1`**. **TC-05 `Passed` (2026-10-19):** WLAN `GUEST-WIFI` (WLAN ID 2, VLAN 20, WPA2-PSK/AES) tạo độc lập, **phát sóng đồng thời** cùng `STAFF-WIFI` — quét từ client mới `GUEST1` thấy **cả hai SSID**; lưu + mở lại `.pkt` vẫn giữ nguyên. **Cập nhật 2026-10-20 — TC-08 `Passed`:** `STAFF1` nhận `192.168.10.102/24`, `GUEST1` nhận `192.168.20.102/24` trên test copy, `SW1 Fa0/3` trunk native 99 mang VLAN 10/20; MAC-table học đúng VLAN 10/20 trên `Fa0/3`. **2026-10-21:** cấu hình được lưu vào official `.pkt`, mở lại và `/renew` cho `STAFF1` `.10.101/24`, `GUEST1` `.20.100/24`.

## 1. Thành phần và model đã chốt

| Thiết bị | Model đã chốt | Số lượng triển khai | Vai trò | Tình trạng xác minh |
| --- | --- | --- | --- | --- |
| Router | `2911` | 01 | Định tuyến liên VLAN (subinterface 802.1Q), **DHCP relay bắt buộc** (§3), ACL cô lập Guest | Model đã chốt; subinterface + relay đã dựng; **ACL chưa kiểm chứng** |
| Switch Layer 2 | `2960-24TT` | 01 | VLAN 10, 20, 99; cổng Switch↔WLC, các cổng còn lại chốt theo model | Model đã chốt; VLAN/trunk đã dựng và đọc lại được |
| WLC | **`WLC-PT`** | 01 | Quản lý tập trung LAP và WLAN/SSID; có trang `Wireless LANs` (SSID, VLAN, WPA2-PSK, WPA2 + RADIUS) | Cổng thật: `Vlan1` (mang IP quản trị; GUI hiển thị `Management`) + `GigabitEthernet0` (uplink). **TC-01 chạy lại `Passed`** (`../evidence/tc-01-wlc-pt-reachability-canvas.png`); **WLAN `STAFF-WIFI` (VLAN 10) và `GUEST-WIFI` (VLAN 20) đã tạo và phát sóng đồng thời — TC-04 + TC-05 `Passed` (2026-10-19)**. **`WLC-PT` là controller trong `.pkt` deliverable** |
| LAP-PT (Lightweight AP) | `LAP-PT` | 01 ở giai đoạn đầu | AP chính, join về WLC; phát cả hai SSID nếu model cho phép | Đã đấu vào `SW1 Fa0/3` (ban đầu access VLAN 99; **từ TC-08 dùng trunk** native 99) và **có IP `192.168.99.30`** (TC-02 `Passed`); cổng thật là `GigabitEthernet0` + `Dot11Radio0` (catalog ghi `Port 0` là sai); **join `WLC-PT` đã kiểm chứng — `LAP1` `Status: Online`** (TC-03 `Passed`, 2026-10-19); **phát đồng thời `STAFF-WIFI` + `GUEST-WIFI` — quét thấy từ `STAFF1` (TC-04) và `GUEST1` (TC-05)** |
| AP mở rộng | cùng `LAP-PT` | +1–2 sau | Chỉ thêm sau khi AP đầu join thành công và client kết nối được | Planned |
| AP dự phòng | `AccessPoint-PT` standalone | Dự phòng | Standalone, chỉ dùng khi chứng minh WLC tập trung bất khả thi + sửa scope | Chưa dùng; quan sát GUI tại `../evidence/audit-ap-pt-port1-security.png` |
| Server-PT | `Server-PT` | 01 | DHCP (đã tạo pool); kiểm tra thêm khả năng RADIUS | DHCP server chạy; **cấp phát thật đã kiểm chứng trong TC-08** (`STAFF1` `.10.102`, `GUEST1` `.20.102` trên test copy; official `/renew` `.10.101`, `.20.100`; kèm giới hạn); cấu hình đã lưu vào official `.pkt` 2026-10-21 |
| Wireless clients | `Laptop-PT` (pilot) | ≥ 4 | 2 Staff + 2 Guest để kiểm thử phân tách và gateway độc lập | Pilot `STAFF1` (NIC vô tuyến) quét thấy `STAFF-WIFI` — TC-04 `Passed`; client Guest `GUEST1` (NIC vô tuyến, thêm 2026-10-19) quét thấy đồng thời hai SSID — TC-05 `Passed`; model cuối chưa chốt |

Các model khác trong palette có khả năng khác nhau — không giả định tương đương. Model cuối cùng đã chốt tại [implementation-notes.md](implementation-notes.md) §2.

## 2. Sơ đồ logic (Mermaid)

```mermaid
flowchart TB
    subgraph Clients["Wireless Clients (triển khai: 2 Staff + 2 Guest)"]
        STAFF1["Staff Client 1<br/>SSID: STAFF-WIFI / VLAN 10"]
        STAFF2["Staff Client 2<br/>SSID: STAFF-WIFI / VLAN 10"]
        GUEST1["Guest Client 1<br/>SSID: GUEST-WIFI / VLAN 20"]
        GUEST2["Guest Client 2<br/>SSID: GUEST-WIFI / VLAN 20"]
    end

    subgraph Wireless["Hạ tầng vô tuyến"]
        LAP1["LAP-PT-1 (AP chính,<br/>triển khai đầu tiên)"]
        LAP2["LAP-PT-2 (mở rộng sau<br/>khi LAP-1 chạy ổn định)"]
        WLC["WLC-PT<br/>Quản lý LAP + WLAN<br/>VLAN 99 Management"]
        APTA["AP standalone (dự phòng,<br/>chỉ dùng khi WLC bất khả thi)"]
    end

    subgraph Wired["Hạ tầng có dây"]
        SW["Switch L2<br/>VLAN 10, 20, 99"]
        RTR["Router<br/>Gateway VLAN + ACL"]
        SRV["Server-PT<br/>DHCP (xác minh trước),<br/>RADIUS (kiểm tra sau)"]
    end

    STAFF1 -. vô tuyến (WPA2-PSK trước) .-> LAP1
    STAFF2 -. vô tuyến (WPA2-PSK trước) .-> LAP1
    GUEST1 -. vô tuyến (WPA2-PSK) .-> LAP1
    GUEST2 -. vô tuyến (WPA2-PSK) .-> LAP1
    LAP2 -. mở rộng sau .-> SW

    LAP1 --- SW
    WLC --- SW
    SRV --- SW
    SW --- RTR
```

> Đường nét đứt là liên kết vô tuyến; đường nét liền là liên kết có dây (dự kiến cáp Copper Straight-Through trong Packet Tracer, loại cáp chốt khi triển khai). AP standalone không nối trong sơ đồ chính vì là dự phòng.

## 3. Kết nối logic và quyết định cần chốt theo model

| Kết nối | Ý nghĩa logic | Quyết định chốt khi triển khai |
| --- | --- | --- |
| Router ↔ Switch | Liên VLAN, gateway `.1` mỗi VLAN | **Đã chốt:** router-on-a-stick trên `2911` — `Gi0/0` = `192.168.99.1/24` (VLAN 99 native) + `Gi0/0.10` (dot1Q 10) + `Gi0/0.20` (dot1Q 20) |
| Switch ↔ WLC | Quản trị WLC (VLAN 99); đường client central (chưa kiểm chứng) | **Management đã chốt trên `WLC-PT`** — `SW1 Fa0/4` trunk native VLAN 99, allowed 10,20,99; `WLC-PT Gi0` uplink (GUI không có tuỳ chọn trunk/VLAN); frame untagged của `Vlan1` rơi vào native 99 ⇒ management ping được (TC-01 `Passed`). **TC-08 xác nhận client VLAN 10/20 đi qua uplink AP `Fa0/3`; `Fa0/4` (WLC) chỉ học MAC management VLAN 99** ⇒ **central switching của WLC chưa kiểm chứng**; cấu hình đã lưu vào official `.pkt` 2026-10-21 |
| Switch ↔ LAP-PT | Uplink AP (quản trị VLAN 99 + tải SSID/VLAN nếu hỗ trợ) | **Ban đầu access VLAN 99 trên `FastEthernet0/3`** cho quản trị (TC-02 `Passed`). **Từ TC-08, `Fa0/3` được đặt trunk** native `99`, allowed `10,20,99`; client MAC VLAN 10/20 học trên `Fa0/3` ⇒ VLAN đi qua uplink AP (**local switching**); cấu hình đã lưu vào official `.pkt` 2026-10-21 |
| Switch ↔ Server-PT | Server tới đúng VLAN phục vụ DHCP (và RADIUS nếu thử) | **Đã chốt:** access VLAN 99 trên `FastEthernet0/2` (theo [ip-vlan-plan.md](ip-vlan-plan.md) §2) |
| LAP-PT ↔ Client | Vô tuyến, SSID ánh xạ tới VLAN | `STAFF-WIFI` → 10: trường VLAN = 10 và **SSID đã phát sóng — quét thấy từ `STAFF1`** (TC-04 `Passed`, 2026-10-19); `GUEST-WIFI` → 20: **SSID đã phát sóng đồng thời — quét thấy cả hai từ `GUEST1`** (TC-05 `Passed`); **associate Staff/Guest bằng WPA2-PSK đã kiểm chứng** (TC-06 + TC-07 `Passed`); **map VLAN đã kiểm chứng trong TC-08** — `STAFF1` `.10.102/24`, `GUEST1` `.20.102/24` trên test copy; official `/renew` `.10.101/24`, `.20.100/24`; MAC đúng VLAN 10/20 **trên uplink AP `Fa0/3`** (local switching); **central switching của WLC chưa kiểm chứng**; cấu hình đã lưu vào official `.pkt` 2026-10-21 |

Chi tiết subnet và gateway chuẩn xem tại [ip-vlan-plan.md](ip-vlan-plan.md) (authoritative).

### 3.1 Đường dữ liệu client: local switching tại AP (TC-08) và central switching chưa kiểm chứng

- `WLC-PT` có tuỳ chọn **central switching** trong trang `Wireless LANs` (xem [implementation-notes.md](implementation-notes.md) §2 và [device-selection-decision.md](device-selection-decision.md)), nhưng **TC-08 chưa kiểm chứng đường central**: trong test copy và official sau khi mở lại, MAC client VLAN 10/20 nằm trên uplink AP `Fa0/3`, không có trên uplink WLC `Fa0/4` ⇒ quan sát phù hợp với **local switching tại AP**.
- `STAFF-WIFI` → VLAN 10 và `GUEST-WIFI` → VLAN 20. Với **local switching**, hai VLAN này đi qua trunk uplink AP `Fa0/3`; nếu chọn **central switching**, cổng WLC `Fa0/4` cũng phải mang đồng thời VLAN 99 (quản trị) + VLAN 10 + VLAN 20.
- Nếu chọn central switching mà để cổng WLC là **access VLAN 99**, lưu lượng Staff/Guest sẽ bị dồn vào VLAN 99. TC-08 hiện đạt qua **trunk uplink AP**, nên **không** chứng minh yêu cầu central; TC-12 vẫn cần kiểm chứng phân đoạn đầy đủ.

**Đã kiểm chứng trên model cũ `WLC-3504` (TC-01, 2026-10-18):** cổng `WLC1 GigabitEthernet1` mặc định là **access VLAN 1** và khi để nguyên thì trunk SW1 drop toàn bộ (không học được MAC nào trên `Fa0/1`). Chuyển nó sang **trunk** thì thông. Phía WLC phải giữ **native VLAN 1** (không phải 99): SVI `Vlan1` của WLC chỉ hoạt động trong VLAN 1 nội bộ, nên frame untagged của nó rơi vào native VLAN 99 của switch.

**Đã kiểm chứng trên `WLC-PT` (model mới, bản test copy):** cổng `GigabitEthernet0` **không có tuỳ chọn VLAN/trunk trong GUI**, nên **không cần** set trunk phía WLC; chỉ cần đặt trunk ở **phía `SW1 Fa0/4`** (native 99, allowed 10,20,99) là management (VLAN 99) thông (TC-01 `Passed`). Câu `addTrunkVlans` (lỗi trên `WLC-3504`) **không áp dụng được** và không cần. **Đã trả lời một phần ở TC-08:** lưu lượng client VLAN 10/20 ra **uplink AP `Fa0/3`** hoạt động — `STAFF1`/`GUEST1` nhận đúng subnet và MAC-table học VLAN 10/20 trên `Fa0/3`; **không** thấy MAC client trên uplink WLC `Fa0/4`, nên **central switching vẫn chưa được kiểm chứng**. **Official `.pkt` đã được cập nhật cấu hình TC-08 ngày 2026-10-21**; sau khi mở lại phải `/renew` cho client.

**Nhánh dự phòng cho central switching:** nếu muốn chứng minh central mà `WLC-PT` không thể mang VLAN 10/20 ra uplink, ghi `Blocked` cho hạng mục central và giữ nguyên kết quả local switching đã có; chỉ chuyển sang AP standalone kèm sửa phạm vi dự án rõ ràng nếu ngay cả đường local cũng không tái hiện được (xem [project-overview.md](project-overview.md) §2). Không tự ý hạ yêu cầu phân đoạn VLAN.

### 3.2 Vì sao DHCP relay bắt buộc

Server-PT ở VLAN 99 nhưng cấp DHCP cho VLAN 10 và VLAN 20 ⇒ router phải có `ip helper-address 192.168.99.20` trên cả hai subinterface client. Chi tiết tại [ip-vlan-plan.md](ip-vlan-plan.md) §3.1. Thiếu relay ⇒ client không có IP ⇒ TC-08 trở đi đều không kiểm thử được.

## 4. Thứ tự bring-up (bắt buộc)

1. **(xong trên model cũ)** Dựng có dây: Router `2911` – Switch `2960-24TT` – Server-PT – WLC `WLC-3504`; trunk cho cổng Switch↔WLC. Đặt IP quản trị cho WLC và kiểm tra management reachability — **TC-01 `Passed`** ngày 2026-10-18 **trên `WLC-3504`**.
2. **(xong — đã thay trong `.pkt` deliverable)** Thay controller bằng **`WLC-PT`** theo [device-selection-decision.md](device-selection-decision.md); đặt Management IPv4 `192.168.99.10/24` trên cổng GUI **`Management`** (API port `Vlan1`) bằng API (`pt_host_ip_config`); nối `WLC2 Gi0` → `SW1 Fa0/4` (trunk native 99, allowed 10,20,99); **TC-01 chạy lại `Passed`** (`../evidence/tc-01-wlc-pt-reachability-canvas.png`). Cấu hình `WLC-3504` giữ lại như lịch sử P1. **Đã thay `WLC-3504` bằng `WLC-PT` trong `.pkt` deliverable** (2026-10-19).
3. **(xong cấu hình)** Cấu hình DHCP scope trên Server-PT + `ip helper-address` trên router (§3.2). Chưa cần client, nhưng phải xong trước TC-08.
4. **(xong một phần)** Nối 01 LAP-PT, kiểm tra LAP nhận IP — **TC-02 `Passed`** ngày 2026-10-18 (`192.168.99.30` tĩnh trên `GigabitEthernet0`, `SW1 Fa0/3` access VLAN 99).
5. **(xong)** LAP-PT join `WLC-PT` (TC-03) — **`Passed` ngày 2026-10-19**: `LAP1` `Status: Online` trong `AP Groups` (nhóm `default-group`), cơ chế **auto-discovery** của PT (không có trường `Primary Controller`; `CAPWAP.PRIMARY_AC = 0.0.0.0` không đọc là bằng chứng). Minh chứng: `../evidence/tc-03-lap-wlc-join.png`. **Lưu ý:** `LAP-PT` **không lưu IP tĩnh vào `.pkt`** — mở lại file phải đặt lại `192.168.99.30/24` thì AP mới rejoin.
6. **(xong)** Tạo WLAN Staff + Guest (TC-04, TC-05) — **`Passed` 2026-10-19**: `STAFF-WIFI` (WLAN ID 1, VLAN 10) quét thấy từ `STAFF1` (TC-04); `GUEST-WIFI` (WLAN ID 2, VLAN 20) tạo độc lập, phát sóng **đồng thời**, quét thấy cả hai SSID từ `GUEST1` (TC-05). **Associate Staff/Guest bằng WPA2-PSK đã `Passed`** (TC-06 + TC-07); **map SSID→VLAN đã `Passed` cho tiêu chí IP đúng subnet (TC-08)** (2026-10-20, kèm giới hạn; đường dữ liệu qua uplink AP `Fa0/3` — local switching, central switching chưa kiểm chứng; cấu hình đã lưu vào official `.pkt` 2026-10-21).
7. **(một phần)** Kiểm tra IP/gateway — **TC-08 `Passed`** (IP đúng subnet; cấu hình đã lưu vào official `.pkt` 2026-10-21); TC-09 (gateway) chưa chạy. Tiếp theo: ACL cô lập (TC-10/TC-11), cuối cùng đối chiếu VLAN + hiển thị ACL (TC-12).
8. Chỉ sau khi Core ổn định mới thử RADIUS (TC-13, Conditional).
9. Chỉ thêm LAP thứ 2–3 sau khi AP đầu và client chạy ổn định.

## 5. Lưu ý kiểm chứng

- [x] Model router/switch/WLC/Server-PT cụ thể và interface thực dùng.
- [x] Controller đã đánh giá và chốt `WLC-PT` (quan sát GUI trực tiếp, có trang `Wireless LANs`).
- [x] **Chạy lại TC-01 trên `WLC-PT`** (bản test copy) — `Passed`, địa chỉ quản trị trên cổng GUI `Management` (API `Vlan1`).
- [x] **Thay `WLC-3504` bằng `WLC-PT` trong `.pkt` deliverable** (Phase C, xong 2026-10-19).
- [x] Chế độ cổng **Switch↔LAP** mang VLAN 10/20: **TC-08** đặt `Fa0/3` trunk native `99`, allowed `10,20,99`; MAC client VLAN 10/20 học trên `Fa0/3` (`STAFF1` `.10.102/24`, `GUEST1` `.20.102/24` trên test copy; official `/renew` `.10.101/24`, `.20.100/24`).
- [ ] Chế độ cổng **Switch↔`WLC-PT`** mang lưu lượng client VLAN 10/20 / **central switching** — **chưa kiểm chứng**. VLAN 99/management đã `Passed` (TC-01); trong TC-08, `Fa0/4` (WLC) chỉ có MAC management, không có MAC client. Cấu hình đã lưu vào official `.pkt` 2026-10-21.
- [x] Đã cấu hình `ip helper-address` cho VLAN 10/20 trỏ về Server-PT (§3.2).
- [x] Chế độ cổng phía switch/router: `Gi0/0`→`Gi0/1` (router, trunk), `Fa0/4` (WLC, trunk native 99 — management), `Fa0/2` (server, access VLAN 99), `Fa0/3` (LAP-PT, **access VLAN 99** ở TC-02; **trunk native 99** từ TC-08).
- [x] LAP-PT join `WLC-PT` — **`Passed` (TC-03)**: cơ chế là **auto-discovery** của PT (không thấy trường `Primary Controller`); `LAP1` `Status: Online` trong `AP Groups`; minh chứng `../evidence/tc-03-lap-wlc-join.png`.
- [x] AP có hỗ trợ 2 SSID đồng thời và map đúng VLAN không. **Đã kiểm chứng:** cả hai WLAN `STAFF-WIFI` (VLAN 10) + `GUEST-WIFI` (VLAN 20) **phát sóng đồng thời** (TC-04 + TC-05 `Passed`); client nhận đúng subnet và MAC-table đúng VLAN 10/20 trên uplink AP `Fa0/3` (TC-08 `Passed`, với giới hạn); **central switching của WLC chưa kiểm chứng**. Cấu hình đã lưu vào official `.pkt` 2026-10-21.
- [ ] Server-PT có đồng thời DHCP + RADIUS không.
- [ ] Mọi kết nối chỉ được coi là đúng sau khi có file `.pkt` và minh chứng kiểm thử.
