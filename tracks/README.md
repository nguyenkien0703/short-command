# Specialization Tracks — **DevSecOps (trục chính)** · **SRE (trục hai)**

> **Cập nhật 09/2026 — đổi trục.** Trước đây folder này lấy MLOps làm lõi. Nay định hướng đã rõ:
> **đào sâu DevSecOps trước, bồi SRE sau, dừng theo đuổi MLOps.**
> Tài liệu này là *big picture + lộ trình 12 tháng*; hai file [devsecops.md](./devsecops.md) và
> [sre.md](./sre.md) là phần đào sâu từng hướng.

## 📌 Quyết định định hướng (ghi lại để khỏi lung lay)

```text
TRỤC CHÍNH  : DevSecOps  — bảo mật hoá toàn vòng đời (Security as Code)
TRỤC HAI    : SRE        — độ tin cậy định lượng (SLO/error budget), bồi từ tháng ~10
NỀN         : DevOps đã có (K8s, CI/CD, IaC, cloud, observability) -> tài sản, không vứt đi
DỪNG        : MLOps/LLMOps làm hướng nghề. File cũ giữ lại làm THAM CHIẾU, không nằm trong lộ trình.
NGÁCH TÙY CHỌN: AI/LLM security — chỉ chạm khi công việc thật đụng tới (xem §6)
```

Vì sao thứ tự này hợp lý: DevSecOps và SRE **dùng chung một bộ nền** (pipeline, K8s, IaC, incident,
observability) nên học cái sau rẻ hơn nhiều. Ngược lại, MLOps đòi hỏi một nền riêng (data/model/GPU/
experiment) — bỏ ra là bỏ đúng thứ *ít cộng hưởng nhất* với hai hướng còn lại.

## 📂 Trong folder này

**Đang theo đuổi**
- **[devsecops.md](./devsecops.md)** — 🎯 trục chính: mindset shift-left, bản đồ security theo từng bước pipeline, trụ cột kỹ năng, lộ trình.
- **[sre.md](./sre.md)** — 🥈 trục hai: SLI/SLO/error budget, incident & postmortem, chaos, capacity.

**Tham chiếu (không nằm trong lộ trình — đọc khi công việc cần)**
- [roadmap-ai-platform-engineer.md](./roadmap-ai-platform-engineer.md) — *(lộ trình cũ, MLOps-core — giữ làm lịch sử quyết định)*
- [mlops.md](./mlops.md) · [ml-serving-k8s.md](./ml-serving-k8s.md) · [llmops-end-to-end.md](./llmops-end-to-end.md) · [fine-tuning-lora.md](./fine-tuning-lora.md) · [vector-db-scale.md](./vector-db-scale.md)
- [ml-concepts-explained.md](./ml-concepts-explained.md) — vẫn hữu ích: đủ vốn khái niệm ML để **bảo mật/vận hành** hệ AI của người khác mà không phải thành MLOps.

## 1. So sánh hai hướng đang theo

| Tiêu chí | **DevSecOps** (chính) | **SRE** (bổ trợ) |
|----------|----------------------|------------------|
| Câu hỏi cốt lõi | "Hệ thống có an toàn & chứng minh được không?" | "Hệ thống có đáng tin không?" |
| Đơn vị quan tâm | Threat, vulnerability, artifact tin cậy, compliance | SLI/SLO, error budget, incident, toil |
| Sản phẩm bạn giao | Security gate trong CI/CD, policy-as-code, SBOM/chữ ký, bằng chứng tuân thủ | Dashboard SLO, runbook, postmortem, hệ tự hồi phục |
| Ai chấm điểm bạn | Auditor, khách hàng doanh nghiệp, quy định pháp lý | Người dùng, on-call, Product |
| Đòn bẩy nền DevOps của bạn | **Rất cao** — bạn đã sở hữu chính pipeline cần bảo mật | **Rất cao** — playbook sự cố bạn viết đã là tư duy SRE |
| Rào cản cần vượt | Tư duy tấn công, app security, compliance | Định lượng (SLO), coding nhiều hơn scripting |
| Chức danh | DevSecOps / Cloud Security / Platform Security Engineer | SRE / Production / Platform Engineer |
| Đường lên Architect | **Security Architect / Cloud Security Architect** | Principal SRE / Reliability Architect |

