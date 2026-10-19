# Packet Tracer

Thư mục chứa file mô phỏng Cisco Packet Tracer (`.pkt`) của dự án, thực hiện trên đúng bản **9.0.1.0858**.

## Hiện trạng

> **Đã có `enterprise-wifi-wlc-lab-v1.pkt`** (cập nhật 2026-10-19) — khung có dây P1 gồm Router `2911` (`R1`), Switch `2960-24TT` (`SW1`), Server-PT (`SRV1`), WLC `WLC-PT` (`WLC2`), LAP-PT (`LAP1`) và ba wireless client `STAFF1`, `STAFF2` + `GUEST1` (`Laptop-PT`) — 9 thiết bị; đã cấu hình VLAN 10/20/99, trunk Switch↔WLC, DHCP relay và **hai WLAN `STAFF-WIFI` (VLAN 10) + `GUEST-WIFI` (VLAN 20), WPA2-PSK — TC-04 + TC-05 + TC-06 `Passed` (TC-06: `STAFF1` + `STAFF2` associate thật vào `STAFF-WIFI`)**. Ảnh canvas kèm tại [../evidence/p1-wired-skeleton-topology.png](../evidence/p1-wired-skeleton-topology.png) (trạng thái P1 ban đầu; chưa gồm client/WLAN).
>
> **TC-01, TC-02, TC-03, TC-04, TC-05 và TC-06 đã `Passed`** (TC-01, TC-02 `2026-10-18`; TC-03 `2026-10-19` — `LAP1` join `WLC2`, `Status: Online`; TC-04 `2026-10-19` — WLAN `STAFF-WIFI` phát sóng, quét thấy từ `STAFF1`, minh chứng [../evidence/tc-04-staff-wlan.png](../evidence/tc-04-staff-wlan.png) + [../evidence/tc-04-wlc-staff-wlan-config.png](../evidence/tc-04-wlc-staff-wlan-config.png) — PSK đã che; TC-05 `2026-10-19` — WLAN `GUEST-WIFI` phát sóng **đồng thời**, quét thấy cả hai SSID từ `GUEST1`, minh chứng [../evidence/tc-05-guest-wlan.png](../evidence/tc-05-guest-wlan.png) + [../evidence/tc-05-wlc-guest-wlan-config.png](../evidence/tc-05-wlc-guest-wlan-config.png) — PSK đã che; TC-06 `2026-10-19` — `STAFF1` + `STAFF2` associate thành công vào `STAFF-WIFI` bằng WPA2-PSK, minh chứng [../evidence/tc-06-staff-associate.png](../evidence/tc-06-staff-associate.png)).
>
> **Lưu ý khi mở file (re-audit 2026-10-19):** `WLC-PT` giữ `192.168.99.10/24` và **hai WLAN `STAFF-WIFI`/`GUEST-WIFI` lưu trong `.pkt`** (mở lại vẫn còn — đã kiểm chứng), nhưng **`LAP-PT` không lưu IP tĩnh** — mở lại file, `LAP1` trở về DHCP và AP hiện `Offline`. Cần **đặt lại tĩnh `192.168.99.30/24` (gw `192.168.99.1`) cho `LAP1`** thì AP mới rejoin (`Status: Online`) và phát lại SSID.
>
> **Đã thay controller:** file nay dùng **`WLC-PT`** (`WLC2`) — model có trang `Wireless LANs` (khác `WLC-3504` cũ, **không có giao diện cấu hình vô tuyến** trong bản 9.0.1.0858). **TC-01 đã chạy lại và `Passed`** (xem [../docs/device-selection-decision.md](../docs/device-selection-decision.md)).
>
> **Đã có 03 wireless client (`STAFF1`, `STAFF2`, `GUEST1`, `Laptop-PT`)** dùng để quét SSID và associate (`GUEST1` phục vụ TC-05; `STAFF2` phục vụ TC-06); các bản `test.pkt` trên máy là bản nháp, đã chặn trong `.gitignore`.
>
> **Lưu ý bảo mật:** PT lưu passphrase WPA2-PSK **dạng rõ** trong XML của `.pkt` (`<WEP_KEY>`) — không chia sẻ file ra ngoài nhóm; khi chụp minh chứng phải che vùng passphrase.

## Quy định khi bổ sung file `.pkt`

1. Chỉ commit file `.pkt` đã mở và kiểm thử được trên đúng bản 9.0.1.0858 (xem [../docs/implementation-notes.md](../docs/implementation-notes.md)).
2. Bắt đầu với **01 LAP-PT**; chỉ thêm AP khi TC-03 và TC-06 đã `Passed` (xem [../tests/test-cases.md](../tests/test-cases.md)).
3. Tên file rõ ràng, tiếng Anh, ví dụ: `enterprise-wifi-wlc-lab-v1.pkt`; hạn chế nhiều bản nháp.
4. Mọi thay đổi topology phải đồng bộ với [../docs/topology.md](../docs/topology.md) và địa chỉ chuẩn tại [../docs/ip-vlan-plan.md](../docs/ip-vlan-plan.md), sau khi cả 3 thành viên thống nhất.
5. Không commit file tạm (`*.pkttmp`, `*.bak.pkt`) — đã loại trừ trong `.gitignore`.

## Cách mở

Xem hướng dẫn tại [../README.md](../README.md#7-cách-mở-dự-án-packet-tracer-khi-đã-có-file-pkt).
