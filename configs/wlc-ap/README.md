# WLC & AP Configs

Lưu bản sao cấu hình/ghi chú cấu hình của WLC và LAP-PT (screenshot GUI WLC, export nếu có, danh sách WLAN/SSID). Người sở hữu: Đặng Văn Hoàng.

## Hiện trạng

> **Hạ tầng WLC đã cấu hình và kiểm chứng — TC-01 `Passed` ngày 2026-10-18.** WLAN/SSID/WPA2 và `Primary Controller` vẫn **chưa làm**; phần đó chỉ thao tác được trên GUI.

### WLC-3504 — hạ tầng (đã cấu hình)

| Hạng mục | Giá trị | Ghi chú |
| --- | --- | --- |
| Model | `WLC-3504` | device type thật đọc từ mô phỏng = **41** |
| Management IPv4 | `192.168.99.10/24` | đặt trên interface **`Vlan1`**, không phải `management` |
| Default gateway | `192.168.99.1` | R1 `Gi0/0` |
| Uplink | `GigabitEthernet1` | **trunk**, native VLAN **1**; nối SW1 `Fa0/1` (native VLAN 99, allowed 10,20,99) |
| MAC `Vlan1` | `00D0.BAD0.7DA9` | chính là MAC SW1 học được trên `Fa0/1` thuộc VLAN 99 |
| MAC `management` | `0003.E479.4739` | interface ảo type 48, **không** mang lưu lượng |
| MAC `virtual` | `0009.7C0C.2770` | IP `0.0.0.0`, mặc định |

**Vì sao native VLAN hai phía khác nhau (1 và 99):** SVI `Vlan1` của WLC chỉ hoạt động trong VLAN 1 nội bộ. Giữ native VLAN 1 phía WLC để frame của `Vlan1` đi **untagged**; SW1 nhận untagged và gán vào native VLAN 99 — Management IPv4 vẫn nằm đúng VLAN 99 theo thiết kế. Đặt native 99 ở cả hai phía đã thử và **thất bại 100%** (WLC không trả lời ARP).

**Đã sửa so với mặc định:** `GigabitEthernet1` mặc định là **access VLAN 1**; nếu để nguyên, frame đi vào VLAN 1 và bị trunk SW1 (allowed 10,20,99) drop — `show mac address-table` không học được MAC nào.

**Chưa giải quyết:** chưa khai báo được danh sách VLAN được phép trên trunk của WLC — `addTrunkVlans('10,20,99')` báo `Invalid arguments for IPC call`. Ảnh hưởng TC-03/TC-08 (lưu lượng WLAN → VLAN 10/20).

Minh chứng: [../../evidence/tc-01-wlc-reachability.png](../../evidence/tc-01-wlc-reachability.png).

## Nội dung dự kiến (làm theo thứ tự)

1. WLC management reachability (TC-01) và LAP-PT join (TC-02, TC-03) — bắt đầu 01 LAP-PT.
2. WLAN `STAFF-WIFI` → VLAN 10 và `GUEST-WIFI` → VLAN 20 với **WPA2-PSK trước** (TC-04→TC-07).
3. RADIUS/WPA2-Enterprise (TC-13, Conditional) chỉ sau khi Core ổn định và khi được hỗ trợ.

AP-PT-A chỉ là dự phòng standalone, không thuộc luồng chính.

## Quy định

- Ghi rõ model WLC/LAP-PT, bản 9.0.1.0858, ngày ghi nhận (xem [../../docs/implementation-notes.md](../../docs/implementation-notes.md)).
- Che PSK và RADIUS secret (`<PSK-REDACTED>`, `<RADIUS-SECRET-REDACTED>`).
- Không khẳng định WLAN phát sóng, map VLAN hay RADIUS chạy được khi chưa có minh chứng tại `../../evidence/`.
- Nếu không export được text, lưu screenshot GUI vào `../../evidence/` và tóm tắt text tại đây.
