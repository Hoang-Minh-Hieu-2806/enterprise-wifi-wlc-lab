# Packet Tracer

Thư mục chứa file mô phỏng Cisco Packet Tracer (`.pkt`) của dự án, thực hiện trên đúng bản **9.0.1.0858**.

## Hiện trạng

> **Deliverable cuối cùng: `final.pkt`** (2026-11-07, 100.297 bytes, SHA-256 `5AA9ADCA…B5CEF`) — topology **enterprise 18 thiết bị / 21 link**:
>
> - **ISP** (Router-PT, Lo0 `8.8.8.8/32`) ↔ **R1** + **R2** (2901, edge + PAT overload, hai path `/30` `203.0.113.0/30` và `203.0.113.4/30`);
> - **R1/R2 ↔ CORE-SW** (3560-24PS, `ip routing`, SVI VLAN 10/20/99, DHCP relay, ACL Guest, default AD 1 + backup AD 254) qua `/30` `10.0.0.0/30` và `10.0.1.0/30`;
> - **CORE-SW ↔ ACCESS-SW** (2960, trunk Fa0/1) ↔ **WLC1** (WLC-3504, mgmt `192.168.99.10`, dynamic interfaces staff/guest) + **LAP-STAFF**, **LAP-GUEST** (LAP-PT);
> - VLAN 99: **DHCP-SRV** (`.20`, service DHCP — dự phòng; client vô tuyến được cấp bởi WLC, xem [../docs/ip-vlan-plan.md](../docs/ip-vlan-plan.md) §4), **RADIUS-SRV** (`.22`, AAA port 1645, client `WLC1`, users staff1/2/3), **Web-SRV** (`.21`), **ADMIN-PC** (`.99.50`, tĩnh);
> - Client wireless: **S-LAP1/2/3** (`STAFF-WIFI`, WPA2-Enterprise/802.1X) và **G-LAP1/2 + G-PHONE** (`GUEST-WIFI`, WPA2-PSK).
>
> Trạng thái chi tiết + kết quả kiểm thử: [../docs/final-implementation.md](../docs/final-implementation.md). Canvas: [../evidence/Final.jpg](../evidence/Final.jpg).
>
> **Kiểm thử final (ping thật qua bridge, 2026-11-05 → 2026-11-07):** 6 client gateway 4/4; Staff → `8.8.8.8` 4/4; G-LAP1 → `8.8.8.8` 4/4; G-LAP1 → `.10.1` và `.99.22` bị ACL chặn 0/4 (đúng design); Staff leases `.104/.103/.106`, Guest `.142/.144/.141`.
>
> **Bỏ `enterprise-wifi-wlc-lab-v1.pkt` (2026-11-07):** file v1 (9 thiết bị, `WLC-PT`, P1 khung có dây) đã bị xoá khỏi repo trong đợt chốt final — lịch sử giai đoạn v1 nằm trong Git history (`git log -- packet-tracer/`) và các docs được đánh dấu "lịch sử v1". Bản `V5.2.pts` không được theo dõi trong Git (xem `.gitignore`).

## Lưu ý khi mở `final.pkt`

1. **DHCP lease không giữ sau khi reopen** — chạy `ipconfig /renew` trên từng client (đặc tính PT).
2. **Association wireless đôi lúc rớt sau khi thao tác nhiều** (limitation PT 9.0.1.0858, chứng kiến 2026-11-06, `../evidence/Final but error.png`): power-cycle AP rồi client, đặt lại DHCP (`0.0.0.0` → dhcp), đợi 10–20 s, `/renew`.
3. **`show ip interface` của PT không hiển thị ACL named** trên `Vlan20` — xem Running Config hoặc `configs/switch/core-sw-running-config.txt`.
4. **Bảo mật:** PSK/secret lưu dạng rõ trong XML `.pkt` — không chia sẻ file ra ngoài nhóm; ảnh chụp phải che passphrase.

## Quy định khi bổ sung file `.pkt`

1. Chỉ commit file `.pkt` đã mở và kiểm thử được trên đúng bản 9.0.1.0858 (xem [../docs/implementation-notes.md](../docs/implementation-notes.md)).
2. Tên file rõ ràng, tiếng Anh — bản deliverable final là `final.pkt`; hạn chế nhiều bản nháp (`test.pkt*` bị chặn trong `.gitignore`).
3. Mọi thay đổi topology phải đồng bộ với [../docs/topology.md](../docs/topology.md), [../docs/final-implementation.md](../docs/final-implementation.md) và địa chỉ chuẩn tại [../docs/ip-vlan-plan.md](../docs/ip-vlan-plan.md), sau khi cả 3 thành viên thống nhất.
4. Không commit file tạm (`*.pkttmp`, `*.bak.pkt`) — đã loại trừ trong `.gitignore`.
5. Cập nhật trạng thái kiểm thử vào [../tests/test-cases.md](../tests/test-cases.md) kèm minh chứng thật.

## Cách mở

Xem hướng dẫn tại [../README.md](../README.md#7-cách-mở-finalpkt).
