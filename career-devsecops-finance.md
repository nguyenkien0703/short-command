# Lộ trình DevSecOps trong ngành Tài chính — Senior → Lead

> Tài liệu này trả lời cụ thể: **làm gì, theo thứ tự nào, bằng chứng gì** để đi từ
> Middle/Senior DevSecOps lên **Senior DevSecOps** rồi **Lead DevSecOps**, trong bối cảnh
> **ngân hàng / fintech / ví điện tử / công ty chứng khoán tại Việt Nam**.
>
> Khác gì [`career-roadmap.md`](./career-roadmap.md)? File kia là bản đồ **chung** (IC vs Management, Staff/Architect).
> File này là **đường đi chi tiết cho đúng 1 nhánh nghề** — có mốc thời gian, deliverable, và domain nghiệp vụ tài chính.
> Nền kỹ thuật DevSecOps xem [`tracks/devsecops.md`](./tracks/devsecops.md).

> ⚠️ **Về phần pháp lý:** các văn bản dưới đây được tổng hợp tới **09/2026** để bạn biết *phải học gì*.
> Trước khi dùng làm căn cứ tuân thủ thật, **luôn đối chiếu bản gốc + hỏi phòng Pháp chế/Tuân thủ** của tổ chức bạn.
> Kỹ năng "biết mình cần xác minh ở đâu" chính là thứ phân biệt Senior với Middle.

