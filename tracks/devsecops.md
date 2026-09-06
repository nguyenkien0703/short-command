# DevSecOps — Bảo mật dịch chuyển vào toàn vòng đời (🎯 trục chính)

> DevSecOps = "Security as Code" — đưa bảo mật vào MỌI giai đoạn (shift-left) thay vì kiểm tra cuối cùng.
> Cốt lõi: bảo mật là **trách nhiệm của mọi người + được tự động hóa**, không phải cổng gác của 1 team riêng.
>
> *Cập nhật 09/2026 — đây là trục chính trong [tracks/README.md](./README.md). Trục bổ trợ: [sre.md](./sre.md).*

## 1. Mindset — shift-left, và "paved road" thay vì cổng chặn

```text
Cũ:   Dev xây xong -> Security team audit cuối -> tìm lỗ hổng -> làm lại (chậm, đối đầu).
DevSecOps: bảo mật TỰ ĐỘNG trong mỗi bước pipeline -> phát hiện sớm -> rẻ hơn nhiều để sửa.
           "Shift left" = đẩy kiểm tra bảo mật về càng sớm càng tốt trong vòng đời.
```
Nguyên tắc: **least privilege, defense in depth, zero trust (không tin mặc định, xác thực mọi thứ), secure by default, assume breach (giả định đã bị xâm nhập → thiết kế để giảm thiệt hại).**

**Bài học vận hành quan trọng nhất (đọc kỹ):** shift-left thất bại khi nó chỉ là *thêm gate*. Dev sẽ
tìm cách đi vòng nếu gate ồn và chậm. Cách làm đúng là **paved road** — con đường mặc định đã an toàn
sẵn (base image đã hardening, template pipeline đã có scan & ký, module Terraform đã đúng chuẩn), đi
theo đường đó thì *không phải nghĩ về bảo mật*; đi ra ngoài mới cần xin ngoại lệ có thời hạn.

```text
Gate  = "cấm anh làm sai"     -> bị né, tạo đối đầu, sinh shadow pipeline.
Paved road = "làm đúng dễ hơn làm sai" -> được chọn tự nguyện, mở rộng được.
Thực tế: cần CẢ HAI, nhưng paved road đi trước, gate chỉ chặn thứ NGHIÊM TRỌNG & MỚI.
```

## 2. Bảo mật theo từng giai đoạn pipeline (bản đồ DevSecOps)

```text
Code        -> SAST (quét source), secret scanning, pre-commit hook, code review bảo mật
Dependencies-> SCA (quét thư viện: CVE, license), SBOM (danh mục thành phần)
Build       -> ký artifact/image (cosign), provenance/attestation (SLSA), quét image (Trivy/Grype)
CI PLATFORM -> chính runner & token là mục tiêu: pin action theo SHA, quyền tối thiểu, OIDC, cách ly job
Registry    -> image signing + admission control (chỉ chạy image đã ký/scan), quét lại khi có CVE mới
Deploy      -> IaC scanning (tfsec/checkov), policy-as-code (Kyverno/OPA/VAP), config an toàn
Runtime     -> runtime security (Falco/Tetragon), network policy, secret mgmt, workload identity
Monitor     -> phát hiện đe dọa, SIEM, audit log, incident response, thu thập bằng chứng tuân thủ
```
> Dòng **CI PLATFORM** là phần được bổ sung sau các vụ 2025–2026: kẻ tấn công không còn chỉ nhắm vào
> code của bạn, mà nhắm vào **nơi build code** — chỗ tập trung nhiều bí mật nhất công ty.

## 3. Các trụ cột kỹ năng

