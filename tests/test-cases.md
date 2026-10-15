# Danh sách ca kiểm thử (Test Cases)

> Toàn bộ test case ở trạng thái **Not Tested**. Cột `Actual result` để trống cho tới khi có kiểm thử thật trên Packet Tracer 9.0.1.0858. Kế hoạch tổng thể và phân loại Core/Conditional xem tại [../docs/testing-plan.md](../docs/testing-plan.md). RADIUS (TC-13) là Conditional: `Blocked` ở đó không làm mất trạng thái hoàn thành của luồng Core.

## TC-01 — WLC management reachability (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-01 |
| **Preconditions** | File `.pkt` mở được trên 9.0.1.0858; WLC đã đấu nối có dây và đặt Management IPv4 theo [../docs/ip-vlan-plan.md](../docs/ip-vlan-plan.md) |
| **Procedure** | 1. Từ một thiết bị cùng/thông được VLAN 99 `ping` tới Management IPv4 của WLC (ví dụ `192.168.99.10`). 2. Ghi model WLC và địa chỉ đã dùng. |
| **Expected result** | Ping tới WLC thành công (reply) |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-01-wlc-reachability.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-02 — LAP-PT nhận IP quản trị hợp lệ (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-02 |
| **Preconditions** | TC-01 đạt hoặc hạ tầng VLAN 99 thông; 01 LAP-PT đã nối dây tới switch |
| **Procedure** | 1. Kiểm tra địa chỉ IP của LAP-PT-1 (DHCP hoặc tĩnh trong `192.168.99.0/24`). 2. Ghi model LAP-PT và cách cấp IP. |
| **Expected result** | LAP-PT có IP hợp lệ thuộc `192.168.99.0/24` và ping được gateway `192.168.99.1` |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-02-lap-ip.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-03 — LAP-PT join WLC (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-03 |
| **Preconditions** | TC-02 đạt; trường `Primary Controller` trên LAP-PT đã trỏ về WLC |
| **Procedure** | 1. Mở GUI/CLI WLC, kiểm tra danh sách AP đã join. 2. Ghi model LAP-PT và trạng thái join. Chỉ thêm AP thứ 2 sau khi TC này và TC-06 đã `Passed` (điều kiện mở rộng AP tại [../docs/testing-plan.md](../docs/testing-plan.md)). |
| **Expected result** | LAP-PT-1 hiển thị Registered/Joined trên WLC |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-03-lap-join.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-04 — WLAN STAFF-WIFI được tạo và phát sóng (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-04 |
| **Preconditions** | TC-03 đạt; WLAN `STAFF-WIFI` đã cấu hình trên trang `Wireless LANs` của WLC |
| **Procedure** | 1. Từ Staff client quét mạng vô tuyến tại vùng phủ LAP-PT-1. 2. Ghi SSID, WLAN ID và VLAN đã cấu hình. |
| **Expected result** | Nhìn thấy `STAFF-WIFI` và associate thử được ở bước TC-06 |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-04-staff-wlan.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-05 — WLAN GUEST-WIFI được tạo và phát sóng (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-05 |
| **Preconditions** | TC-03 đạt; WLAN `GUEST-WIFI` đã cấu hình độc lập với `STAFF-WIFI` |
| **Procedure** | 1. Từ Guest client quét mạng vô tuyến tại vùng phủ LAP-PT-1. 2. Xác nhận hai WLAN tồn tại đồng thời (nếu model cho phép). |
| **Expected result** | Nhìn thấy `GUEST-WIFI` đồng thời với `STAFF-WIFI` (hoặc ghi rõ giới hạn model nếu không) |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-05-guest-wlan.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-06 — Staff associate bằng WPA2-PSK (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-06 |
| **Preconditions** | TC-04 đạt; `STAFF-WIFI` dùng WPA2-PSK |
| **Procedure** | 1. Kết nối Staff client 1 và 2 vào `STAFF-WIFI` bằng PSK đúng (che secret khi chụp). 2. Ghi trạng thái association. |
| **Expected result** | Cả hai Staff client associate thành công |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-06-staff-associate.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-07 — Guest associate bằng WPA2-PSK (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-06 |
| **Preconditions** | TC-05 đạt; `GUEST-WIFI` dùng WPA2-PSK |
| **Procedure** | 1. Kết nối Guest client 1 và 2 vào `GUEST-WIFI` bằng PSK đúng (che secret khi chụp). 2. Ghi trạng thái association. |
| **Expected result** | Cả hai Guest client associate thành công |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-07-guest-associate.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-08 — Client nhận IP đúng subnet (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-07, REQ-08 |
| **Preconditions** | TC-06, TC-07 đạt; DHCP trên Server-PT đã cấu hình cho VLAN 10/20 |
| **Procedure** | 1. Chạy `ipconfig` trên 1 Staff và 1 Guest client. 2. Đối chiếu với [../docs/ip-vlan-plan.md](../docs/ip-vlan-plan.md). |
| **Expected result** | Staff nhận IP `192.168.10.0/24`; Guest nhận IP `192.168.20.0/24` |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-08-staff-ipconfig.png`, `evidence/tc-08-guest-ipconfig.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-09 — Client tới được gateway (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-09 |
| **Preconditions** | TC-08 đạt |
| **Procedure** | 1. Từ Staff `ping 192.168.10.1`. 2. Từ Guest `ping 192.168.20.1`. |
| **Expected result** | Ping tới gateway của đúng VLAN thành công (reply) |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-09-ping-gateway.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-10 — Guest không tới được Staff (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-11 |
| **Preconditions** | TC-08 đạt; ACL đã áp trên router (explicit deny Guest→Staff + permit tối thiểu) |
| **Procedure** | 1. Từ Guest (`192.168.20.0/24`) `ping` tới IP Staff client và `192.168.10.1`. 2. Ghi kết quả trước/sau ACL. |
| **Expected result** | Ping từ Guest tới Staff thất bại sau ACL (Request timed out / Destination unreachable) |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-10-guest-to-staff.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-11 — Guest không tới được Management (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-11 |
| **Preconditions** | TC-08 đạt; ACL đã áp; địa chỉ quản trị chốt theo [../docs/ip-vlan-plan.md](../docs/ip-vlan-plan.md) |
| **Procedure** | 1. Từ Guest `ping` tới địa chỉ quản trị (ví dụ WLC `192.168.99.10`). 2. Chạy `show access-lists` / `show running-config` trên router. |
| **Expected result** | Guest không tới được VLAN 99; ACL hiển thị khớp [../docs/security-design.md](../docs/security-design.md) |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-11-guest-to-mgmt.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-12 — VLAN đúng + Staff chỉ giữ truy cập cho phép + ACL hiển thị (Core)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core |
| **Requirement** | REQ-07, REQ-11 |
| **Preconditions** | Switch đã cấu hình VLAN; router đã áp ACL |
| **Procedure** | 1. Chạy `show vlan brief` (hoặc lệnh tương đương đúng model). 2. Kiểm tra cổng theo [../docs/topology.md](../docs/topology.md). 3. Từ Staff kiểm tra chỉ còn đúng truy cập nội bộ cho phép theo phạm vi đã chốt trước trong [../docs/implementation-notes.md](../docs/implementation-notes.md) (§4). |
| **Expected result** | VLAN 10/20/99 tồn tại, cổng đúng thiết kế đã chốt; Staff không vượt phạm vi cho phép |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-12-show-vlan.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-13 — RADIUS/WPA2-Enterprise cho Staff (Conditional)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Conditional — chỉ thử sau khi Core ổn định và khi Server-PT + model WLC hỗ trợ |
| **Requirement** | REQ-12 |
| **Preconditions** | TC-06 `Passed`; RADIUS đã cấu hình trên Server-PT và WLC (IP + shared secret, che khi chụp) |
| **Procedure** | 1. Kết nối Staff client bằng tài khoản RADIUS đúng. 2. Ghi kết quả associate và IP nhận được. Nếu Server-PT/model không hỗ trợ, ghi `Blocked` kèm lý do. |
| **Expected result** | Associate thành công qua RADIUS (hoặc `Blocked` có lý do nếu không hỗ trợ) |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-13-radius.png` (dự kiến, chưa có) |
| **Status** | Not Tested |

## TC-14 — Sai credential bị từ chối (Core cho PSK, Conditional cho RADIUS)

| Hạng mục | Nội dung |
| --- | --- |
| **Loại** | Core (WPA2-PSK) + Conditional (RADIUS) |
| **Requirement** | REQ-10 |
| **Preconditions** | TC-06/TC-07 đạt (PSK); TC-13 đã thử (cho phần RADIUS, nếu có) |
| **Procedure** | 1. Kết nối bằng PSK sai → phải bị từ chối. 2. Nếu đã tới TC-13, kết nối bằng tài khoản RADIUS sai → phải bị từ chối. |
| **Expected result** | Sai PSK bị từ chối, không nhận IP nội bộ (bắt buộc); sai tài khoản RADIUS bị từ chối (nếu đã triển khai RADIUS) |
| **Actual result** | _Not Tested_ |
| **Evidence filename** | `evidence/tc-14-invalid-psk.png`, `evidence/tc-14-invalid-radius.png` (dự kiến, chưa có; file RADIUS chỉ cần khi TC-13 được thử) |
| **Status** | Not Tested |

> Research-only (không có test case): WPA3, EAP-TLS/PEAP, roaming, nhiễu RF, đo throughput — chỉ làm nền báo cáo.
