# Threat Model Template (STRIDE) — có ví dụ đã điền cho hệ thống tài chính

> Threat model = **phân tích bề mặt tấn công TRƯỚC khi xây**, không phải pentest sau khi xong.
> Mục tiêu không phải liệt kê mọi mối đe dọa — mà là **tìm ra vài mối đe dọa quan trọng nhất và
> quyết định làm gì với chúng**, có ghi lại để người sau hiểu.
>
> Đọc kèm: [`../career-devsecops-finance.md`](../career-devsecops-finance.md) (§3.2 luồng tiền 7 chặng) ·
> [`adr-template.md`](./adr-template.md) (mỗi quyết định lớn sinh ra từ threat model → 1 ADR)

## Khi nào làm

```text
BẮT BUỘC: hệ thống mới có luồng tiền · kênh mới hướng Internet · xử lý PII/dữ liệu thẻ/sinh trắc học
          · tích hợp đối tác mới · thay đổi ranh giới tin cậy (trust boundary)
NÊN LÀM : thay đổi kiến trúc lớn · trước kỳ đánh giá PCI/ISO · sau một sự cố (để tìm lớp lỗ hổng)
KHÔNG CẦN: sửa bug, đổi config, thêm field vào API sẵn có (trừ khi field đó là dữ liệu nhạy cảm)
```

## Quy trình 5 bước (một buổi 90 phút là đủ cho lần đầu)

```text
1. VẼ  — sơ đồ luồng dữ liệu (DFD): thực thể ngoài, tiến trình, kho dữ liệu, luồng, RANH GIỚI TIN CẬY.
2. HỎI — với mỗi phần tử, chạy qua 6 chữ STRIDE. Đừng bỏ qua kho dữ liệu và luồng.
3. XẾP — mỗi mối đe dọa: khả năng × ảnh hưởng. Chỉ ~5–10 cái đáng bàn tiếp.
4. QUYẾT— mỗi mối đe dọa chọn 1 trong 4: Giảm thiểu / Chuyển giao / Tránh / Chấp nhận (có người ký).
5. THEO— biến các "Giảm thiểu" thành ticket có người phụ trách & hạn. Không có ticket = chưa xong.
```

> ⚠️ **Lỗi phổ biến nhất:** vẽ sơ đồ theo *công nghệ* (pod, service, DB) thay vì theo *ranh giới tin cậy*
> và *dòng tiền*. Trong hệ thống tài chính, mối đe dọa nằm ở chỗ tiền đổi chủ và chỗ dữ liệu vượt ranh giới,
> không nằm ở chỗ nhiều microservice nhất.

## STRIDE — nhắc nhanh

| Chữ | Mối đe dọa | Vi phạm thuộc tính | Câu hỏi tự vấn |
|---|---|---|---|
| **S** — Spoofing | Giả mạo danh tính | Xác thực (Authentication) | Ai đó có giả làm người/dịch vụ khác được không? |
| **T** — Tampering | Sửa đổi trái phép | Toàn vẹn (Integrity) | Dữ liệu/lệnh có bị sửa trên đường hoặc lúc lưu không? |
| **R** — Repudiation | Chối bỏ hành vi | Chống chối bỏ | Có chứng minh được ai đã làm gì, khi nào không? |
| **I** — Information Disclosure | Lộ thông tin | Bảo mật (Confidentiality) | Dữ liệu có lộ ra ngoài phạm vi cần biết không? |
| **D** — Denial of Service | Từ chối dịch vụ | Sẵn sàng (Availability) | Có làm hệ thống ngừng/chậm được không? |
| **E** — Elevation of Privilege | Leo thang đặc quyền | Ủy quyền (Authorization) | Có làm được việc vượt quyền của mình không? |

**Bổ sung riêng cho tài chính** (STRIDE không phủ hết — đây là chỗ Senior thật khác "Senior tool"):

| Mối đe dọa | Câu hỏi tự vấn |
|---|---|
| **Trùng lặp giao dịch** | Retry ở bất kỳ tầng nào (client, LB, service mesh, job, MQ) có tạo ra 2 bút toán không? |
| **Nhất quán một phần** | Ghi nợ thành công nhưng ghi có thất bại thì sao? Có bù trừ không? Ai phát hiện? |
| **Đi đường vòng control** | Có đường nào chạm tới core/DB mà không qua tầng kiểm tra rủi ro không? (job, batch, tool nội bộ, vendor) |
| **Nội gián** | Người vận hành có tự mình hoàn tất được một giao dịch không? (vi phạm SoD) |
| **Lệch không phát hiện** | Nếu đối soát lệch 1 giao dịch/ngày, sau bao lâu có người biết? |