| Trụ cột | Nội dung | Công cụ (2026) |
|---------|----------|----------------|
| **Application security** | SAST/DAST, secure coding, OWASP Top 10:2025, ASVS | Semgrep, SonarQube, CodeQL, ZAP |
| **Supply chain security** | dependency, SBOM, ký & xác minh, provenance | Trivy, Grype, Syft (CycloneDX/SPDX), cosign/Sigstore, SLSA, in-toto |
| **CI/CD platform security** | bảo vệ chính pipeline & runner | pin SHA, OIDC, `permissions:` tối thiểu, harden-runner, tách runner |
| **Secrets management** | không hardcode, rotation, tiến tới secretless | Vault, External Secrets, SOPS, cloud secret manager, gitleaks/TruffleHog |
| **Policy as Code** | enforce quy tắc tự động | **Kyverno** (CNCF graduated 3/2026), OPA/Gatekeeper, **ValidatingAdmissionPolicy (CEL, GA từ K8s 1.30)**, Conftest |
| **Infra & container security** | IaC scan, hardening, image tối thiểu | checkov, tfsec/trivy-config, kube-bench, distroless/minimal base |
| **Runtime security** | phát hiện & chặn hành vi bất thường | Falco (graduated), Tetragon, Tracee, KubeArmor |
| **Identity & Zero Trust** | IAM least-privilege, mTLS, workload identity | SPIFFE/SPIRE, service mesh, OIDC federation, cloud workload identity |
| **Cloud security posture** | misconfig, quyền dư thừa, drift | Prowler, ScoutSuite, CSPM/CNAPP của cloud |
| **Compliance & governance** | CIS/SOC2/ISO 27001/PCI, **EU CRA**, **Luật BVDLCN VN** | CIS Benchmarks, NIST SSDF, OpenSCAP, evidence automation |
| **Threat modeling** | phân tích bề mặt tấn công trước khi xây | STRIDE, attack tree, data flow diagram |

## 4. Kỹ năng: bạn ĐÃ CÓ vs CẦN THÊM

**Đã có (nền DevOps — đây là lợi thế lớn, đừng xem nhẹ):** CI/CD (nơi nhúng security), K8s (network
policy, RBAC, pod security — bạn đã viết trong playbook), IaC, secrets (Vault/SOPS), image scanning
(Trivy). Người làm security thuần **không có** nền này; đó là lý do bạn vào mảng này với vị thế tốt.

**Cần bổ sung:**
```text
- Tư duy tấn công (offensive mindset): hiểu kẻ tấn công nghĩ gì -> threat modeling có chất lượng.
- OWASP Top 10:2025 + ASVS: vốn app security nền tảng (mảng yếu nhất của người từ infra).
- Supply chain sâu: SBOM, SLSA, ký & xác minh provenance, verify tại admission.
- Bảo mật chính CI/CD: token, OIDC, cách ly runner, script injection trong workflow.
- Policy as Code: viết Kyverno/OPA/CEL policy, triển khai từ audit -> enforce mà không gãy sản xuất.
- Zero Trust & workload identity: mTLS, SPIFFE, bỏ static credential.
- Compliance frameworks: đủ để THIẾT KẾ cho tuân thủ & tự động thu thập bằng chứng.
- Incident response bảo mật: containment, forensic cơ bản, timeline, xử lý lộ secret.
```

## 5. Bối cảnh & ưu tiên kỹ thuật 2026

### 5.1 Supply chain là mặt trận số một
OWASP Top 10:**2025** thêm hạng mục mới **A03 — Software Supply Chain Failures** (tỷ lệ gặp cao nhất
nhưng gần như không có CVE để quét → scanner không cứu được bạn), và **Security Misconfiguration lên #2**.

Hai vụ nên đọc kỹ vì nó thay đổi cách làm CI:
```text
tj-actions/changed-files (3/2025): action bị chèn mã đọc BỘ NHỚ tiến trình Runner.Worker
                                   -> moi OIDC token ngay trong lúc workflow chạy.
Shai-Hulud (npm, 9/2025; V2 11/2025, dư chấn 2026): worm TỰ NHÂN BẢN — nhặt được npm token trong
                                   môi trường CI thì tự publish phiên bản độc cho mọi package nó với tới.
=> Bài học: (1) CI runner là PRODUCTION; (2) token phải ngắn hạn & phạm vi hẹp;
   (3) một dependency bị chiếm là đủ để lan ra toàn tổ chức qua chính pipeline.
```

