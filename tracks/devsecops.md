# DevSecOps — Bảo mật dịch chuyển vào toàn vòng đời

> DevSecOps = "Security as Code" — đưa bảo mật vào MỌI giai đoạn (shift-left) thay vì kiểm tra cuối cùng.
> Cốt lõi: bảo mật là **trách nhiệm của mọi người + được tự động hóa**, không phải cổng gác của 1 team riêng.

## 1. Mindset — shift-left & tự động hóa

```text
Cũ:   Dev xây xong -> Security team audit cuối -> tìm lỗ hổng -> làm lại (chậm, đối đầu).
DevSecOps: bảo mật TỰ ĐỘNG trong mỗi bước pipeline -> phát hiện sớm -> rẻ hơn nhiều để sửa.
           "Shift left" = đẩy kiểm tra bảo mật về càng sớm càng tốt trong vòng đời.
```
Nguyên tắc: **least privilege, defense in depth, zero trust (không tin mặc định, xác thực mọi thứ), secure by default, assume breach (giả định đã bị xâm nhập → thiết kế để giảm thiệt hại).**

## 2. Bảo mật theo từng giai đoạn pipeline (bản đồ DevSecOps)

```text
Code        -> SAST (quét source), secret scanning, pre-commit hook, code review bảo mật
Dependencies-> SCA (quét thư viện: CVE, license), SBOM (danh mục thành phần)
Build       -> ký artifact/image (cosign), build tin cậy (SLSA), quét image (Trivy/Grype)
Registry    -> image signing + admission control (chỉ chạy image đã ký/scan)
Deploy      -> IaC scanning (tfsec/checkov), policy-as-code (OPA/Kyverno), config an toàn
Runtime     -> runtime security (Falco), network policy, secret management, RASP
Monitor     -> phát hiện đe dọa (GuardDuty), SIEM, audit log, incident response
```

## 3. Các trụ cột kỹ năng

| Trụ cột | Nội dung | Công cụ |
|---------|----------|---------|
| **Application security** | SAST/DAST, secure coding, OWASP Top 10 | SonarQube, Semgrep, Snyk, ZAP |
| **Supply chain security** | dependency, SBOM, ký & xác minh, provenance | Trivy, Grype, Syft, cosign, SLSA |
| **Secrets management** | không hardcode, rotation, vault | Vault, Sealed Secrets, SOPS, cloud secret mgr |
| **Policy as Code** | enforce quy tắc tự động | OPA/Gatekeeper, Kyverno, Conftest |
| **Infra & container security** | IaC scan, hardening, image tối thiểu | tfsec, checkov, kube-bench, distroless |
| **Runtime security** | phát hiện hành vi bất thường | Falco, Tetragon, Aqua/Sysdig |
| **Identity & Zero Trust** | IAM least-privilege, mTLS, workload identity | SPIFFE/SPIRE, service mesh, OIDC |
| **Compliance & governance** | CIS/PCI/SOC2/GDPR, audit | Cloud Config, Security Hub, OpenSCAP |
| **Threat modeling** | phân tích bề mặt tấn công trước khi xây | STRIDE, attack trees |

## 4. Kỹ năng: bạn ĐÃ CÓ vs CẦN THÊM
**Đã có (nền DevOps):** CI/CD (nơi nhúng security), K8s (network policy, RBAC, pod security — bạn đã viết), IaC, secrets (Vault/SOPS bạn đã có), image scanning (Trivy đã nhắc).

**Cần bổ sung:**
```text
- Tư duy tấn công (offensive mindset): hiểu kẻ tấn công nghĩ gì -> threat modeling.
- OWASP Top 10 + kiến thức app security nền tảng.
- Supply chain security sâu: SBOM, SLSA, ký & xác minh provenance (đang cực nóng sau các vụ
  tấn công chuỗi cung ứng: SolarWinds, log4j, xz).
- Policy as Code: viết policy OPA/Kyverno enforce tự động.
- Zero Trust architecture: workload identity, mTLS, phân đoạn.
- Compliance frameworks: hiểu SOC2/ISO/PCI/GDPR đủ để thiết kế cho tuân thủ.
- Incident response bảo mật: forensic, containment.
```

## 5. Liên hệ AI/ML (rất mới & nóng) — "MLSecOps / AI Security"
Nếu bạn làm AI, đây là ngách **cực kỳ mới** ít người biết — lợi thế lớn:
```text
- Data security & governance: training data có PII? có bị đầu độc (data poisoning)? lineage/audit?
- Model supply chain: model tải từ HuggingFace có an toàn? (model có thể chứa code độc qua pickle) ->
  quét model, dùng safetensors, verify nguồn.
- Model theft / extraction: bảo vệ model khỏi bị trích xuất qua API.
- LLM security (OWASP Top 10 for LLM):
    * Prompt injection (kẻ xấu chèn lệnh vào input)
    * Insecure output handling (output LLM chạy như code -> nguy hiểm)
    * Data leakage (LLM lộ dữ liệu training/context)
    * Excessive agency (LLM agent có quá nhiều quyền hành động)
- Guardrails: lọc input/output, chống jailbreak, PII redaction.
- AI governance & compliance: EU AI Act, model card, audit quyết định của AI.
```
→ **MLSecOps = MLOps + Security**, thị trường gần như trống. Bạn đang ở vị trí hiếm để chiếm.

## 6. Lộ trình học
```text
1. Nền security: OWASP Top 10, threat modeling (STRIDE), least privilege/zero trust.
2. Nhúng vào pipeline bạn đã có: SAST (Semgrep) + SCA (Trivy) + secret scan + IaC scan (checkov)
   vào CI -> fail build khi có lỗ hổng nghiêm trọng.
3. Supply chain: tạo SBOM (Syft), ký image (cosign), admission control chỉ cho image tin cậy.
4. Policy as Code: viết Kyverno/OPA policy cho cluster (đã có nền k8s security).
5. Runtime: Falco phát hiện hành vi bất thường; network policy default-deny.
6. (AI) MLSecOps: model scanning, LLM guardrails, data governance, OWASP LLM Top 10.
7. Compliance: map hệ thống với 1 framework (SOC2/CIS) + tự động thu thập bằng chứng.
```

## 7. Thị trường & đòn bẩy
- Cầu bảo mật tăng mạnh (quy định chặt hơn, tấn công nhiều hơn). Người biết **cả DevOps lẫn security** rất hiếm.
- Chức danh: DevSecOps Engineer, Security Engineer, Cloud Security, Platform Security, **AI/ML Security**.
- Đòn bẩy: security là mối quan tâm cấp C-level (rủi ro pháp lý/uy tín) → tiếng nói có trọng lượng.
- Con đường Architect: **Security Architect** / Cloud Security Architect.

## 8. Giao thoa với MLOps & SRE
- **↔ MLOps:** MLSecOps (data/model/LLM security, governance) — nhúng security vào ML pipeline. Xem [mlops.md](./mlops.md).
- **↔ SRE:** "resilience" gồm cả chịu tấn công; security incident dùng chung quy trình incident/postmortem của SRE. Assume-breach ≈ assume-failure.

## Tóm tắt
- DevSecOps = shift-left + tự động hóa bảo mật trong mọi bước pipeline (Security as Code).
- Nền DevOps của bạn đã chứa nhiều mảnh (CI/CD, k8s security, secrets, scanning); cần thêm offensive mindset, supply chain, policy-as-code, zero trust, compliance.
- Với AI: **MLSecOps / AI Security** (prompt injection, model supply chain, data governance) là ngách trống & giá trị cao.