---

## Template (copy phần dưới)

```markdown
# TM-<năm>-<số> — Threat Model: <tên hệ thống/luồng>

| | |
|---|---|
| **Phiên bản** | 1.0 |
| **Ngày** | YYYY-MM-DD |
| **Người làm** | <tên> · **Người review**: <tên, nên có 1 người ngoài team> |
| **Hệ thống** | <tên> · **Cấp độ ATTT**: <1–5 theo NĐ 85/2016 & TT 09> |
| **Khung tuân thủ áp dụng** | <TT 09 / TT 50 / QĐ 2345 / PCI DSS / SWIFT CSCF / Luật BVDLCN> |
| **Trạng thái** | Nháp / Đã review / Đã duyệt |

## 1. Phạm vi & giả định
**Trong phạm vi:** <liệt kê thành phần>
**Ngoài phạm vi:** <liệt kê + lý do — quan trọng để auditor biết bạn không bỏ sót vô ý>
**Giả định:** <ví dụ: "hạ tầng mạng đã phân vùng theo chuẩn X", "vendor Y đã có chứng nhận Z">
> Mỗi giả định là một rủi ro tiềm ẩn. Nếu giả định sai, threat model này sai.

## 2. Tài sản cần bảo vệ (xếp theo giá trị)
| Tài sản | Loại | Vì sao kẻ tấn công muốn |
|---|---|---|
| Khả năng khởi tạo giao dịch | Chức năng | Chiếm đoạt tiền trực tiếp |
| Dữ liệu khách hàng (PII) | Dữ liệu cá nhân | Bán, lừa đảo, tống tiền |
| <...> | | |

## 3. Sơ đồ luồng dữ liệu (DFD) + ranh giới tin cậy
<sơ đồ ASCII hoặc link ảnh — PHẢI vẽ rõ ranh giới tin cậy bằng đường nét đứt>

## 4. Bảng mối đe dọa
| # | Phần tử | STRIDE | Mối đe dọa (kịch bản CỤ THỂ) | Khả năng | Ảnh hưởng | Mức | Xử lý | Control | Chủ sở hữu | Hạn |
|---|---|---|---|:--:|:--:|:--:|---|---|---|---|
| T1 | | | | Thấp/TB/Cao | Thấp/TB/Cao | | Giảm thiểu/Chấp nhận/... | | | |

> Viết kịch bản CỤ THỂ: "kẻ tấn công có token của user A gọi được endpoint /transfer với accountId của user B"
> chứ không phải "thiếu kiểm soát truy cập".

## 5. Rủi ro chấp nhận
| # | Rủi ro tồn dư | Lý do chấp nhận | Người ký | Hạn xem lại |
|---|---|---|---|---|

## 6. Việc phải làm (ra ticket ngay)
| # | Việc | Ticket | Người | Hạn |
|---|---|---|---|---|

## 7. Lịch xem lại
Xem lại khi: <thay đổi kiến trúc / thêm tích hợp / sau sự cố> — hoặc định kỳ <6/12> tháng.
```

---

## Ví dụ ĐÃ ĐIỀN — API chuyển tiền nhanh 24/7

> Ví dụ rút gọn nhưng **thật về mặt kỹ thuật**. Dùng làm mẫu cho lần đầu tự làm.
> Tên hệ thống là hư cấu; đừng sao chép nguyên si vào tổ chức của bạn mà chưa đối chiếu kiến trúc thật.

### TM-2026-003 — Threat Model: API chuyển tiền nhanh 24/7 (liên ngân hàng)

| | |
|---|---|
| **Hệ thống** | `transfer-api` (kênh Mobile Banking) → Core Banking → NAPAS |
| **Cấp độ ATTT** | Cấp 3 |
| **Khung áp dụng** | TT 09/2020, TT 50/2024, QĐ 2345, Luật BVDLCN 2025 |