**Giao thoa (học một, lợi hai):** cùng một quy trình incident (security incident = một loại sự cố
production, dùng chung severity/on-call/blameless postmortem) · cùng một triết lý *assume breach ≈
assume failure* · cùng dùng policy-as-code & admission control (chặn cấu hình nguy hiểm **và** chặn
cấu hình kém tin cậy) · cùng nền observability (audit log & detection ngồi trên chính stack metrics/log/trace).

```text
        DevOps của bạn (K8s · CI/CD · IaC · cloud · observability)   <- ĐÃ CÓ
                 |                              |
        + DevSecOps (an toàn, chứng minh được)  + SRE (đáng tin, đo được)
                 \                              /
                  =>  Platform/Security Engineer -> Security Architect
                      (vị thế hiếm: hiểu hệ thống ĐỦ SÂU để bảo mật nó, không chỉ đi quét)
```

## 2. Bối cảnh 2026 — cái gì đang đẩy nhu cầu DevSecOps

> Phần này quyết định *học gì trước*. Mỗi mục: bối cảnh → **việc cần làm được**.

**a) Supply chain là mặt trận nóng nhất.** OWASP Top 10:2025 thêm hẳn hạng mục mới
**A03 — Software Supply Chain Failures** (tỷ lệ gặp cao nhất nhưng gần như không có CVE để mà quét),
và Security Misconfiguration nhảy lên **#2**. Thực tế 2025–2026 khớp: vụ **tj-actions/changed-files**
(3/2025) trích OIDC token ngay từ bộ nhớ tiến trình runner; worm **Shai-Hulud** trên npm (9/2025, biến
thể V2 11/2025, còn dư chấn sang 2026) tự nhân bản bằng chính token npm nó nhặt được trong CI.
→ **Cần làm được:** pin action theo commit SHA, token least-privilege & ngắn hạn, chặn post-install
script, SBOM cho mọi build, và coi **CI runner là production** (nơi có nhiều bí mật nhất công ty).

**b) Artifact phải chứng minh được nguồn gốc.** SBOM (CycloneDX/SPDX) + ký bằng **Sigstore/cosign** +
**provenance SLSA L3** đã thành mặc định của pipeline trưởng thành, không còn là "điểm cộng".
→ **Cần làm được:** sinh SBOM tự động → ký image → admission control **chỉ chạy image đã ký & đã quét**.

**c) Quy định đã tới hạn — và Việt Nam có luật riêng.**
- **EU CRA**: từ **11/9/2026** bắt buộc báo cáo lỗ hổng đang bị khai thác (cảnh báo sớm 24h) tới ENISA/CSIRT; **11/12/2027** áp dụng đầy đủ (CE marking).
- **Việt Nam**: **Luật Bảo vệ dữ liệu cá nhân (91/2025/QH15)** + **NĐ 356/2025/NĐ-CP** hiệu lực **1/1/2026**, thay thế NĐ 13/2023 — quyền chủ thể dữ liệu, đánh giá tác động, hồ sơ xử lý dữ liệu.
→ **Cần làm được:** biết dữ liệu cá nhân nằm ở đâu trong hệ thống (data map/lineage), tự động **thu thập bằng chứng** tuân thủ thay vì làm tay trước mỗi kỳ audit. Đây là phần khiến DevSecOps có tiếng nói ở cấp C-level.

**d) Danh tính máy (non-human identity) là chu vi mới.** Khoảng **2/3 tổ chức vẫn sống bằng static
credential**; hướng dịch chuyển là **secretless / zero standing privilege**: mTLS + **SPIFFE/SPIRE**,
workload identity của cloud, token ngắn hạn thay API key vĩnh viễn.
→ **Cần làm được:** bỏ long-lived secret khỏi CI và khỏi service (OIDC federation → cloud role), phát
hành & xoay vòng danh tính workload tự động.

**e) Runtime-first.** Thị trường CNAPP đang hội tụ về **runtime-first** (Forrester Wave Q1/2026); phía
open-source, **Falco** (CNCF graduated) cùng **Tetragon/Tracee** (eBPF) đã đủ chín cho production, và
**Kyverno graduated 3/2026** — policy-as-code chính thức là hạ tầng chuẩn, không phải thử nghiệm.
→ **Cần làm được:** default-deny network policy, Pod Security Standards, Kyverno/OPA enforce, Falco/
Tetragon phát hiện hành vi bất thường lúc chạy.

