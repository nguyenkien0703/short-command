# Design Doc: <Tên hệ thống / tính năng / thay đổi>

> **Design Doc (còn gọi RFC / Technical Spec)** = thiết kế & thuyết phục TRƯỚC KHI xây.
> Mục tiêu: người đọc (team + sếp) hiểu bạn xây gì, tại sao thế, đánh đổi ra sao, rủi ro gì — và đồng ý.
> Viết đủ để ra quyết định, KHÔNG viết tiểu thuyết. Xóa mọi dòng hướng dẫn (`>`) khi dùng thật.

---

- **Tác giả:** <tên>   **Trạng thái:** `Draft` | `In Review` | `Approved` | `Implemented`
- **Reviewers:** <người cần duyệt>   **Ngày:** YYYY-MM-DD
- **Ticket/Epic liên quan:**

## 0. TL;DR (tóm tắt 3-5 dòng)

> Viết CUỐI CÙNG nhưng đặt ĐẦU TIÊN. Sếp bận chỉ đọc phần này. Nêu: vấn đề, giải pháp chọn, tác động chính.

Vấn đề → giải pháp → kết quả kỳ vọng, trong vài câu.

## 1. Bối cảnh & Vấn đề (Background & Problem)

> Tại sao làm việc này BÂY GIỜ? Đau ở đâu? Có số liệu càng tốt (vd "latency p99 = 2s, khách rời 15%").

- Tình trạng hiện tại:
- Vấn đề / pain point (kèm số liệu nếu có):
- Vì sao cần giải quyết bây giờ (tác động business):

## 2. Mục tiêu & Ngoài phạm vi (Goals & Non-Goals)

> Non-Goals cực quan trọng — nó chặn scope creep và câu hỏi lạc đề của reviewer.

**Goals (đo được):**
- [ ]
- [ ]

**Non-Goals (KHÔNG làm trong lần này):**
-

## 3. Yêu cầu (Requirements)

**Chức năng (Functional):**
-

**Phi chức năng (Non-Functional) — thường quyết định kiến trúc:**
| Yêu cầu | Mục tiêu cụ thể |
|---------|-----------------|
| Scale (RPS/users/data) | |
| Latency | |
| Availability (số 9) | |
| Bảo mật / tuân thủ | |
| Ngân sách / chi phí | |
| Thời gian ra mắt | |

## 4. Giải pháp đề xuất (Proposed Solution)

> Phần chính. Mô tả kiến trúc rõ ràng. CÓ SƠ ĐỒ (ASCII/diagram) — 1 hình bằng 1000 chữ.

### 4.1 Tổng quan kiến trúc
```text
<Sơ đồ luồng: component nào, nối với nhau ra sao, data đi đâu>
Ví dụ:
  Client -> CloudFront -> ALB -> ECS (service X) -> RDS
                                      -> SQS -> Worker -> S3
```

### 4.2 Thành phần chính
- **<Component>**: vai trò, công nghệ, vì sao chọn.
-

### 4.3 Luồng dữ liệu / xử lý chính
> Mô tả 1-2 luồng quan trọng nhất (happy path + 1 edge case).

### 4.4 Data model / API (nếu có)
> Schema chính, endpoint chính, thay đổi contract.

## 5. Các phương án thay thế (Alternatives Considered)

> BẮT BUỘC. Cho thấy bạn không chọn bừa. Mỗi phương án: mô tả ngắn + vì sao KHÔNG chọn.
> (Mỗi quyết định lớn ở đây nên tách thành 1 ADR riêng.)

| Phương án | Ưu | Nhược | Vì sao không chọn |
|-----------|----|----|-------------------|
| A (đã chọn) | | | (được chọn) |
| B | | | |
| C (buy/managed?) | | | |

## 6. Đánh đổi (Trade-offs)

> Trung thực về cái giá của lựa chọn. Sếp/reviewer tin bạn hơn khi bạn tự nêu mặt trái.

- Ta được: ___  |  Ta hy sinh: ___
- Đây là one-way / two-way door:

## 7. Các mặt cắt ngang (Cross-cutting concerns)

> Reviewer giỏi sẽ hỏi đúng những mục này. Chuẩn bị trước.

- **Bảo mật:** authn/authz, mã hóa, secrets, bề mặt tấn công, phân quyền.
- **Độ tin cậy / HA / DR:** failure modes, blast radius, backup, RPO/RTO, rollback.
- **Khả năng quan sát (Observability):** metrics/logs/traces nào, alert gì, SLO.
- **Hiệu năng & Scale:** điểm nghẽn, cách scale, giới hạn.
- **Chi phí:** ước tính chi phí hạ tầng + vận hành; so với hiện tại.
- **Vận hành:** ai vận hành, runbook, độ phức tạp thêm vào.
- **Migration / tương thích ngược:** ảnh hưởng hệ thống đang chạy, cách chuyển đổi.

## 8. Kế hoạch triển khai (Rollout Plan)

> Chia nhỏ, giảm rủi ro. Không big-bang.

- [ ] Giai đoạn 1 (MVP / POC):
- [ ] Giai đoạn 2:
- [ ] Chiến lược release: feature flag / canary / blue-green
- [ ] Cách rollback nếu hỏng:
- [ ] Tiêu chí thành công (đo bằng gì):

## 9. Rủi ro & Câu hỏi mở (Risks & Open Questions)

- **Rủi ro:** <mô tả> → **Giảm thiểu:** <cách>
- **Câu hỏi mở (cần input):**
  - [ ]

## 10. Ước tính công sức & mốc thời gian (Effort & Timeline)
| Hạng mục | Ước tính | Người |
|----------|----------|-------|
| | | |

## 11. Phụ lục (Appendix)
- POC/benchmark, link tham khảo, ADR liên quan, số liệu chi tiết.

---
> **Checklist trước khi trình sếp/review:**
> - [ ] TL;DR đọc là hiểu ngay, không cần đọc hết.
> - [ ] Goals đo được; Non-Goals rõ ràng.
> - [ ] Có ≥ 2 phương án thay thế + lý do loại.
> - [ ] Đã nêu trade-off & rủi ro TRUNG THỰC (không giấu mặt trái).
> - [ ] Có ước tính chi phí + kế hoạch rollback.
> - [ ] Có sơ đồ kiến trúc.
> - [ ] Mục cross-cutting (bảo mật/DR/observability/cost) đã cân nhắc.