**Việc cần làm được (checklist CI hardening):**
```text
[ ] Pin action/image theo commit SHA hoặc digest (kèm comment version), không dùng tag di động.
[ ] permissions: mặc định read-only ở cấp workflow, nâng quyền từng job khi thật cần.
[ ] Bỏ secret dài hạn -> OIDC federation sang cloud role (token ngắn hạn, có điều kiện).
[ ] Không nội suy dữ liệu không tin cậy (tiêu đề PR, tên branch...) vào lệnh shell -> script injection.
[ ] Job xử lý code chưa review (PR từ fork) KHÔNG chạm secret production; tách runner.
[ ] Chặn/ giám sát post-install script khi cài dependency; dùng lockfile & cài ở chế độ CI.
[ ] Egress control cho runner (chặn exfiltration) + audit ai đổi được workflow.
```

### 5.2 Artifact phải chứng minh được nguồn gốc (SBOM → ký → verify)
```text
SBOM (Syft -> CycloneDX/SPDX)  : biết trong artifact có gì; quét LẠI khi CVE mới xuất hiện.
Ký (cosign/Sigstore, keyless)  : chứng minh ai tạo ra nó — KÝ THEO DIGEST, không ký theo tag.
Provenance (SLSA, in-toto)     : chứng minh nó được build ra sao, ở đâu.
   L1 có provenance · L2 provenance được nền tảng build ký · L3 build cách ly, khoá ký ngoài tầm với
   của bước build do người dùng định nghĩa  -> mục tiêu thực tế của một pipeline trưởng thành: L3.
Verify tại admission (Kyverno verifyImages) : phần NHIỀU NGƯỜI BỎ QUÊN — ký mà không verify thì
   chữ ký chỉ là trang trí.
```

### 5.3 Kubernetes: policy-as-code đã là hạ tầng chuẩn
- **Kyverno graduated (CNCF, 3/2026)** — policy-as-code hết thời "thử nghiệm".
- **ValidatingAdmissionPolicy** (CEL, GA từ K8s 1.30): chạy **trong API server**, không cần webhook →
  bớt một điểm chết & bớt gánh TLS/latency. Dùng cho luật đơn giản, dành Kyverno/OPA cho luật phức tạp
  (verify chữ ký, mutate, generate).
- **Pod Security Standards**: triển khai đúng cách là `baseline` ở chế độ **audit/warn** toàn cụm →
  sửa dần → `enforce`; `restricted` cho namespace production.
- Kèm theo: NetworkPolicy **default-deny** rồi mở dần, RBAC dọn theo quyền *thực dùng*.

### 5.4 Runtime-first (eBPF)
Thị trường CNAPP đang hội tụ về **runtime-first**: quét tĩnh cho ra hàng nghìn CVE, còn runtime trả lời
được câu hỏi đắt giá hơn — *cái nào đang thực sự chạy và bị chạm tới*.
```text
Falco    : rule engine rộng nhất, CNCF graduated — thiên PHÁT HIỆN.
Tetragon : eBPF của Cilium — phát hiện + có khả năng CHẶN (enforcement) ở kernel.
Tracee   : Aqua, eBPF, mạnh về forensic/behavioral signature.
=> Chọn 1 cái, viết được rule của riêng mình (shell trong container, ghi file lạ, kết nối lạ),
   và nối vào đúng kênh alert của on-call — cài xong để đó thì vô nghĩa.
```

### 5.5 Danh tính máy (non-human identity) là chu vi mới
Khoảng **2/3 tổ chức vẫn sống bằng static credential**. Hướng đi: **secretless / zero standing privilege**
— mTLS + **SPIFFE/SPIRE** hoặc workload identity của cloud, token ngắn hạn thay API key vĩnh viễn.
Bài toán thực tế của bạn: *bao nhiêu secret trong cụm có thể xoá hẳn nếu chuyển sang workload identity?*

### 5.6 Compliance đã tới hạn — và có phần riêng cho Việt Nam
```text
EU CRA : 11/9/2026 bắt buộc báo cáo lỗ hổng đang bị khai thác (cảnh báo sớm 24h) tới ENISA/CSIRT;
         11/12/2027 áp dụng đầy đủ (CE marking). Ảnh hưởng mọi sản phẩm số bán vào EU.
Việt Nam: Luật Bảo vệ dữ liệu cá nhân 91/2025/QH15 + NĐ 356/2025/NĐ-CP, hiệu lực 1/1/2026,
         thay NĐ 13/2023 — quyền chủ thể dữ liệu, đánh giá tác động, hồ sơ xử lý dữ liệu.
Chuẩn kỹ thuật để bám: NIST SSDF (SP 800-218) cho quy trình phát triển, CIS Benchmarks cho cấu hình.
```
**Kỹ năng đáng giá nhất ở mục này không phải "thuộc luật", mà là *evidence automation*:** mỗi control →
một kiểm tra tự động → báo cáo sinh ra liên tục, thay vì cả team cắm mặt gom screenshot trước kỳ audit.

