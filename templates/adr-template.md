# ADR-NNNN: <Tiêu đề ngắn gọn quyết định>

> **ADR (Architecture Decision Record)** = ghi lại MỘT quyết định kiến trúc quan trọng + lý do.
> Ngắn gọn (1-2 trang). Bất biến: đã "Accepted" thì không sửa — muốn đổi thì viết ADR mới thay thế nó.
> Xóa mọi dòng hướng dẫn (blockquote `>`) khi dùng thật.

---

- **Trạng thái:** `Proposed` | `Accepted` | `Deprecated` | `Superseded by ADR-XXXX`
- **Ngày:** YYYY-MM-DD
- **Người quyết định / phê duyệt:** <tên, vai trò>
- **Người liên quan (consulted/informed):** <team/người>
- **Tags:** <networking | k8s | security | cost | ...>

## 1. Bối cảnh (Context)

> Vấn đề gì cần giải quyết? Ràng buộc nào (kỹ thuật, business, thời gian, ngân sách, tuân thủ, team hiện có)?
> Nêu SỰ THẬT & RÀNG BUỘC, chưa nói giải pháp. Người đọc phải hiểu "tại sao cần quyết định này".

- Vấn đề:
- Ràng buộc bắt buộc (hard constraints):
- Yêu cầu phi chức năng liên quan (scale/latency/availability/cost/compliance):
- Giả định (assumptions):

## 2. Các phương án đã cân nhắc (Options Considered)

> Liệt kê ÍT NHẤT 2-3 phương án thực sự (kể cả "không làm gì"). Mỗi cái nêu ưu/nhược + đánh đổi.
> Đây là phần cho thấy bạn tư duy như architect, không phải chọn bừa.

### Phương án A — <tên>
- Mô tả:
- Ưu:
- Nhược / rủi ro:
- Chi phí (tiền + vận hành + độ phức tạp):

### Phương án B — <tên>
- Mô tả:
- Ưu:
- Nhược / rủi ro:
- Chi phí:

### Phương án C — <tên / "Giữ nguyên / không làm gì">
- Mô tả:
- Ưu / Nhược:

> (Tùy chọn) Bảng so sánh nhanh theo tiêu chí quan trọng nhất:

| Tiêu chí | A | B | C |
|----------|---|---|---|
| Chi phí | | | |
| Độ tin cậy | | | |
| Độ phức tạp vận hành | | | |
| Thời gian triển khai | | | |
| Bảo mật | | | |

## 3. Quyết định (Decision)

> Nêu RÕ RÀNG chọn phương án nào và **TẠI SAO** — gắn với ràng buộc/ưu tiên ở mục 1.
> Một câu cốt lõi: "Chúng tôi chọn ___ vì ___, chấp nhận đánh đổi ___."

**Chọn: Phương án <X>.**

Lý do chính:
-
-

Đây là quyết định **one-way door / two-way door** (khó/dễ đảo ngược): <ghi rõ + hàm ý>.

## 4. Hệ quả (Consequences)

> Điều gì trở nên tốt hơn, và cái GIÁ phải trả là gì? Trung thực về mặt trái.

**Tích cực:**
-

**Tiêu cực / đánh đổi chấp nhận:**
-

**Rủi ro & cách giảm thiểu (mitigation):**
-

**Việc cần làm tiếp (follow-up):**
- [ ]

## 5. Tham chiếu (References)
- Design Doc liên quan:
- ADR liên quan / bị thay thế:
- Link tài liệu, benchmark, POC:

---
> **Mẹo review trước khi trình sếp:** đọc lại mục 3 — nếu không nối được "quyết định ↔ ràng buộc ở mục 1",
> nghĩa là lập luận còn yếu. Sếp sẽ hỏi đúng chỗ đó.