## 📑 Mục lục
- [0. Vì sao DevSecOps tài chính là một nghề riêng](#0-vì-sao-devsecops-tài-chính-là-một-nghề-riêng)
  - [0.1 Bạn ngồi ở đâu: mô hình Ba tuyến phòng vệ](#01-bạn-ngồi-ở-đâu-mô-hình-ba-tuyến-phòng-vệ)
  - [0.2 Phân biệt các vai lân cận](#02-phân-biệt-các-vai-lân-cận-đừng-nhận-nhầm-việc)
- [1. Thang cấp bậc — định nghĩa hết mơ hồ](#1-thang-cấp-bậc--định-nghĩa-hết-mơ-hồ)
- [2. Ma trận năng lực Senior vs Lead (tự chấm điểm)](#2-ma-trận-năng-lực-senior-vs-lead-tự-chấm-điểm)
- [3. DOMAIN NGHIỆP VỤ TÀI CHÍNH — phần quan trọng nhất](#3-domain-nghiệp-vụ-tài-chính--phần-quan-trọng-nhất)
  - [3.1 Bản đồ hệ thống một ngân hàng](#31-bản-đồ-hệ-thống-một-ngân-hàng)
  - [3.2 Luồng tiền và điểm rủi ro tương ứng](#32-luồng-tiền-và-điểm-rủi-ro-tương-ứng)
  - [3.3 20 khái niệm nghiệp vụ bắt buộc phải hiểu](#33-20-khái-niệm-nghiệp-vụ-bắt-buộc-phải-hiểu)
  - [3.4 Bản đồ tuân thủ Việt Nam + quốc tế](#34-bản-đồ-tuân-thủ-việt-nam--quốc-tế)
  - [3.5 Dịch điều khoản → control → bằng chứng tự động](#35-dịch-điều-khoản--control--bằng-chứng-tự-động-kỹ-năng-đắt-nhất)
  - [3.6 Phân cấp độ hệ thống thông tin (1–5)](#36-phân-cấp-độ-hệ-thống-thông-tin-15--thứ-quyết-định-control-bắt-buộc)
- [4. Giai đoạn 1: → Senior DevSecOps (12–18 tháng)](#4-giai-đoạn-1--senior-devsecops-1218-tháng)
- [5. Giai đoạn 2: Senior → Lead DevSecOps (12–18 tháng)](#5-giai-đoạn-2-senior--lead-devsecops-1218-tháng)
- [6. Vận hành thực chiến — bốn thứ tài liệu khác không nói](#6-vận-hành-thực-chiến--bốn-thứ-tài-liệu-khác-không-nói)
  - [6.1 SLA vá lỗi & ma trận ưu tiên](#61-sla-vá-lỗi--ma-trận-ưu-tiên-đừng-dùng-cvss-làm-ưu-tiên)
  - [6.2 Triển khai guardrail mà không gây sự cố](#62-triển-khai-guardrail-mà-không-gây-sự-cố)
  - [6.3 Nhịp của tổ chức tài chính](#63-nhịp-của-tổ-chức-tài-chính--đề-xuất-đúng-lúc-thì-được-duyệt)
  - [6.4 Ngôn ngữ rủi ro — cấu trúc 5 bước](#64-ngôn-ngữ-rủi-ro--cấu-trúc-5-bước-để-nói-chuyện-với-lãnh-đạo)
- [7. Portfolio: 12 hiện vật chứng minh năng lực](#7-portfolio-12-hiện-vật-chứng-minh-năng-lực)
- [8. Kế hoạch 90 ngày đầu tiên (bắt đầu từ thứ Hai)](#8-kế-hoạch-90-ngày-đầu-tiên-bắt-đầu-từ-thứ-hai)
- [9. Chứng chỉ — xếp theo ROI trong ngành tài chính](#9-chứng-chỉ--xếp-theo-roi-trong-ngành-tài-chính)
- [10. Phỏng vấn: câu hỏi thật + cách trả lời](#10-phỏng-vấn-câu-hỏi-thật--cách-trả-lời)
- [11. 10 cái bẫy khiến bạn kẹt ở Senior](#11-10-cái-bẫy-khiến-bạn-kẹt-ở-senior)
- [12. Tự đo tiến độ hằng quý](#12-tự-đo-tiến-độ-hằng-quý)
- [13. Từ điển Anh–Việt (tra cứu & phỏng vấn)](#13-từ-điển-anhviệt-để-tra-cứu--phỏng-vấn)
- [14. Tài nguyên học](#14-tài-nguyên-học)
- [Mẫu văn bản đi kèm](#mẫu-văn-bản-đi-kèm)

---

## 0. Vì sao DevSecOps tài chính là một nghề riêng

DevSecOps ở công ty sản phẩm thường tối ưu: *ship nhanh, giảm CVE, ít ma sát*.
DevSecOps ở tổ chức tài chính tối ưu thêm **ba thứ mà nơi khác không có**:

```text
1. TIỀN THẬT & KHÔNG THỂ HOÀN TÁC.
   Bug ở app thường = user khó chịu. Bug ở luồng thanh toán = mất tiền thật, ghi nợ sai,
   hoặc chuyển 2 lần. Không có "rollback" cho một lệnh chuyển tiền đã settle.
   -> Hệ quả kỹ thuật: idempotency, đối soát (reconciliation), maker-checker, audit trail
      bất biến trở thành YÊU CẦU BẢO MẬT, không phải chuyện của riêng dev.

2. CƠ QUAN QUẢN LÝ LÀ MỘT "STAKEHOLDER" THẬT.
   NHNN, Bộ Công an, kiểm toán nội bộ, QSA (PCI), SWIFT assessor — họ sẽ ĐÒI BẰNG CHỨNG.
   -> Hệ quả kỹ thuật: mọi control phải SINH RA BẰNG CHỨNG TỰ ĐỘNG (evidence-as-code).
      Một control chạy tốt nhưng không chứng minh được = coi như không có.

3. HẬU QUẢ PHI KỸ THUẬT LỚN HƠN HẬU QUẢ KỸ THUẬT.
   Sự cố -> đình chỉ dịch vụ, phạt tiền, mất giấy phép, lên báo, mất niềm tin.
   -> Hệ quả nghề nghiệp: người dịch được "rủi ro kỹ thuật -> rủi ro kinh doanh & pháp lý"
      có giá trị gấp nhiều lần người chỉ chạy được scanner.
```

**Đây chính là đòn bẩy nghề nghiệp của bạn.** Số người biết Kubernetes + CI/CD rất đông.
Số người biết Kubernetes + CI/CD **+ hiểu luồng đối soát ngân hàng + đọc được Thông tư 09/50 và biến nó
thành Kyverno policy** thì cực hiếm. Thị trường trả giá cho giao điểm đó, không phải cho từng mảnh riêng lẻ.

```text
        Kỹ thuật DevOps/K8s          Bảo mật (AppSec/CloudSec)
                 \                          /
                  \                        /
                   \____   BẠN Ở ĐÂY  ____/
                        \              /
                         \            /
                     Nghiệp vụ + Tuân thủ tài chính

Thiếu góc 1 -> "auditor biết nói tech", không tự làm được.
Thiếu góc 2 -> DevOps thường, security chỉ là plugin.
Thiếu góc 3 -> "DevSecOps giỏi nhưng không hiểu ngân hàng" — làm được Senior, KHÓ lên Lead.
```

> **Kết luận định hướng:** con đường lên Lead của bạn **không phải** là học thêm 5 công cụ scan.
> Nó là lấp góc thứ 3 (nghiệp vụ + tuân thủ) trong khi giữ vững góc 1 và 2.

### 0.1 Bạn ngồi ở đâu: mô hình Ba tuyến phòng vệ

Mọi tổ chức tài chính vận hành theo **Three Lines of Defense**. Không hiểu mô hình này thì bạn sẽ liên tục
"va" vào người khác mà không hiểu vì sao — và sẽ đề xuất sai cửa.

```text
TUYẾN 1 — VẬN HÀNH & SỞ HỮU RỦI RO        ◄── BẠN Ở ĐÂY
  Khối CNTT, Trung tâm hạ tầng, Platform, DevSecOps, các team ứng dụng
  Vai trò: TỰ nhận diện và TỰ kiểm soát rủi ro trong hệ thống mình vận hành.

TUYẾN 2 — GIÁM SÁT RỦI RO & TUÂN THỦ
  Phòng An toàn thông tin (ISO), Quản lý rủi ro hoạt động, Tuân thủ, DPO
  Vai trò: ĐẶT chuẩn, THẨM ĐỊNH, PHÊ DUYỆT ngoại lệ, báo cáo cơ quan quản lý.

TUYẾN 3 — KIỂM TOÁN
  Kiểm toán nội bộ + kiểm toán độc lập / QSA / thanh tra NHNN
  Vai trò: KIỂM TRA cả tuyến 1 và tuyến 2. Ra "audit finding" có thời hạn khắc phục.
```

**Bốn hiểu lầm phổ biến — và thực tế:**

| Bạn hay nghĩ | Thực tế trong tổ chức tài chính |
|---|---|
| "Tôi làm DevSecOps nên tôi là security, tôi đặt chuẩn" | DevSecOps thường ở **tuyến 1**. Bạn *thực thi* chuẩn; tuyến 2 *đặt* chuẩn. Muốn đổi chuẩn phải **thuyết phục tuyến 2 bằng dữ liệu**, không tự quyết |
| "Chặn build khi có CVE Critical là xong việc" | Chặn mà **không có quy trình ngoại lệ có chủ sở hữu và có hạn** thì business sẽ vượt mặt bạn, hoặc bạn thành kẻ cản trở. Xem [`templates/risk-acceptance-template.md`](./templates/risk-acceptance-template.md) |
| "Audit finding là việc của Compliance" | Finding thuộc **hệ thống bạn vận hành = việc của bạn**. Đóng finding đúng hạn là chỉ số được nhìn tận cấp Ban điều hành — và là cơ hội hiển thị lớn nhất của bạn |
| "Cứ làm tốt kỹ thuật rồi sẽ được ghi nhận" | Thứ được ghi nhận là **rủi ro giảm được, đo được, có bằng chứng**. Việc không có bằng chứng = việc không tồn tại |

> 🔑 **Đòn bẩy lớn nhất để lên Lead:** trở thành người *duy nhất* nói trôi chảy **cả hai ngôn ngữ** —
> ngôn ngữ pipeline/hạ tầng (tuyến 1) và ngôn ngữ rủi ro/điều khoản (tuyến 2).
> Người đó tự nhiên thành đầu mối. Đầu mối thì thành Lead.

### 0.2 Phân biệt các vai lân cận (đừng nhận nhầm việc)

Trong ngân hàng, ranh giới vai trò rõ hơn ở công ty sản phẩm. Nhận nhầm việc = làm nhiều mà không được ghi nhận.

```text
DevSecOps Engineer : bảo mật TRONG pipeline & hạ tầng. Guardrail, automation, golden path.  ◄ bạn
AppSec Engineer    : bảo mật TRONG code ứng dụng. Threat model sâu, secure code review, hỗ trợ dev sửa.
SecOps / SOC       : phát hiện & phản ứng. SIEM, threat hunting, trực 24/7, điều tra sự cố.
GRC / Compliance   : khung, chính sách, ánh xạ điều khoản, làm việc với NHNN/QSA/kiểm toán.
Security Architect : thiết kế mô hình bảo mật cho hệ thống mới, quyết định one-way-door.
Fraud / Risk tech  : chống gian lận GIAO DỊCH — rất riêng của tài chính, không phải infosec truyền thống.
IT Audit           : kiểm tra bạn. Không phải đối thủ — hiểu cách họ làm việc thì đời bạn dễ hơn nhiều.
```

**Lead DevSecOps phải giao tiếp trôi chảy với cả 7 vai trên**, và thường kiêm một phần Security Architect.
Đây là lý do kỹ năng mềm quyết định ngưỡng Senior → Lead nhiều hơn kỹ thuật.

---

## 1. Thang cấp bậc — định nghĩa hết mơ hồ

| | **Middle DevSecOps** | **Senior DevSecOps** | **Lead DevSecOps** | **Head / Security Architect** |
|---|---|---|---|---|
| **Phạm vi** | 1 pipeline / 1 nhóm tool | 1 domain: toàn bộ pipeline + cluster + cloud của 1-2 hệ thống | Cả chương trình bảo mật của kỹ thuật: nhiều team, nhiều hệ thống | Toàn tổ chức, tham gia quyết định rủi ro cấp CIO/CRO |
| **Câu hỏi bạn trả lời** | "Chạy scan này thế nào?" | "Kiến trúc bảo mật cho hệ thống thanh toán mới ra sao?" | "Chiến lược bảo mật kỹ thuật 12–18 tháng tới là gì, ưu tiên gì, bỏ gì?" | "Khẩu vị rủi ro của tổ chức là gì và kiến trúc phải như thế nào?" |
| **Đầu ra chính** | Config, script, ticket đã đóng | Design doc, threat model, guardrail tự động, policy | Roadmap, chuẩn, ngân sách/nhân sự kỹ thuật, đại diện trước auditor | Chiến lược, khung quản trị, quan hệ với cơ quan quản lý |
| **Tự chủ** | Được giao việc rõ | Tự định nghĩa vấn đề trong domain | Tự đặt ưu tiên cho nhiều domain | Đặt vấn đề cho cả tổ chức |
| **Đo bằng** | Task hoàn thành, MTTR của mình | Số lớp lỗ hổng bị **loại bỏ**, chất lượng thiết kế | Năng lực **của team**, độ phủ control, kết quả audit | Vị thế rủi ro của tổ chức |
| **Thời gian ở cấp** | 1–3 năm | 2–4 năm | 3+ năm | — |
| **Bài kiểm tra "đã đủ chưa"** | Không cần ai kèm | Kiến trúc sư/PM tìm bạn **trước khi** thiết kế, không phải sau | Bạn nghỉ 3 tuần, chương trình bảo mật vẫn chạy đúng hướng | Cơ quan quản lý/BLĐ hỏi ý kiến bạn về rủi ro |

**Ba ngưỡng chuyển cấp — đây là chỗ hầu hết mọi người mắc kẹt:**

```text
NGƯỠNG 1 (Middle -> Senior): TỪ "CHẠY CÔNG CỤ" SANG "THIẾT KẾ HỆ THỐNG KIỂM SOÁT"
  Middle: cài Trivy vào pipeline, fail build khi CRITICAL.
  Senior: thiết kế cả vòng đời lỗ hổng — ai sở hữu, SLA vá theo mức độ nghiêm trọng,
          ngoại lệ được duyệt thế nào & hết hạn ra sao, ai chịu rủi ro tồn dư,
          làm sao chứng minh với kiểm toán, và làm sao KHÔNG chặn 40 team release.

NGƯỠNG 2 (Senior -> Lead): TỪ "TÔI LÀM ĐƯỢC" SANG "TỔ CHỨC LÀM ĐƯỢC MÀ KHÔNG CẦN TÔI"
  Senior: tự tay threat model cho hệ thống thanh toán mới — làm rất tốt.
  Lead: xây quy trình + template + đào tạo để 6 team tự threat model đạt chuẩn,
        bạn chỉ review các hệ thống cấp độ 4-5. Thành công đo bằng việc bạn KHÔNG phải có mặt.

NGƯỠNG 3 (Lead -> Head): TỪ "CHƯƠNG TRÌNH KỸ THUẬT" SANG "QUẢN TRỊ RỦI RO"
  Lead: tối đa hóa độ phủ control với nguồn lực có sẵn.
  Head: quyết định CHẤP NHẬN rủi ro nào, đánh đổi với mục tiêu kinh doanh, chịu trách nhiệm.
```

> Ghi nhớ: **Lead DevSecOps ≠ Senior DevSecOps giỏi hơn.** Nó là công việc khác về bản chất — nhân bản
> năng lực qua người khác và qua hệ thống. Nếu bạn lên Lead mà vẫn là người giỏi nhất về mọi thứ và
> ai cũng phải chờ bạn, bạn đang làm **Senior với title Lead** và sẽ kiệt sức.

**Lead DevSecOps ở tổ chức tài chính còn có 3 việc mà nơi khác không có:**
1. **Đứng trước kiểm toán/QSA/thanh tra NHNN** — giải thích control, đưa bằng chứng, bảo vệ thiết kế.
2. **Bảo mật bên thứ ba** — ngân hàng phụ thuộc rất nhiều vendor (core banking, switch, eKYC, SMS/OTP). Đánh giá và ràng buộc họ là việc của bạn.
3. **Ngồi trong hội đồng thay đổi (CAB) và hội đồng rủi ro** — có quyền và trách nhiệm chặn/thông qua thay đổi lên hệ thống trọng yếu.

---

## 2. Ma trận năng lực Senior vs Lead (tự chấm điểm)

Chấm 1–5 (1 = chưa biết · 2 = biết khái niệm · 3 = làm được khi có hướng dẫn · 4 = tự chủ, làm ở production · 5 = thiết kế được chuẩn & dạy người khác).
**Senior cần trung bình ≥ 4 ở nhóm A–D và ≥ 3 ở nhóm E–F. Lead cần ≥ 4 ở tất cả, và ≥ 4.5 ở nhóm F.**

### A. Nền tảng kỹ thuật (bạn đã có phần lớn)
| Năng lực | Senior | Lead | Điểm |
|---|:--:|:--:|:--:|
| Kubernetes production (RBAC, NetworkPolicy, Pod Security, multi-tenant) | 4 | 4 | |
| CI/CD & GitOps (pipeline as code, ArgoCD, môi trường tách biệt) | 4 | 4 | |
| IaC (Terraform module, state, drift detection) | 4 | 4 | |
| Cloud/Hạ tầng (AWS hoặc on-prem/VMware — phần lớn NH VN là hybrid) | 4 | 4 | |
| Linux, mạng (phân vùng, firewall, TLS, DNS, proxy) | 4 | 4 | |
| Observability (metrics/logs/traces + chi phí của nó) | 3 | 4 | |

### B. Bảo mật ứng dụng & chuỗi cung ứng
| Năng lực | Senior | Lead | Điểm |
|---|:--:|:--:|:--:|
| SAST/DAST/SCA: **tinh chỉnh, giảm false positive**, không chỉ bật lên | 4 | 4 | |
| Secret scanning + quy trình xoay vòng khi lộ (chưa lộ vẫn phải diễn tập) | 4 | 4 | |
| SBOM (CycloneDX/SPDX), quản lý theo vòng đời, truy vấn khi có CVE 0-day | 4 | 5 | |
| Ký & xác minh artifact (cosign/Sigstore), provenance, SLSA level | 4 | 4 | |
| Admission control chặn image không ký / không quét (Kyverno/OPA) | 4 | 4 | |
| Threat modeling (STRIDE) cho hệ thống có luồng tiền | 4 | 5 | |
| OWASP Top 10 + **OWASP API Top 10** (ngân hàng = API-first, cực quan trọng) | 4 | 4 | |
| Quản lý lỗ hổng: SLA, ngoại lệ, rủi ro tồn dư, báo cáo cho lãnh đạo | 4 | 5 | |

### C. Bảo mật hạ tầng & vận hành
| Năng lực | Senior | Lead | Điểm |
|---|:--:|:--:|:--:|
| Secrets management (Vault: dynamic secret, transit, PKI, namespace) | 4 | 4 | |
| Quản lý khóa & HSM — **bắt buộc trong ngân hàng** (PIN block, key ceremony) | 3 | 4 | |
| Zero Trust / workload identity (SPIFFE, mTLS, OIDC federation, không dùng long-lived key) | 4 | 4 | |
| Phân vùng mạng theo yêu cầu tuân thủ (CDE của PCI, SWIFT secure zone) | 4 | 5 | |
| Runtime security (Falco/Tetragon) + xử lý cảnh báo thật, không để nhiễu | 3 | 4 | |
| Bảo mật đặc quyền (PAM, jump host, session recording — auditor luôn hỏi) | 3 | 4 | |
| Ứng cứu sự cố bảo mật: containment, forensic, chuỗi bằng chứng, báo cáo cơ quan quản lý | 3 | 5 | |
| DR/BCP: diễn tập chuyển site, RTO/RPO theo cấp độ hệ thống | 3 | 4 | |

### D. Nghiệp vụ tài chính *(phần bạn thấy mù mờ nhất — xem mục 3)*
| Năng lực | Senior | Lead | Điểm |
|---|:--:|:--:|:--:|
| Hiểu kiến trúc core banking + hệ thống kênh + switch thanh toán | 3 | 4 | |
| Hiểu luồng tiền end-to-end và điểm rủi ro của từng chặng | 4 | 5 | |
| Idempotency, đối soát, cut-off/EOD, settlement vs clearing | 4 | 5 | |
| Nhận diện gian lận & chống rửa tiền (đủ để thiết kế control, không cần làm analyst) | 3 | 4 | |
| Phân loại dữ liệu: PII, dữ liệu thẻ (PAN/CVV), dữ liệu sinh trắc học, dữ liệu giao dịch | 4 | 5 | |
| Phân loại cấp độ hệ thống thông tin (cấp 1–5) và control tương ứng | 3 | 5 | |

### E. Tuân thủ & quản trị
| Năng lực | Senior | Lead | Điểm |
|---|:--:|:--:|:--:|
| Đọc điều khoản pháp lý → dịch thành control kỹ thuật | 3 | 5 | |
| Compliance/evidence as code (tự động thu thập bằng chứng) | 4 | 5 | |
| Làm việc với kiểm toán nội bộ / QSA / thanh tra | 2 | 4 | |
| Đánh giá bảo mật bên thứ ba (vendor core banking, eKYC, SMS) | 2 | 4 | |
| Đăng ký rủi ro, rủi ro tồn dư, quy trình chấp nhận rủi ro có ký duyệt | 2 | 4 | |

### F. Lãnh đạo & ảnh hưởng *(quyết định việc lên Lead)*
| Năng lực | Senior | Lead | Điểm |
|---|:--:|:--:|:--:|
| Viết: design doc, ADR, threat model, postmortem, báo cáo cho lãnh đạo | 4 | 5 | |
| Thuyết phục không quyền lực (dev không báo cáo cho bạn nhưng phải nghe bạn) | 3 | 5 | |
| Kèm cặp & nâng cấp người khác (Security Champion program) | 3 | 5 | |
| Ưu tiên hóa & nói KHÔNG có lý lẽ (nguồn lực luôn thiếu) | 3 | 5 | |
| Dịch tech → rủi ro kinh doanh (nói được ngôn ngữ của CRO/CIO) | 3 | 5 | |
| Xử lý xung đột Dev ↔ Security (đây là việc chính của Lead) | 2 | 5 | |
| Tuyển dụng & phỏng vấn kỹ thuật bảo mật | 1 | 4 | |

> **Cách dùng bảng:** chấm thật, đừng nương tay. Lấy **3 dòng thấp nhất so với cột mục tiêu** làm
> trọng tâm quý tới. Chấm lại mỗi quý, lưu vào git để thấy đường tiến.

---

## 3. DOMAIN NGHIỆP VỤ TÀI CHÍNH — phần quan trọng nhất

Đây là phần bạn nói "mù mờ", và cũng là phần tạo khác biệt lớn nhất. Kỹ thuật bạn có thể học từ tài liệu.
**Nghiệp vụ tài chính thì phải học có hệ thống, vì nó quyết định control nào là bắt buộc.**

### 3.1 Bản đồ hệ thống một ngân hàng

```text
                              ┌───────────────────────────────┐
   KHÁCH HÀNG ───────────────►│  KÊNH (Channel Layer)         │
   (app, web, ATM, POS,       │  Mobile/Internet Banking,     │  ◄── Bề mặt tấn công LỚN NHẤT
    QR, chi nhánh)            │  API Gateway, Open API/BaaS   │      (public, đông user)
                              └───────────┬───────────────────┘
                                          │
                              ┌───────────▼───────────────────┐
                              │  LỚP TÍCH HỢP                 │
                              │  ESB / API Management /       │  ◄── Nơi rò rỉ dữ liệu & thiếu
                              │  Message Queue (Kafka/MQ)     │      xác thực service-to-service
                              └───────────┬───────────────────┘
                    ┌─────────────────────┼─────────────────────┐
                    ▼                     ▼                     ▼
        ┌───────────────────┐  ┌────────────────────┐  ┌──────────────────┐
        │ CORE BANKING      │  │ HỆ THỐNG THANH TOÁN│  │ HỆ THỐNG HỖ TRỢ  │
        │ (T24/Flexcube/    │  │ Payment Switch,    │  │ eKYC, LOS/LMS,   │
        │  Symbols/nội bộ)  │  │ Card Mgmt, ATM/POS,│  │ CRM, AML/CFT,    │
        │ • Sổ cái, tài khoản│  │ SWIFT, NAPAS,     │  │ Fraud engine,    │
        │ • Bút toán kép    │  │ CITAD/IBPS, QR     │  │ Kho dữ liệu, BI  │
        │ ⚠ CẤP ĐỘ CAO NHẤT │  │ ⚠ TIỀN RỜI TỔ CHỨC │  │ ⚠ PII & sinh trắc│
        └───────────────────┘  └────────────────────┘  └──────────────────┘
                    │                     │                     │
                    └─────────────────────┼─────────────────────┘
                                          ▼
                              ┌───────────────────────────────┐
                              │  HẠ TẦNG & DR                 │
                              │  DC chính + DR site, HSM,     │
                              │  backup, mạng phân vùng       │
                              └───────────────────────────────┘
```

**Điều một Senior phải nắm về mỗi khối:**

| Khối | Đặc điểm khiến bảo mật khác biệt | Câu hỏi bạn phải trả lời được |
|---|---|---|
| **Core banking** | Thường là hệ thống đóng của vendor, khó vá, không container hóa được, cửa sổ bảo trì cực hẹp | "Không vá được thì bù bằng control gì?" (phân vùng, ảo hóa vá lỗi, giám sát chặt, PAM) |
| **Kênh (Mobile/Internet Banking)** | Public, chịu tấn công liên tục, nơi áp dụng TT 50 & QĐ 2345 | "Xác thực mạnh theo hạn mức giao dịch được enforce ở tầng nào? Chống bot/credential stuffing ra sao?" |
| **API Gateway / Open API** | Nơi lộ nhiều nhất theo OWASP API Top 10 (BOLA — truy cập nhầm tài nguyên người khác) | "Kiểm tra ủy quyền theo từng đối tượng (object-level authz) nằm ở đâu? Rate limit theo user hay theo IP?" |
| **Payment switch / thẻ** | Trong phạm vi PCI DSS (CDE). Dữ liệu PAN, chạm HSM | "Ranh giới CDE ở đâu? Cái gì làm CDE phình ra? Tokenization đặt ở đâu?" |
| **SWIFT** | Phải tuân thủ CSCF, có "secure zone" riêng, chứng thực hằng năm | "Luồng back-office ↔ SWIFT đã được bảo vệ chưa? (v2026 bắt buộc control 2.4)" |
| **eKYC / sinh trắc học** | Dữ liệu cá nhân **nhạy cảm** theo Luật BVDLCN 2025 — vi phạm rất nặng | "Dữ liệu sinh trắc lưu ở đâu, mã hóa bằng khóa nào, ai truy cập được, giữ bao lâu?" |
| **Kho dữ liệu / BI** | Chỗ rò rỉ hàng loạt phổ biến nhất: bản sao production đầy PII cho analyst | "Dữ liệu xuống môi trường thấp có được che/giả lập không? Ai duyệt?" |
| **DR site** | Auditor luôn hỏi. Thường bảo mật kém hơn DC chính — kẻ tấn công biết điều đó | "DR có cùng mức control với DC chính không? Lần diễn tập gần nhất là khi nào?" |

### 3.2 Luồng tiền và điểm rủi ro tương ứng

Học thuộc luồng này. Đây là thứ giúp bạn threat model **đúng chỗ có tiền**, không phải chỗ dễ scan.

```text
[1] KHỞI TẠO          Khách bấm "Chuyển 50 triệu"
     │                RỦI RO: chiếm tài khoản, malware trên máy khách, kỹ nghệ xã hội,
     │                        thay đổi tham số phía client, phát lại (replay)
     │                CONTROL: xác thực mạnh theo hạn mức (QĐ 2345: sinh trắc học khớp CCCD
     │                         với GD > 10tr/lần hoặc > 20tr/ngày), gắn thiết bị (device binding),
     │                         phát hiện root/jailbreak, chống chụp màn hình, TLS pinning
     ▼
[2] XÁC THỰC & PHÊ DUYỆT   OTP / sinh trắc / soft token / chữ ký số
     │                RỦI RO: chặn OTP (SIM swap), bỏ qua bước xác thực do lỗi logic,
     │                        chấp nhận sinh trắc bị giả mạo (deepfake)
     │                CONTROL: kiểm tra sống (liveness), ràng buộc OTP với ĐÚNG nội dung
     │                         giao dịch (what-you-see-is-what-you-sign), giới hạn số lần thử
     ▼
[3] KIỂM TRA RỦI RO   Fraud engine, hạn mức, sàng lọc AML
     │                RỦI RO: bỏ qua bằng cách đi đường vòng, luật quá lỏng, engine chết
     │                        thì "fail open" (cho qua hết)
     │                CONTROL: fail CLOSED, không đường vòng, giám sát chính engine, phiên bản
     │                         hóa bộ luật như code
     ▼
[4] GHI SỔ            Core banking ghi nợ/có (bút toán kép)
     │                RỦI RO: GHI 2 LẦN (retry không idempotent!), ghi một nửa (nợ mà không có),
     │                        chèn bút toán trái phép qua truy cập DB trực tiếp
     │                CONTROL: khóa idempotency, giao dịch có tính nguyên tử/saga có bù trừ,
     │                         CẤM truy cập DB production trực tiếp (đây là control bảo mật, không
     │                         phải quy ước dev), maker-checker cho bút toán thủ công
     ▼
[5] GỬI ĐI            NAPAS / CITAD / SWIFT / ví đối tác
     │                RỦI RO: sửa lệnh trên đường truyền, khóa ký bị đánh cắp, giả mạo tổ chức đối tác
     │                CONTROL: mTLS + ký ở tầng nghiệp vụ, khóa nằm trong HSM, secure zone,
     │                         danh sách trắng IP, giới hạn tần suất theo kênh
     ▼
[6] ĐỐI SOÁT & QUYẾT TOÁN   Cuối ngày (EOD), file đối soát với đối tác
     │                RỦI RO: LỆCH KHÔNG AI PHÁT HIỆN = mất tiền âm thầm; file đối soát bị sửa
     │                CONTROL: đối soát tự động + cảnh báo khi lệch, file có ký & checksum,
     │                         quy trình xử lý lệch có thời hạn
     ▼
[7] BẰNG CHỨNG        Log, audit trail, lưu trữ dài hạn
                      RỦI RO: log bị xóa/sửa (kẻ tấn công luôn dọn dấu vết), log chứa PII/PAN,
                              không đủ log để điều tra
                      CONTROL: log chỉ-ghi-thêm, đẩy sang SIEM ngoài tầm với của quản trị hệ thống,
                               che dữ liệu nhạy cảm khi ghi log, thời hạn lưu theo quy định
```

> **Bài tập bắt buộc (làm trong 2 tuần):** chọn MỘT luồng thật ở nơi bạn làm (ví dụ: chuyển tiền nhanh 24/7),
> vẽ đúng 7 chặng này với tên hệ thống thật, và điền control thật đang có + control còn thiếu.
> **Bản vẽ đó chính là hiện vật portfolio số 1 của bạn** và nó sẽ gây ấn tượng hơn bất kỳ chứng chỉ nào.

### 3.3 20 khái niệm nghiệp vụ bắt buộc phải hiểu

Không hiểu những khái niệm này, bạn sẽ đề xuất control sai chỗ và bị dev/nghiệp vụ coi thường.

| # | Khái niệm | Nghĩa ngắn gọn | Vì sao DevSecOps phải quan tâm |
|---|---|---|---|
| 1 | **Idempotency** | Gọi lại nhiều lần vẫn ra 1 kết quả | Retry ở tầng hạ tầng (LB, service mesh, job) có thể tạo giao dịch trùng → **thiết kế hạ tầng là rủi ro tài chính** |
| 2 | **Bút toán kép** (double-entry) | Mọi bút toán có nợ và có, tổng = 0 | Hiểu vì sao "sửa nhanh trong DB" là điều cấm kỵ tuyệt đối |
| 3 | **Đối soát** (reconciliation) | So khớp sổ nội bộ với đối tác | Sai lệch = chỉ báo phát hiện gian lận. Bảo vệ tính toàn vẹn của job đối soát |
| 4 | **Cut-off / EOD** | Mốc chốt ngày làm việc | Cửa sổ bảo trì, deploy, backup xoay quanh mốc này. Deploy sai giờ = sự cố |
| 5 | **Clearing vs Settlement** | Bù trừ (tính ra ai nợ ai) vs quyết toán (chuyển tiền thật) | Xác định thời điểm giao dịch không thể đảo ngược |
| 6 | **T+n** | Số ngày làm việc tới khi quyết toán | Cửa sổ để phát hiện & xử lý sự cố |
| 7 | **Maker–Checker (4 mắt)** | Người làm ≠ người duyệt | Áp dụng cho cả **thay đổi hạ tầng**: PR phải có người duyệt khác, không tự merge |
| 8 | **Chống chối bỏ** (non-repudiation) | Không thể phủ nhận đã làm | Vì sao audit log phải bất biến và ký |
| 9 | **Phân tách nhiệm vụ** (SoD) | Một người không nắm trọn quy trình | Ảnh hưởng trực tiếp thiết kế RBAC: dev không được có quyền production |
| 10 | **Hạn mức & phân tầng giao dịch** | Giao dịch to → xác thực mạnh hơn | Logic này phải enforce phía server, không tin client |
| 11 | **Chargeback / khiếu nại** | Khách đòi lại tiền | Cần log & bằng chứng đủ để giải quyết tranh chấp |
| 12 | **KYC / eKYC** | Định danh khách hàng | Sinh dữ liệu sinh trắc học — loại nhạy cảm nhất theo luật |
| 13 | **AML / CFT** | Chống rửa tiền & tài trợ khủng bố | Có nghĩa vụ báo cáo; sàng lọc danh sách trừng phạt không được lỗi âm thầm |
| 14 | **Ngày giá trị** (value date) | Ngày ghi nhận hiệu lực | Lỗi múi giờ/NTP = lỗi tài chính. **NTP là dịch vụ bảo mật trọng yếu** |
| 15 | **Nostro/Vostro** | Tài khoản của mình ở NH khác / ngược lại | Nền tảng của thanh toán quốc tế & luồng SWIFT |
| 16 | **ISO 20022** | Chuẩn bản tin thanh toán mới (thay MT) | Bản tin giàu dữ liệu hơn → nhiều PII hơn trong luồng, cần che khi log |
| 17 | **Tokenization vs Mã hóa** | Thay bằng token vô nghĩa vs mã hóa có khóa | Cách chính để **thu hẹp phạm vi PCI**. Kiến thức tiết kiệm tiền thật |
| 18 | **Cấp độ hệ thống thông tin (1–5)** | Phân loại theo quy định VN | Quyết định control bắt buộc. Nhầm cấp = nhầm toàn bộ thiết kế |
| 19 | **RTO / RPO** | Thời gian & lượng dữ liệu được phép mất | Ngân hàng thường yêu cầu RTO rất ngắn cho hệ thống trọng yếu — chi phối kiến trúc DR |
| 20 | **Rủi ro tồn dư & chấp nhận rủi ro** | Rủi ro còn lại sau control, được ai đó ký nhận | Cách Senior/Lead xử lý cái "không sửa được ngay" một cách chuyên nghiệp |

> **Cách học nhanh nhất:** xin ngồi cùng 1 buổi với đội **Vận hành thẻ**, 1 buổi với đội **Đối soát**,
> 1 buổi với đội **Quản lý rủi ro vận hành**. Ba buổi cà phê đó dạy bạn nhiều hơn 3 tháng đọc tài liệu,
> **và tạo quan hệ liên phòng ban — thứ bắt buộc phải có để lên Lead.**

### 3.4 Bản đồ tuân thủ Việt Nam + quốc tế

Bạn không cần thuộc lòng. Bạn cần **biết cái nào áp dụng cho hệ thống nào, và nó đòi control gì**.

#### Việt Nam — bắt buộc

| Văn bản | Áp dụng cho | Yêu cầu cốt lõi liên quan tới bạn |
|---|---|---|
| **[Thông tư 09/2020/TT-NHNN](https://thuvienphapluat.vn/van-ban/Tien-te-Ngan-hang/Thong-tu-09-2020-TT-NHNN-an-toan-he-thong-thong-tin-trong-hoat-dong-ngan-hang-455885.aspx)** (hiệu lực 01/01/2021, thay TT 18/2018) | TCTD, chi nhánh NH nước ngoài, trung gian thanh toán, công ty thông tin tín dụng | **Xương sống nghề của bạn.** Phân loại hệ thống theo cấp độ; quản lý tài sản; quản lý truy cập & tài khoản đặc quyền; an toàn vận hành & quản lý thay đổi; tách môi trường phát triển/kiểm thử/vận hành; nhật ký & giám sát; quản lý sự cố; sao lưu & dự phòng thảm họa (có diễn tập định kỳ); kiểm tra đánh giá ATTT định kỳ |
| **[Thông tư 50/2024/TT-NHNN](https://thuvienphapluat.vn/van-ban/Tien-te-Ngan-hang/Thong-tu-50-2024-TT-NHNN-an-toan-bao-mat-cung-cap-dich-vu-truc-tuyen-nganh-Ngan-hang-327954.aspx)** (hiệu lực 01/01/2025) | Dịch vụ trực tuyến: Internet/Mobile Banking, trung gian thanh toán | Xác thực theo phân loại giao dịch; **không cho ứng dụng ghi nhớ mật khẩu**; **không gửi SMS/email chứa link** (trừ khi khách yêu cầu); kiểm tra thiết bị; mã hóa dữ liệu; đánh giá ATTT trước khi đưa vào vận hành và định kỳ |
| **Quyết định 2345/QĐ-NHNN** (từ 01/07/2024) | Giao dịch trực tuyến của cá nhân | Xác thực **sinh trắc học khớp dữ liệu CCCD gắn chip** cho giao dịch vượt ngưỡng (>10 triệu/lần hoặc >20 triệu/ngày) |
| **[Luật Bảo vệ dữ liệu cá nhân 91/2025/QH15](https://baochinhphu.vn/luat-bao-ve-du-lieu-ca-nhan-chinh-thuc-co-hieu-luc-tu-ngay-mai-1-1-2026-102251231155609721.htm)** + **NĐ 356/2025/NĐ-CP** (hiệu lực **01/01/2026**) | Mọi tổ chức xử lý dữ liệu cá nhân | Nâng cấp từ NĐ 13/2023 lên cấp **luật**: cơ sở pháp lý cho việc xử lý, quyền của chủ thể dữ liệu (biết, đồng ý, truy cập, sửa, xóa), đánh giá tác động, hồ sơ chuyển dữ liệu ra nước ngoài, nghĩa vụ thông báo vi phạm. **Dữ liệu sinh trắc học & tài chính = nhạy cảm, mức bảo vệ cao nhất** |
| **Luật An toàn thông tin mạng 86/2015** + **NĐ 85/2016** + **TCVN 11930:2017** | Hệ thống thông tin theo cấp độ | Khung phân loại **cấp độ 1–5** và danh mục control kỹ thuật tương ứng. Hệ thống ngân hàng trọng yếu thường ở **cấp độ 3–5** |
| **Luật An ninh mạng 2018** + **NĐ 53/2022** | Doanh nghiệp cung cấp dịch vụ trên mạng | Yêu cầu lưu trữ dữ liệu trong nước với một số loại dữ liệu → **ràng buộc kiến trúc cloud** |
| **Luật Dữ liệu 60/2024/QH15** (hiệu lực 01/07/2025) | Quản trị dữ liệu quốc gia | Phân loại, chia sẻ, chất lượng dữ liệu; nền tảng cho các nghị định sau |

#### Quốc tế — áp dụng tùy nghiệp vụ

| Khung | Khi nào áp dụng | Điểm nóng hiện tại |
|---|---|---|
| **[PCI DSS v4.0.1](https://blog.pcisecuritystandards.org/now-is-the-time-for-organizations-to-adopt-the-future-dated-requirements-of-pci-dss-v4-x)** | Có xử lý/lưu/truyền dữ liệu thẻ | **51 yêu cầu "future-dated" đã bắt buộc từ 31/03/2025.** Đáng chú ý: **6.3.2** (kiểm kê phần mềm tự phát triển + thành phần bên thứ ba → thực chất là SBOM), **6.4.3 + 11.6.1** (kiểm kê script phía client + phát hiện sửa đổi trang thanh toán, chống e-skimming), **11.3.1.1** (xử lý cả lỗ hổng không-cao), **8.4.2** (MFA cho mọi truy cập vào CDE), **12.3.1** (phân tích rủi ro có mục tiêu) |
| **[SWIFT CSCF v2026](https://www.swift.com/myswift/services/training/swift-training-catalogue/browse-swift-training-catalogue/swift-customer-security-programme-v2026)** | Có kết nối SWIFT | 32 control; **2.4 "Back Office Data Flow Security" chuyển từ khuyến nghị sang BẮT BUỘC** — luồng giữa back-office và vùng SWIFT phải được mã hóa/phân đoạn/xác thực/giám sát. Mở rộng phạm vi sang customer connector (API, middleware, file transfer) |
| **ISO/IEC 27001:2022** | Chứng nhận hệ thống quản lý ATTT | Annex A mới có các control gắn thẳng với DevSecOps: **8.25** vòng đời phát triển an toàn, **8.28** lập trình an toàn, **8.31** tách môi trường, **5.23** an toàn dịch vụ đám mây |
| **SOC 2 Type II** | Bán dịch vụ B2B (fintech cho ngân hàng) | Chú trọng **hiệu lực của control theo thời gian** → đúng chỗ evidence-as-code phát huy |
| **[DORA](https://www.eiopa.europa.eu/digital-operational-resilience-act-dora_en)** (EU, áp dụng từ 17/01/2025) | Có khách hàng/pháp nhân EU | Quản trị rủi ro CNTT, **sổ đăng ký nhà cung cấp ICT bên thứ ba**, báo cáo sự cố theo thời hạn ngặt, **TLPT (kiểm thử xâm nhập theo kịch bản đe dọa) ít nhất 3 năm/lần** cho tổ chức trọng yếu |
| **NIST SSDF (SP 800-218)** | Khung tham chiếu tốt | Ngôn ngữ chuẩn để mô tả "quy trình phát triển an toàn" trước auditor |
| **SLSA v1.0+** | Bảo mật chuỗi cung ứng build | Thang đo mức độ tin cậy của quy trình build — dùng làm mục tiêu roadmap |
| **EU AI Act / OWASP LLM Top 10** | Nếu triển khai AI/LLM cho nghiệp vụ | Chấm điểm tín dụng bằng AI = hệ thống rủi ro cao → nghĩa vụ giải trình, ghi log, giám sát |

### 3.5 Dịch điều khoản → control → bằng chứng tự động (kỹ năng đắt nhất)

**Đây là kỹ năng phân biệt rõ nhất Senior với Lead trong ngành tài chính.** Hãy làm thành thạo mẫu 5 cột này:

| Điều khoản | Ý định thật | Control kỹ thuật | Thực thi tự động ở đâu | Bằng chứng sinh ra |
|---|---|---|---|---|
| TT 09 — tách môi trường phát triển/kiểm thử/vận hành | Code chưa duyệt không được chạm dữ liệu thật | Tách cluster/tài khoản cloud/VPC; cấm chia sẻ credential; dữ liệu production không xuống môi trường thấp | Terraform module theo môi trường + Kyverno chặn image từ registry sai + DLP trên job export dữ liệu | Báo cáo drift Terraform hằng ngày, log từ chối của Kyverno, log job che dữ liệu |
| TT 09 — quản lý tài khoản đặc quyền | Không ai tự ý vào production | Quyền theo yêu cầu, có hạn (JIT), qua PAM, ghi hình phiên | Vault dynamic credential + jump host + phê duyệt trong quy trình | Log cấp quyền theo thời gian + bản ghi phiên + báo cáo rà soát quyền hằng quý |
| TT 50 — không lưu mật khẩu trong app | Chống chiếm tài khoản khi mất máy | Vô hiệu hóa autofill/remember, xóa dữ liệu nhạy cảm khi vào nền | Kiểm tra tự động trong pipeline mobile (quét cấu hình + kiểm thử) | Kết quả test tự động lưu theo mỗi bản phát hành |
| QĐ 2345 — xác thực sinh trắc theo hạn mức | Chống chuyển tiền trái phép giá trị lớn | Kiểm tra hạn mức phía **server**, gắn với đúng nội dung giao dịch | Kiểm thử hợp đồng API trong CI (test case cho từng ngưỡng hạn mức) | Báo cáo kiểm thử + log quyết định xác thực (đã che PII) |
| Luật BVDLCN — dữ liệu sinh trắc | Bảo vệ mức cao nhất + quyền xóa | Mã hóa bằng khóa riêng trong HSM/KMS, danh sách truy cập tối thiểu, có vòng đời lưu trữ | Phân loại dữ liệu như code + chính sách IAM sinh tự động + job xóa theo hạn | Báo cáo tồn kho dữ liệu + log truy cập + biên bản xóa |
| PCI 6.3.2 — kiểm kê thành phần phần mềm | Biết mình đang chạy cái gì khi có CVE 0-day | Sinh SBOM cho mọi build, lưu tập trung, truy vấn được | Syft trong CI → đẩy vào kho SBOM (Dependency-Track) | SBOM theo từng artifact + truy vấn "ai đang dùng thư viện X" trong vài phút |
| PCI 11.6.1 — phát hiện sửa trang thanh toán | Chống e-skimming | Kiểm kê script phía client + giám sát toàn vẹn + CSP chặt | Kiểm tra CSP trong CI + giám sát tổng kiểm tra ngoài luồng | Cảnh báo có mốc thời gian + kiểm kê script đã duyệt |
| SWIFT 2.4 — luồng back-office | Chặn đường vòng vào vùng SWIFT | mTLS + phân đoạn mạng + xác thực & giám sát mọi luồng | NetworkPolicy/firewall as code + kiểm thử luồng tự động | Sơ đồ luồng sinh tự động + báo cáo kiểm thử phân đoạn |

**Quy tắc vàng:** *Nếu một control không tự sinh ra bằng chứng, thì đến kỳ kiểm toán bạn sẽ phải làm thủ công —
và đó là lúc mọi người ghét bảo mật.* Lead giỏi được nhận diện bằng việc **kỳ kiểm toán trở nên nhàm chán**.

**Cách đọc một thông tư cho hiệu quả** (kỹ năng meta, ít ai dạy):
```text
1. ĐỌC ĐIỀU 1–3 TRƯỚC: phạm vi, đối tượng áp dụng, giải thích từ ngữ.
   -> Trả lời câu quan trọng nhất: "cái này CÓ áp dụng cho hệ thống của tôi không?"
2. TÌM ĐỘNG TỪ BẮT BUỘC: "phải", "tối thiểu", "không được", "định kỳ".
   Chúng là control. Câu mô tả chung chung thì không phải.
3. TÌM CON SỐ: tần suất (hằng năm/quý), thời hạn lưu trữ, ngưỡng giá trị.
   Con số = thứ auditor sẽ đo bạn. Ghi lại hết.
4. TÌM "THEO QUY ĐỊNH CỦA PHÁP LUẬT": đây là con trỏ sang văn bản khác — đi theo nó.
5. VỚI MỖI ĐIỀU KHOẢN BẮT BUỘC, HỎI 3 CÂU:
   (a) Hiện ta có làm không? (b) Nếu có, bằng chứng nằm ở đâu? (c) Sinh bằng chứng đó tự động được không?
6. GHI VÀO BẢNG 5 CỘT Ở TRÊN. Sau 20 dòng, bạn hiểu thông tư đó hơn phần lớn đồng nghiệp.
```

### 3.6 Phân cấp độ hệ thống thông tin (1–5) — thứ quyết định control bắt buộc

Đây là khái niệm **đặc thù Việt Nam** mà tài liệu quốc tế không có, và là chỗ nhiều kỹ sư mơ hồ nhất.
Theo Luật An toàn thông tin mạng 86/2015, **NĐ 85/2016** và **TCVN 11930**, mọi hệ thống thông tin
được phân thành 5 cấp độ theo mức độ hậu quả nếu bị xâm hại. **Cấp độ quyết định danh mục control bắt buộc.**

| Cấp độ | Ý nghĩa (tóm lược) | Ví dụ trong ngân hàng | Hệ quả với bạn |
|:--:|---|---|---|
| **1** | Hậu quả nhỏ, phạm vi hẹp | Trang thông tin nội bộ, công cụ phụ trợ | Control cơ bản |
| **2** | Ảnh hưởng quyền lợi tổ chức/cá nhân | Hệ thống nội bộ không chạm tiền/PII nhạy cảm | Control cơ bản + quản lý truy cập |
| **3** | Ảnh hưởng nghiêm trọng tới sản xuất kinh doanh, quyền lợi nhiều người | **Phần lớn hệ thống ngân hàng trọng yếu**: Internet/Mobile Banking, hệ thống thanh toán, kho dữ liệu KH | Đầy đủ: phân vùng, giám sát, DR có diễn tập, kiểm tra ATTT định kỳ, quản lý thay đổi chặt |
| **4** | Ảnh hưởng tới lợi ích công cộng, trật tự xã hội | Hệ thống thanh toán quy mô lớn, hạ tầng dùng chung | Như cấp 3 + yêu cầu khắt khe hơn về kiểm tra, giám sát, báo cáo |
| **5** | Ảnh hưởng quốc phòng, an ninh quốc gia | Hạ tầng thông tin trọng yếu quốc gia | Mức cao nhất, có yêu cầu riêng của cơ quan chuyên trách |

> ⚠️ Bảng này là **bản đồ định hướng, không phải căn cứ pháp lý**. Hồ sơ đề xuất cấp độ phải do đơn vị
> lập và được cấp có thẩm quyền phê duyệt. Luôn đối chiếu NĐ 85/2016 + TCVN 11930 + hướng dẫn nội bộ.

**Vì sao Senior phải nắm chắc phần này:**

```text
1. NHẦM CẤP ĐỘ = NHẦM TOÀN BỘ THIẾT KẾ. Hệ thống cấp 3 mà làm control cấp 1 -> audit finding chắc chắn.
   Ngược lại, bọc cấp 4 cho một công cụ nội bộ -> lãng phí và bị coi là không hiểu việc.
2. CẤP ĐỘ LÀ CƠ SỞ ĐỂ BẠN NÓI "KHÔNG" CÓ TRỌNG LƯỢNG.
   "Việc này không làm được vì hệ thống ở cấp 3, theo quy định phải có X" — mạnh hơn "em thấy rủi ro".
3. CẤP ĐỘ QUYẾT ĐỊNH NGÂN SÁCH & ƯU TIÊN. Khi bạn xếp hạng 40 hệ thống để phân bổ nguồn lực (việc của Lead),
   cấp độ là trục xếp hạng đầu tiên, trước cả "có tiền không / có PII không".
4. ĐÂY LÀ CÂU HỎI PHỎNG VẤN LỌC NGƯỜI. Ứng viên nói được "hệ thống tôi phụ trách ở cấp 3, nên chúng tôi
   phải có A, B, C" thì lập tức khác hẳn ứng viên chỉ kể tên công cụ.
```

**Bài tập:** liệt kê mọi hệ thống bạn chạm tới, gắn cấp độ dự kiến + lý do, rồi **đem đi đối chiếu với
phòng ATTT (tuyến 2)**. Cuộc trao đổi đó vừa sửa hiểu biết của bạn, vừa là lần tiếp xúc tốt với tuyến 2.

---

## 4. Giai đoạn 1: → Senior DevSecOps (12–18 tháng)

Mục tiêu cuối giai đoạn: **bạn sở hữu trọn vẹn tư thế bảo mật kỹ thuật của một domain, và kiến trúc sư
tìm bạn TRƯỚC khi thiết kế.**

### Quý 1 — Xây nền & làm chủ luồng tiền
```text
KỸ THUẬT
  [ ] Chuẩn hóa pipeline bảo mật cho 1 ứng dụng thật, đủ 5 lớp:
      secret scan -> SCA -> SAST -> IaC scan -> image scan, kèm CHÍNH SÁCH FAIL rõ ràng
      (đừng bật hết rồi fail mọi thứ — sẽ bị tắt trong 2 tuần. Bắt đầu: chỉ fail secret + CRITICAL có bản vá).
  [ ] Tinh chỉnh: đưa false positive < 20% trước khi mở rộng sang app khác. Đây là bài học quan trọng nhất.
  [ ] Sinh SBOM cho mọi build, đẩy vào Dependency-Track. Tự kiểm tra: "thư viện X đang chạy ở đâu?" -> trả lời < 5 phút.

NGHIỆP VỤ  ⭐ trọng tâm quý này
  [ ] Vẽ luồng tiền 7 chặng (mục 3.2) cho MỘT dịch vụ thật, tên hệ thống thật.
  [ ] Cà phê với đội Đối soát, đội Vận hành thẻ, đội Rủi ro vận hành (mỗi đội 1 buổi).
  [ ] Nắm 20 khái niệm ở mục 3.3 tới mức giải thích được cho người khác.
  [ ] Xác định hệ thống bạn phụ trách thuộc CẤP ĐỘ mấy và vì sao.

ĐẦU RA: 1 sơ đồ luồng tiền + phân tích khoảng trống · 1 pipeline mẫu · 1 kho SBOM đang chạy.
```

### Quý 2 — Chuỗi cung ứng & policy as code
```text
KỸ THUẬT
  [ ] Ký image bằng cosign (khóa để trong KMS, KHÔNG để trong CI variable).
  [ ] Kyverno/OPA admission: chặn image không ký, chặn container chạy quyền root, chặn hostPath,
      bắt buộc resource limit & security context. Triển khai theo 3 bước: audit -> cảnh báo -> chặn.
  [ ] NetworkPolicy default-deny cho 1 namespace trọng yếu (làm được ở 1 chỗ rồi mới nhân rộng).
  [ ] Vault: chuyển từ secret tĩnh sang dynamic credential cho ít nhất 1 database.
  [ ] Diễn tập lộ secret: giả định 1 khóa bị lộ -> đo thời gian phát hiện & xoay vòng. Ghi lại số liệu.

NGHIỆP VỤ & TUÂN THỦ
  [ ] Đọc TT 09 và TT 50 (đọc thật, bản gốc). Lập bảng 5 cột mục 3.5 cho ÍT NHẤT 15 điều khoản.
  [ ] Xác định phạm vi PCI (nếu có thẻ): vẽ ranh giới CDE và chỉ ra cái gì làm nó phình ra.

ĐẦU RA: bộ policy trên git + báo cáo diễn tập xoay khóa + bảng "điều khoản → control" 15 dòng.
```

### Quý 3 — Threat modeling & thiết kế
```text
KỸ THUẬT
  [ ] Threat model STRIDE cho 1 hệ thống có luồng tiền. Không dùng template rỗng — bám vào 7 chặng của bạn.
      -> Dùng templates/threat-model-template.md (có ví dụ đã điền cho API chuyển tiền để đối chiếu).
  [ ] Viết 3 ADR bảo mật thật (dùng templates/adr-template.md). Ví dụ đề tài:
      "Tokenization tại tầng nào để thu hẹp phạm vi PCI", "Workload identity thay vì static key",
      "Fail-open hay fail-closed khi fraud engine chết".
  [ ] Runtime: Falco/Tetragon trên 1 cluster, tinh chỉnh tới mức < 5 cảnh báo nhiễu/ngày.
      Cảnh báo không ai xử lý thì tệ hơn không có cảnh báo.
  [ ] Tham gia hoặc chủ trì 1 phiên ứng cứu sự cố bảo mật (kể cả diễn tập trên bàn giấy).

ẢNH HƯỞNG
  [ ] Trình bày 1 chủ đề cho đội phát triển — chọn góc "làm sao đỡ khổ cho bạn", đừng chọn góc "quy định bắt buộc".
  [ ] Bắt đầu Security Champion: tìm 1 dev ở mỗi team sẵn sàng làm cầu nối.

ĐẦU RA: 1 threat model hoàn chỉnh · 3 ADR · Falco đã tinh chỉnh · 1 buổi chia sẻ.
```

### Quý 4 — Bằng chứng, kiểm toán & mở rộng
```text
  [ ] Evidence as code: dashboard tự sinh trạng thái control (bao nhiêu % service đã ký image,
      có NetworkPolicy, đạt SLA vá lỗi). Xuất được báo cáo cho kiểm toán KHÔNG cần làm tay.
  [ ] Xây quy trình quản lý lỗ hổng có SLA + đường ngoại lệ CÓ HẠN (ngoại lệ phải tự hết hạn).
      -> Bảng SLA & ma trận ưu tiên: xem mục 6.1. Mẫu hồ sơ ngoại lệ: templates/risk-acceptance-template.md.
  [ ] Tham gia ít nhất 1 kỳ kiểm toán/đánh giá thật với vai trò người cung cấp bằng chứng.
  [ ] Nhân rộng pipeline chuẩn ra 5+ service.
  [ ] Viết postmortem cho 1 sự cố bảo mật (kể cả sự cố nhỏ) theo hướng không đổ lỗi.

ĐẦU RA: dashboard tuân thủ · quy trình quản lý lỗ hổng đã được duyệt · 5+ service theo chuẩn.
CHECKLIST SẴN SÀNG LÊN SENIOR:
  [ ] Bạn tự ra được quyết định bảo mật cho domain mà không cần hỏi ai.
  [ ] Kiến trúc sư/PM mời bạn vào từ giai đoạn THIẾT KẾ, không phải trước khi go-live 1 tuần.
  [ ] Bạn đã LOẠI BỎ được ít nhất 1 LỚP lỗ hổng (không phải vá từng cái).
  [ ] Bạn giải thích được cho một người không rành kỹ thuật vì sao một rủi ro là quan trọng.
  [ ] Có ít nhất 1 việc bạn làm được auditor chấp nhận làm bằng chứng.
```

---

## 5. Giai đoạn 2: Senior → Lead DevSecOps (12–18 tháng)

Mục tiêu: **chuyển từ "tôi bảo vệ hệ thống" sang "tổ chức tự bảo vệ được, theo chuẩn tôi đặt ra".**

> Cảnh báo lớn nhất: ở giai đoạn này, **làm nhiều việc kỹ thuật hơn sẽ KHÔNG đưa bạn lên Lead.**
> Nó thậm chí cản đường — vì bạn trở thành nút thắt cổ chai, và tổ chức sợ mất bạn khỏi vị trí hiện tại.

### Quý 5–6 — Đòn bẩy: golden path & Security Champion
```text
  [ ] XÂY GOLDEN PATH: template dịch vụ mà team mới dùng là đã đạt ~80% yêu cầu bảo mật MẶC ĐỊNH
      (pipeline chuẩn, base image cứng hóa, policy, secret management, log chuẩn, dashboard).
      Thước đo thành công: team mới lên production ĐÚNG CHUẨN mà không hỏi bạn câu nào.
  [ ] CHÍNH THỨC HÓA SECURITY CHAMPION: mỗi team 1 người, có đào tạo, có quyền, được ghi nhận trong đánh giá.
      Đây là cơ chế nhân bản mạnh nhất của Lead — 1 bạn thành 10 bạn.
  [ ] Chuyển threat modeling thành DỊCH VỤ TỰ PHỤC VỤ: template + hướng dẫn + buổi review định kỳ.
      Bạn chỉ trực tiếp làm cho hệ thống cấp độ 4–5.
  [ ] Bắt đầu đo và công bố CHỈ SỐ: thời gian trung bình để vá theo mức độ, % phủ golden path,
      số ngoại lệ quá hạn, thời gian phát hiện. Không đo thì không quản trị được, và không thuyết phục được ai.
```

### Quý 7–8 — Chiến lược, tuân thủ & bên thứ ba
```text
  [ ] VIẾT CHIẾN LƯỢC BẢO MẬT KỸ THUẬT 18 THÁNG: hiện trạng -> mục tiêu -> khoảng trống ->
      lộ trình theo quý -> nguồn lực cần -> **cái gì CHỦ ĐỘNG KHÔNG LÀM và vì sao**.
      Mục cuối cùng là thứ khiến bản chiến lược đáng tin. Trình bày cho lãnh đạo kỹ thuật.
  [ ] Sở hữu MỘT khung tuân thủ đầu-cuối (PCI DSS hoặc SWIFT CSCF hoặc ISO 27001): từ phân tích
      khoảng trống -> khắc phục -> chứng thực. Đứng ra làm đầu mối kỹ thuật trước assessor.
  [ ] Xây quy trình ĐÁNH GIÁ BẢO MẬT BÊN THỨ BA: bộ câu hỏi, yêu cầu bằng chứng, điều khoản hợp đồng
      (quyền kiểm toán, nghĩa vụ báo cáo sự cố, yêu cầu SBOM), phân tầng rủi ro nhà cung cấp.
      Ngân hàng phụ thuộc rất nhiều vendor → đây là mảng thiếu người nghiêm trọng và rất được coi trọng.
  [ ] Xây/nâng cấp QUY TRÌNH ỨNG CỨU SỰ CỐ BẢO MẬT: phân loại mức độ, cây leo thang, ngưỡng báo cáo
      cơ quan quản lý, mẫu thông báo, chuỗi bằng chứng. Diễn tập trên bàn giấy 1 lần/quý với cả
      Pháp chế, Truyền thông, Nghiệp vụ (không chỉ kỹ thuật).
```

### Quý 9–10 — Vị thế lãnh đạo
```text
  [ ] Ngồi trong CAB / hội đồng kiến trúc / hội đồng rủi ro với tư cách tiếng nói bảo mật.
  [ ] Kèm cặp 1–2 người lên Senior — CÓ KẾT QUẢ NHÌN THẤY ĐƯỢC (họ được thăng cấp/nhận scope lớn hơn).
      Đây là bằng chứng thuyết phục nhất cho việc bạn sẵn sàng làm Lead.
  [ ] Tham gia tuyển dụng: thiết kế vòng phỏng vấn kỹ thuật bảo mật, phỏng vấn thật.
  [ ] Trình bày rủi ro cho cấp lãnh đạo bằng NGÔN NGỮ KINH DOANH:
      ❌ "Chúng ta có 43 lỗ hổng CRITICAL."
      ✅ "3 lỗ hổng nằm trên luồng thanh toán, khai thác được từ Internet, có thể dẫn tới giao dịch
          trái phép và nghĩa vụ báo cáo NHNN. Cần 2 tuần của 2 kỹ sư để xử lý. 40 lỗ hổng còn lại
          nằm ở hệ thống nội bộ, đề xuất xếp lịch quý sau."
  [ ] Ảnh hưởng ra ngoài tổ chức: chia sẻ ở meetup/cộng đồng, viết bài. Trong ngành tài chính,
      danh tiếng bên ngoài làm tăng trọng lượng tiếng nói bên trong một cách rõ rệt.

CHECKLIST SẴN SÀNG LÊN LEAD:
  [ ] Bạn nghỉ 3 tuần, chương trình bảo mật vẫn chạy đúng hướng (thử thật đi — đó là bài test).
  [ ] Có chuẩn/golden path mang tên bạn mà nhiều team đang dùng.
  [ ] Bạn đã đại diện kỹ thuật trước kiểm toán/assessor và bảo vệ được thiết kế.
  [ ] Có người bạn kèm đã lên cấp.
  [ ] Bạn nói KHÔNG với những việc đúng, có lý lẽ, và không bị mất uy tín.
  [ ] Lãnh đạo hỏi ý bạn về ưu tiên và ngân sách, không chỉ về kỹ thuật.
  [ ] Bạn đã xử lý ít nhất 1 xung đột Dev↔Security ra kết quả cả hai bên chấp nhận.
```

### Bạn đang xây cái gì? — cấu trúc một đội DevSecOps trưởng thành

Biết đích đến giúp bạn xếp ưu tiên. Đây là 5 mảng một đội DevSecOps ngân hàng trưởng thành phải phủ —
ở tổ chức nhỏ một người kiêm nhiều mảng, nhưng **mảng nào không có người thì đó là khoảng trống rủi ro**:

```text
Lead DevSecOps
 ├─ Pipeline & chuỗi cung ứng   : golden pipeline, SBOM, ký artifact, quản trị công cụ quét
 ├─ Cloud & container security  : policy-as-code, hardening, phân vùng, posture (CSPM)
 ├─ Định danh & bí mật          : IAM/PAM/Vault, SoD, rà soát quyền định kỳ
 ├─ Phát hiện & phản ứng        : phối hợp SOC — use case, runbook, diễn tập
 └─ Tự động hóa tuân thủ        : ánh xạ control, thu thập bằng chứng, dashboard tuân thủ
Vệ tinh: Security Champions trong từng team dev (bạn đào tạo & dẫn dắt, KHÔNG quản lý trực tiếp)
```

> Khi bạn tự chấm "mình đang phủ mảng nào, mảng nào trống", bạn có ngay nội dung cho bản đề xuất
> tăng người hoặc cho lộ trình 18 tháng — bằng ngôn ngữ mà lãnh đạo hiểu.

---

## 6. Vận hành thực chiến — bốn thứ tài liệu khác không nói

Bốn mục dưới đây là chỗ **lý thuyết gặp thực tế**. Nắm được chúng, bạn tránh được hầu hết
những cú vấp khiến người giỏi kỹ thuật vẫn bị đánh giá là "chưa chín".

### 6.1 SLA vá lỗi & ma trận ưu tiên (đừng dùng CVSS làm ưu tiên)

Sai lầm phổ biến: xếp ưu tiên theo **điểm CVSS**. CVSS đo mức nghiêm trọng *lý thuyết*, không đo
rủi ro *thật* trong kiến trúc của bạn. Một CVE 9.8 trong thư viện không nằm trên đường request
ít nguy hiểm hơn một CVE 6.5 ở API hướng Internet chạm dữ liệu khách hàng.

**Công thức ưu tiên dùng được ngay:**

```text
Mức ưu tiên = Mức nghiêm trọng × Độ phơi nhiễm × Độ nhạy dữ liệu × Cấp độ hệ thống
                                        ▲
                  yếu tố quan trọng nhất, và là yếu tố hay bị bỏ qua nhất
```

| Độ phơi nhiễm | Nghĩa | Hệ số |
|---|---|:--:|
| Hướng Internet, không cần xác thực | Ai cũng chạm được | ×3 |
| Hướng Internet, cần xác thực | Cần tài khoản hợp lệ | ×2 |
| Nội bộ, nhiều bên gọi | Cần đã vào được mạng trong | ×1 |
| Nội bộ, chỉ 1–2 dịch vụ gọi, có NetworkPolicy | Cần chiếm dịch vụ khác trước | ×0.5 |
| **Không nằm trên đường thực thi** (thư viện có nhưng không gọi tới) | Về lý thuyết không khai thác được | ×0.2 |

> Yếu tố cuối cùng gọi là **reachability**. Công cụ SCA hiện đại có phân tích này. Bật nó lên
> thường giảm 60–80% số lỗ hổng "phải xử lý" — và đó là cách bạn lấy lại thiện cảm của đội phát triển.

**Bảng SLA mẫu** (điều chỉnh theo tổ chức, nhưng phải có **con số** và phải được tuyến 2 duyệt):

| Mức ưu tiên | Hệ thống cấp 3–5, hướng Internet | Hệ thống nội bộ | Hành động khi quá hạn |
|---|:--:|:--:|---|
| **Nghiêm trọng** (đang bị khai thác ngoài thực địa) | **24 giờ** | 72 giờ | Kích hoạt quy trình sự cố, báo cáo lãnh đạo trong ngày |
| **Cao** | 7 ngày | 30 ngày | Leo thang tới chủ sở hữu hệ thống; chặn release tính năng mới |
| **Trung bình** | 30 ngày | 90 ngày | Đưa vào backlog sprint kế tiếp, báo cáo hằng tháng |
| **Thấp** | 90 ngày | Chu kỳ nâng cấp thường kỳ | Theo dõi, không leo thang |

```text
BỐN QUY TẮC LÀM SLA SỐNG ĐƯỢC:
1. ĐỒNG HỒ CHẠY TỪ LÚC PHÁT HIỆN, không phải lúc ai đó nhận ticket. Nếu không, người ta sẽ trì hoãn nhận.
2. QUÁ HẠN PHẢI CÓ HỆ QUẢ TỰ ĐỘNG (chặn release tính năng mới, không chặn hotfix). Không có hệ quả = không có SLA.
3. NGOẠI LỆ PHẢI QUA HỒ SƠ, không qua tin nhắn. Dùng templates/risk-acceptance-template.md.
4. ĐO VÀ CÔNG BỐ. "MTTR vá lỗi mức Cao: 11 ngày, mục tiêu 7" — con số công khai tạo áp lực lành mạnh
   tốt hơn mọi email nhắc nhở.
```

### 6.2 Triển khai guardrail mà không gây sự cố

> ⚠️ **Bài học đắt nhất của nghề này:** một guardrail chặn nhầm lúc 9h sáng ngày lương gây thiệt hại
> **lớn hơn** lỗ hổng mà nó chặn. Bạn sẽ mất quyền triển khai control trong 6 tháng sau đó.

**Quy trình 4 bước bắt buộc cho mọi control mới:**

```text
BƯỚC 1 — QUAN SÁT (2–4 tuần)
  Bật ở chế độ audit/dry-run. KHÔNG chặn gì. Thu thập: nếu bật chặn thì sẽ chặn bao nhiêu, cái gì.
  -> Đây là dữ liệu bạn cần để thuyết phục, và để không tự bắn vào chân mình.

BƯỚC 2 — CẢNH BÁO (2–4 tuần)
  Vẫn cho qua, nhưng cảnh báo cho team sở hữu + cho họ hạn sửa. Kèm HƯỚNG DẪN SỬA CỤ THỂ,
  không chỉ báo lỗi. Tốt nhất: đưa luôn dòng lệnh/PR mẫu để sửa.

BƯỚC 3 — CHẶN CÓ NGOẠI LỆ (từ đây trở đi)
  Bật chặn cho service MỚI trước, service CŨ có lộ trình. Danh sách miễn trừ tạm thời có hạn,
  ghi trong git, tự hết hạn.

BƯỚC 4 — CHẶN TOÀN BỘ
  Chỉ khi tỉ lệ vi phạm đã gần 0. Nếu bật chặn mà 40% build đỏ, bạn đã làm sai bước 1–3.
```

**Ba thứ mọi guardrail phải có trước khi bật chặn:**

| Thứ | Vì sao | Kiểm tra bằng câu hỏi |
|---|---|---|
| **Kill-switch** | Sự cố lúc 2h sáng, người trực phải tắt được mà không cần bạn | "Người trực có tắt được trong 5 phút không? Có runbook không?" |
| **Người trực & đường leo thang** | Control là hệ thống production, phải có người chịu trách nhiệm | "Ai nhận cảnh báo khi policy engine chết?" |
| **Chế độ suy giảm an toàn** | Policy engine chết → cụm ngừng nhận deploy? | "Fail-open hay fail-closed? Quyết định này đã được ai duyệt?" |

> **Fail-open hay fail-closed?** Không có câu trả lời chung — nó phụ thuộc *cái gì hỏng*:
> · **Fraud engine chết → fail CLOSED** (thà không cho giao dịch còn hơn cho giao dịch gian lận đi qua).
> · **Admission controller chết → cân nhắc fail-open có cảnh báo** (chặn mọi deploy giữa sự cố có thể
> ngăn chính bản vá khắc phục sự cố đó). Đây là loại đánh đổi bạn phải **viết ADR** và cho tuyến 2 duyệt,
> không phải quyết một mình.

### 6.3 Nhịp của tổ chức tài chính — đề xuất đúng lúc thì được duyệt

Ngân hàng có nhịp riêng. Cùng một đề xuất, đúng mùa thì được duyệt, lệch mùa thì chờ một năm.

```text
ĐÓNG BĂNG THAY ĐỔI (change freeze)
  Tết Nguyên đán · cuối năm tài chính · mùa quyết toán · các đợt cao điểm giao dịch
  -> Không đưa thay đổi lớn vào các cửa sổ này. Lập kế hoạch LÙI LẠI trước freeze, không phải sau.
  -> Mẹo: đặt hạn ngoại lệ TRƯỚC kỳ freeze, đừng để hạn rơi vào giữa freeze.

CHU KỲ KIỂM TOÁN
  Kiểm toán nội bộ, chứng thực PCI/SWIFT hằng năm, kiểm tra ATTT định kỳ
  -> Đây là ĐÒN BẨY của bạn: đề xuất gắn với finding hoặc với kỳ chứng thực sắp tới dễ được duyệt gấp bội.
  -> Biết lịch audit trước 6 tháng = biết trước cửa sổ cơ hội.

MÙA NGÂN SÁCH
  Thường lập kế hoạch quý 3–4 cho năm sau
  -> Đề xuất mua công cụ/tăng người phải vào ĐÚNG mùa này. Lệch mùa = chờ 12 tháng.
  -> Chuẩn bị từ 2 quý trước: thu thập số liệu chứng minh nhu cầu.

CHU KỲ BÁO CÁO
  Báo cáo rủi ro/tuân thủ hằng quý lên hội đồng
  -> Chỗ để bảo mật có tiếng nói ở cấp cao. Xin một slot 5 phút đều đặn còn giá trị hơn
     một bài trình bày hoành tráng mỗi năm một lần.
```

### 6.4 Ngôn ngữ rủi ro — cấu trúc 5 bước để nói chuyện với lãnh đạo

Đây là kỹ năng có ROI cao nhất ở ngưỡng Senior → Lead, và luyện được.

```text
❌ CÁCH NÓI KHIẾN BẠN BỊ TỪ CHỐI:
   "Cluster chưa có admission policy, cần triển khai Kyverno gấp."
   -> Lãnh đạo nghe: một kỹ sư muốn thêm một công cụ nữa. Không có cơ sở để quyết.

✅ CẤU TRÚC 5 BƯỚC:
   1. HIỆN TRẠNG (sự thật, không cảm tính)
      "Hiện bất kỳ ai có quyền deploy đều chạy được container privileged trên node production."
   2. KỊCH BẢN XẤU (câu chuyện cụ thể, có thật về mặt kỹ thuật)
      "Một tài khoản dev bị chiếm là đủ để tiếp cận toàn bộ node của kênh số, nơi có dữ liệu khách hàng."
   3. KHẢ NĂNG × ẢNH HƯỞNG (định lượng hết mức có thể)
      "Khả năng: trung bình — đã có 2 vụ lộ token trong 12 tháng qua.
       Ảnh hưởng: nghiêm trọng — dữ liệu KH + gián đoạn dịch vụ + nghĩa vụ báo cáo theo quy định."
   4. PHƯƠNG ÁN & CHI PHÍ (luôn đưa lựa chọn, đừng đưa tối hậu thư)
      "Phương án A: policy chặn ở admission, 3 tuần × 1 kỹ sư, đã kiểm thử không ảnh hưởng workload hiện tại.
       Phương án B: chỉ giám sát + cảnh báo, 1 tuần, nhưng không ngăn được."
   5. RỦI RO TỒN DƯ NẾU KHÔNG LÀM (đưa quyết định về đúng người)
      "Nếu chưa làm, rủi ro tồn dư ở mức Cao và cần chủ sở hữu hệ thống ký chấp nhận có thời hạn."
```

**Ba nguyên tắc đi kèm:**

```text
1. ĐỪNG BAO GIỜ CHỈ NÓI "KHÔNG".
   Nói: "Không theo cách này, vì rủi ro X. Có 2 cách khác: A (an toàn, chậm hơn 2 tuần),
   B (nhanh, nhưng cần chấp nhận rủi ro Y có thời hạn)." Rồi để chủ sở hữu rủi ro CHỌN và KÝ.
   -> Bạn chuyển từ "người cản trở" thành "người đưa lựa chọn". Khác biệt rất lớn.

2. COI TUYẾN 2 LÀ ĐỒNG MINH, KHÔNG PHẢI RÀO CẢN.
   Họ chịu trách nhiệm trước cơ quan quản lý. Nếu bạn giúp họ có bằng chứng đẹp và báo cáo dễ,
   họ sẽ chống lưng cho đề xuất của bạn. Đây là mối quan hệ giá trị nhất trong nghề này.

3. KHÔNG BAO GIỜ GIẤU SỰ CỐ. Ngân hàng tha thứ cho sự cố, không tha thứ cho việc giấu.
   Giấu một lần là mất toàn bộ uy tín đã xây nhiều năm.
```

---

## 7. Portfolio: 12 hiện vật chứng minh năng lực

Thăng cấp trong ngành tài chính là quyết định **có bằng chứng**. Hãy chuẩn bị sẵn 12 thứ này
(che thông tin nhạy cảm để dùng được cả trong phỏng vấn bên ngoài):

```text
NHÓM SENIOR
  1. Sơ đồ luồng tiền 7 chặng + phân tích khoảng trống control  ⭐ ấn tượng nhất
  2. Threat model STRIDE hoàn chỉnh cho 1 hệ thống có tiền
  3. 3–5 ADR bảo mật có nêu rõ đánh đổi (không phải "chọn X vì X tốt")
  4. Bảng "điều khoản pháp lý → control → bằng chứng" (mục 3.5) ≥ 20 dòng
  5. Bộ policy-as-code trên git + số liệu trước/sau khi triển khai
  6. Báo cáo diễn tập xoay vòng secret (có số đo thời gian)
  7. 1 postmortem bảo mật không đổ lỗi, có hành động phòng ngừa đã hoàn thành

NHÓM LEAD
  8. Bản chiến lược bảo mật kỹ thuật 18 tháng (có mục "không làm gì và vì sao")
  9. Tài liệu golden path + số team đã áp dụng
 10. Chương trình Security Champion: cấu trúc, giáo trình, kết quả
 11. Kết quả một kỳ chứng thực/kiểm toán bạn dẫn dắt (PCI/SWIFT/ISO)
 12. Bộ chỉ số theo thời gian: MTTR vá lỗi, độ phủ, ngoại lệ quá hạn — kèm biểu đồ xu hướng đi lên
```

> Lưu tất cả vào một repo riêng tư có cấu trúc rõ ràng. Khi đến kỳ đánh giá hoặc phỏng vấn,
> bạn không phải nhớ lại — bạn chỉ mở ra. **Người có hồ sơ luôn thắng người có trí nhớ tốt.**

**Công thức viết một dòng thành tích** (dùng cho CV, hồ sơ thăng cấp, và câu trả lời phỏng vấn):

```text
[Hành động kỹ thuật]  →  [thay đổi ĐO ĐƯỢC]  →  [ý nghĩa rủi ro / tiền / tuân thủ]
                                                          ▲
                            vế thứ ba là vế duy nhất lãnh đạo thật sự nghe
```

> ❌ "Triển khai Trivy và Kyverno cho cluster."
> ✅ "Chuẩn hóa quét image + admission control cho 42 dịch vụ: **100% workload production chạy image đã ký
> và đã quét**, loại bỏ hoàn toàn lớp rủi ro 'image không rõ nguồn gốc' — **đóng finding kiểm toán
> A-2025-17 trước hạn 6 tuần**."

Câu thứ hai và câu thứ nhất mô tả **cùng một công việc**. Chỉ câu thứ hai đưa bạn lên cấp.
Ở ngân hàng, thăng cấp là một **hồ sơ được chuẩn bị trước 2 quý**, không phải một cuộc trò chuyện —
nên hãy viết dòng thành tích **ngay khi làm xong**, đừng đợi tới kỳ đánh giá mới nhớ lại.

---

## 8. Kế hoạch 90 ngày đầu tiên (bắt đầu từ thứ Hai)

Nếu bạn chỉ làm được một phần của tài liệu này, hãy làm phần này.

```text
TUẦN 1–2  · Định vị
  [ ] Tự chấm điểm ma trận mục 2. Lưu vào git. Chọn 3 dòng yếu nhất.
  [ ] Liệt kê mọi hệ thống bạn chạm tới, gắn nhãn: cấp độ mấy? có tiền không? có PII không?
      có dữ liệu thẻ không? khung tuân thủ nào áp dụng?
  [ ] Đọc mục lục TT 09/2020 và TT 50/2024 (chưa cần đọc kỹ, cần biết nó nói về cái gì).

TUẦN 3–6  · Nghiệp vụ  ⭐ đây là phần bạn thấy mù mờ — đánh thẳng vào nó
  [ ] Vẽ luồng tiền 7 chặng cho 1 dịch vụ thật. Vẽ tay cũng được. Sai cũng được.
  [ ] Đem bản vẽ đi hỏi 3 người: 1 dev của hệ thống đó, 1 người đội đối soát, 1 người đội rủi ro.
      Câu hỏi mở đầu: "Em vẽ luồng này để hiểu rủi ro, anh/chị xem em sai chỗ nào ạ?"
      -> Vừa học được nghiệp vụ, vừa xây quan hệ. Không ai từ chối câu hỏi này.
  [ ] Sửa lại bản vẽ. Đánh dấu control đang có (xanh) / còn thiếu (đỏ).

TUẦN 7–10 · Thắng nhanh một trận có sức thuyết phục
  [ ] Chọn 1 khoảng trống MÀU ĐỎ vừa sức, sửa dứt điểm, ĐO trước/sau.
      Ưu tiên loại "loại bỏ cả một lớp lỗ hổng": ví dụ chặn secret vào git ở mức tổ chức,
      hoặc bắt buộc security context qua admission control.
  [ ] Viết 1 ADR cho quyết định đó, nêu rõ đánh đổi.

TUẦN 11–13 · Hiển thị
  [ ] Trình bày 20 phút cho đội kỹ thuật: bản đồ luồng tiền + rủi ro + việc đã làm + kế hoạch tiếp.
  [ ] Xin phản hồi từ sếp trực tiếp: "Để em được xem là Senior/Lead, còn thiếu gì ạ?"
      Ghi lại nguyên văn. Đó là bản đồ đường đi CHÍNH XÁC cho tổ chức của bạn — quý hơn mọi tài liệu chung.
  [ ] Bắt đầu bảng "điều khoản → control", điền 10 dòng đầu.
```

### Biến thể: nếu bạn MỚI vào một tổ chức tài chính

Kế hoạch trên dành cho người **đã ở trong** tổ chức. Nếu bạn vừa chuyển việc, thứ tự khác hẳn —
sai lầm chết người nhất của người mới là *sửa trước khi hiểu*:

```text
NGÀY 1–30 — HIỂU, TUYỆT ĐỐI ĐỪNG SỬA
  [ ] Vẽ được sơ đồ dòng tiền + danh mục hệ thống trọng yếu. HỎI, đừng đoán.
  [ ] Biết hệ thống nào thuộc phạm vi PCI (CDE) / SWIFT / chứa dữ liệu cá nhân / ở cấp độ mấy.
  [ ] Đọc: chính sách ATTT nội bộ, báo cáo kiểm toán gần nhất, danh sách finding đang mở, sổ rủi ro,
      sổ ngoại lệ. -> Đây là bản đồ CHÍNH XÁC về việc tổ chức đang đau ở đâu.
  [ ] Gặp 1-1: trưởng nhóm dev chính, phòng ATTT (tuyến 2), SOC, vận hành, kiểm toán nội bộ, đội fraud.
  [ ] Ghi lại "10 điều khiến tôi ngạc nhiên". Sau 3 tháng bạn sẽ quen và MẤT VĨNH VIỄN góc nhìn này —
      đây là tài sản chỉ người mới mới có.

NGÀY 31–60 — MỘT THẮNG LỢI NHỎ, CHẮC CHẮN
  [ ] Chọn việc thỏa cả 3: rủi ro rõ ràng + ít gây ma sát + đo được.
      Gợi ý tốt: bật secret scanning ở chế độ cảnh báo · che dữ liệu ở môi trường non-prod ·
      đóng một finding đã mở lâu ngày mà không ai nhận.
  [ ] Làm xong, đo, báo cáo ngắn có số liệu.
  [ ] ⚠️ TUYỆT ĐỐI KHÔNG bật chế độ CHẶN ở bất kỳ cổng nào trong 60 ngày đầu.
      Bạn chưa đủ vốn tín nhiệm để chịu một sự cố do mình gây ra.

NGÀY 61–90 — ĐỀ XUẤT HƯỚNG ĐI
  [ ] Trình 1 tài liệu: hiện trạng rủi ro + 3 ưu tiên 12 tháng + cách đo.
  [ ] Có 1 người ủng hộ ở tuyến 2 và 1 trưởng nhóm dev sẵn sàng làm thí điểm.
  [ ] Khởi động chương trình dài hạn đầu tiên.
```

---

## 9. Chứng chỉ — xếp theo ROI trong ngành tài chính

Chứng chỉ **không làm bạn lên cấp**, nhưng nó giúp: qua vòng lọc CV, tạo áp lực học có hệ thống,
và tăng độ tin cậy trước auditor. Thứ tự đề xuất:

| Ưu tiên | Chứng chỉ | Vì sao đáng | Lưu ý |
|---|---|---|---|
| ⭐⭐⭐ | **CKS** (Certified Kubernetes Security Specialist) | Trực tiếp với công việc, thực hành thật, hiếm người có | Cần CKA trước. ROI cao nhất cho bạn |
| ⭐⭐⭐ | **CISSP** | "Tấm vé" của ngành tài chính. Ngôn ngữ quản trị & rủi ro — thứ bạn cần cho Lead | Cần 5 năm kinh nghiệm. Học chính là học tư duy Lead |
| ⭐⭐ | **AWS Security Specialty** (hoặc tương đương cloud bạn dùng) | Nếu tổ chức dùng cloud nhiều | Bạn đã có nền AWS trong repo |
| ⭐⭐ | **PCI DSS ISA** (Internal Security Assessor) | Cực đắt giá nếu có nghiệp vụ thẻ. Rất ít người có | Cần tổ chức tài trợ |
| ⭐⭐ | **ISO 27001 Lead Implementer / Lead Auditor** | Học cách nghĩ của auditor = biết cách chuẩn bị bằng chứng | Nhanh, thực dụng |
| ⭐ | **CCSP** | Nếu đi sâu cloud security & quản trị | Trùng lặp một phần với CISSP |
| ⭐ | **OSCP** | Tư duy tấn công thật sự | Tốn thời gian; chỉ làm nếu bạn thích offensive |
| — | CEH | Ít giá trị thực tế, chủ yếu để qua bộ lọc HR ở một số nơi | Bỏ qua nếu được chọn |

> **Chiến lược tối ưu cho bạn:** `CKA → CKS` (năm 1) → `CISSP` (năm 2, khi đủ kinh nghiệm) →
> thêm khung tuân thủ theo đúng nghiệp vụ nơi bạn làm (PCI ISA nếu có thẻ, ISO LI nếu không).

---

## 10. Phỏng vấn: câu hỏi thật + cách trả lời

### Vòng Senior DevSecOps
| Câu hỏi | Họ đang kiểm tra | Cách trả lời tạo khác biệt |
|---|---|---|
| "Bạn nhúng bảo mật vào CI/CD thế nào?" | Có phải chỉ biết bật tool | Nói về **chính sách fail, tinh chỉnh false positive, và cách không chặn 40 team** — đừng liệt kê tên tool |
| "Có CVE CRITICAL trong thư viện đang chạy production, bạn làm gì?" | Quy trình & sự bình tĩnh | Xác định phạm vi ảnh hưởng bằng **SBOM** → đánh giá khả năng khai thác thật (có reachable không) → biện pháp giảm thiểu tạm thời → vá → hậu kiểm. Nhấn "khả năng khai thác" chứ không phải "điểm CVSS" |
| "Thiết kế bảo mật cho API chuyển tiền" | Có hiểu nghiệp vụ không | Đi theo **7 chặng luồng tiền**. Nhắc idempotency, hạn mức phía server, chống phát lại, đối soát. Đây là chỗ ứng viên thường lộ ra là không hiểu ngân hàng |
| "Dev nói control của bạn làm chậm release, xử lý sao?" | Kỹ năng hợp tác | Đưa số liệu, tách control chặn vs cảnh báo, đề xuất golden path để bảo mật thành đường dễ đi nhất |
| "Kubernetes multi-tenant cho ngân hàng, bạn cô lập thế nào?" | Chiều sâu kỹ thuật | Namespace + NetworkPolicy default-deny + RBAC + Pod Security + quota + admission + **và** nói rõ khi nào phải tách CLUSTER (ví dụ ranh giới CDE) |

### Vòng Lead DevSecOps
| Câu hỏi | Họ đang kiểm tra | Cách trả lời tạo khác biệt |
|---|---|---|
| "Bạn có 3 kỹ sư và 40 hệ thống. Ưu tiên thế nào?" | Tư duy Lead | Phân tầng theo **rủi ro kinh doanh** (có tiền / có PII / hướng Internet), đầu tư vào **đòn bẩy** (golden path, champion) thay vì phủ tuyến tính, và nói rõ **cái gì chấp nhận không làm** |
| "Làm sao bạn thuyết phục lãnh đạo đầu tư cho bảo mật?" | Kỹ năng kinh doanh | Ngôn ngữ rủi ro: xác suất × tác động, nghĩa vụ pháp lý, chi phí nếu xảy ra vs chi phí phòng ngừa. Gắn với mốc kiểm toán sắp tới |
| "Kể một lần bạn SAI về một quyết định bảo mật." | Sự trưởng thành | Kể thật một control quá chặt bị vô hiệu hóa/đi đường vòng, và bạn đã học được gì. Tránh kể chuyện giả vờ khiêm tốn |
| "Kiểm toán phát hiện thiếu bằng chứng cho control X. Bạn làm gì?" | Tư duy tuân thủ | Không cãi; xác nhận khoảng trống → biện pháp bù đắp ngay → tự động hóa việc sinh bằng chứng để không tái diễn |
| "Nói về một lần bạn nâng cấp được người khác." | Đòn bẩy | Kể cụ thể: ai, làm gì, kết quả đo được của HỌ. Đây là câu hỏi quan trọng nhất của vòng Lead |
| "Vendor core banking từ chối cho kiểm thử xâm nhập. Bạn xử lý sao?" | Thực tế ngành | Yêu cầu bằng chứng thay thế (chứng chỉ, báo cáo bên thứ ba), thêm điều khoản hợp đồng, control bù đắp phía mình (phân đoạn, giám sát), ghi nhận rủi ro tồn dư và trình cấp có thẩm quyền ký chấp nhận |

---

## 11. 10 cái bẫy khiến bạn kẹt ở Senior

```text
1.  BẬT MỌI SCANNER, FAIL MỌI BUILD.
    -> Bị vô hiệu hóa trong 1 tháng. Bắt đầu bằng cảnh báo, tinh chỉnh, rồi mới chặn.

2.  ĐO BẰNG SỐ LỖ HỔNG TÌM ĐƯỢC.
    -> Sai chỉ số. Đo bằng số lỗ hổng ĐÃ ĐÓNG đúng SLA và số LỚP lỗ hổng đã loại bỏ.

3.  LÀM CẢNH SÁT, KHÔNG LÀM ĐỒNG ĐỘI.
    -> Dev sẽ giấu bạn. Khi bảo mật là kẻ cản đường, người ta đi đường vòng.
       Nguyên tắc: mỗi khi bạn thêm một rào, hãy đồng thời gỡ một ma sát khác.

4.  CHỈ HỌC CÔNG CỤ, KHÔNG HỌC NGHIỆP VỤ.
    -> Đây chính là điều bạn đang lo, và đúng là nút thắt lớn nhất. Công cụ thay đổi mỗi 2 năm;
       hiểu biết về đối soát và luồng tiền thì dùng cả sự nghiệp.

5.  COI TUÂN THỦ LÀ VIỆC CỦA NGƯỜI KHÁC.
    -> Trong ngành tài chính, người dịch được điều khoản thành kiến trúc là người được lên Lead.

6.  ÔM HẾT VIỆC VÌ "TỰ LÀM NHANH HƠN".
    -> Trần cứng ngăn bạn lên Lead. Trước khi tự làm, hỏi: "việc này có nên thành template không?"

7.  KHÔNG VIẾT.
    -> Ở cấp cao, ảnh hưởng nhân qua CHỮ VIẾT. Không có ADR/design doc = không có bằng chứng năng lực.

8.  BỎ QUA BÊN THỨ BA.
    -> Ngân hàng phụ thuộc vendor rất nặng. Đây là mảng thiếu người và rất được coi trọng.

9.  KHÔNG BAO GIỜ NÓI KHÔNG.
    -> Nhận hết = làm hời hợt hết. Lead được đánh giá bằng chất lượng ưu tiên hóa.

10. CHỜ ĐƯỢC GIAO TITLE RỒI MỚI LÀM VIỆC CỦA CẤP ĐÓ.
    -> Ngược hoàn toàn. Thăng cấp là sự CÔNG NHẬN việc bạn ĐÃ làm ở cấp trên. Làm trước, title theo sau.
```

---

## 12. Tự đo tiến độ hằng quý

Mỗi quý, tự trả lời 8 câu này bằng **bằng chứng cụ thể**, không phải cảm giác. Lưu vào git.

```text
[ ] Quý này tôi loại bỏ được LỚP lỗ hổng nào? (không phải vá bao nhiêu cái)
[ ] Tôi hiểu thêm phần nghiệp vụ tài chính nào mà quý trước chưa hiểu?
[ ] Tôi viết ra được gì mà người khác đang dùng? (ADR, chuẩn, template, policy)
[ ] Ai giỏi lên nhờ tôi quý này? Đo bằng gì?
[ ] Tôi đã nói KHÔNG với việc gì, và có bảo vệ được lý do không?
[ ] Chỉ số của tôi thay đổi thế nào? (MTTR vá lỗi, độ phủ golden path, ngoại lệ quá hạn)
[ ] Tôi có tham gia quyết định nào ở tầm cao hơn cấp hiện tại không?
[ ] Nếu tôi nghỉ 3 tuần, cái gì sẽ đứng lại? (Danh sách này càng ngắn, bạn càng gần Lead)
```

**Chỉ số gợi ý cho bảng theo dõi cá nhân:**

| Chỉ số | Ý nghĩa | Hướng tốt |
|---|---|---|
| Thời gian vá trung bình theo mức nghiêm trọng | Hiệu lực vận hành | ↓ |
| % service theo golden path | Đòn bẩy | ↑ |
| Số ngoại lệ quá hạn | Kỷ luật quy trình | ↓ về 0 |
| Thời gian trả lời "thư viện X đang chạy ở đâu" | Độ trưởng thành SBOM | ↓ dưới 5 phút |
| % bằng chứng kiểm toán sinh tự động | Trưởng thành tuân thủ | ↑ |
| Số phát hiện lặp lại giữa 2 kỳ kiểm toán | Có sửa gốc rễ không | ↓ về 0 |
| Số người bạn kèm được lên cấp | Sẵn sàng làm Lead | ↑ |

**Bộ chỉ số đầy đủ khi bạn làm Lead** (7 nhóm — dùng để quản trị và để báo cáo lên hội đồng):

| Nhóm | Chỉ số | Vì sao lãnh đạo quan tâm |
|---|---|---|
| **Bao phủ** | % dịch vụ có đủ cổng kiểm tra · % workload production chạy image đã ký · % team trên golden path | Trả lời "ta kiểm soát được bao nhiêu phần?" |
| **Tốc độ xử lý** | MTTR lỗ hổng theo mức so với SLA · tuổi trung bình lỗ hổng tồn · số lỗ hổng quá hạn | Ánh xạ thẳng vào yêu cầu quản lý lỗ hổng của PCI và vào audit finding |
| **Chất lượng** | Tỉ lệ false positive · số lỗ hổng lọt ra production · số lỗ hổng **lặp lại cùng loại** | Chống bệnh "cài công cụ rồi bỏ đó"; lặp lại = đào tạo chưa hiệu quả |
| **Ma sát với dev** | Thời gian build tăng thêm · số build bị chặn nhầm · mức hài lòng của dev | **Bảo mật làm chậm dev = bảo mật sẽ bị vượt mặt.** Rất ít Lead đo cái này — đo được là lợi thế |
| **Tuân thủ** | Finding mở/đóng đúng hạn · số control tự sinh bằng chứng · số ngoại lệ đang mở & **quá hạn** | Ngôn ngữ của Ban điều hành và cơ quan quản lý |
| **Sẵn sàng ứng cứu** | MTTD/MTTR sự cố bảo mật · số kịch bản đã diễn tập · thời gian trả lời "ai bị ảnh hưởng bởi CVE X" | Chứng minh năng lực **chống chịu**, không chỉ phòng ngừa |
| **Quyền hạn** | Số tài khoản quyền cao · số phiên khẩn cấp (break-glass)/tháng và tỉ lệ đã hậu kiểm · số quyền bị cắt sau rà soát | Rủi ro nội gián — mối lo lớn nhất của ngành tài chính |

> **Quy tắc trình bày:** mỗi chỉ số phải kèm **ngưỡng và xu hướng**.
> "85%" là con số vô nghĩa. "85%, mục tiêu 95%, quý trước 60%" mới là thông tin để ra quyết định.

---

## 13. Từ điển Anh–Việt (để tra cứu & phỏng vấn)

Tài liệu này viết bằng tiếng Việt để bạn *hiểu*. Nhưng khi **tra cứu tài liệu, thi chứng chỉ và đi phỏng vấn**
(nhất là với ngân hàng nước ngoài hoặc công ty gia công), bạn cần đúng thuật ngữ tiếng Anh.

| Tiếng Việt (dùng trong tài liệu này) | Tiếng Anh | Ghi chú khi dùng |
|---|---|---|
| Ba tuyến phòng vệ | Three Lines of Defense | Nói đúng tên tiếng Anh trong phỏng vấn ngân hàng |
| Đối soát | Reconciliation | Google từ này để đọc tài liệu kỹ thuật |
| Bút toán kép | Double-entry bookkeeping | |
| Chốt ngày / cuối ngày | Cut-off / End-of-Day (EOD) | |
| Bù trừ / Quyết toán | Clearing / Settlement | Hai khái niệm khác nhau, đừng gộp |
| Bốn mắt / người làm–người duyệt | Four-eyes principle / Maker-Checker | |
| Phân tách nhiệm vụ | Segregation of Duties (SoD) | Auditor dùng từ này liên tục |
| Chống chối bỏ | Non-repudiation | |
| Rủi ro tồn dư | Residual risk | |
| Chấp nhận rủi ro | Risk acceptance | |
| Chủ sở hữu rủi ro | Risk owner | Người ký, không phải người làm |
| Phát hiện của kiểm toán | Audit finding | |
| Bằng chứng tuân thủ | Compliance evidence / Audit evidence | |
| Đóng băng thay đổi | Change freeze / Change blackout | |
| Hội đồng thay đổi | Change Advisory Board (CAB) | |
| Con đường vàng | Golden path / Paved road | Từ "paved road" phổ biến hơn ở công ty Mỹ |
| Rào chắn tự động | Guardrail | |
| Kiểm soát nạp vào cụm | Admission control | |
| Chính sách dạng mã | Policy as Code | |
| Bằng chứng dạng mã | Evidence as Code / Compliance as Code | |
| Danh mục thành phần phần mềm | SBOM (Software Bill of Materials) | |
| Khả năng bị chạm tới | Reachability | Yếu tố giảm tải lỗ hổng nhiều nhất |
| Định danh khối lượng công việc | Workload identity | |
| Quyền cấp theo nhu cầu, có hạn | Just-in-Time (JIT) access | |
| Quản lý truy cập đặc quyền | Privileged Access Management (PAM) | |
| Vùng dữ liệu thẻ | Cardholder Data Environment (CDE) | Thuật ngữ PCI, phải thuộc |
| Mã hóa thay thế bằng token | Tokenization | Cách chính để thu hẹp phạm vi PCI |
| Thu hẹp phạm vi | Scope reduction | Từ khóa vàng khi nói chuyện với QSA |
| Đánh giá tác động xử lý dữ liệu | DPIA (Data Protection Impact Assessment) | |
| Kiểm thử theo kịch bản đe dọa | Threat-Led Penetration Testing (TLPT) | Thuật ngữ DORA |
| Mô hình hóa mối đe dọa | Threat modeling | |
| Ranh giới tin cậy | Trust boundary | |
| Bán kính ảnh hưởng | Blast radius | |
| Khóa idempotency | Idempotency key | |
| Công tắc người chết | Dead man's switch | Cảnh báo khi job *không* chạy |
| Rà soát quyền định kỳ | Access recertification / User access review | Auditor rất hay hỏi |
| Hồ sơ nhà cung cấp ICT | Register of Information (RoI) | Thuật ngữ DORA |
| Chuyên gia đánh giá PCI | QSA (Qualified Security Assessor) / ISA (Internal) | |

> **Mẹo phỏng vấn:** khi trả lời, dùng thuật ngữ tiếng Anh cho *khái niệm chuẩn* (SoD, CDE, residual risk)
> nhưng giải thích bằng tiếng Việt cho *bối cảnh cụ thể*. Người phỏng vấn nhận ra ngay bạn đọc tài liệu gốc,
> không phải học qua tóm tắt.

---

## 14. Tài nguyên học

**Nghiệp vụ tài chính (đọc trước, đây là phần bạn thiếu):**
- Tài liệu công khai của **NAPAS** về luồng chuyển mạch & đối soát; tài liệu **ISO 20022** (mục Business Model).
- Blog kỹ thuật của các ngân hàng số & fintech (Monzo, Starling, Stripe, Adyen) — họ viết rất kỹ về idempotency, đối soát, thiết kế sổ cái. **Đọc mục "ledger" và "reconciliation" của Stripe/Modern Treasury.**
- Cổng thông tin **Ngân hàng Nhà nước** (sbv.gov.vn) và **Tạp chí Ngân hàng** — bài phân tích tác động của các thông tư mới.
- Nói chuyện với người thật ở đội Đối soát & Rủi ro vận hành — **hiệu quả hơn mọi tài liệu**.

**Tuân thủ (đọc bản gốc, đừng đọc tóm tắt của bên thứ ba):**
- [Thông tư 09/2020/TT-NHNN](https://thuvienphapluat.vn/van-ban/Tien-te-Ngan-hang/Thong-tu-09-2020-TT-NHNN-an-toan-he-thong-thong-tin-trong-hoat-dong-ngan-hang-455885.aspx) · [Thông tư 50/2024/TT-NHNN](https://thuvienphapluat.vn/van-ban/Tien-te-Ngan-hang/Thong-tu-50-2024-TT-NHNN-an-toan-bao-mat-cung-cap-dich-vu-truc-tuyen-nganh-Ngan-hang-327954.aspx)
- [Luật Bảo vệ dữ liệu cá nhân 2025 + NĐ 356/2025](https://baochinhphu.vn/luat-bao-ve-du-lieu-ca-nhan-chinh-thuc-co-hieu-luc-tu-ngay-mai-1-1-2026-102251231155609721.htm) · [Phân tích tác động tới ngành ngân hàng](https://tapchinganhang.gov.vn/luat-bao-ve-du-lieu-ca-nhan-nam-2025-va-nhung-tac-dong-doi-voi-linh-vuc-ngan-hang-tai-viet-nam-17211.html)
- [PCI Security Standards Council — blog & tài liệu v4.x](https://blog.pcisecuritystandards.org/now-is-the-time-for-organizations-to-adopt-the-future-dated-requirements-of-pci-dss-v4-x) · [SWIFT CSCF v2026](https://www.swift.com/myswift/services/training/swift-training-catalogue/browse-swift-training-catalogue/swift-customer-security-programme-v2026) · [DORA (EIOPA)](https://www.eiopa.europa.eu/digital-operational-resilience-act-dora_en)
- NIST SP 800-218 (SSDF) · CIS Benchmarks · ISO/IEC 27001:2022 Annex A

**Kỹ thuật DevSecOps:**
- *Threat Modeling: Designing for Security* — Adam Shostack (kinh điển, đọc chương STRIDE).
- *Securing DevOps* — Julien Vehent · *Container Security* — Liz Rice.
- *Alice and Bob Learn Application Security* — Tanya Janca (nền AppSec dễ vào).
- OWASP: Top 10 · **API Security Top 10** · ASVS · SAMM · Cheat Sheet Series.
- Tài liệu Kyverno, OPA Gatekeeper, Sigstore/cosign, SLSA, Dependency-Track.

**Lãnh đạo & tuân thủ:**
- *Staff Engineer* — Will Larson · *The Manager's Path* — Camille Fournier (chương Tech Lead).
- *How to Measure Anything in Cybersecurity Risk* — Hubbard & Seiersen — **đọc quyển này nếu muốn lên Lead**: dạy cách nói về rủi ro bằng con số thay vì màu đỏ/vàng/xanh.
- *The Phoenix Project* / *The Unicorn Project* — hiểu tâm lý tổ chức khi bảo mật va vào tốc độ.

---

## Mẫu văn bản đi kèm

Tài liệu này nhắc nhiều lần tới các văn bản bạn "phải viết được". Ba mẫu dưới đây có sẵn trong repo,
kèm **ví dụ đã điền cho bối cảnh ngân hàng** — copy về dùng ngay:

| Mẫu | Dùng khi | Có gì bên trong |
|---|---|---|
| [`templates/threat-model-template.md`](./templates/threat-model-template.md) | Trước khi xây hệ thống có luồng tiền / kênh mới / tích hợp đối tác | Quy trình 5 bước · STRIDE + 5 mối đe dọa đặc thù tài chính · **ví dụ đã điền: API chuyển tiền nhanh 24/7** với DFD, ranh giới tin cậy và 10 mối đe dọa có control |
| [`templates/risk-acceptance-template.md`](./templates/risk-acceptance-template.md) | Khi không thể sửa ngay nhưng vẫn phải đi tiếp | 4 nguyên tắc bất di bất dịch · khi nào **không** được dùng · **ví dụ đã điền** · cách vận hành sổ ngoại lệ (việc của Lead) |
| [`templates/adr-template.md`](./templates/adr-template.md) | Mọi quyết định kỹ thuật khó đảo ngược | Context → Options → Decision → Trade-off & Consequence |

> Ba loại văn bản này chính là **bằng chứng năng lực** ở ngưỡng Senior → Lead. Kỹ thuật thì khó chứng minh
> sau khi rời tổ chức; một threat model tốt và một hồ sơ ngoại lệ viết chuẩn thì mang đi phỏng vấn được.

---

## Tóm tắt một trang

```text
VẤN ĐỀ CỦA BẠN không phải thiếu kỹ thuật. Nền DevOps/K8s/Cloud của bạn đã đủ mạnh.
Cái thiếu là GÓC THỨ BA: nghiệp vụ tài chính + tuân thủ. Đó là lý do lộ trình thấy "mù mờ".

ĐỂ LÊN SENIOR (12–18 tháng):
  Chuyển từ "chạy công cụ" sang "thiết kế hệ thống kiểm soát".
  Việc quan trọng nhất: vẽ được luồng tiền 7 chặng của hệ thống thật + threat model nó
  + dịch được TT 09/TT 50 thành control có bằng chứng tự động.

ĐỂ LÊN LEAD (12–18 tháng tiếp):
  Chuyển từ "tôi làm được" sang "tổ chức làm được mà không cần tôi".
  Việc quan trọng nhất: golden path + Security Champion + chiến lược 18 tháng có mục "không làm gì"
  + dẫn dắt một kỳ chứng thực + kèm được người lên Senior.

BẮT ĐẦU THỨ HAI NÀY:
  1. Chấm điểm ma trận mục 2, chọn 3 dòng yếu nhất.
  2. Vẽ luồng tiền 7 chặng cho 1 dịch vụ thật — vẽ tay cũng được.
  3. Đem đi hỏi 3 người (dev / đối soát / rủi ro): "em vẽ sai chỗ nào ạ?"

Ba việc đó, làm trong 6 tuần, sẽ xóa hết cảm giác mù mờ hiện tại.
```