#### 1. Phạm vi & giả định
- **Trong phạm vi:** app mobile → API Gateway → `transfer-api` → fraud engine → core banking adapter → hàng đợi gửi NAPAS → job đối soát EOD.
- **Ngoài phạm vi:** nội bộ core banking (hệ thống vendor, có threat model riêng TM-2025-011); hạ tầng NAPAS phía đối tác.
- **Giả định:** (a) mạng đã phân vùng, `transfer-api` không nói chuyện trực tiếp với Internet; (b) khóa ký gửi NAPAS nằm trong HSM; (c) app mobile đã bật TLS pinning.

#### 2. Tài sản
| Tài sản | Loại | Vì sao kẻ tấn công muốn |
|---|---|---|
| Khả năng khởi tạo lệnh chuyển tiền | Chức năng | Chiếm đoạt tiền trực tiếp — giá trị cao nhất |
| Khóa ký bản tin gửi NAPAS | Khóa mật mã | Giả mạo lệnh ở quy mô lớn |
| Số dư & lịch sử giao dịch | PII + dữ liệu tài chính | Lừa đảo có mục tiêu, tống tiền |
| Nhật ký giao dịch (audit trail) | Bằng chứng | Xóa dấu vết sau khi chiếm đoạt |

#### 3. DFD & ranh giới tin cậy

```text
   ╔═══ RANH GIỚI: THIẾT BỊ KHÁCH (KHÔNG TIN CẬY) ═══╗
   ║  App Mobile                                     ║
   ╚════════════════════╤════════════════════════════╝
                        │ HTTPS + TLS pinning
   ╔════════════════════▼═══ RANH GIỚI: VÙNG DMZ ════╗
   ║  WAF  ──►  API Gateway (xác thực, rate limit)   ║
   ╚════════════════════╤════════════════════════════╝
                        │ mTLS
   ╔════════════════════▼═══ RANH GIỚI: VÙNG ỨNG DỤNG ═══════════════╗
   ║  transfer-api ──► fraud-engine ──► core-adapter                 ║
   ║       │                                  │                      ║
   ║       └──► [Kho: idempotency-store]      │                      ║
   ║                                          │                      ║
   ║            [Hàng đợi: napas-outbound] ◄──┘                      ║
   ║                     │                                            ║
   ║              napas-sender ──► HSM (ký)                           ║
   ╚═════════════════════╤═══════════════════════════════════════════╝
                         │ leased line + mTLS
   ╔═════════════════════▼═══ RANH GIỚI: ĐỐI TÁC ════╗
   ║  NAPAS                                          ║
   ╚═════════════════════════════════════════════════╝

   ╔═══ RANH GIỚI: VÙNG QUẢN TRỊ ════╗
   ║  Jump host / PAM ──► mọi vùng   ║  ◄── ranh giới hay bị quên nhất
   ╚═════════════════════════════════╝
```

#### 4. Bảng mối đe dọa (rút gọn — 10 dòng quan trọng nhất)

