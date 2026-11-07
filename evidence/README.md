# Evidence

Thư mục lưu minh chứng kiểm thử trên bản Cisco Packet Tracer **9.0.1.0858**: ảnh chụp canvas, ảnh cửa sổ Command Prompt / kết quả lệnh. Người duy trì: Hoàng Minh Hiếu (nhóm trưởng).

> **Lưu ý 2026-11-07 (final build):** phiên v1 (`enterprise-wifi-wlc-lab-v1.pkt`, controller `WLC-PT`) đã được thay bằng **`final.pkt`** (topology enterprise 18 thiết bị: ISP + R1/R2 + CORE-SW + ACCESS-SW + WLC1 + 2 AP + 3 server + 6 wireless client + ADMIN-PC). Toàn bộ ảnh minh chứng từng bước v1 (cũ: `p1-wired-skeleton-topology.png`, `tc-01-*` → `tc-08-*`, các ảnh `audit-*`) đã bị **xóa khỏi repo cùng file v1 trong lần dọn dẹp cuối** — vẫn tồn tại trong lịch sử Git nếu cần đối chiếu. Minh chứng final dùng tên không theo mã TC (xem dưới).

## Minh chứng final (hiện có trong repo)

| File | Nội dung | Tham chiếu |
| --- | --- | --- |
| [`Final.jpg`](Final.jpg) | **Canvas Packet Tracer** chụp 2026-11-06: **topology final 18 thiết bị đầy đủ link, mọi liên kết up** (ISP, R1, R2, CORE-SW, ACCESS-SW, WLC1, LAP-STAFF, LAP-GUEST, RADIUS-SRV, DHCP-SRV, ADMIN-PC, Web-SRV, S-LAP1/2/3, G-LAP1/2, G-PHONE) | [../docs/final-implementation.md](../docs/final-implementation.md) |
| [`Final 2.png`](Final%202.png) | **Canvas Packet Tracer** — bản chụp lại của topology final, trạng thái ổn định | same as above |
| [`Final but error.png`](Final%20but%20error.png) | Bản chụp tại thời điểm **bất ổn mô phỏng**: một wireless association tạm rớt — lưu để **ghi nhận limitation của PT với wireless**; trạng thái đã hồi phục trên `Final.jpg` / `Final 2.png` sau recovery | Mục *Limitations* của [../docs/final-implementation.md](../docs/final-implementation.md) |

## Kết quả kiểm thử final (2026-11-05 → 2026-11-07)

Trình bày chi tiết tại [../tests/test-cases.md](../tests/test-cases.md) và [../docs/final-implementation.md](../docs/final-implementation.md). Tóm tắt:

- **Gateway**: cả 6 wireless client + ADMIN-PC ping gateway riêng của mình, 4/4 (0% loss).
- **Internet (PAT)**: Staff (S-LAP1/2/3) → `8.8.8.8` 4/4; Guest G-LAP1 → `8.8.8.8` 4/4 (đạt sau một lần retry — thỉnh thoảng PT đổi 50% loss khi wireless stack bất ổn; xem limitations).
- **ACL cô lập Guest**: G-LAP1 → `192.168.10.1` **0/4** (blocked) ✓; G-LAP1 → `192.168.99.22` **0/4** (blocked) ✓; DHCP vẫn permit qua `bootps`.
- Không tạo screenshot giả; canvas dùng làm minh chứng hiện trường, kết quả ping nêu bằng số trong tài liệu (đã chạy thật qua PT bridge).

## Quy định

1. Tên file tiếng Anh, gắn với nội dung (`final-*`, `audit-*` cho ảnh quan sát); không dùng mã TC cho ảnh canvas hiện trường.
2. Ảnh đọc được: thấy rõ thiết bị/link liên quan và kết quả chính; che PSK/password/tài khoản thật.
3. Chỉ commit minh chứng của lần test thật trên đúng bản 9.0.1.0858.
4. Không dùng ảnh minh họa ngoài đồ án, không chỉnh sửa kết quả.
5. Mỗi file phải được tham chiếu từ [../tests/test-cases.md](../tests/test-cases.md), [../docs/final-implementation.md](../docs/final-implementation.md) hoặc [../docs/testing-plan.md](../docs/testing-plan.md).