**f) Thị trường.** Cloud security là kỹ năng **tuyển nhiều nhất** và cũng là khoảng trống lớn thứ hai
(chỉ sau AI); vị trí cloud security mất **65+ ngày** mới tuyển được. Người **vừa biết vận hành vừa biết
bảo mật** (Terraform/K8s/CI-CD + security) được trả cao hơn analyst bảo mật thuần khoảng **20–40%**.
→ **Đúng chỗ bạn đang đứng.** Lợi thế của bạn không phải "biết security", mà là **biết security trên
chính hệ thống bạn đang vận hành**.

**g) (Tùy chọn) AI security — ngách, không phải trục.** OWASP đã có **Top 10 for LLM Apps (2025)** và
**Top 10 for Agentic Applications (2026)**; rủi ro MCP server (tool poisoning, endpoint không xác thực)
đang là chủ đề nóng 2026. Bạn **không cần làm MLOps** để chạm mảng này — nó là *application security cho
một loại ứng dụng mới*. Chỉ đầu tư khi công việc thật có LLM/agent.

## 3. Lộ trình 12 tháng (DevSecOps trước, SRE sau)

> Nguyên tắc: **mỗi giai đoạn phải đẻ ra một artifact thật** (pipeline/policy/tài liệu) trên hệ thống
> đang làm. Học chay mảng này rất nhanh quên.

```text
T0        Kiểm kê & threat model
T1-T3     Shift-left trong pipeline        \
T4-T6     Supply chain & artifact trust     |  DevSecOps (trục chính)
T7-T8     Runtime & policy-as-code          |
T9-T10    Cloud security & compliance      /
T11-T12   SRE lớp bổ trợ (SLO/incident/chaos)
```

**T0 — Kiểm kê (2 tuần).** Vẽ bản đồ 1 hệ thống thật đang vận hành: luồng dữ liệu, nơi chứa secret,
ai/cái gì có quyền gì. Làm **threat model STRIDE** cho nó → ra danh sách rủi ro xếp hạng. Đây là tài liệu
nền cho mọi việc phía sau (và là thứ rất đáng đưa vào ADR).

**T1–T3 — Shift-left trong pipeline (nền tảng).**
- SAST (Semgrep) + SCA (Trivy/Grype) + secret scanning (gitleaks) + IaC scan (checkov/tfsec) vào CI, có **gate**: nghiêm trọng → fail build.
- Chống nhiễu: baseline lỗi cũ, chỉ chặn cái *mới* — pipeline hay chết vì báo động giả hơn vì thiếu tool.
- **Hardening chính CI/CD** (bài học 2025–2026): pin action theo SHA, quyền token tối thiểu, tách runner cho job tin cậy, bỏ secret dài hạn → OIDC.
- OWASP Top 10:2025 + ASVS đọc song song để có vốn app security.

**T4–T6 — Supply chain & niềm tin vào artifact.**
- SBOM tự động (Syft → CycloneDX/SPDX), lưu trữ & **quét lại khi có CVE mới**.
- Ký image bằng **cosign/Sigstore**, sinh provenance hướng **SLSA L3**.
- Admission control: cluster **chỉ chấp nhận image đã ký + đã quét** (Kyverno verifyImages).
- Base image tối thiểu (distroless/chainguard-style), vá & rebuild tự động.

**T7–T8 — Runtime & policy-as-code (đúng thế mạnh K8s của bạn).**
- Kyverno/OPA: enforce Pod Security Standards, cấm privileged/hostPath, bắt buộc resource limit & label.
- NetworkPolicy default-deny + phân đoạn; RBAC dọn về least-privilege thật sự.
- Falco hoặc Tetragon: rule phát hiện shell trong container, ghi file lạ, kết nối bất thường → nối vào alert.
- Secrets: Vault/External Secrets, xoay vòng tự động; **workload identity (SPIFFE/SPIRE hoặc cloud-native)** thay static key.

**T9–T10 — Cloud security & compliance.**
- CSPM/misconfiguration (Prowler/ScoutSuite hoặc dịch vụ cloud), IAM least-privilege có phương pháp (phân tích quyền thực dùng).
- Chọn **một** framework để map: CIS Benchmark → rồi SOC2/ISO 27001 nếu công ty cần; với thị trường VN thêm **Luật BVDLCN 91/2025 + NĐ 356/2025** (dữ liệu cá nhân nằm đâu, cơ sở pháp lý xử lý, quyền xoá/truy cập).
- **Tự động hoá bằng chứng**: control → kiểm tra tự động → báo cáo. Đây là kỹ năng hiếm và rất "được giá".

