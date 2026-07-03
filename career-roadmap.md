# Career Roadmap — DevOps/SRE → Architect (IC track) → Leadership

> Lộ trình phát triển sự nghiệp từ Middle DevOps đi lên, **nhấn mạnh nhánh IC/Architect (kỹ thuật sâu)**,
> kèm bản đồ nhánh Management để bạn chọn có chủ đích về sau.
> Đọc kèm: [`devops-fast-track.md`](./devops-fast-track.md) (kỹ năng kỹ thuật) · [`k8s-operations-playbook.md`](./k8s-operations-playbook.md) · [`cloud/aws/`](./cloud/aws/)

## 📑 Mục lục
- [0. Nguyên lý cốt lõi](#0-nguyên-lý-cốt-lõi-mỗi-cấp--đổi-thứ-bạn-tối-ưu)
- [1. Bản đồ hai nhánh: IC vs Management](#1-bản-đồ-hai-nhánh-ic-vs-management)
- [2. Nhánh IC/Architect — đi sâu](#2-nhánh-icarchitect--đi-sâu-trọng-tâm)
- [3. Kỹ năng ra quyết định kiến trúc](#3-kỹ-năng-ra-quyết-định-kiến-trúc-lõi-của-architect)
- [4. Skill Matrix theo cấp (tự chấm điểm)](#4-skill-matrix-theo-cấp-tự-chấm-điểm)
- [5. Kỹ năng mềm — nút thắt thật sự](#5-kỹ-năng-mềm--nút-thắt-thật-sự)
- [6. Nhánh Management (để biết & chọn sau)](#6-nhánh-management-để-biết--chọn-sau)
- [7. Checklist "sẵn sàng lên cấp"](#7-checklist-sẵn-sàng-lên-cấp)
- [8. Kế hoạch 12 tháng (Middle/Senior → Staff/Architect)](#8-kế-hoạch-12-tháng-middlesenior--staffarchitect)
- [9. Cách đi nhanh nhất (meta)](#9-cách-đi-nhanh-nhất-meta)
- [10. Sách & tài nguyên](#10-sách--tài-nguyên)

---

## 0. Nguyên lý cốt lõi: mỗi cấp = đổi "thứ bạn tối ưu"

Sai lầm lớn nhất: nghĩ lên cấp = "giỏi kỹ thuật hơn". Thực tế **giá trị của bạn đổi bản chất** ở mỗi ngưỡng.

```text
Middle    -> tối ưu: HOÀN THÀNH TỐT task được giao (execute)
Senior    -> tối ưu: GIẢI QUYẾT VẤN ĐỀ độc lập (own outcome, tự quyết kỹ thuật, không cần dắt tay)
Staff/Lead-> tối ưu: TÁC ĐỘNG QUA NGƯỜI KHÁC + hướng kỹ thuật (multiply, set standard)
Principal -> tối ưu: KẾT QUẢ KỸ THUẬT CẤP TỔ CHỨC (định hình kiến trúc & chuẩn cho nhiều team)
```

Điểm mấu chốt của IC track cấp cao: **scope of impact mở rộng** (1 task → 1 hệ thống → nhiều team → cả tổ chức), trong khi vẫn giữ chiều sâu kỹ thuật.

---

## 1. Bản đồ hai nhánh: IC vs Management

```text
                         Senior Engineer
                        /               \
        ┌──────────────┘                 └──────────────┐
        ▼  IC TRACK (kỹ thuật sâu)          MANAGEMENT TRACK (con người) ▼
   Staff Engineer / Tech Lead            Engineering Manager
        │                                     │
   Senior Staff / Principal Architect    Senior Manager / Director
        │                                     │
   Distinguished Engineer / Fellow       VP Engineering / Head of
        │                                     │
        └──────────── (thường ngang cấp lương/ảnh hưởng) ─────────┘
```

**Sự thật quan trọng:** Manager KHÔNG phải "cấp trên" của Staff Engineer — nó là **nghề khác**, cùng thang ảnh hưởng. IC track lên rất cao (Principal/Distinguished có tiếng nói ngang Director). Bạn chọn IC = đúng nếu bạn **yêu giải quyết vấn đề kỹ thuật khó** và muốn giữ chiều sâu.

> Bạn đang chọn IC/Architect → phần 2, 3, 4 là trọng tâm. Nhưng đọc phần 6 để biết nhánh kia, vì Staff+ vẫn cần kỹ năng "lãnh đạo không quyền lực".

---

## 2. Nhánh IC/Architect — đi sâu (trọng tâm)

### 2.1 Các cấp và scope

| Cấp | Scope tác động | Bạn được đánh giá bởi |
|-----|----------------|------------------------|
| **Senior** | 1 hệ thống/service, tự chủ | Giải quyết vấn đề khó độc lập, chất lượng, tin cậy |
| **Staff / Tech Lead** | 1 team / vài hệ thống liên quan | Định hướng kỹ thuật, nâng cả team, giải bài toán mơ hồ |
| **Principal / Architect** | Nhiều team / 1 domain lớn | Kiến trúc xuyên hệ thống, chuẩn kỹ thuật, quyết định one-way-door |
| **Distinguished / Fellow** | Toàn tổ chức / ngành | Định hình chiến lược kỹ thuật, ảnh hưởng ngoài công ty |

### 2.2 Cái phân biệt Staff+ (không phải "code giỏi hơn")

- **Giải bài toán MƠ HỒ.** Senior giải bài rõ ràng. Staff nhận bài "hệ thống chậm và tốn kém, làm gì đó đi" — tự định nghĩa vấn đề, phạm vi, giải pháp.
- **Tư duy hệ thống & blast radius.** Thấy được 1 thay đổi lan tới đâu, failure mode nào, hệ quả 2-3 năm.
- **Đòn bẩy (leverage).** Không tự làm mọi thứ — viết chuẩn, dựng platform/tooling, mentor để **10 người làm tốt hơn** thay vì tự làm việc của 2 người.
- **Judgment kỹ thuật.** Ra quyết định đánh đổi đúng với thông tin thiếu. Đây là tài sản quý nhất, đến từ **reps + postmortem + phản tư**.
- **Ảnh hưởng không quyền lực.** Thuyết phục team/sếp bằng lập luận & uy tín, không bằng chức danh.
- **"Glue work" kỹ thuật.** Ghép các mảnh, gỡ bế tắc liên team, thấy bức tranh lớn mà không ai thấy.

### 2.3 Nền kỹ thuật cần liên tục đào sâu (T-shaped)

```text
Chiều SÂU (chọn 1-2 làm thế mạnh cốt lõi):
  - Kubernetes & container orchestration (bạn đang xây - tốt)
  - Cloud architecture (AWS sâu - đang làm)
  - Networking (từ CIDR tới service mesh, hybrid)
  - Reliability/SRE (SLO, capacity, DR, chaos)

Chiều RỘNG (đủ để ra quyết định & nói chuyện với mọi team):
  - Security (zero-trust, IAM, mã hóa, compliance)
  - Data (database, streaming, data platform, cost của data)
  - Observability (metrics/logs/traces, cost của observability)
  - CI/CD & Developer Experience (platform engineering)
  - Cost/FinOps (chi phí là quyết định kiến trúc)
  - Một chút software engineering (đọc code, viết tool, hiểu app team)
```

> Nguyên tắc: **sâu 1-2 cột trụ để có tiếng nói chuyên gia, rộng phần còn lại để ra quyết định hệ thống.** Architect chỉ rộng mà không sâu → "kiến trúc sư PowerPoint", không ai nghe.

### 2.4 Từ "vận hành" sang "định hình platform"

DevOps middle thường mắc kẹt ở "chạy việc". Để lên Architect, chuyển tư duy:
```text
Vận hành từng hệ thống   ->  Xây PLATFORM để nhiều team tự phục vụ
Sửa từng sự cố           ->  Loại bỏ CẢ LỚP sự cố (thiết kế lại, guardrail, automation)
Làm task nhanh           ->  Đặt CHUẨN (golden path) để 10 team làm nhanh & đúng
"Tôi biết cách làm"      ->  "Tôi khiến team không cần hỏi tôi vẫn làm đúng"
```
Đây chính là **Platform Engineering** — nơi giá trị Architect DevOps cao nhất.

---

## 3. Kỹ năng ra quyết định kiến trúc (lõi của Architect)

Đây là kỹ năng được trả lương cao nhất. Khung tư duy áp dụng cho mọi quyết định:

```text
1. XUẤT PHÁT TỪ RÀNG BUỘC, không từ công nghệ.
   Non-functional requirements quyết định kiến trúc: scale, latency, availability (mấy số 9?),
   compliance, budget, team hiện có, timeline. Công nghệ là hệ quả, không phải điểm bắt đầu.

2. MỌI THỨ LÀ ĐÁNH ĐỔI. Không có "tốt nhất", chỉ có "phù hợp nhất với ràng buộc NÀY".
   Trục đánh đổi: Cost ↔ Reliability ↔ Performance ↔ Security ↔ Complexity ↔ Time-to-market.
   Nêu rõ bạn đang tối ưu trục nào và HY SINH trục nào.

3. ONE-WAY vs TWO-WAY DOOR (Amazon).
   Quyết định khó đảo (chọn cloud, data model, ngôn ngữ core, chia service) -> cân nhắc kỹ, nhiều dữ liệu.
   Quyết định dễ đảo -> quyết nhanh, học từ thực tế, đừng phân tích tê liệt.

4. BUILD vs BUY, MANAGED vs SELF-HOSTED.
   Mặc định chọn managed/buy, TRỪ KHI có lý do rõ (chi phí ở quy mô, đặc thù, khóa nhà cung cấp).
   Mỗi thứ tự vận hành = nợ kỹ thuật + chi phí người vĩnh viễn.

5. CONWAY'S LAW. Kiến trúc sẽ phản chiếu cơ cấu giao tiếp của tổ chức.
   Thiết kế hệ thống HỢP với tổ chức thật (Team Topologies), không phải tổ chức lý tưởng.

6. CHỐNG OVER-ENGINEERING. Độ phức tạp là kẻ thù của độ tin cậy.
   Match complexity với nhu cầu THẬT hôm nay + tăng trưởng THẤY ĐƯỢC, không phải "biết đâu sau này".
   "Boring technology" thắng "shiny new" trong hầu hết trường hợp production.

7. THIẾT KẾ CHO THẤT BẠI. Mọi thứ sẽ hỏng. Blast radius nhỏ, graceful degradation, tự hồi phục.

8. GHI LẠI BẰNG ADR (Architecture Decision Record):
   Context (bối cảnh & ràng buộc) -> Options (đã cân nhắc gì) -> Decision -> Trade-off & Consequence.
   -> Người sau hiểu "tại sao", tránh tranh cãi lại, và bạn xây được uy tín có hệ thống.
```

**Ở quy mô doanh nghiệp, thêm:**
- Multi-account/multi-tenant governance, compliance & audit (không phải afterthought).
- **Migration strategy** — thực tế hiếm khi greenfield; biết migrate hệ thống đang chạy (strangler fig, 7R).
- **Standardization & golden path** — cho phép tốc độ ở quy mô mà không hỗn loạn.
- **FinOps** — chi phí là ràng buộc kiến trúc first-class, không phải hóa đơn cuối tháng.
- **Cân bằng chuẩn hóa ↔ tự chủ team** — quá cứng thì kìm hãm, quá lỏng thì hỗn loạn.

> Bài luyện: mỗi quyết định kỹ thuật bạn gặp, ép mình viết 1 ADR ngắn. Sau 6 tháng bạn sẽ tư duy kiến trúc tự nhiên.

---

## 4. Skill Matrix theo cấp (tự chấm điểm)

Chấm mỗi dòng 1-5 (1=chưa có, 5=dạy được người khác). Chỗ thấp so với cấp mục tiêu = việc cần làm.

| Năng lực | Senior | Staff | Principal |
|----------|:------:|:-----:|:---------:|
| Chiều sâu kỹ thuật (1-2 domain) | Vững 1 domain | Chuyên gia, người khác hỏi | Định hình domain |
| Chiều rộng (nhiều domain) | Biết dùng | Ra quyết định liên domain | Chiến lược đa domain |
| Ra quyết định kiến trúc + ADR | Cho 1 hệ thống | Cho nhiều hệ thống/team | Xuyên tổ chức |
| Xử lý bài toán mơ hồ | Bài rõ ràng | Tự định nghĩa vấn đề | Đặt đúng vấn đề cho tổ chức |
| Đòn bẩy (tooling/platform/chuẩn) | Tự động hóa việc mình | Nâng cả team | Nâng nhiều team |
| Ảnh hưởng không quyền lực | Trong team | Liên team | Toàn tổ chức + bên ngoài |
| Viết & giao tiếp (design doc) | Rõ ràng | Thuyết phục | Định hình tư duy người khác |
| Mentoring | 1-2 người | Cả team | Nâng cả cấp Senior/Staff |
| Business/cost acumen | Hiểu cost cơ bản | Gắn tech với business | Ra quyết định business-tech |
| Reliability/SRE mindset | Cho service mình | Cho team | Chuẩn cho tổ chức |

---

## 5. Kỹ năng mềm — nút thắt thật sự

90% engineer kẹt ở đây, không phải ở code. Kể cả IC track, từ Staff trở lên **kỹ năng mềm quyết định**, không phải kỹ thuật.

- **Viết tốt.** Design doc, ADR, postmortem, RFC. Ở cấp cao, ảnh hưởng của bạn nhân qua **chữ viết** (không phải code). Đây là kỹ năng đòn bẩy #1. Luyện: viết mỗi tuần.
- **Giao tiếp & influence without authority.** Thuyết phục bằng lập luận + dữ liệu + uy tín, không bằng chức danh. Trình bày cho cả kỹ sư lẫn sếp non-tech.
- **Business acumen.** Dịch tech → rủi ro/chi phí/doanh thu/thời gian. Sếp không quan tâm "p99 latency", họ quan tâm "khách rời web vì chậm → mất tiền".
- **Ownership & độ tin cậy.** Nói được làm được, đều đặn. Đây là **currency of promotion** — người ta trao scope lớn cho người họ TIN.
- **Mentoring & nâng người khác.** Ở Staff+, thành công đo bằng "team/tổ chức làm được gì nhờ bạn", không phải "bạn làm được gì".
- **Xử lý bất đồng & cho feedback khó.** Tranh luận kỹ thuật lành mạnh, disagree-and-commit.
- **Prioritization & nói KHÔNG.** Biết cái gì KHÔNG làm quan trọng hơn làm nhiều.

---

## 6. Nhánh Management (để biết & chọn sau)

Ngay cả khi chọn IC, hiểu nhánh này giúp bạn cộng tác tốt & giữ cửa mở.

```text
Engineering Manager : 1:1 đều đặn, feedback, hiring/interview, performance management
                      (kể cả xử lý người yếu), delegation, tạo psychological safety,
                      own roadmap & delivery, phát triển sự nghiệp của report. Hands-on ít.
Senior Mgr/Director : quản lý các MANAGER (không phải engineer), org design, ngân sách & headcount,
                      chiến lược nhiều quý, OKR, cross-functional (Product/Sales/Finance).
VP/Head/Trưởng div. : chiến lược tổ chức, văn hóa, đại diện, quyết định lớn (build/buy/M&A),
                      hiring lãnh đạo, gắn kỹ thuật với mục tiêu công ty.
```

**Dấu hiệu bạn HỢP Management:** thấy vui khi người khác lớn lên hơn khi tự giải bài; giỏi gỡ xung đột; kiên nhẫn với con người & sự mơ hồ tổ chức. **Nếu chưa chắc → ở IC lâu hơn**, thử Tech Lead (nửa kỹ thuật nửa dẫn dắt) để nếm thử.

> Chuyển từ IC sang Manager luôn có thể làm sau. Nhưng đừng lên Manager chỉ vì "đó là bước tiếp theo" — nhiều người giỏi khổ vì bỏ thứ họ yêu để làm thứ họ không hợp.

---

## 7. Checklist "sẵn sàng lên cấp"

**Middle → Senior:**
```text
[ ] Tự own một service/hệ thống end-to-end, không cần dắt tay.
[ ] Debug production khó dưới áp lực, tự tìm ra root cause.
[ ] Ra quyết định kỹ thuật cho phạm vi của mình, biết trade-off.
[ ] Người mới hỏi bạn về mảng bạn phụ trách.
[ ] Code/config bạn viết là chuẩn tham khảo cho team.
```

**Senior → Staff / Tech Lead:**
```text
[ ] Nhận bài toán MƠ HỒ và tự định nghĩa vấn đề + giải pháp.
[ ] Tác động vượt ra ngoài việc của riêng bạn (nâng cả team).
[ ] Viết design doc/ADR mà người khác dùng để ra quyết định.
[ ] Dẫn dắt kỹ thuật một dự án liên nhiều người.
[ ] Mentor được Junior/Middle lên tay.
[ ] Được liên team tìm đến khi có vấn đề khó.
```

**Staff → Principal / Architect:**
```text
[ ] Kiến trúc/chuẩn của bạn ảnh hưởng NHIỀU team.
[ ] Ra được quyết định one-way-door với thông tin thiếu, và đúng.
[ ] Định hình hướng kỹ thuật của cả một domain lớn.
[ ] Xây platform/chuẩn khiến nhiều team làm nhanh & đúng hơn.
[ ] Tiếng nói của bạn được cân nhắc ngang cấp Director trong quyết định lớn.
```

---

## 8. Kế hoạch 12 tháng (Middle/Senior → Staff/Architect)

```text
Quý 1 — Củng cố chiều sâu + bắt đầu viết
  - Chọn 1-2 domain trụ (K8s + AWS đang làm) -> đào tới mức chuyên gia (đã có tài liệu).
  - Bắt đầu viết: 1 ADR/design doc cho MỖI quyết định kỹ thuật ở việc thật.
  - Lấy 1 cert nền (SAA hoặc CKA) làm forcing function.

Quý 2 — Mở rộng scope + đòn bẩy
  - Nhận (hoặc tự đề xuất) một vấn đề LỚN HƠN việc thường ngày (vd: "giảm 30% cost hạ tầng",
    "chuẩn hóa CI/CD cho các team", "dựng platform self-service").
  - Thay vì tự làm: viết chuẩn/tooling/automation để team khác dùng.
  - Mentor 1 người. Trình bày 1 chủ đề cho team (visibility).

Quý 3 — Ảnh hưởng liên team + business
  - Dẫn dắt kỹ thuật 1 dự án chạm nhiều team.
  - Ngồi họp với Product/Finance -> học ràng buộc business & cost thật.
  - Viết 1 RFC/đề xuất kiến trúc được cấp trên xem xét.
  - Xây "incident/decision journal" -> tích lũy judgment.

Quý 4 — Thể hiện ở cấp mục tiêu ("act the level")
  - Own một quyết định kiến trúc one-way-door có tác động rõ.
  - Có sponsor (không chỉ mentor) nói tốt về bạn trong phòng thăng chức.
  - Hồ sơ: các ADR/design doc + kết quả đo được (cost giảm, uptime tăng, team nhanh hơn).
  - Đề xuất thăng cấp với BẰNG CHỨNG bạn đã LÀM việc của cấp đó rồi.
```

Nguyên tắc: **promotion là công nhận việc bạn ĐÃ làm ở cấp cao hơn, không phải cơ hội để thử.** Làm trước, title theo sau.

---

## 9. Cách đi nhanh nhất (meta)

```text
1. "ACT THE LEVEL" - làm việc của cấp trên MỘT bậc trước khi có title.
2. OWN thứ có TÁC ĐỘNG BUSINESS rõ và đo được (cost, uptime, tốc độ team).
3. XÂY UY TÍN & LÒNG TIN - tài sản quý nhất, currency of promotion. Nói được làm được, đều đặn.
4. TÌM SPONSOR, không chỉ mentor. Sponsor nói tốt về bạn TRONG phòng ra quyết định.
5. VISIBILITY - việc vô hình không được thưởng. Viết, trình bày, chia sẻ (như repo này).
6. HỌC BUSINESS, không chỉ tech. Hiểu công ty kiếm tiền & ưu tiên thế nào.
7. JUDGMENT = reps + reflection. Mỗi quyết định & sự cố -> rút bài học, GHI LẠI.
8. DẠY LẠI - nén kiến thức 2 lần & xây danh tiếng chuyên gia.
```

**Lỗ hổng phổ biến của DevOps muốn lên cao (tự soi):**
- Giỏi tool, yếu **thiết kế hệ thống & trade-off** → luyện viết design doc, học kiến trúc.
- Yếu **viết & giao tiếp** → ép viết ADR/postmortem/RFC mỗi tuần.
- Không hiểu **business/cost** → học FinOps, ngồi họp với Product/Finance.
- Ôm hết việc, không **delegate/mentor/tạo đòn bẩy** → trần cứng chặn lên Staff.

---

## 10. Sách & tài nguyên

**IC/Architect track:**
- *Staff Engineer: Leadership Beyond the Management Track* — Will Larson (kim chỉ nam nhánh IC).
- *The Staff Engineer's Path* — Tanya Reilly.
- *Fundamentals of Software Architecture* — Mark Richards & Neal Ford.
- *Designing Data-Intensive Applications* — Martin Kleppmann (kinh điển về hệ thống dữ liệu).
- *Site Reliability Engineering* + *The SRE Workbook* — Google (miễn phí online).
- *Team Topologies* — Skelton & Pais (kiến trúc ↔ tổ chức).
- *Building Microservices* — Sam Newman.

**Ra quyết định & tư duy:**
- ADR: xem `adr.github.io` và mẫu của Michael Nygard.
- *Thinking in Systems* — Donella Meadows.
- Blog: martinfowler.com, AWS Architecture Blog, các engineering blog (Netflix, Uber, Cloudflare).

**Khi cân nhắc Management (sau này):**
- *The Manager's Path* — Camille Fournier.
- *An Elegant Puzzle* — Will Larson.

**Cộng đồng & cập nhật:** CNCF, KubeCon talks, các RFC/design doc công khai của công ty lớn, đọc postmortem public (học judgment từ sự cố người khác).

---

### Lời cuối
Bạn đang đi đúng: chọn IC/Architect để giữ chiều sâu, và **xây một body-of-work công khai** (chính repo tài liệu này) — đó vừa là học, vừa là visibility, vừa là bằng chứng năng lực. Ba việc quan trọng nhất từ giờ: **(1) đào sâu 1-2 domain tới mức chuyên gia, (2) viết — ADR/design doc mỗi tuần, (3) mở rộng scope: từ "làm task" sang "xây chuẩn & platform cho người khác".** Làm đều 3 việc đó, con đường lên Staff/Principal là tất yếu.