| # | Phần tử | STRIDE | Kịch bản cụ thể | KN | AH | Mức | Xử lý | Control |
|---|---|---|---|:--:|:--:|:--:|---|---|
| T1 | App → Gateway | **S** | Kẻ tấn công có được token phiên qua malware trên máy khách, gọi `/transfer` thay khách | TB | Cao | **Cao** | Giảm thiểu | Ràng buộc thiết bị; xác thực sinh trắc học lại theo QĐ 2345 khi vượt ngưỡng; phát hiện root/jailbreak; token thời hạn ngắn |
| T2 | `transfer-api` | **T** | Sửa `amount` hoặc `toAccount` sau bước xác thực OTP (OTP không ràng buộc nội dung giao dịch) | TB | **Cao** | **Cao** | Giảm thiểu | OTP ký trên **hash của nội dung giao dịch** (what-you-see-is-what-you-sign); server không nhận lại amount từ client sau bước xác thực |
| T3 | `transfer-api` → core | *(ngoài STRIDE)* | Client/LB/mesh retry khi timeout → **2 bút toán** cho 1 lệnh | **Cao** | **Cao** | **Nghiêm trọng** | Giảm thiểu | `Idempotency-Key` bắt buộc, lưu ở `idempotency-store` với TTL > timeout dài nhất; core adapter kiểm tra trước khi ghi; **tắt retry tự động ở tầng mesh cho route này** |
| T4 | `fraud-engine` | **D** | Fraud engine quá tải/chết → hệ thống "fail open" cho qua hết | TB | **Cao** | **Cao** | Giảm thiểu | **Fail closed** + hàng đợi chờ + cảnh báo; có kill-switch thủ công phải 2 người phê duyệt |
| T5 | Mọi service | **E** | Container chạy privileged/root, một RCE trong `transfer-api` → chiếm node → chạm HSM client | Thấp | **Cao** | **Cao** | Giảm thiểu | Admission policy chặn privileged/root/hostPath; NetworkPolicy default-deny; `napas-sender` chạy namespace riêng, không cùng node pool |
| T6 | Nhật ký | **R** | Quản trị viên có quyền xóa/sửa log để che dấu vết | Thấp | **Cao** | **TB** | Giảm thiểu | Log đẩy sang SIEM **ngoài tầm với** của quản trị hệ thống; chỉ-ghi-thêm; đối chiếu số lượng bản ghi |
| T7 | Nhật ký | **I** | Log ghi đầy đủ số tài khoản + số dư → lộ PII qua hệ thống log dùng chung | **Cao** | TB | **Cao** | Giảm thiểu | Che dữ liệu khi ghi log (thư viện chuẩn); kiểm tra tự động trong CI phát hiện log PII; phân quyền truy cập log |
| T8 | `napas-outbound` | **T** | Sửa bản tin trong hàng đợi trước khi ký | Thấp | **Cao** | **TB** | Giảm thiểu | Ký **tại nguồn** trước khi vào hàng đợi; xác minh chữ ký ở `napas-sender`; ACL trên hàng đợi |
| T9 | Job đối soát EOD | *(ngoài STRIDE)* | Job đối soát chết âm thầm → lệch không ai phát hiện trong nhiều ngày | TB | **Cao** | **Cao** | Giảm thiểu | Cảnh báo khi job **không chạy** (dead-man's switch), không chỉ khi job lỗi; ngưỡng lệch tự động báo |
| T10 | Vùng quản trị | **E** | DevOps dùng quyền production tự thực hiện trọn vẹn một giao dịch (vi phạm SoD) | Thấp | **Cao** | **Cao** | Giảm thiểu | Quyền JIT có hạn qua PAM; ghi hình phiên; cấm truy cập DB production trực tiếp; cảnh báo hành vi bất thường của chính người vận hành |

> Chú ý phân bố: **T3, T4, T9 không đến từ STRIDE** mà từ hiểu biết nghiệp vụ tài chính.
> Đây chính là phần khiến threat model của bạn khác với threat model do một AppSec thuần túy làm.

#### 5. Rủi ro chấp nhận
| # | Rủi ro tồn dư | Lý do chấp nhận | Người ký | Hạn xem lại |
|---|---|---|---|---|
| A1 | Chưa phát hiện được deepfake ở bước sinh trắc học | Giải pháp liveness nâng cao đang đấu thầu; tạm bù bằng hạn mức thấp hơn cho khách mới | Giám đốc khối Kênh số | 30/06/2026 |

#### 6. Việc phải làm
| # | Việc | Ticket | Người | Hạn |
|---|---|---|---|---|
| 1 | Tắt retry tự động ở service mesh cho route `/transfer` (T3) | SEC-1041 | <tên> | 2 tuần |
| 2 | Chuyển fraud engine sang fail-closed + kill-switch 2 người duyệt (T4) | SEC-1042 | <tên> | 3 tuần |
| 3 | Thêm kiểm tra CI phát hiện log chứa PII (T7) | SEC-1043 | <tên> | 4 tuần |
| 4 | Dead-man's switch cho job đối soát EOD (T9) | SEC-1044 | <tên> | 2 tuần |

---

## Mẹo để threat model không thành "tài liệu chết"

```text
1. LÀM CÙNG DEV, đừng làm một mình rồi gửi kết quả. Buổi 90 phút có dev + kiến trúc sư + bạn.
   Giá trị lớn nhất của threat model là CUỘC HỘI THOẠI, không phải file.
2. RA TICKET NGAY trong buổi đó. Không có ticket = không có gì xảy ra.
3. GIỚI HẠN 10 MỐI ĐE DỌA. Bảng 60 dòng không ai đọc và không ai sửa.
4. XEM LẠI khi kiến trúc đổi. Threat model của kiến trúc cũ tệ hơn không có, vì tạo cảm giác an toàn giả.
5. LƯU VÀO GIT cạnh code, không để trong wiki/ổ chia sẻ. Nó phải đổi cùng code.
```
