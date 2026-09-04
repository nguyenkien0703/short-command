# Lộ trình Senior DevSecOps → Lead DevSecOps (domain Tài chính — Ngân hàng/Fintech/Thanh toán)

> Tài liệu này trả lời đúng 3 câu hỏi, không nói chung chung:
> **(1)** Senior DevSecOps trong ngành tài chính *cụ thể* phải làm được gì?
> **(2)** Lead DevSecOps khác Senior ở chỗ nào — và làm sao chứng minh mình đã ở cấp đó?
> **(3)** Nghiệp vụ tài chính nào bắt buộc phải hiểu, vì nó **đổi cách bạn thiết kế security**?
>
> Đọc kèm: [`devsecops.md`](./devsecops.md) (nền kỹ thuật) · [`../career-roadmap.md`](../career-roadmap.md) (khung thăng tiến chung) · [`sre.md`](./sre.md)

## 📑 Mục lục
- [0. Đọc tài liệu này thế nào](#0-đọc-tài-liệu-này-thế-nào)
- [1. Bạn ngồi ở đâu trong một tổ chức tài chính](#1-bạn-ngồi-ở-đâu-trong-một-tổ-chức-tài-chính-quan-trọng-nhất)
- [2. Nghiệp vụ tài chính tối thiểu phải hiểu](#2-nghiệp-vụ-tài-chính-tối-thiểu-phải-hiểu)
- [3. Bản đồ tuân thủ → control kỹ thuật → bằng chứng](#3-bản-đồ-tuân-thủ--control-kỹ-thuật--bằng-chứng)
- [4. Ba cấp: định nghĩa dứt khoát](#4-ba-cấp-định-nghĩa-dứt-khoát)
- [5. Senior DevSecOps — làm được gì thì gọi là Senior](#5-senior-devsecops--làm-được-gì-thì-gọi-là-senior)
- [6. Lead DevSecOps — làm được gì thì gọi là Lead](#6-lead-devsecops--làm-được-gì-thì-gọi-là-lead)
- [7. Kế hoạch 24 tháng theo quý (có deliverable & số đo)](#7-kế-hoạch-24-tháng-theo-quý-có-deliverable--số-đo)
- [8. Bộ chỉ số (KPI) của DevSecOps trong ngân hàng](#8-bộ-chỉ-số-kpi-của-devsecops-trong-ngân-hàng)
- [9. Hồ sơ bằng chứng để được thăng cấp](#9-hồ-sơ-bằng-chứng-để-được-thăng-cấp)
- [10. Chứng chỉ: thứ tự ưu tiên và lý do](#10-chứng-chỉ-thứ-tự-ưu-tiên-và-lý-do)
- [11. Kỹ năng "chính trị tổ chức" đặc thù ngân hàng](#11-kỹ-năng-chính-trị-tổ-chức-đặc-thù-ngân-hàng)
- [12. Mẫu văn bản bạn phải viết được](#12-mẫu-văn-bản-bạn-phải-viết-được)
- [13. 90 ngày đầu khi vào một tổ chức tài chính](#13-90-ngày-đầu-khi-vào-một-tổ-chức-tài-chính)
- [14. Cạm bẫy đặc thù ngành tài chính](#14-cạm-bẫy-đặc-thù-ngành-tài-chính)
- [15. Tài nguyên](#15-tài-nguyên)

---

## 0. Đọc tài liệu này thế nào

Ba thứ tách bạch, đừng trộn:

```text
NĂNG LỰC KỸ THUẬT   -> làm được việc gì bằng tay (phần 5, 6)
NGHIỆP VỤ TÀI CHÍNH -> hiểu tiền chạy thế nào & rủi ro ở đâu (phần 2)   <-- đây là thứ tạo khác biệt
QUYỀN & PHẠM VI     -> bạn quyết định được gì, ai nghe bạn (phần 1, 4, 6)
```

Sự thật phũ phàng: trong ngân hàng, **người giỏi kỹ thuật nhất thường KHÔNG lên Lead**. Lead thuộc về người
*nói được ngôn ngữ rủi ro*, *đứng được trước kiểm toán*, và *làm cho 5 team dev khác đi nhanh hơn mà vẫn tuân thủ*.
Kỹ thuật là điều kiện cần, không phải điều kiện đủ.

---

## 1. Bạn ngồi ở đâu trong một tổ chức tài chính (quan trọng nhất)

Ngân hàng/công ty tài chính vận hành theo mô hình **Ba tuyến phòng vệ (Three Lines of Defense)**. Không hiểu cái này
thì bạn sẽ liên tục "va" vào người khác mà không biết vì sao.

```text
TUYẾN 1 — Vận hành & sở hữu rủi ro (bạn ở đây)
   Khối CNTT / Trung tâm hạ tầng / Platform / DevSecOps / App teams
   -> TỰ nhận diện, TỰ kiểm soát rủi ro trong hệ thống mình vận hành.

TUYẾN 2 — Giám sát rủi ro & tuân thủ (Risk / Compliance / Information Security Officer)
   Ban Quản lý rủi ro hoạt động, phòng An toàn thông tin (ISO), Tuân thủ (Compliance), DPO
   -> ĐẶT chuẩn, THẨM ĐỊNH, PHÊ DUYỆT ngoại lệ, báo cáo NHNN.

TUYẾN 3 — Kiểm toán nội bộ (Internal Audit) + Kiểm toán độc lập/QSA/NHNN thanh tra
   -> KIỂM TRA cả tuyến 1 và 2. Ra "audit finding" có thời hạn khắc phục.
```

**Hệ quả trực tiếp lên sự nghiệp của bạn:**

| Điều bạn hay hiểu sai | Thực tế trong ngân hàng |
|---|---|
| "Security team là tuyến 2, tôi làm DevSecOps nên tôi là security" | DevSecOps thường ở **tuyến 1**. Bạn *thực thi* chuẩn, tuyến 2 *đặt* chuẩn. Muốn đổi chuẩn phải thuyết phục tuyến 2. |
| "Tôi chặn build có CVE Critical là xong" | Chặn mà không có **quy trình ngoại lệ + risk acceptance có chủ sở hữu và hạn** thì business sẽ vượt mặt bạn, hoặc bạn thành kẻ cản trở. |
| "Audit finding là việc của Compliance" | Finding thuộc **hệ thống bạn vận hành** = việc của bạn. Đóng finding đúng hạn là KPI được nhìn thấy tận cấp Ban điều hành. |
| "Tôi cứ làm tốt kỹ thuật rồi được ghi nhận" | Ở ngân hàng, thứ được ghi nhận là **rủi ro được giảm, đo được, có bằng chứng**. Việc không có bằng chứng = việc không tồn tại. |

> 🔑 **Đòn bẩy lớn nhất để lên Lead:** trở thành người *duy nhất* nói được cả hai ngôn ngữ — ngôn ngữ pipeline (tuyến 1)
> và ngôn ngữ rủi ro/điều khoản (tuyến 2). Người đó tự động thành đầu mối, và đầu mối thì thành Lead.

**Phân biệt các vai lân cận (đừng nhận nhầm việc):**

```text
DevSecOps Engineer  : security TRONG pipeline & hạ tầng. Guardrail, automation, golden path.
AppSec Engineer     : security TRONG code ứng dụng. Threat model, code review sâu, pentest hỗ trợ dev.
SecOps / SOC        : phát hiện & phản ứng (SIEM, IR, threat hunting). Trực 24/7.
GRC / Compliance    : khung, chính sách, ánh xạ điều khoản, làm việc với NHNN/QSA/kiểm toán.
Security Architect  : thiết kế mô hình bảo mật cho hệ thống mới, quyết định one-way-door.
Fraud / Risk tech   : chống gian lận giao dịch (rất riêng của tài chính — không phải infosec truyền thống).
```
Lead DevSecOps phải **giao tiếp trôi chảy với cả 5 vai trên**, và thường kiêm một phần Security Architect.

---

## 2. Nghiệp vụ tài chính tối thiểu phải hiểu

Đây là phần bạn nói bị "mù mờ". Nói thẳng: **nghiệp vụ tài chính không phải trang trí CV — nó đổi cách bạn threat model.**

### 2.1 Dòng tiền — học theo "tiền đi đâu"

```text
KÊNH KHÁCH HÀNG          KÊNH XỬ LÝ                       KÊNH THANH TOÁN LIÊN NGÂN HÀNG
Mobile App / Internet  ┐                              ┌─ NAPAS (chuyển nhanh 24/7, thẻ nội địa)
Web / USSD / ATM / POS ├─> API Gateway / Channel ─> Core Banking ─┼─ IBPS/CITAD (NHNN, giá trị lớn)
Ví điện tử / QR (VietQR)┘        │                    (sổ cái)    ├─ SWIFT (quốc tế)
                                 ├─> Card Switch / Issuer ──────> Visa/Mastercard/JCB
                                 ├─> Payment Gateway (merchant)
                                 ├─> LOS/LMS (cho vay), eKYC, CIC
                                 └─> Data Warehouse / Báo cáo NHNN / AML
```

Bạn phải trả lời được, cho hệ thống mình phụ trách:
1. **Giao dịch đi qua những hop nào?** Hop nào chạm tiền thật, hop nào chỉ đọc?
2. **Ở hop nào dữ liệu nhạy cảm xuất hiện ở dạng rõ?** (PAN thẻ, CVV, PIN block, OTP, sinh trắc học, CCCD)
3. **Nếu hop đó bị chiếm, kẻ tấn công rút được tiền hay chỉ đọc được dữ liệu?** → đó là blast radius thật.
4. **Ai đối soát và bao lâu một lần?** Đối soát (reconciliation) là "phao cứu sinh" cuối cùng của ngân hàng.

### 2.2 Từ vựng nghiệp vụ bắt buộc thuộc

| Khái niệm | Nghĩa ngắn | Vì sao DevSecOps phải quan tâm |
|---|---|---|
| **Core banking** | Hệ thống sổ cái, tài khoản (T24, Flexcube, tự xây) | Vùng cấm. Change rất chậm, cửa sổ bảo trì hẹp, ai chạm cũng phải có phê duyệt. |
| **Đối soát (reconciliation)** | So khớp giao dịch giữa các hệ thống/đối tác cuối ngày | Log & audit trail của bạn là đầu vào đối soát. Mất log = không đối soát được = tổn thất thật. |
| **Idempotency** | Gọi lại 1 request không tạo 2 giao dịch | Retry của bạn ở tầng infra/gateway có thể **nhân đôi tiền**. Đây là sự cố kinh điển. |
| **Non-repudiation** | Chống chối bỏ giao dịch | Cần log bất biến, ký số, đồng hồ chuẩn (NTP) — không phải "nice to have". |
| **Maker–Checker (dual control)** | Người làm ≠ người duyệt | Áp cho cả CI/CD: người push ≠ người approve deploy production. |
| **SoD (Segregation of Duties)** | Tách quyền để không ai làm trọn chuỗi | Dev không được có quyền prod; người cấp quyền ≠ người dùng quyền. Kiểm toán soi mục này đầu tiên. |
| **Break-glass** | Quyền khẩn cấp có kiểm soát | Phải có: phê duyệt, thời hạn tự hết, ghi log toàn bộ phiên, hậu kiểm bắt buộc. |
| **CDE (Cardholder Data Environment)** | Vùng chứa/xử lý dữ liệu thẻ | Phân vùng mạng đúng = giảm phạm vi PCI DSS = tiết kiệm hàng tỉ chi phí audit. **Đây là đòn bẩy business rõ nhất của bạn.** |
| **Tokenization / masking** | Thay PAN bằng token, che số thẻ | Cách hợp pháp để "kéo" hệ thống ra khỏi phạm vi PCI. |
| **HSM** | Thiết bị phần cứng giữ khóa & xử lý PIN | Khóa thanh toán KHÔNG nằm trong Vault/KMS phần mềm. Hiểu ranh giới này. |
| **AML / KYC / eKYC / CIC** | Phòng chống rửa tiền, định danh KH | Sinh ra yêu cầu lưu trữ, truy vết, và dữ liệu cực nhạy (sinh trắc học). |
| **RTO / RPO** | Thời gian & dữ liệu được phép mất khi thảm họa | NHNN & nội bộ ấn định cứng; diễn tập DR định kỳ là nghĩa vụ, không phải dự án. |
| **Change freeze** | Đóng băng thay đổi (Tết, cuối quý/năm, ngày lễ, sự kiện lớn) | Kế hoạch triển khai của bạn phải né các cửa sổ này. Không biết = trượt tiến độ. |
| **RCSA / KRI / risk register** | Tự đánh giá rủi ro, chỉ số rủi ro chính, sổ rủi ro | Cách tuyến 2 "nhìn" công việc của bạn. Đưa được control của mình vào đây = có quyền lực. |
| **Residual risk / risk acceptance** | Rủi ro còn lại sau kiểm soát, và việc chấp nhận nó | Bạn không "cấm" — bạn *trình bày rủi ro còn lại* để chủ sở hữu rủi ro ký chấp nhận. |

### 2.3 Threat model đặc thù tài chính (khác hẳn công ty internet)

```text
1. GIAN LẬN GIAO DỊCH (fraud) — kẻ tấn công không cần hack hệ thống, chỉ cần lừa khách hàng.
   -> Bạn góp phần: chống chiếm đoạt phiên, ràng buộc thiết bị, log đủ để đội fraud phân tích.
2. NỘI BỘ (insider) — rủi ro số 1 bị đánh giá thấp nhất. Admin DB, DevOps có quyền prod.
   -> SoD, PAM, ghi hình phiên, quyền JIT, cảnh báo hành vi bất thường của chính người vận hành.
3. CHUỖI CUNG ỨNG — thư viện, image, vendor tích hợp, đối tác thanh toán.
   -> SBOM, ký artifact, đánh giá bảo mật nhà cung cấp (third-party risk) — ngân hàng bắt buộc làm.
4. TẤN CÔNG VÀO KÊNH THANH TOÁN — SWIFT/NAPAS/card switch. Ít nhưng thiệt hại thảm khốc.
   -> Vùng cách ly, jump host, giám sát riêng, không dùng chung công cụ với vùng thường.
5. RÒ RỈ DỮ LIỆU KHÁCH HÀNG — bây giờ có chế tài pháp lý thật (xem phần 3).
   -> Phân loại dữ liệu, mã hóa, masking ở non-prod, DLP, kiểm soát export từ DWH.
6. SẴN SÀNG DỊCH VỤ — downtime ngân hàng lên báo, NHNN hỏi, mất uy tín.
   -> Bảo mật KHÔNG được đánh đổi bằng downtime. Guardrail phải fail-safe, không fail-closed mù quáng.
```

> ⚠️ Điểm này phân biệt Senior thật với "Senior tool": một guardrail chặn nhầm lúc 9h sáng ngày lương
> gây thiệt hại lớn hơn CVE mà nó chặn. **Mọi control bạn bật phải có: chế độ cảnh báo trước khi chặn,
> kill-switch, và người trực.**

---

## 3. Bản đồ tuân thủ → control kỹ thuật → bằng chứng

Đây là "bảng dịch" bạn nên thuộc. Nó biến điều khoản pháp lý thành việc bạn build được.

### 3.1 Khung áp dụng tại Việt Nam

| Văn bản | Phạm vi | Điều bạn phải nhớ |
|---|---|---|
| **TT 09/2020/TT-NHNN** (hiệu lực 01/01/2021) | Tổ chức tín dụng, trung gian thanh toán, NAPAS, CIC... | Yêu cầu **tối thiểu** về ATTT: phân loại hệ thống theo cấp độ, quản lý tài sản, kiểm soát truy cập, nhật ký, sao lưu, an toàn mạng, phòng chống mã độc, quản lý thay đổi. Đây là nền pháp lý cho gần như mọi control bạn làm. |
| **TT 50/2024/TT-NHNN** (hiệu lực 01/01/2025, sửa đổi bởi **TT 77/2025**) | An toàn, bảo mật cho **dịch vụ trực tuyến** ngành ngân hàng | Xác thực khách hàng (OTP + sinh trắc học khi thiết bị lạ/lần đầu), tối thiểu 5 biện pháp bảo vệ dữ liệu khách hàng (Điều 19), cấm gửi SMS/email chứa hyperlink (trừ theo yêu cầu KH). Ảnh hưởng trực tiếp thiết kế kênh số. |
| **Luật Bảo vệ dữ liệu cá nhân số 91/2025/QH15** (hiệu lực **01/01/2026**) | Toàn bộ tổ chức xử lý DLCN | Nâng khung của NĐ 13/2023 lên cấp Luật: quyền của chủ thể dữ liệu (biết, đồng ý, truy cập, chỉnh sửa, xóa), nghĩa vụ của bên kiểm soát/xử lý, đánh giá tác động, thông báo vi phạm. → Sinh ra việc thật: **phân loại dữ liệu, mask ở non-prod, quản lý retention, quy trình xóa theo yêu cầu, log truy cập DLCN**. |
| **Luật An toàn thông tin mạng · Luật An ninh mạng · NĐ 53/2022** | Chung | Phân loại cấp độ hệ thống, lưu trữ dữ liệu trong nước với một số loại dữ liệu/dịch vụ. Ảnh hưởng lựa chọn cloud & vùng lưu trữ. |

> 📌 Pháp lý thay đổi nhanh (TT 50 đã có sửa đổi sau 1 năm). **Luôn kiểm tra bản mới nhất trên
> thuvienphapluat.vn / sbv.gov.vn trước khi trích dẫn trong tài liệu thiết kế.** Nêu sai số hiệu văn bản
> trước mặt tuyến 2 làm mất uy tín rất nhanh.

### 3.2 Khung quốc tế

| Khung | Khi nào áp | Điểm chạm DevSecOps |
|---|---|---|
| **PCI DSS v4.0.1** | Có xử lý/lưu/truyền dữ liệu thẻ | **Req 6** là sân nhà của bạn: 6.2.x phát triển an toàn + đào tạo dev; **6.3.1** duy trì *inventory* phần mềm tự phát triển và **thành phần bên thứ ba** (≈ SBOM, bắt buộc từ 31/03/2025); 6.3.2/6.3.3 vá lỗ hổng theo mức độ; **6.4.2** giải pháp tự động (WAF) cho web public-facing. Thêm Req 7/8 (quyền, MFA), 10 (log), 11 (quét & pentest), 12 (chính sách). **Phân vùng mạng để thu hẹp CDE = giá trị business lớn nhất bạn tạo ra.** |
| **SWIFT CSP – CSCF v2026** | Có kết nối SWIFT | v2026: **26 control bắt buộc / 6 khuyến nghị**; **Control 2.4 (Back Office Data Flow Security) chuyển từ khuyến nghị → BẮT BUỘC**, mở rộng phạm vi ra middleware/file transfer/back-office; connector khách hàng vào phạm vi; siết yêu cầu mã hóa & độ dài khóa. Đánh giá độc lập hằng năm. |
| **ISO/IEC 27001:2022** | Chứng nhận hệ thống quản lý ATTT | Annex A 8.25–8.31 nói thẳng về vòng đời phát triển an toàn, tách môi trường, kiểm thử. Đây là nơi bạn "gắn" pipeline vào ISMS. |
| **SOC 2 Type II** | Bán dịch vụ cho đối tác/nước ngoài | Chấm điểm **tính nhất quán của control qua thời gian** → ép bạn tự động hóa thu thập bằng chứng, không làm thủ công trước kỳ audit. |
| **NIST SSDF (SP 800-218)** | Khung tham chiếu SDLC an toàn | Ngôn ngữ chung tốt nhất để trình bày pipeline của bạn với tuyến 2. Dùng để map: PO/PS/PW/RV. |
| **DORA (EU)** | Nếu có pháp nhân/khách hàng EU | Chống chịu vận hành số: quản lý rủi ro ICT, báo cáo sự cố, kiểm thử chống chịu, rủi ro bên thứ ba. |
| **CIS Benchmarks / NIST 800-53** | Chuẩn cấu hình & control | Nguồn để viết policy-as-code (Kyverno/OPA) và baseline hardening. |

### 3.3 Bảng dịch: điều khoản → bạn build gì → kiểm toán hỏi gì

| Yêu cầu (nguồn) | Control kỹ thuật bạn xây | Bằng chứng phải xuất được |
|---|---|---|
| Phát triển an toàn (PCI 6.2, ISO A.8.25, SSDF PW) | SAST + secret scan + SCA bắt buộc trong CI, chặn theo ngưỡng | Cấu hình pipeline (as code), log 90 ngày build bị chặn, tỉ lệ repo có gate |
| Inventory phần mềm & thành phần (PCI 6.3.1) | Sinh **SBOM** (Syft/CycloneDX) mỗi build, lưu tập trung, đối chiếu CVE liên tục | Kho SBOM có thể truy vấn "dịch vụ nào đang dùng log4j 2.14?" trong < 5 phút |
| Vá lỗ hổng theo mức độ (PCI 6.3.3) | SLA vá theo severity + dashboard tuổi lỗ hổng + tự tạo ticket | Báo cáo MTTR theo severity, danh sách quá hạn + risk acceptance kèm chữ ký |
| Chỉ chạy phần mềm tin cậy (SLSA, ISO A.8.28) | Ký image (cosign) + admission control chỉ cho image đã ký & đã quét | Policy Kyverno đang enforce + log từ chối deploy |
| Kiểm soát truy cập, SoD, dual control (TT 09, PCI 7/8) | RBAC least-privilege, JIT access, PAM, tách người push/người approve | Ma trận quyền, log phê duyệt deploy prod, báo cáo rà soát quyền định kỳ |
| Nhật ký & giám sát (TT 09, PCI 10) | Log tập trung, bất biến, đồng bộ NTP, retention theo quy định, use case SIEM | Truy vết đầy đủ một thay đổi prod: ai, khi nào, phê duyệt bởi ai, rollback ra sao |
| Bảo vệ dữ liệu cá nhân (Luật 91/2025, TT 50 Đ.19) | Phân loại dữ liệu, mã hóa at-rest/in-transit, **masking dữ liệu ở non-prod**, kiểm soát export | Sơ đồ luồng dữ liệu, chứng minh non-prod không chứa dữ liệu thật |
| Sao lưu & DR (TT 09) | Backup có mã hóa, kiểm thử restore định kỳ, IaC dựng lại vùng | Biên bản diễn tập restore/DR gần nhất + thời gian đạt được vs RTO/RPO |
| Quản lý thay đổi (TT 09, ISO) | CI/CD gắn ticket, phê duyệt tự động hóa, rollback tự động | Chuỗi liên kết ticket → commit → build → approve → deploy |
| Bảo mật luồng back-office (SWIFT 2.4) | Phân đoạn mạng, mã hóa file transfer, kiểm tra toàn vẹn | Sơ đồ phân vùng + cấu hình mã hóa + kết quả đánh giá độc lập |

> 💡 **Nguyên tắc vàng:** thiết kế control sao cho **bằng chứng tự sinh ra** (từ pipeline, từ policy engine),
> không phải đi thu thập thủ công trước kỳ audit. Đây chính là "Continuous Control Monitoring" — và là
> deliverable ghi điểm mạnh nhất ở cấp Senior→Lead.

---

## 4. Ba cấp: định nghĩa dứt khoát

```text
MIDDLE DevOps/DevSecOps -> tối ưu: LÀM XONG control được giao, đúng chuẩn có sẵn.
SENIOR DevSecOps        -> tối ưu: SỞ HỮU một miền rủi ro end-to-end, tự ra quyết định đánh đổi,
                                   không cần ai dắt, và các team dev tin dùng thứ bạn làm.
LEAD DevSecOps          -> tối ưu: KẾT QUẢ QUA NGƯỜI KHÁC + đại diện kỹ thuật trước tuyến 2/tuyến 3.
                                   Bạn quyết chuẩn, ưu tiên, ngân sách công cụ, và chịu trách nhiệm
                                   trạng thái tuân thủ của toàn bộ pipeline tổ chức.
```

| Tiêu chí | Senior DevSecOps | Lead DevSecOps |
|---|---|---|
| Phạm vi | 1–2 miền (VD: pipeline security + container security) cho vài hệ thống | Toàn bộ SDLC của nhiều team/khối; sở hữu "golden path" |
| Bài toán | Rõ ràng hoặc bán mơ hồ: "làm cho tất cả repo có SBOM" | Mơ hồ hoàn toàn: "giảm rủi ro ATTT của kênh số mà không làm chậm ra sản phẩm" |
| Quyết định | Chọn công cụ/thiết kế control trong miền của mình | Chọn **chiến lược**: build vs buy, đánh đổi tốc độ/rủi ro, thứ tự ưu tiên, dừng cái gì |
| Với tuyến 2 | Trả lời được câu hỏi của họ | **Đàm phán** chuẩn với họ; đề xuất sửa chuẩn khi chuẩn sai thực tế |
| Với kiểm toán | Cung cấp bằng chứng khi được hỏi | **Đứng tên** trước finding, cam kết kế hoạch khắc phục, bảo vệ phương án |
| Với sự cố | Xử lý kỹ thuật, tìm root cause | **Incident commander** cho sự cố bảo mật; quyết định "ngắt dịch vụ hay chịu rủi ro" |
| Với con người | Mentor 1–2 người | Xây đội (2–8 người), phân việc, đánh giá, tuyển, phát triển năng lực; nâng năng lực security cho **dev** |
| Với tiền | Biết chi phí công cụ mình dùng | Đề xuất & bảo vệ **ngân sách**; tính ROI ("thu hẹp CDE giảm X phạm vi audit") |
| Đơn vị đo | Control chạy được, lỗ hổng giảm | Rủi ro tồn dư toàn tổ chức giảm, audit finding đóng đúng hạn, dev velocity không giảm |
| Câu hỏi tự kiểm | "Tôi có tự giải quyết được không cần hỏi ai?" | "Nếu tôi nghỉ 1 tháng, hệ thống chuẩn & con người có tự chạy đúng không?" |

**Ranh giới thật sự giữa hai cấp — 3 dấu hiệu:**
1. Bạn được **hỏi ý kiến trước khi tổ chức ra quyết định lớn** (chọn cloud, mở kênh mới, mua nền tảng), chứ không phải được thông báo sau.
2. Có ít nhất một chuẩn/nền tảng **mang tên bạn** mà nhiều team đang dùng hằng ngày, và nó chạy tốt kể cả khi bạn nghỉ phép.
3. Bạn **nói không** với một yêu cầu của business kèm phương án thay thế, và tổ chức chấp nhận lập luận của bạn.

---

## 5. Senior DevSecOps — làm được gì thì gọi là Senior

### 5.1 Năng lực kỹ thuật (checklist tự chấm 1–5)

```text
A. BẢO MẬT PIPELINE (bắt buộc, phải ở mức 5)
[ ] Thiết kế được gate SAST/SCA/secret/IaC/container cho một tổ chức, KHÔNG chỉ cho 1 repo:
    - ngưỡng chặn theo severity + theo mức độ nhạy của hệ thống (kênh số ≠ trang tin nội bộ)
    - cơ chế "cảnh báo trước, chặn sau" (baseline period) để không làm sập luồng dev đang chạy
    - allowlist/suppression có HẠN và có CHỦ SỞ HỮU, tự hết hạn
[ ] Xử lý được vấn đề số 1 của mọi triển khai security: TIẾNG ỒN (false positive).
    Triage, tuning rule, ưu tiên theo khả năng khai thác thật (reachability), không đổ hết cho dev.
[ ] Bảo mật CHÍNH pipeline (không chỉ dùng pipeline để quét): quyền của runner, tách runner
    prod/non-prod, bảo vệ branch, ký commit, chống pipeline poisoning & dependency confusion.

B. CHUỖI CUNG ỨNG
[ ] SBOM tự động mỗi build (Syft/CycloneDX), lưu tập trung, truy vấn được theo CVE — PCI 6.3.1.
[ ] Ký artifact/image (cosign) + verify ở admission (Kyverno/OPA) → chỉ image tin cậy được chạy.
[ ] Base image nội bộ đã hardened + quy trình vá & ép nâng cấp toàn tổ chức khi có CVE nóng.
[ ] Quy trình phản ứng "log4shell": trả lời trong 1 giờ "hệ thống nào bị ảnh hưởng" — DIỄN TẬP trước.

C. HẠ TẦNG & KUBERNETES
[ ] Policy-as-code enforce: no privileged, no hostPath, read-only rootfs, resource limit, image tin cậy.
[ ] NetworkPolicy default-deny + phân vùng mạng phục vụ THU HẸP PCI SCOPE (nói được bằng tiền).
[ ] mTLS/workload identity (SPIFFE/mesh), bỏ static credential giữa các service.
[ ] Runtime detection (Falco/Tetragon) có use case thật, tinh chỉnh đủ để không spam.

D. QUẢN LÝ BÍ MẬT & MÃ HÓA (tài chính soi rất kỹ)
[ ] Vault/KMS: dynamic secret, rotation tự động, không secret tĩnh trong CI.
[ ] Hiểu ranh giới HSM: khóa thanh toán/PIN nằm ở đâu, ai được dùng, quy trình key ceremony.
[ ] Mã hóa at-rest/in-transit + quản lý vòng đời khóa + kịch bản lộ khóa (rotate mà không downtime).
[ ] Masking/tokenization để non-prod KHÔNG chứa dữ liệu khách hàng thật.

E. TRUY CẬP & DANH TÍNH
[ ] Least privilege thật (không phải "*"), rà soát quyền định kỳ có bằng chứng.
[ ] JIT/just-in-time access + break-glass có hạn, có hậu kiểm.
[ ] SoD trong CI/CD: người viết code ≠ người duyệt deploy prod. Enforce bằng công cụ, không bằng niềm tin.

F. PHÁT HIỆN & ỨNG CỨU
[ ] Log tập trung, bất biến, retention đúng quy định; audit trail đủ để dựng lại một giao dịch.
[ ] Viết được use case phát hiện (không chỉ cài SIEM): VD "service account prod gọi API lạ lúc 3h sáng".
[ ] Tham gia IR thật: containment, forensic cơ bản, viết postmortem không đổ lỗi.

G. NGHIỆP VỤ & TUÂN THỦ
[ ] Threat model (STRIDE) cho một luồng giao dịch thật, ra được danh sách control ưu tiên.
[ ] Ánh xạ được control ↔ điều khoản (TT 09/TT 50/PCI/ISO) mà không cần hỏi Compliance.
[ ] Viết được risk acceptance memo đúng ngôn ngữ tuyến 2 (xem phần 12).
```

### 5.2 Bằng chứng "đã ở cấp Senior" (chọn 3–4 cái, làm cho xong, đo được)

| Dự án | Số đo để chứng minh |
|---|---|
| Golden pipeline bảo mật áp cho ≥10 repo | % repo có gate, thời gian build tăng thêm < X phút, số secret rò rỉ giảm về 0 |
| Nền tảng SBOM + trả lời CVE nóng | Thời gian trả lời "ai bị ảnh hưởng": từ vài ngày → < 1 giờ |
| Ký & verify image toàn cluster | 100% workload prod chạy image đã ký; số deploy bị chặn/tháng |
| Thu hẹp phạm vi PCI bằng phân vùng + tokenization | Số hệ thống ra khỏi CDE, ước tính chi phí/công sức audit giảm |
| Xóa dữ liệu thật khỏi non-prod | Số môi trường đã mask, số dataset nhạy cảm còn lại = 0 |
| Tự động thu thập bằng chứng audit | Giờ công chuẩn bị audit giảm từ N ngày → M giờ |

> Mỗi dự án → **1 ADR + 1 design doc** (dùng [`../templates/`](../templates/)). Không có tài liệu thì đến kỳ
> thăng cấp bạn không chứng minh được gì.

---

## 6. Lead DevSecOps — làm được gì thì gọi là Lead

### 6.1 Bốn thứ Lead sở hữu mà Senior không

**(1) Chiến lược & lộ trình bảo mật SDLC (12–18 tháng)**
Không phải danh sách công cụ. Là: rủi ro nào tổ chức đang chấp nhận → cái nào giảm trước → tốn bao nhiêu → đo bằng gì.
Trình bày được cho CIO/CISO trong 10 slide, và cho kiểm toán trong 3 trang.

**(2) Chuẩn & golden path cho toàn bộ dev**
Bạn không đi quét từng repo. Bạn làm cho **con đường an toàn trở thành con đường dễ nhất**:
template dịch vụ, thư viện chuẩn, pipeline dùng chung, môi trường có sẵn guardrail.
Chỉ số của Lead: *"tỉ lệ team đi trên golden path"*, không phải *"số lỗ hổng tôi tự sửa"*.

**(3) Quan hệ với tuyến 2, tuyến 3 và nhà cung cấp**
- Đàm phán chuẩn với ISO/Compliance; đề xuất sửa chuẩn lỗi thời bằng dữ liệu.
- Đứng tên khắc phục audit finding; cam kết mốc và giữ đúng mốc.
- Đánh giá & chọn công cụ (build vs buy), làm việc với vendor, kiểm soát rủi ro bên thứ ba.
- Hỗ trợ kỳ đánh giá PCI/SWIFT/ISO: chuẩn bị bằng chứng, dẫn phiên phỏng vấn kỹ thuật.

**(4) Con người**
Phân việc theo năng lực, mentor có lộ trình, tuyển & phỏng vấn, đánh giá hiệu suất, giữ người.
Và quan trọng nhất trong DevSecOps: **nâng năng lực bảo mật cho hàng trăm dev** (đào tạo secure coding —
cũng là yêu cầu của PCI 6.2.2), champion program, chứ không phải tự mình gánh.

### 6.2 Checklist "sẵn sàng lên Lead"

```text
[ ] Bạn đã dẫn dắt ít nhất 1 chương trình chạm ≥3 team và kéo dài ≥2 quý, có kết quả đo được.
[ ] Có 1 nền tảng/chuẩn do bạn thiết kế đang được dùng rộng, vận hành ổn khi bạn vắng mặt.
[ ] Bạn đã đứng trước kiểm toán/QSA/thanh tra và bảo vệ được thiết kế của mình.
[ ] Bạn đã làm incident commander (hoặc phó) cho ít nhất 1 sự cố bảo mật thật.
[ ] Bạn đã mentor ≥2 người lên rõ rệt, và đã tham gia phỏng vấn tuyển người.
[ ] Bạn đã từ chối một yêu cầu rủi ro cao kèm phương án thay thế, và tổ chức đi theo phương án của bạn.
[ ] Bạn trình bày được bằng NGÔN NGỮ TIỀN & RỦI RO, không phải bằng tên công cụ.
[ ] Lãnh đạo tìm bạn khi có quyết định lớn — TRƯỚC khi quyết, không phải sau.
[ ] Bạn có sponsor (cấp Giám đốc khối/CISO) sẵn sàng nói tốt về bạn trong phòng họp bổ nhiệm.
```

### 6.3 Cấu trúc một đội DevSecOps trưởng thành (để bạn biết mình đang xây gì)

```text
Lead DevSecOps
 ├─ Pipeline & Supply chain      : golden pipeline, SBOM, ký artifact, quản trị công cụ quét
 ├─ Cloud & Container security   : policy-as-code, hardening, phân vùng, posture (CSPM)
 ├─ Identity & Secrets           : IAM/PAM/Vault, SoD, rà soát quyền
 ├─ Detection & Response (phối hợp SOC): use case, runbook, diễn tập
 └─ Compliance automation        : ánh xạ control, thu thập bằng chứng tự động, dashboard tuân thủ
Vệ tinh: Security Champions trong từng team dev (bạn đào tạo & dẫn dắt, không quản lý trực tiếp)
```

---

## 7. Kế hoạch 24 tháng theo quý (có deliverable & số đo)

> Giả định điểm xuất phát: Middle/Senior DevOps vững K8s + cloud + CI/CD (đúng hồ sơ của bạn).
> Nếu đã làm DevSecOps rồi, bỏ Q1 và dồn lên.

### GIAI ĐOẠN 1 — Trở thành DevSecOps thực thụ (Q1–Q2)

```text
Q1 — Nền security + nghiệp vụ tài chính
  Học   : OWASP Top 10 · STRIDE threat modeling · least privilege/zero trust · đọc TT 09 & TT 50 (đọc THẬT,
          không đọc tóm tắt) · sơ đồ dòng tiền của chính tổ chức mình.
  Làm   : - Vẽ sơ đồ luồng dữ liệu + threat model cho 1 hệ thống bạn đang vận hành.
          - Nhúng secret scanning + SCA vào 2–3 pipeline (chế độ CẢNH BÁO trước).
          - Lập bảng ánh xạ: control hiện có ↔ điều khoản TT 09.
  Đo    : có 1 threat model được tuyến 2 đọc & góp ý; 0 secret mới bị commit từ tháng thứ 3.
  Viết  : ADR "chọn công cụ quét cho pipeline" (nêu rõ đánh đổi false positive vs coverage).

Q2 — Bảo mật pipeline ở quy mô + chuỗi cung ứng
  Học   : SBOM/SLSA · cosign · Kyverno/OPA · CIS Benchmark K8s.
  Làm   : - Chuyển gate từ cảnh báo → chặn theo severity, có quy trình ngoại lệ có hạn.
          - SBOM tự động + kho tra cứu CVE.
          - Ký image + admission verify trên môi trường non-prod, rồi prod.
  Đo    : ≥80% repo trọng yếu có gate; trả lời "ai dùng thư viện X" < 1 giờ.
  Viết  : design doc "Golden pipeline bảo mật" — trình lên cấp quản lý.
```

### GIAI ĐOẠN 2 — Senior DevSecOps (Q3–Q6)

```text
Q3 — Hạ tầng, danh tính, bí mật
  Làm   : - Policy-as-code enforce baseline toàn cluster (privileged, hostPath, image không ký...).
          - NetworkPolicy default-deny cho ≥1 namespace trọng yếu.
          - Vault: bỏ secret tĩnh trong CI, bật rotation cho ít nhất 1 loại credential quan trọng.
          - Rà soát & cắt quyền dư (bắt đầu bằng quyền prod của chính đội DevOps — làm gương).
  Đo    : số quyền admin prod giảm X%; 100% workload mới chạy non-root.
  Nghiệp vụ: ngồi cùng đội Fraud/Vận hành 1 buổi — hiểu họ điều tra một giao dịch nghi vấn thế nào.

Q4 — Dữ liệu & tuân thủ (đòn bẩy tài chính mạnh nhất)
  Làm   : - Phân loại dữ liệu + MASK toàn bộ non-prod (Luật 91/2025 + TT 50 Đ.19).
          - Đề xuất phương án thu hẹp CDE (phân vùng/tokenization) kèm ước tính chi phí audit giảm.
          - Tự động hóa thu thập bằng chứng cho 5–10 control hay bị hỏi nhất.
  Đo    : giờ chuẩn bị audit giảm ≥50%; số dataset thật ở non-prod = 0.
  Viết  : ADR "thu hẹp phạm vi PCI" — đây là tài liệu ghi điểm với cả CISO lẫn CFO.

Q5 — Phát hiện, ứng cứu & diễn tập
  Làm   : - 3–5 use case phát hiện gắn với rủi ro thật (insider, service account bất thường, deploy lạ).
          - Runbook IR cho 2 kịch bản: lộ credential CI, image độc lên prod.
          - Diễn tập bàn giấy (tabletop) với SOC + app team. Diễn tập restore/DR.
  Đo    : MTTD/MTTR của 2 kịch bản đã diễn tập; biên bản diễn tập nộp được cho kiểm toán.

Q6 — Kết tinh & đòi hỏi cấp Senior
  Làm   : - Đóng gói mọi thứ thành "golden path" có tài liệu, để team khác tự dùng không cần hỏi bạn.
          - Mentor 1 người kế nhiệm mảng pipeline.
          - Trình bày 1 chủ đề cho toàn khối CNTT (visibility).
  Đề xuất thăng cấp: gói hồ sơ theo phần 9. Nguyên tắc: xin thăng cấp cho việc ĐÃ LÀM.
```

### GIAI ĐOẠN 3 — Lead DevSecOps (Q7–Q8+)

```text
Q7 — Mở rộng ảnh hưởng ra ngoài phạm vi kỹ thuật
  Làm   : - Xây & dẫn dắt chương trình Security Champions (mỗi team dev 1 người).
          - Chủ trì đào tạo secure coding (đáp ứng PCI 6.2.2) — đo bằng số lỗ hổng lặp lại giảm.
          - Nhận đứng tên khắc phục ≥1 audit finding lớn, cam kết mốc và giữ mốc.
          - Xây dashboard tuân thủ/rủi ro cho lãnh đạo (phần 8) — cập nhật tự động.
  Đo    : % team đi trên golden path; số finding đóng đúng hạn; lỗ hổng lặp lại giảm.

Q8 — Chiến lược, ngân sách, con người
  Làm   : - Viết lộ trình bảo mật SDLC 12–18 tháng + đề xuất ngân sách công cụ có ROI.
          - Chủ trì 1 quyết định build-vs-buy lớn (VD: nền tảng ASPM/CSPM/secret management).
          - Tham gia tuyển dụng, thiết kế vòng phỏng vấn kỹ thuật cho vị trí DevSecOps.
          - Làm incident commander cho 1 sự cố bảo mật thật (hoặc diễn tập quy mô lớn).
  Đo    : rủi ro tồn dư giảm (theo risk register của tuyến 2 — số liệu ngoài tầm kiểm soát cá nhân bạn,
          nên nó chứng minh bạn đã tác động ở cấp tổ chức).

Q9+ — Giữ nhịp: mỗi quý phải có (a) 1 rủi ro lớn được loại bỏ có hệ thống,
      (b) 1 người trong đội lên tay rõ rệt, (c) 1 tài liệu định hình quyết định của tổ chức.
```

---

## 8. Bộ chỉ số (KPI) của DevSecOps trong ngân hàng

Đây là thứ Lead dùng để quản trị và để báo cáo. Senior nên bắt đầu đo sớm cho phạm vi của mình.

| Nhóm | Chỉ số | Vì sao lãnh đạo quan tâm |
|---|---|---|
| **Bao phủ** | % repo/dịch vụ có đủ gate (SAST/SCA/secret/IaC) · % workload prod chạy image đã ký · % team trên golden path | Trả lời "chúng ta kiểm soát được bao nhiêu phần?" |
| **Tốc độ xử lý** | MTTR lỗ hổng theo severity vs SLA · tuổi trung bình lỗ hổng tồn · số lỗ hổng quá hạn | Ánh xạ trực tiếp vào PCI 6.3.3 và audit finding |
| **Chất lượng** | Tỉ lệ false positive · số lỗ hổng "thoát" ra prod · lỗ hổng lặp lại cùng loại (đo hiệu quả đào tạo) | Chống việc "cài công cụ rồi bỏ đó" |
| **Ma sát với dev** | Thời gian build tăng thêm · số build bị chặn nhầm · điểm hài lòng của dev | **Bảo mật làm chậm dev = bảo mật sẽ bị vượt mặt.** Lead phải đo cái này. |
| **Tuân thủ** | Số audit finding mở/đóng đúng hạn · số control tự động thu thập bằng chứng · số ngoại lệ (risk acceptance) đang mở & quá hạn | Ngôn ngữ của Ban điều hành và NHNN |
| **Sẵn sàng ứng cứu** | MTTD/MTTR sự cố bảo mật · số kịch bản đã diễn tập · thời gian trả lời "ai bị ảnh hưởng bởi CVE X" | Chứng minh năng lực chống chịu, không chỉ phòng ngừa |
| **Quyền hạn** | Số tài khoản quyền cao · số phiên break-glass/tháng & tỉ lệ hậu kiểm · số quyền bị cắt sau rà soát | Rủi ro insider — mối lo lớn nhất của tài chính |

> Quy tắc trình bày: **mỗi chỉ số phải kèm ngưỡng và xu hướng**. "85%" vô nghĩa; "85%, mục tiêu 95%, quý trước 60%" mới có nghĩa.

---

## 9. Hồ sơ bằng chứng để được thăng cấp

Ở ngân hàng, thăng cấp là một **hồ sơ**, không phải một cuộc trò chuyện. Chuẩn bị trước 2 quý:

```text
1. TÓM TẮT 1 TRANG: bạn đã giảm rủi ro gì, đo bằng số nào, ai hưởng lợi.
2. 3–5 DỰ ÁN LỚN: mỗi cái 1 trang — bối cảnh, ràng buộc (gồm ràng buộc pháp lý), phương án đã cân nhắc,
   quyết định, kết quả đo được, bài học.
3. TÀI LIỆU: các ADR/design doc/threat model bạn viết mà người khác dùng để ra quyết định.
4. TÁC ĐỘNG TỚI NGƯỜI KHÁC: ai bạn mentor & họ tiến bộ thế nào; đào tạo bạn tổ chức; golden path bao nhiêu team dùng.
5. TUÂN THỦ: finding bạn đóng, kỳ audit bạn hỗ trợ, control bạn tự động hóa.
6. SỰ CỐ: vai trò của bạn, postmortem, biện pháp loại bỏ CẢ LỚP sự cố (không phải vá một lần).
7. THƯ/PHẢN HỒI từ tuyến 2, từ trưởng nhóm dev, từ đối tác — bằng chứng xã hội rất có trọng lượng.
```

**Cách viết mỗi dòng thành tích (công thức):**
`[Hành động kỹ thuật] → [thay đổi đo được] → [ý nghĩa rủi ro/tiền/tuân thủ]`

> ❌ "Triển khai Trivy và Kyverno cho cluster."
> ✅ "Chuẩn hóa quét image + admission control cho 42 dịch vụ: 100% workload prod chạy image đã ký & đã quét,
> loại bỏ hoàn toàn lớp rủi ro 'image không rõ nguồn gốc' — đóng finding kiểm toán A-2024-17 trước hạn 6 tuần."

---

## 10. Chứng chỉ: thứ tự ưu tiên và lý do

Chứng chỉ **không làm bạn giỏi hơn**, nhưng trong ngành tài chính nó có 2 tác dụng thật: qua vòng hồ sơ, và
làm "forcing function" ép bạn học có hệ thống. Thứ tự đề xuất cho hồ sơ của bạn (nền K8s + AWS):

| Ưu tiên | Chứng chỉ | Vì sao & khi nào |
|---|---|---|
| 1 | **CKS** (Certified Kubernetes Security Specialist) | Sát việc nhất với nền của bạn, thi thực hành. Làm ngay giai đoạn 1–2. |
| 2 | **AWS Certified Security – Specialty** (hoặc tương đương Azure/GCP theo cloud tổ chức dùng) | Chứng minh chiều sâu cloud security — thứ ngân hàng đang thiếu người. |
| 3 | **CISSP** hoặc **CCSP** | CISSP là "tấm vé" ngôn ngữ chung với tuyến 2/CISO ở ngân hàng; rất được coi trọng khi lên **Lead**. CCSP nếu bạn đi sâu cloud. |
| 4 | **PCIP / PCI ISA** | Đặc thù tài chính. ISA cho phép bạn tham gia đánh giá nội bộ — cực kỳ giá trị nếu tổ chức có CDE. |
| 5 | **CISM** | Nếu bạn xác định đi nhánh quản lý (Lead → Head of Security). Ngôn ngữ quản trị rủi ro. |
| Tùy chọn | **OSCP** / các khóa DevSecOps thực hành (CDP...) | OSCP cho tư duy tấn công (rất hữu ích cho threat modeling) nhưng tốn thời gian — chỉ làm nếu bạn thật sự thích offensive. |
| Bổ trợ | **ISO 27001 Lead Implementer/Auditor** | Rẻ, nhanh, và nói đúng ngôn ngữ tuyến 2. Đòn bẩy tốt cho bước lên Lead. |

> Nguyên tắc: **tối đa 2 chứng chỉ/năm**, và mỗi chứng chỉ phải đi kèm 1 dự án thật áp dụng ngay.
> Chứng chỉ không có dự án đi kèm = giấy.

---

## 11. Kỹ năng "chính trị tổ chức" đặc thù ngân hàng

Đây là phần quyết định bạn có lên Lead hay không, và gần như không tài liệu kỹ thuật nào nói.

**1. Nói bằng rủi ro, không bằng công cụ.**
```text
❌ "Cluster chưa có Kyverno, cần triển khai gấp."
✅ "Hiện bất kỳ ai có quyền deploy đều chạy được container privileged trên node prod.
    Một tài khoản dev bị chiếm là đủ để tiếp cận toàn bộ node của kênh số.
    Khả năng xảy ra: trung bình. Ảnh hưởng: nghiêm trọng (dữ liệu KH + gián đoạn dịch vụ).
    Biện pháp: policy chặn ở admission, triển khai 3 tuần, không ảnh hưởng workload hiện tại (đã kiểm thử).
    Nếu không làm: rủi ro tồn dư ở mức cao, cần chủ sở hữu hệ thống ký chấp nhận."
```
Cấu trúc luôn là: **hiện trạng → kịch bản xấu → khả năng × ảnh hưởng → phương án & chi phí → rủi ro tồn dư nếu không làm.**

**2. Đừng bao giờ chỉ nói "không".** Nói: "Không theo cách này, vì rủi ro X. Có 2 cách khác: A (an toàn, chậm hơn 2 tuần), B (nhanh, cần chấp nhận rủi ro Y có thời hạn)." Rồi để chủ sở hữu rủi ro chọn và **ký**.

**3. Coi tuyến 2 là đồng minh, không phải rào cản.** Họ chịu trách nhiệm trước NHNN. Nếu bạn giúp họ có bằng chứng đẹp và báo cáo dễ, họ sẽ chống lưng cho đề xuất của bạn. Đây là mối quan hệ có giá trị nhất trong sự nghiệp DevSecOps ngành tài chính.

**4. Hiểu nhịp của ngân hàng.** Change freeze (Tết, cuối năm tài chính, mùa quyết toán), chu kỳ audit, mùa lập ngân sách. Đề xuất công cụ vào đúng mùa ngân sách thì được duyệt; lệch mùa thì chờ 1 năm.

**5. Chuẩn bị cho phòng phỏng vấn kiểm toán.** Trả lời ngắn, đúng câu hỏi, đưa bằng chứng, **không suy đoán**, không nói "chắc là có". Không biết thì nói "tôi sẽ xác nhận và gửi trong ngày" — rồi gửi đúng hạn.

**6. Visibility có kiểm soát.** Ngân hàng kỵ khoe khoang. Cách tạo uy tín đúng: tài liệu tốt, số liệu trung thực, báo cáo đều đặn, và **không bao giờ giấu sự cố**. Giấu một lần là mất hết.

---

## 12. Mẫu văn bản bạn phải viết được

### 12.1 Risk Acceptance / Đề nghị ngoại lệ (bắt buộc thuộc)

```markdown
# RA-2026-014 — Ngoại lệ: dịch vụ X chạy image có CVE-XXXX-YYYY (High)

## 1. Bối cảnh
Dịch vụ X (thuộc kênh Internet Banking, cấp độ 3 theo TT 09) phụ thuộc thư viện Z v1.2,
chưa có bản vá tương thích. Gate CI hiện chặn build theo ngưỡng High.

## 2. Rủi ro
- Kịch bản khai thác: <mô tả cụ thể, có khả thi trong kiến trúc hiện tại không>
- Khả năng xảy ra: Thấp — thành phần không nằm trên đường request từ internet (đã xác minh bằng <bằng chứng>)
- Ảnh hưởng nếu xảy ra: Cao — truy cập dữ liệu khách hàng
- Rủi ro tồn dư sau biện pháp giảm thiểu: **Trung bình**

## 3. Biện pháp giảm thiểu tạm thời (đã triển khai)
- NetworkPolicy giới hạn egress của pod
- Rule phát hiện tại SIEM: <ID rule>
- Giám sát tăng cường: <dashboard/alert>

## 4. Kế hoạch xử lý dứt điểm
- Nâng cấp lên Z v2.0: dự kiến 30/11/2026, người phụ trách: <tên>
- Điểm kiểm tra giữa kỳ: 31/10/2026

## 5. Thời hạn ngoại lệ
Có hiệu lực đến **30/11/2026**. Tự động hết hiệu lực, KHÔNG tự động gia hạn.

## 6. Phê duyệt
| Vai trò | Tên | Ngày |
|---|---|---|
| Chủ sở hữu hệ thống (tuyến 1) | | |
| An toàn thông tin (tuyến 2) | | |
| Quản lý rủi ro hoạt động | | |
```

### 12.2 Các tài liệu khác phải có trong tay nghề

| Tài liệu | Khi nào dùng | Mẹo |
|---|---|---|
| **Threat model** (STRIDE trên sơ đồ luồng dữ liệu) | Trước khi build hệ thống/kênh mới | Bám dòng tiền, không bám công nghệ |
| **ADR** | Mọi quyết định kỹ thuật khó đảo | Dùng [`../templates/adr-template.md`](../templates/adr-template.md) |
| **Design doc bảo mật** | Đề xuất nền tảng/chuẩn mới | Luôn có mục "ảnh hưởng tới tốc độ dev" và "chi phí vận hành" |
| **Postmortem không đổ lỗi** | Sau mọi sự cố | Kết luận phải là "loại bỏ cả lớp sự cố", không phải "nhắc nhở anh em cẩn thận" |
| **Kế hoạch khắc phục audit finding** | Khi nhận finding | Mốc thời gian ít nhưng phải giữ được; trượt mốc tệ hơn mốc dài |
| **Báo cáo trạng thái tuân thủ hằng quý** | Cho lãnh đạo | 1 trang, xu hướng, 3 rủi ro lớn nhất, 3 việc đang làm |

---

## 13. 90 ngày đầu khi vào một tổ chức tài chính

```text
NGÀY 1–30 — HIỂU, ĐỪNG SỬA
  [ ] Vẽ được sơ đồ dòng tiền & danh mục hệ thống trọng yếu (hỏi, đừng đoán).
  [ ] Biết hệ thống nào thuộc CDE / thuộc phạm vi SWIFT / chứa DLCN.
  [ ] Đọc: chính sách ATTT nội bộ, báo cáo audit gần nhất, danh sách finding đang mở, risk register.
  [ ] Gặp 1-1: trưởng nhóm dev chính, ISO/tuyến 2, SOC, vận hành, kiểm toán nội bộ, đội fraud.
  [ ] Ghi lại "10 điều khiến tôi ngạc nhiên" — sau 3 tháng bạn sẽ quen và mất góc nhìn này.

NGÀY 31–60 — CHỌN MỘT THẮNG LỢI NHỎ, CHẮC CHẮN
  [ ] Chọn việc: rủi ro rõ + ít gây ma sát + đo được. Gợi ý tốt: secret scanning, hoặc mask non-prod,
      hoặc đóng 1 finding đang mở lâu ngày.
  [ ] Làm xong, đo, báo cáo ngắn gọn có số liệu.
  [ ] TUYỆT ĐỐI không bật chế độ chặn ở bất cứ gate nào trong 60 ngày đầu.

NGÀY 61–90 — ĐỀ XUẤT HƯỚNG ĐI
  [ ] Trình 1 tài liệu: hiện trạng rủi ro SDLC + 3 ưu tiên 12 tháng + cách đo.
  [ ] Có được 1 người ủng hộ ở tuyến 2 và 1 trưởng nhóm dev sẵn sàng làm thí điểm.
  [ ] Bắt đầu chương trình dài hạn đầu tiên.
```

---

## 14. Cạm bẫy đặc thù ngành tài chính

| Cạm bẫy | Hậu quả | Cách tránh |
|---|---|---|
| Bật gate chặn ngay từ đầu | Dev vượt mặt bạn, uy tín mất, bạn thành "kẻ cản trở" | Cảnh báo → đo tiếng ồn → tinh chỉnh → mới chặn, và chặn theo mức độ nhạy hệ thống |
| Chỉ giỏi công cụ, không hiểu nghiệp vụ | Mãi ở Senior, không ai mời vào phòng quyết định | Ép mình vẽ được dòng tiền & threat model theo giao dịch, không theo hạ tầng |
| Coi tuân thủ là việc phiền | Bỏ lỡ đòn bẩy quyền lực lớn nhất trong ngành | Tuân thủ là *ngân sách* và *quyền phủ quyết* của bạn — dùng nó |
| Làm bằng chứng thủ công trước kỳ audit | Mỗi năm mất hàng trăm giờ, và không bền vững | Tự động hóa thu thập bằng chứng ngay từ khi thiết kế control |
| Ngoại lệ không có hạn & không có chủ | Rủi ro tích tụ âm thầm, kiểm toán phát hiện → finding lớn | Mọi ngoại lệ: có chủ sở hữu, có hạn, tự hết hiệu lực, review hằng quý |
| Ôm hết việc bảo mật về mình | Trần cứng ở Senior; đội security thành nút cổ chai | Golden path + Security Champions: làm cho dev tự làm đúng |
| Dữ liệu thật ở môi trường test | Vi phạm Luật BVDLCN & PCI, rủi ro rò rỉ cao nhất thực tế | Mask/tổng hợp dữ liệu; coi đây là việc ưu tiên hàng đầu |
| Không tính đến change freeze & mùa audit | Kế hoạch trượt, mất uy tín cam kết | Lấy lịch freeze/audit của tổ chức trước khi lập roadmap |
| Chạy theo mọi CVE | Kiệt sức, dev mệt, rủi ro thật vẫn còn | Ưu tiên theo khả năng khai thác thật + độ nhạy hệ thống, không theo điểm CVSS đơn thuần |
| Giấu/giảm nhẹ sự cố | Mất sạch uy tín, có thể mất việc & vi phạm nghĩa vụ báo cáo | Báo cáo sớm, trung thực, kèm phương án. Ngân hàng tha thứ sự cố, không tha thứ che giấu |

---

## 15. Tài nguyên

**Nghiệp vụ & tuân thủ (đọc bản gốc, không đọc tóm tắt):**
- Thông tư [09/2020/TT-NHNN](https://thuvienphapluat.vn/van-ban/Tien-te-Ngan-hang/Thong-tu-09-2020-TT-NHNN-an-toan-he-thong-thong-tin-trong-hoat-dong-ngan-hang-455885.aspx) — an toàn HTTT trong hoạt động ngân hàng.
- Thông tư [50/2024/TT-NHNN](https://thuvienphapluat.vn/van-ban/Tien-te-Ngan-hang/Thong-tu-50-2024-TT-NHNN-an-toan-bao-mat-cung-cap-dich-vu-truc-tuyen-nganh-Ngan-hang-327954.aspx) (+ TT 77/2025 sửa đổi) — dịch vụ trực tuyến.
- [Luật Bảo vệ dữ liệu cá nhân 91/2025/QH15](https://baochinhphu.vn/luat-bao-ve-du-lieu-ca-nhan-chinh-thuc-co-hieu-luc-tu-ngay-mai-1-1-2026-102251231155609721.htm) — hiệu lực 01/01/2026.
- PCI DSS v4.0.1 — tải bản gốc tại PCI SSC Document Library; đọc kỹ Requirement 6, 7, 8, 10, 11.
- [SWIFT Customer Security Programme — CSCF v2026](https://www.swift.com/myswift/customer-security-programme-csp).
- NIST **SP 800-218 (SSDF)** — khung SDLC an toàn, dùng làm ngôn ngữ chung với tuyến 2.
- ISO/IEC 27001:2022 Annex A (8.25–8.31) — vòng đời phát triển an toàn.

**Kỹ thuật:**
- OWASP: Top 10, **ASVS**, SAMM, Cheat Sheet Series, Top 10 for LLM (nếu chạm AI).
- CIS Benchmarks (Kubernetes, Docker, Linux, cloud) — nguồn viết policy.
- SLSA framework · Sigstore/cosign · Syft/Grype/Trivy · Kyverno & OPA Gatekeeper · Falco/Tetragon.
- *Threat Modeling: Designing for Security* — Adam Shostack.
- *Alice and Bob Learn Application Security* — Tanya Janca.
- *Securing DevOps* — Julien Vehent.
- *Container Security* — Liz Rice · *Kubernetes Security and Observability*.

**Lãnh đạo & sự nghiệp:**
- *The Manager's Path* — Camille Fournier (nếu đi tiếp nhánh quản lý sau Lead).
- *Staff Engineer* — Will Larson (nếu muốn giữ nhánh IC: Security Architect).
- *How to Measure Anything in Cybersecurity Risk* — Hubbard & Seiersen (**đọc cuốn này** — nó dạy bạn nói chuyện rủi ro bằng số, đúng thứ ngân hàng cần).
- Verizon DBIR hằng năm · các postmortem/sự cố công bố công khai ngành tài chính.

---

## Tóm tắt 10 dòng

```text
1. DevSecOps ở ngân hàng nằm ở TUYẾN 1. Muốn có quyền, phải nói được ngôn ngữ của TUYẾN 2 (rủi ro, điều khoản).
2. Nghiệp vụ tài chính = hiểu DÒNG TIỀN. Threat model theo giao dịch, không theo hạ tầng.
3. Thuộc bảng dịch: điều khoản (TT 09/TT 50/Luật 91-2025/PCI/SWIFT) -> control -> BẰNG CHỨNG.
4. Senior = sở hữu một miền rủi ro end-to-end, tự quyết đánh đổi, dev tin dùng thứ mình làm.
5. Lead  = sở hữu chuẩn + con người + ngân sách + trách nhiệm trước kiểm toán, tạo kết quả QUA người khác.
6. Đòn bẩy business rõ nhất của bạn: THU HẸP PHẠM VI PCI, XÓA DỮ LIỆU THẬT KHỎI NON-PROD,
   TỰ ĐỘNG HÓA BẰNG CHỨNG AUDIT. Ba việc này quy ra tiền được ngay.
7. Không bao giờ chặn trước khi cảnh báo. Bảo mật làm sập luồng dev là bảo mật sẽ bị vô hiệu hóa.
8. Mọi ngoại lệ phải có CHỦ SỞ HỮU + HẠN + tự hết hiệu lực.
9. Chứng chỉ theo thứ tự: CKS -> AWS Security Specialty -> CISSP/CCSP -> PCI ISA. Mỗi cái kèm 1 dự án thật.
10. Thăng cấp = hồ sơ bằng chứng cho việc BẠN ĐÃ LÀM ở cấp đó. Làm trước, title theo sau.
```