**T11–T12 — Bồi SRE lên chính hệ đã bảo mật.**
- SLI/SLO + error budget cho 1 service thật; alert theo **burn rate** (giảm nhiễu — cùng bài toán với alert bảo mật).
- Observability chuẩn **OpenTelemetry** (vendor-neutral, đang là mặc định 2026); dashboard error budget.
- Incident: runbook, on-call, **blameless postmortem** — dùng chung quy trình cho cả sự cố bảo mật.
- Chaos/game day; và bản bảo mật của nó: **purple-team game day** (giả lập tấn công → đo thời gian phát hiện & phản ứng).
- Nâng coding: viết 1 controller/operator hoặc tool tự động hoá toil (SRE thiên software).

**Xuyên suốt cả năm:** ① tư duy tấn công — lab/CTF nhẹ hằng tuần (đọc được lỗ hổng mới hiểu được phòng
thủ); ② mỗi quyết định thật → 1 **ADR** ([templates/](../templates/)); ③ ghi lại vào chính repo này.

## 4. ✅ Cột mốc — bằng chứng năng lực (không phải "đã đọc xong")

```text
[ ] Threat model + data map cho 1 hệ thống thật, có xếp hạng rủi ro.
[ ] CI có security gate thật sự chặn được release xấu, tỷ lệ báo động giả thấp (dev không né).
[ ] Không còn long-lived secret trong CI (OIDC/workload identity), token least-privilege.
[ ] Mọi image production đều có SBOM + chữ ký; cluster từ chối image không ký (chứng minh bằng demo).
[ ] Policy-as-code enforce PSS + default-deny network policy trên cluster thật.
[ ] Runtime detection có rule tự viết, từng bắt được ít nhất 1 hành vi bất thường (kể cả trong diễn tập).
[ ] Map được hệ thống với 1 framework + tự động thu thập bằng chứng.
[ ] (SRE) SLO + error budget + burn-rate alert cho 1 service; 1 postmortem blameless đã viết.
[ ] Chạy được 1 purple-team game day: đo MTTD/MTTR cho một kịch bản tấn công.
Capstone: "secure-by-default platform" — repo mẫu: pipeline có gate + SBOM/ký + policy + runtime
          detection + SLO dashboard, dựng bằng IaC/GitOps, kèm tài liệu threat model & compliance map.
```

## 5. 🎓 Cert & tài nguyên (chọn lọc, không ôm đồm)

**Cert theo thứ tự đáng làm nhất với nền của bạn**
```text
1. CKS (cần CKA trước)          — sát việc nhất, chứng minh security trên K8s. Ưu tiên #1.
2. Cloud security specialty     — AWS Security Specialty (hoặc tương đương cloud bạn dùng).
3. Offensive nhẹ (tùy chọn)     — CPTS/PNPT/BSCP để có "credibility tấn công"; OSCP chỉ khi thực sự đi sâu.
4. Security+ / CCSP / CISSP     — giá trị hồ sơ & đường architect/quản lý, làm sau, không phải bây giờ.
```
> Cert chỉ là *chất xúc tác*. Thứ được trả tiền là **hệ thống bạn đã làm cho an toàn** — capstone ở §4 nặng ký hơn mọi tấm bằng.

**Tài nguyên lõi**
- **Sách bắc cầu hai hướng:** *Building Secure and Reliable Systems* (Google, free) — đúng chỗ giao giữa SRE và security.
- *Container Security* (Liz Rice) · *Kubernetes Security* (CNCF/NSA-CISA hardening guide) · *Threat Modeling* (Adam Shostack).
- Chuẩn & framework: **OWASP Top 10:2025**, OWASP ASVS, **SLSA**, **NIST SSDF (SP 800-218)**, CIS Benchmarks.
- SRE: *SRE Book* + *SRE Workbook* (Google, free); OpenTelemetry docs.
- (Nếu chạm AI) OWASP **Top 10 for LLM Apps 2025** + **Top 10 for Agentic Applications 2026**.

## 6. Còn mấy file MLOps/LLMOps trong folder thì sao?

Không xoá — **giữ làm tham chiếu**, vì hai lý do: (1) công việc hiện tại vẫn có thể đụng hệ AI của người
khác; (2) khi đó bạn vào với **tư cách DevSecOps/SRE**, không phải MLOps:

