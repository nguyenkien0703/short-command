# Templates — ADR & Design Doc

Bộ template chuẩn để **luyện tập tư duy kiến trúc** và **trình bày lên sếp/team**.

## Dùng cái nào khi nào?

| Tình huống | Dùng |
|-----------|------|
| Ghi lại **một quyết định** kỹ thuật + lý do (chọn A thay vì B) | [ADR](./adr-template.md) |
| Đề xuất/thiết kế **một giải pháp/hệ thống** trước khi build | [Design Doc](./design-doc-template.md) |

```text
Quy tắc ngón tay cái:
- ADR       = NGẮN (1-2 trang). Trả lời "quyết định GÌ và TẠI SAO". Bất biến theo thời gian.
- Design Doc= DÀI hơn. Trả lời "xây CÁI GÌ, NHƯ THẾ NÀO, đánh đổi ra sao" TRƯỚC khi code.
- 1 Design Doc thường SINH RA nhiều ADR (mỗi quyết định lớn trong đó = 1 ADR).
```

## Vì sao viết những cái này (đọc trước khi bỏ qua)

- **Luyện judgment kiến trúc** — ép bạn nêu rõ ràng buộc, phương án, đánh đổi thay vì "cảm tính".
- **Currency of promotion** — Staff/Architect được đánh giá qua **chữ viết** nhiều hơn code. Đây là bằng chứng năng lực.
- **Giảm tranh cãi lại** — quyết định có ghi lý do thì người sau không đào lại từ đầu.
- **Trình sếp** — sếp cần thấy bạn cân nhắc trade-off, rủi ro, chi phí — không phải "em thấy nên làm vậy".

## Cách luyện tập
```text
1. Mỗi quyết định kỹ thuật thật ở công việc -> viết 1 ADR (dù nhỏ).
2. Mỗi dự án/thay đổi lớn -> viết 1 Design Doc TRƯỚC khi làm.
3. Đọc lại sau 3-6 tháng -> đối chiếu "consequence" thực tế với dự đoán -> rút judgment.
4. Đánh số ADR tăng dần (ADR-0001, 0002...) lưu trong repo -> thành "decision log" của bạn/team.
```
