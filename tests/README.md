# Tests

Thư mục kiểm thử của dự án (bản 9.0.1.0858).

- [test-cases.md](test-cases.md) — 14 ca kiểm thử: TC-01→TC-12 và TC-14 (PSK) là Core; TC-13 và phần RADIUS của TC-14 là Conditional.
- Kế hoạch tổng thể và phân loại Core/Conditional/Research-only xem tại [../docs/testing-plan.md](../docs/testing-plan.md).

## Hiện trạng

**⚠ Quản lý này cho giai đoạn v1 (cũ).** Kể từ 2026-11-07, deliverable là `packet-tracer/final.pkt` (topology enterprise 18 thiết bị); file `v1.pkt` + ảnh `tc-01→tc-08` đã bị xoá khỏi repo (giữ trong lịch sử Git). Checkout status tại [../docs/final-implementation.md](../docs/final-implementation.md) mục 7 và chính bảng tại [test-cases.md](test-cases.md):

**Final build (2026-11-05 → 2026-11-07, ping thật qua PT bridge):**

- **TC-09 Passed** — 6 wireless client ping gateway 4/4.
- **TC-10 Passed (1/3 guest)** — G-LAP1 → 192.168.10.1 bị chặn 0/4; G-LAP2/G-PHONE chưa chạy bài này riêng.
- **TC-11 Passed (1/3 guest + config)** — G-LAP1 → 192.168.99.22 bị chặn 0/4; DHCP vẫn permit; ACL binding có trong running/startup.
- **TC-12 Passed (config-level)** — `configs/switch/core-sw-running-config.txt` khớp design; screenshot show-vlan riêng chưa có.
- **TC-13 Configured / chưa chứng minh exchange end-to-end** — host + WLC khớp port 1645; không có minh chứng EAP.
- **TC-14 Not Tested** — chưa thử PSK/tài khoản sai.

**Giai đoạn v1 (lịch sử): TC-01 → TC-08 đã `Passed`** (TC-01/TC-02 `2026-10-18` trên `WLC-PT`; TC-03 `2026-10-19` LAP join `Status: Online`; TC-04 + TC-05 `2026-10-19`; TC-06 + TC-07 `2026-10-19`; TC-08 `2026-10-20` trên test copy, lưu official `2026-10-21`). Minh chứng từng bộ cũ đã bị xoá khỏi working tree; chi tiết nằm trong `git log -- evidence/` và [../docs/final-implementation.md](../docs/final-implementation.md).

## Quy định

1. Chạy theo thứ tự TC-01→TC-12, TC-14 (PSK) trước; TC-13 sau cùng.
2. Không điền `Actual result` khi chưa test thật; `Blocked` kèm lý do được chấp nhận cho TC-13 mà không ảnh hưởng Core.
3. Mỗi `Passed` phải có file evidence tồn tại trong `../evidence/`.
4. Mọi đổi kỳ vọng (`Expected result`) phải có lý do và đồng bộ với `docs/`.
