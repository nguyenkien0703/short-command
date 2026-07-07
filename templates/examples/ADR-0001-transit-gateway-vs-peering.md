# ADR-0001: Dùng Transit Gateway thay cho VPC Peering để kết nối multi-VPC

> Đây là ADR MẪU đã điền hoàn chỉnh — dùng làm tham chiếu khi viết ADR thật của bạn.

---

- **Trạng thái:** Accepted
- **Ngày:** 2026-07-03
- **Người quyết định / phê duyệt:** Nguyen Kien (DevOps), Head of Infrastructure
- **Người liên quan:** Team Platform, Team Security
- **Tags:** networking, aws, multi-account

## 1. Bối cảnh (Context)

- **Vấn đề:** Công ty đang có 8 VPC (prod/staging/dev × vài product) trên 3 AWS account, dự kiến tăng lên ~20 VPC trong 12 tháng. Các VPC cần giao tiếp nội bộ với nhau và với on-premises (qua VPN).
- **Ràng buộc bắt buộc:**
  - CIDR các VPC đã được quy hoạch không chồng lấn (10.x.0.0/16 mỗi môi trường).
  - Phải quản lý kết nối tập trung, có audit (yêu cầu Security).
  - Ngân sách hạ tầng mạng < $X/tháng.
- **Yêu cầu phi chức năng:** thêm VPC mới không được kéo theo cấu hình thủ công O(n²); phải scale tới hàng chục VPC.
- **Giả định:** số VPC sẽ tiếp tục tăng; nhu cầu kết nối on-prem sẽ mở rộng.

## 2. Các phương án đã cân nhắc (Options Considered)

### Phương án A — VPC Peering (full mesh)
- Mô tả: peer từng cặp VPC trực tiếp.
- Ưu: đơn giản, **miễn phí** (chỉ trả data transfer), latency thấp nhất.
- Nhược: **không bắc cầu** (A-B, B-C không cho A-C); số kết nối tăng O(n²) — 20 VPC cần ~190 peering; quản lý route thủ công ở mọi VPC; không tích hợp on-prem tập trung.

### Phương án B — Transit Gateway (hub-and-spoke)
- Mô tả: 1 TGW làm hub, mọi VPC + VPN on-prem attach vào.
- Ưu: **bắc cầu**, thêm VPC chỉ cần 1 attachment (O(n)); route table tập trung; tích hợp VPN/DX; chia sẻ qua RAM cho multi-account; audit dễ.
- Nhược: **tính phí theo attachment/giờ + GB xử lý**; thêm 1 điểm cần thiết kế HA; latency thêm 1 hop nhỏ.

### Phương án C — Giữ nguyên Peering + chấp nhận thủ công
- Mô tả: tiếp tục peering, thêm tay khi cần.
- Ưu: không đổi gì ngay, không phát sinh phí TGW.
- Nhược: nợ vận hành tăng nhanh, sẽ phải migrate đau hơn khi đã 20 VPC.

| Tiêu chí | A (Peering) | B (TGW) | C (giữ nguyên) |
|----------|:-----------:|:-------:|:--------------:|
| Chi phí trực tiếp | Thấp nhất | Trung bình | Thấp |
| Chi phí vận hành (scale) | Cao (O(n²)) | Thấp (O(n)) | Rất cao |
| Bắc cầu / on-prem tập trung | Không | Có | Không |
| Audit/quản lý tập trung | Kém | Tốt | Kém |
| Phù hợp khi tăng tới 20+ VPC | Không | Có | Không |

## 3. Quyết định (Decision)

**Chọn: Phương án B — Transit Gateway.**

Lý do chính:
- Ràng buộc "scale tới hàng chục VPC, không cấu hình O(n²)" loại A và C.
- Yêu cầu Security về quản lý & audit tập trung khớp với mô hình hub của TGW.
- Chi phí TGW chấp nhận được so với chi phí người + rủi ro của peering full-mesh ở quy mô mục tiêu.

Đây là quyết định **two-way door** ở mức vừa: có thể migrate về peering cho vài VPC cụ thể nếu cần tối ưu latency/cost, nhưng làm hub từ đầu tránh phải re-architect sau.

## 4. Hệ quả (Consequences)

**Tích cực:**
- Thêm VPC mới = 1 attachment, không đụng các VPC khác.
- Kết nối on-prem (VPN) tập trung tại TGW.
- Route & audit tập trung.

**Tiêu cực / đánh đổi chấp nhận:**
- Phát sinh phí TGW (attachment + data processing) — đã tính vào ngân sách.
- Thêm 1 hop → latency tăng không đáng kể cho use case hiện tại.

**Rủi ro & giảm thiểu:**
- TGW thành điểm phụ thuộc trọng yếu → dùng TGW có sẵn HA của AWS (regional), thiết kế route dự phòng; giám sát qua CloudWatch.
- Chi phí data processing bất ngờ → bật cost alert, xem lại sau 1 quý.

**Follow-up:**
- [ ] Thiết kế TGW route tables (phân đoạn prod/non-prod).
- [ ] Chia sẻ TGW qua RAM cho các account.
- [ ] Viết Design Doc chi tiết cho migration từ peering hiện có.

## 5. Tham chiếu (References)
- Design Doc: "Multi-VPC Connectivity Redesign" (link).
- AWS Docs: Transit Gateway, RAM.
- File nội bộ: [cloud/aws/01-networking-vpc.md](../../cloud/aws/01-networking-vpc.md)
