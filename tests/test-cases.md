# Danh sách ca kiểm thử (Test Cases)

> Tất cả ca kiểm thử ở trạng thái **Not tested**. Cột `Actual result` để trống cho tới khi có kiểm thử thật. Kế hoạch tổng thể xem tại [../docs/testing-plan.md](../docs/testing-plan.md).

## TC-01 — AP đăng ký / được WLC quản lý

| Hạng mục | Nội dung |
| --- | --- |
| **Preconditions** | File `.pkt` mở được; WLC và AP đã đấu nối theo [../docs/topology.md](../docs/topology.md) |
| **Steps** | 1. Mở GUI/CLI WLC. 2. Kiểm tra danh sách AP đã join. 3. Ghi model AP và trạng thái join. |
| **Expected results** | 2–3 AP hiển thị trạng thái Registered/Joined trên WLC |
| **Actual results** | _Not tested_ |
| **Evidence paths** | `evidence/tc-01-ap-registration.png` (dự kiến, chưa có) |
| **Status** | Not tested |

## TC-02 — SSID khả dụng đúng

| Hạng mục | Nội dung |
| --- | --- |
| **Preconditions** | TC-01 đạt; WLC đã cấu hình SSID |
| **Steps** | 1. Từ wireless client quét mạng vô tuyến. 2. Ghi các SSID nhìn thấy tại 2 vị trí AP khác nhau. |
| **Expected results** | Nhìn thấy `STAFF-WIFI` và `GUEST-WIFI` |
| **Actual results** | _Not tested_ |
| **Evidence paths** | `evidence/tc-02-ssid-availability.png` (dự kiến, chưa có) |
| **Status** | Not tested |

## TC-03 — Client nhận IP đúng subnet

| Hạng mục | Nội dung |
| --- | --- |
| **Preconditions** | TC-02 đạt; DHCP đã cấu hình (trên router hoặc server) |
| **Steps** | 1. Kết nối 1 client vào `STAFF-WIFI`, 1 client vào `GUEST-WIFI`. 2. Chạy `ipconfig` trên mỗi client. 3. Đối chiếu với [../docs/ip-vlan-plan.md](../docs/ip-vlan-plan.md). |
| **Expected results** | Client Staff nhận IP `192.168.10.0/24`; client Guest nhận IP `192.168.20.0/24` |
| **Actual results** | _Not tested_ |
| **Evidence paths** | `evidence/tc-03-staff-ipconfig.png`, `evidence/tc-03-guest-ipconfig.png` (dự kiến, chưa có) |
| **Status** | Not tested |

## TC-04 — Kết nối tới gateway

| Hạng mục | Nội dung |
| --- | --- |
| **Preconditions** | TC-03 đạt |
| **Steps** | 1. Từ client Staff `ping 192.168.10.1`. 2. Từ client Guest `ping 192.168.20.1`. |
| **Expected results** | Ping tới gateway của đúng VLAN thành công (reply) |
| **Actual results** | _Not tested_ |
| **Evidence paths** | `evidence/tc-04-ping-gateway.png` (dự kiến, chưa có) |
| **Status** | Not tested |

## TC-05a — Xác thực thành công (nơi được hỗ trợ)

| Hạng mục | Nội dung |
| --- | --- |
| **Preconditions** | SSID đã cấu hình xác thực (PSK hoặc Enterprise/RADIUS nếu khả thi) |
| **Steps** | 1. Kết nối client với thông tin xác thực đúng (PSK đúng hoặc tài khoản RADIUS đúng). 2. Kiểm tra trạng thái kết nối và IP nhận được. |
| **Expected results** | Kết nối thành công, nhận đúng IP theo VLAN |
| **Actual results** | _Not tested_ |
| **Evidence paths** | `evidence/tc-05a-auth-success.png` (dự kiến, chưa có) |
| **Status** | Not tested |

## TC-05b — Xác thực thất bại bị từ chối (nơi được hỗ trợ)

| Hạng mục | Nội dung |
| --- | --- |
| **Preconditions** | Như TC-05a |
| **Steps** | 1. Kết nối với PSK sai hoặc tài khoản RADIUS sai. 2. Ghi thông báo từ chối. |
| **Expected results** | Kết nối bị từ chối, không nhận IP nội bộ |
| **Actual results** | _Not tested_ |
| **Evidence paths** | `evidence/tc-05b-auth-failure.png` (dự kiến, chưa có) |
| **Status** | Not tested |

## TC-06 — Guest bị hạn chế truy cập mạng Staff

| Hạng mục | Nội dung |
| --- | --- |
| **Preconditions** | TC-03 đạt; ACL (nếu triển khai) đã áp trên router |
| **Steps** | 1. Từ client Guest (`192.168.20.0/24`) `ping` tới IP client Staff (`192.168.10.0/24`) và tới gateway Staff `192.168.10.1`. 2. Ghi kết quả. |
| **Expected results** | Ping từ Guest tới Staff thất bại (Request timed out / Destination unreachable) sau khi áp ACL |
| **Actual results** | _Not tested_ |
| **Evidence paths** | `evidence/tc-06-guest-isolation.png` (dự kiến, chưa có) |
| **Status** | Not tested |

## TC-07 — Kiểm chứng VLAN

| Hạng mục | Nội dung |
| --- | --- |
| **Preconditions** | Switch đã cấu hình VLAN |
| **Steps** | 1. Chạy `show vlan brief` (hoặc lệnh tương đương trên đúng model). 2. Kiểm tra cổng trunk/access theo [../docs/topology.md](../docs/topology.md). |
| **Expected results** | VLAN 10 (STAFF), 20 (GUEST), 99 (MANAGEMENT) tồn tại; cổng trunk/access đúng thiết kế |
| **Actual results** | _Not tested_ |
| **Evidence paths** | `evidence/tc-07-show-vlan.png` (dự kiến, chưa có) |
| **Status** | Not tested |

## TC-08 — Kiểm chứng ACL và bảo vệ mạng Management

| Hạng mục | Nội dung |
| --- | --- |
| **Preconditions** | Router đã áp ACL (nếu triển khai); thiết bị quản trị ở VLAN 99 |
| **Steps** | 1. Từ Guest `ping` tới địa chỉ quản trị (ví dụ `192.168.99.10` — địa chỉ ví dụ, sẽ chốt theo [../docs/ip-vlan-plan.md](../docs/ip-vlan-plan.md)). 2. Chạy `show access-lists` / `show running-config` trên router. |
| **Expected results** | Guest không truy cập được VLAN 99; ACL hiển thị và khớp thiết kế tại [../docs/security-design.md](../docs/security-design.md) |
| **Actual results** | _Not tested_ |
| **Evidence paths** | `evidence/tc-08-acl-management.png` (dự kiến, chưa có) |
| **Status** | Not tested |