### 5.7 (Tùy chọn) AI/LLM security — ngách, không phải trục
Nếu hệ thống bạn bảo vệ có LLM/agent, đây là **AppSec cho một loại ứng dụng mới** — bạn *không cần* làm
MLOps để chạm nó:
```text
- OWASP Top 10 for LLM Apps (2025): prompt injection, insecure output handling, system prompt leakage,
  excessive agency (thừa chức năng / thừa quyền / thừa tự chủ), unbounded consumption, vector weakness.
- OWASP Top 10 for Agentic Applications (2026): goal hijacking, tool misuse, memory/context poisoning.
- MCP server (2026): endpoint không xác thực, tool poisoning (chỉ dẫn độc giấu trong mô tả tool),
  token lộ qua transport không mã hoá -> coi mỗi MCP server như một dịch vụ production có auth & log.
- Model supply chain: model tải về có thể chứa mã độc qua pickle -> ưu tiên safetensors, quét & verify nguồn.
- Data: training/context data có PII không? lineage? (nối thẳng với Luật BVDLCN ở §5.6)
```
Xem thêm phần guardrails trong [llmops-end-to-end.md](./llmops-end-to-end.md) khi cần chi tiết kỹ thuật.

## 6. Lộ trình học (bản chi tiết của lộ trình 12 tháng trong [README](./README.md))

```text
0. Kiểm kê: vẽ data flow + threat model STRIDE cho 1 hệ thống THẬT -> danh sách rủi ro xếp hạng.
1. Nền security: OWASP Top 10:2025, ASVS, least privilege/zero trust/assume breach.
2. Shift-left vào pipeline đã có: Semgrep (SAST) + Trivy (SCA) + gitleaks (secret) + checkov (IaC),
   baseline lỗi cũ, chỉ FAIL với lỗi MỚI & nghiêm trọng.
3. Hardening chính CI/CD theo checklist §5.1 (đây là phần "rẻ mà giá trị cao" nhất hiện nay).
4. Supply chain: SBOM (Syft) -> ký (cosign) -> provenance SLSA -> VERIFY tại admission (Kyverno).
5. Policy as Code: PSS baseline audit -> enforce; NetworkPolicy default-deny; RBAC least-privilege;
   luật đơn giản dùng ValidatingAdmissionPolicy (CEL), luật phức tạp dùng Kyverno.
6. Runtime: Falco/Tetragon với rule tự viết -> alert vào kênh on-call; diễn tập 1 kịch bản thật.
7. Identity: bỏ long-lived secret (OIDC/workload identity), tiến tới SPIFFE/mTLS nếu hệ đủ lớn.
8. Cloud posture: Prowler/CSPM, phân tích quyền thực dùng để cắt IAM dư thừa.
9. Compliance: map 1 framework (CIS -> SOC2/ISO nếu cần) + Luật BVDLCN; tự động hoá bằng chứng.
10. Offensive nhẹ, đều đặn: lab/CTF hằng tuần — đọc được lỗ hổng thì phòng thủ mới có chiều sâu.
```
**Cách học hiệu quả nhất ở mảng này:** mỗi mục trên → làm trên hệ thống thật đang vận hành → viết 1 ADR
([templates/](../templates/)). Bộ ADR đó chính là hồ sơ năng lực khi phỏng vấn vị trí security.

## 7. Bẫy thường gặp (đọc trước khi triển khai, đỡ mất uy tín)

