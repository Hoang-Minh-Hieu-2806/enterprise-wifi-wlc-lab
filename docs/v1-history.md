# Lịch sử giai đoạn v1 (trước 2026-11-01)

> Tài liệu này gom toàn bộ trạng thái của **phiên v1** (`packet-tracer/enterprise-wifi-wlc-lab-v1.pkt`, 9 thiết bị, controller `WLC-PT`) — đã được thay thế hoàn toàn bởi bản final: [final-implementation.md](final-implementation.md). File `v1.pkt` và ảnh minh chứng v1 (`tc-01-*` → `tc-08-*`, `p1-*`, `audit-*`) đã bị xoá khỏi working tree ngày 2026-11-07 (vẫn xem được trong Git history: `git log --all -- evidence/ packet-tracer/`). Các tài liệu `[topology.md](topology.md)`, `[ip-vlan-plan.md](ip-vlan-plan.md)`, `[security-design.md](security-design.md)`, `[device-selection-decision.md](device-selection-decision.md)` hiện mô tả bản final — nội dung v1 gốc của chúng nằm rải rác trong Git history.

## 1. Kiến trúc v1 (khung P1)

- 9 thiết bị trong `v1.pkt`: `R1` (2911, router-on-a-stick trên `Gi0/0.10/.20` + `ip helper-address`) · `SW1` (2960-24TT) · `SRV1` (Server-PT DHCP `192.168.99.20`) · `WLC2` (`WLC-PT`, mgmt `192.168.99.10` trên interface `Vlan1`) · `LAP1` (`LAP-PT`, quản trị VLAN 99) · clients `STAFF1`, `STAFF2`, `GUEST1` (+ `GUEST2` trong phiên WIP TC-07) — tất cả `Laptop-PT`.
- VLAN 10/20/99 (cùng subnet với final); trunk `SW1 Fa0/3` (LAP) và `Fa0/4` (WLC), native 99.
- Đường dữ liệu client là **local switching tại AP** qua uplink `Fa0/3`; **central switching của WLC chưa kiểm chứng**.

## 2. Kết quả kiểm thử v1 (mốc thời gian)

| TC | Nội dung | Kết quả | Ngày |
| --- | --- | --- | --- |
| TC-01 | WLC management reachability (ping `192.168.99.10`) | **Passed** — chạy lại trên `WLC-PT` sau khi đổi controller | 2026-10-18 |
| TC-02 | LAP-PT nhận IP quản trị (`192.168.99.30/24` tĩnh runtime) | **Passed** | 2026-10-18 |
| TC-03 | LAP-PT join `WLC-PT` (`AP Groups`, `Status: Online`, auto-discovery) | **Passed** | 2026-10-19 |
| TC-04 | WLAN `STAFF-WIFI` (ID 1, VLAN 10, WPA2-PSK/AES) tạo + phát sóng | **Passed** | 2026-10-19 |
| TC-05 | WLAN `GUEST-WIFI` (ID 2, VLAN 20) tạo + phát **đồng thời** hai SSID | **Passed** | 2026-10-19 |
| TC-06 | 2 Staff client associate `STAFF-WIFI` bằng WPA2-PSK | **Passed** | 2026-10-19 |
| TC-07 | 2 Guest client associate `GUEST-WIFI` bằng WPA2-PSK | **Passed** (phiên WIP có `GUEST2`; không lưu vào deliverable) | 2026-10-19 |
| TC-08 | Client nhận DHCP đúng subnet (test copy `.10.102`/`.20.102`; official `/renew` `.10.101`/`.20.100`) | **Passed** kèm giới hạn (central switching, `serverPool`, lease `/renew`) — cấu hình lưu vào `v1.pkt` | 2026-10-20 → lưu 2026-10-21 |

TC-09 → TC-14 ở giai đoạn v1 = **Not Tested** (được hoàn thiện trên bản final — xem [final-implementation.md](final-implementation.md) §7).

## 3. Quyết định controller giai đoạn v1

- Audit 2026-10-18 phát hiện **tab Config** của `WLC-3504` còn thiếu trang lớp vô tuyến (chỉ `GLOBAL/Settings` + `INTERFACE`), nên đổi `WLC-3504` → `WLC-PT` để tạo WLAN. Decision record đầy đủ: [device-selection-decision.md](device-selection-decision.md) (nay có banner SUPERSEDED vì final dùng lại `WLC-3504` qua **web GUI riêng** của controller).

## 4. Lưu ý vận hành v1 (được kế thừa/thay đổi ở final)

- `LAP-PT` không lưu IP tĩnh + trạng thái join vào `.pkt` — mở lại phải đặt lại IP (`v1`); final dùng DHCP cho AP qua WLC.
- DHCP lease không giữ sau reopen — cần `/renew` (còn đúng trên final).
- `serverPool` factory của Server-PT không xoá được vĩnh viễn, giới hạn `.99.30–.30` (còn đúng trên final).
- PSK lưu dạng rõ trong XML `.pkt` (field `<WEP_KEY>`) — che khi chụp, không commit (còn đúng trên final).
