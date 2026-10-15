# Kế hoạch kiểm thử (Core / Conditional / Research-only)

> Toàn bộ test case ở trạng thái **Not Tested**. Cột kết quả thực tế (`Actual result`) để trống cho tới khi có kiểm thử thật. Chi tiết từng bước xem tại [../tests/test-cases.md](../tests/test-cases.md).

## 1. Phân loại test

- **Core (bắt buộc, TC-01→TC-12, TC-14 cho WPA2-PSK):** quyết định đạt/không đạt của đồ án. Thứ tự ưu tiên: WLC + LAP-PT + WLAN + WPA2-PSK + IP/gateway + VLAN/ACL trước.
- **Conditional (TC-13 RADIUS, phần RADIUS của TC-14):** chỉ thử sau khi Core ổn định và khi Server-PT + model WLC hỗ trợ. `Blocked` kèm lý do được chấp nhận và không làm mất trạng thái hoàn thành của Core.
- **Research-only:** WPA3, EAP-TLS/PEAP, roaming, nhiễu RF, throughput — không có test case, chỉ làm nền báo cáo.

## 2. Ma trận truy vết (test coverage)

| Test ID | Loại | Yêu cầu liên quan | Nội dung kiểm thử |
| --- | --- | --- | --- |
| TC-01 | Core | REQ-01 | WLC management reachability |
| TC-02 | Core | REQ-02 | LAP-PT nhận IP quản trị hợp lệ |
| TC-03 | Core | REQ-03 | LAP-PT join WLC |
| TC-04 | Core | REQ-04 | WLAN `STAFF-WIFI` được tạo và phát sóng |
| TC-05 | Core | REQ-05 | WLAN `GUEST-WIFI` được tạo và phát sóng |
| TC-06 | Core | REQ-06 | Staff associate bằng WPA2-PSK |
| TC-07 | Core | REQ-06 | Guest associate bằng WPA2-PSK |
| TC-08 | Core | REQ-07, REQ-08 | Client nhận IP đúng subnet (DHCP Server-PT) |
| TC-09 | Core | REQ-09 | Client tới được gateway tương ứng |
| TC-10 | Core | REQ-11 | Guest không tới được Staff |
| TC-11 | Core | REQ-11 | Guest không tới được Management |
| TC-12 | Core | REQ-07, REQ-11 | VLAN tồn tại đúng + Staff chỉ giữ truy cập cho phép + ACL hiển thị |
| TC-13 | Conditional | REQ-12 | RADIUS/WPA2-Enterprise cho Staff (thử sau cùng) |
| TC-14 | Core + Conditional | REQ-10 | Sai credential bị từ chối (PSK là Core; RADIUS là Conditional) |

## 3. Điều kiện chung và thứ tự chạy

1. File `.pkt` đúng bản 9.0.1.0858; topology đúng [topology.md](topology.md); địa chỉ đúng [ip-vlan-plan.md](ip-vlan-plan.md).
2. Chạy theo thứ tự TC-01→TC-12, TC-14 (PSK) trước; TC-13 và phần RADIUS của TC-14 sau cùng.
3. Bắt đầu với 01 LAP-PT; chỉ thêm AP khi TC-03 + TC-06 đã `Passed`.
4. Biết SSID, PSK kiểm thử, tài khoản RADIUS kiểm thử (nếu tới TC-13). **Không commit secret thật** — che khi chụp minh chứng.
5. Công cụ: `ping`, `ipconfig`, `show vlan brief`, `show access-lists` / `show running-config` (đúng model), quét wireless trên client, GUI WLC.

## 4. Quy định ghi kết quả và minh chứng

- Mỗi test case gồm: ID, loại, requirement, preconditions, procedure, expected, actual, evidence filename, status.
- `Actual result` để trống hoặc ghi `Not Tested` cho tới khi test thật.
- `Status` chỉ dùng: `Not Tested` / `Passed` / `Failed` / `Blocked` (ghi rõ lý do nếu `Blocked`, ví dụ model không hỗ trợ).
- Minh chứng đặt tại `../evidence/` theo tên trong test case (ví dụ `evidence/tc-01-wlc-reachability.png`); file phải tồn tại khi claim `Passed`.
- Không chỉnh `Expected result` để khớp lỗi thực tế; nếu kỳ vọng sai, cập nhật tài liệu thiết kế và nêu lý do.

## 5. Tiêu chí đạt (pass criteria)

- Core TC-01→TC-12 và TC-14 (PSK) phải `Passed` mới công nhận luồng cơ bản hoàn thành.
- TC-13 (RADIUS): `Passed` là cộng thêm; `Failed`/`Blocked` kèm lý do và evidence được chấp nhận mà không ảnh hưởng Core.
- TC-10/TC-11: Guest→Staff/Management phải không tới được sau ACL; TC-09 (gateway của chính Guest) vẫn thông.
- Mọi `Failed`/`Blocked` phải có nguyên nhân và hướng xử lý, không giấu lỗi.