| Bẫy | Vì sao chết | Cách tránh |
|-----|-------------|------------|
| Bật full scanner ngay ngày đầu | Hàng nghìn finding cũ, build đỏ, dev tắt gate | Baseline nợ cũ, chỉ chặn lỗi **mới** + severity cao |
| Quét mà không có người sửa | Báo cáo chất đống, rủi ro không giảm | Mỗi finding phải có **owner + SLA sửa**; đo *thời gian vá*, không đo *số lỗi tìm được* |
| Ký image nhưng không verify | Chữ ký thành trang trí | Bật admission verify, chứng minh bằng demo deploy image không ký bị từ chối |
| Policy để audit-mode mãi mãi | Không có tác dụng thật | Đặt hạn chuyển sang enforce ngay từ đầu, có quy trình ngoại lệ **có thời hạn** |
| Alert bảo mật ồn | Đúng bệnh alert fatigue của SRE | Ưu tiên theo *khả năng khai thác thực tế* (đang chạy? có exposed? có exploit?), không theo CVSS thuần |
| "Compliance theater" | Có giấy tờ, không có an toàn | Control phải là **kiểm tra tự động**, bằng chứng sinh liên tục |
| Bỏ quên chính CI/CD | Nơi nhiều secret nhất lại ít được bảo vệ nhất | §5.1 |
| Security thành người gác cổng | Bị né, sinh shadow IT | Paved road (§1), làm đúng phải là đường dễ đi nhất |

## 8. Thị trường & đòn bẩy
- **Cloud security là kỹ năng được tuyển nhiều nhất** và là khoảng trống lớn thứ hai (sau AI); vị trí
  cloud security mất **65+ ngày** mới tuyển được, khoảng trống nhân lực an ninh mạng toàn cầu ~**4,8 triệu**.
- Người **vừa vận hành được vừa bảo mật được** (K8s/Terraform/CI-CD + security) được trả cao hơn analyst
  bảo mật thuần khoảng **20–40%**.
- Chức danh: DevSecOps Engineer, Cloud/Platform Security Engineer, Security Engineer, Product Security.
- Đòn bẩy: security là mối quan tâm cấp C-level (rủi ro pháp lý & uy tín) → tiếng nói có trọng lượng,
  và từ 2026 có thêm sức ép **quy định** (CRA, Luật BVDLCN) — người giải được bài toán này rất hiếm.
- Đường Architect: **Security Architect / Cloud Security Architect**.

## 9. Giao thoa với SRE (trục bổ trợ — xem [sre.md](./sre.md))

```text
Incident        : sự cố bảo mật là MỘT LOẠI sự cố production -> dùng chung severity, on-call,
                  incident command, blameless postmortem. Học một quy trình, dùng cho cả hai.
Triết lý        : assume breach  ≈  assume failure.
Định lượng      : error budget (SRE) ~ risk budget (Sec) — chấp nhận rủi ro có kiểm soát,
                  thay vì đòi "an toàn tuyệt đối" (bất khả thi, và làm chậm sản phẩm).
Alert           : burn-rate alerting của SRE là cách chữa alert fatigue cho cả alert bảo mật.
Diễn tập        : game day (chaos) ~ purple team — đo MTTD/MTTR cho kịch bản tấn công.
Chiều ngược lại : chính security control của bạn phải ĐÁNG TIN — admission webhook chết là cả cụm
                  không deploy được; scanner treo là pipeline đứng. Security thêm điểm chết mới,
                  nên phải thiết kế theo tư duy SRE (timeout, failurePolicy, fail-open vs fail-closed).
```

## Tóm tắt
- DevSecOps = shift-left + tự động hoá bảo mật trong mọi bước pipeline, và **paved road** quan trọng hơn gate.
- Nền DevOps của bạn đã chứa nhiều mảnh (CI/CD, K8s security, secrets, scanning); cần thêm offensive
  mindset, app security, supply chain (SBOM/ký/SLSA + **verify**), policy-as-code, workload identity, compliance.
- Ưu tiên 2026: **bảo vệ chính CI/CD**, chứng minh nguồn gốc artifact, policy-as-code đã chín (Kyverno
  graduated, VAP/CEL), runtime-first bằng eBPF, secretless identity, và tự động hoá bằng chứng tuân thủ
  (EU CRA, Luật BVDLCN 1/1/2026).
- AI/LLM security là **ngách tùy chọn** — vào với vai AppSec, không cần làm MLOps.