| Tình huống thật | Đọc file nào | Vai của bạn |
|-----------------|--------------|-------------|
| Cần hiểu team AI đang nói gì (train/inference/RAG…) | [ml-concepts-explained.md](./ml-concepts-explained.md) | đủ vốn từ để không bị dắt |
| Phải deploy/vận hành hệ serving model trên K8s | [ml-serving-k8s.md](./ml-serving-k8s.md) | vận hành & reliability (SRE) |
| Hệ thống có LLM/RAG/agent cần bảo mật | [llmops-end-to-end.md](./llmops-end-to-end.md) (phần guardrails) + [devsecops.md](./devsecops.md) §5 | **AppSec cho ứng dụng LLM** (DevSecOps) |
| Muốn xem lại vì sao từng chọn MLOps | [roadmap-ai-platform-engineer.md](./roadmap-ai-platform-engineer.md) | lịch sử quyết định |

Còn lại ([mlops.md](./mlops.md), [fine-tuning-lora.md](./fine-tuning-lora.md), [vector-db-scale.md](./vector-db-scale.md)):
**không cần đọc theo lộ trình.** Đừng thấy tiếc mà học dở dang — chi phí lớn nhất của việc đổi trục là
tiếp tục chia trí cho trục cũ.

---

## Nguồn tham khảo (đợt cập nhật 09/2026)

- OWASP Top 10:2025 — hạng mục mới A03 Software Supply Chain Failures: [owasp.org/Top10/2025](https://owasp.org/Top10/2025/0x00_2025-Introduction/) · [phân tích của Endor Labs](https://www.endorlabs.com/learn/owasp-top-10-adds-a03-2025-software-supply-chain-failures)
- Tấn công chuỗi cung ứng npm/CI: [Unit 42 — Shai-Hulud](https://unit42.paloaltonetworks.com/npm-supply-chain-attack/) · [CISA alert 09/2025](https://www.cisa.gov/news-events/alerts/2025/09/23/widespread-supply-chain-compromise-impacting-npm-ecosystem)
- EU Cyber Resilience Act — mốc 11/9/2026 & 11/12/2027: [European Commission](https://digital-strategy.ec.europa.eu/en/policies/cyber-resilience-act) · [FOSSA timeline](https://fossa.com/blog/cra-compliance-timeline/)
- Luật Bảo vệ dữ liệu cá nhân 91/2025/QH15 & NĐ 356/2025 (hiệu lực 1/1/2026): [Báo Chính phủ](https://baochinhphu.vn/luat-bao-ve-du-lieu-ca-nhan-chinh-thuc-co-hieu-luc-tu-ngay-mai-1-1-2026-102251231155609721.htm) · [Thư viện pháp luật](https://thuvienphapluat.vn/chinh-sach-phap-luat-moi/vn/ho-tro-phap-luat/chinh-sach-moi/102230/nghi-dinh-13-2023-nd-cp-ve-bao-ve-du-lieu-ca-nhan-het-hieu-luc-tu-01-01-2026)
- Non-human identity / secretless / SPIFFE: [spiffe.io](https://spiffe.io/) · [NHI Mgmt Group](https://nhimg.org/articles/secretless-access-and-passwordless-identity-reduce-nhi-risk/)
- Runtime-first & policy-as-code: [CNCF — Kyverno graduation 24/3/2026](https://www.cncf.io/announcements/2026/03/24/cloud-native-computing-foundation-announces-kyvernos-graduation/) · [so sánh Falco/Tetragon/Tracee](https://1337skills.com/blog/2026-06-24-ebpf-runtime-security-2026-falco-tetragon-tracee/)
- Thị trường & kỹ năng: [2026 Cloud Security Report](https://www.cybersecurity-insiders.com/2026-cloud-security-report-closing-the-cloud-complexity-gap/) · [DevSecOps salary guide 2026](https://www.kore1.com/devsecops-engineer-salary-guide/)
- AI/agent security (ngách tùy chọn): [OWASP GenAI Security Project](https://genai.owasp.org/) · [Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)
- Xu hướng SRE 2026 (OTel, AI-assisted incident response): [Elastic](https://www.elastic.co/blog/2026-observability-trends-generative-ai-opentelemetry) · [Dynatrace](https://www.dynatrace.com/news/blog/sre-best-practices-platform-engineering-trends/)
