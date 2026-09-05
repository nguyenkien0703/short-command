# Templates — ADR · Design Doc · Threat Model · Risk Acceptance

Bộ template chuẩn để **luyện tập tư duy kiến trúc** và **trình bày lên sếp/team**.

## Dùng cái nào khi nào?

| Tình huống | Dùng |
|-----------|------|
| Ghi lại **một quyết định** kỹ thuật + lý do (chọn A thay vì B) | [ADR](./adr-template.md) |
| Đề xuất/thiết kế **một giải pháp/hệ thống** trước khi build | [Design Doc](./design-doc-template.md) |
| Phân tích **rủi ro bảo mật** trước khi xây (hệ thống có tiền/PII/tích hợp mới) | [Threat Model](./threat-model-template.md) |
| **Không sửa được ngay** một rủi ro nhưng vẫn phải đi tiếp | [Risk Acceptance](./risk-acceptance-template.md) |

```text
Quy tắc ngón tay cái:
- ADR          = NGẮN (1-2 trang). Trả lời "quyết định GÌ và TẠI SAO". Bất biến theo thời gian.
- Design Doc   = DÀI hơn. Trả lời "xây CÁI GÌ, NHƯ THẾ NÀO, đánh đổi ra sao" TRƯỚC khi code.
- Threat Model = "Ai có thể phá cái này, bằng cách nào, ta làm gì?" — làm TRƯỚC khi build, không phải sau.
- Risk Accept. = "Rủi ro còn đó, AI chịu, ĐẾN BAO GIỜ, trong lúc chờ thì bù bằng gì?"

Quan hệ giữa chúng:
  1 Design Doc  -> sinh ra nhiều ADR (mỗi quyết định lớn = 1 ADR)
  1 Threat Model-> sinh ra ticket khắc phục; cái nào chưa làm được -> 1 Risk Acceptance
```

> Hai mẫu Threat Model và Risk Acceptance **có ví dụ đã điền cho bối cảnh ngân hàng**
> (API chuyển tiền nhanh 24/7; ngoại lệ CVE chưa có bản vá). Xem lộ trình dùng chúng ở
> [`../career-devsecops-finance.md`](../career-devsecops-finance.md).

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
