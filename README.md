# short-command — Sổ tay DevOps → AI Platform Engineer

> Bộ tài liệu cá nhân: từ câu lệnh tra nhanh hằng ngày tới vận hành K8s, kiến trúc cloud,
> AI/ML Platform Engineering, và lộ trình sự nghiệp. Viết bằng tiếng Việt để tra cứu & học nhanh.
>
> Triết lý: **học gắn việc thật · viết lại điều đã học · xây body-of-work công khai.**

---

## 🚀 Bắt đầu từ đâu?

| Bạn muốn... | Đọc |
|-------------|-----|
| Tra nhanh 1 câu lệnh (docker/k8s/git/aws/network...) | [cheatsheet.md](./cheatsheet.md) |
| Dùng thành thạo Claude Code (command/hook/MCP/subagent) | [claude-code-guide.md](./claude-code-guide.md) |
| Vận hành / xử lý sự cố cluster K8s | [k8s-operations-playbook.md](./k8s-operations-playbook.md) |
| **Ôn thi chứng chỉ Kubernetes (CKA/CKS)** | [certs/k8s/](./certs/k8s/README.md) |
| Học AWS để thi cert (SAA/SAP/SysOps/DevOps) | [cloud/aws/](./cloud/aws/README.md) |
| Định hình hướng chuyên môn hoá (DevSecOps → SRE) | [tracks/](./tracks/README.md) |
| Phát triển sự nghiệp lên Architect/Lead | [career-roadmap.md](./career-roadmap.md) |
| Đi **Senior → Lead DevSecOps** trong ngành **tài chính/ngân hàng** | [career-devsecops-finance.md](./career-devsecops-finance.md) |
| Luyện viết ADR / design doc (trình sếp) | [templates/](./templates/README.md) |

---

## 📂 Toàn bộ tài liệu theo nhóm

### ⚡ Tra cứu nhanh
- **[cheatsheet.md](./cheatsheet.md)** — 31 nhóm lệnh: Docker, Compose, Helm, kubectl, grep, troubleshooting Linux, Git, Database, SSH, HTTP/SSL, jq/yq, IaC, cloud CLI, monitoring, message queue, nginx, firewall, profiling, ArgoCD, CI/CD, secrets, systemd, container runtime, service mesh, k8s TUI...
- **[docker.txt](./docker.txt)** — bộ alias shell gốc.
- **[claude-code-guide.md](./claude-code-guide.md)** — toàn bộ slash command, CLI flags, custom command, CLAUDE.md, subagent, skill, hook, MCP, permission mode, git worktree, headless/CI, quản lý chi phí, phím tắt, IDE integration + tips&tricks dùng Claude Code hiệu quả trong công việc.

### ☸️ Kubernetes — vận hành & production
- **[k8s-operations-playbook.md](./k8s-operations-playbook.md)** — playbook sự cố (Pod/Node/Network/Storage/Control-plane/Scaling) theo cấu trúc *triệu chứng → chẩn đoán → xử lý → root cause → phòng ngừa*; backup/DR; reliability; observability; security; lộ trình DevOps Lead.
- **[k8s-production-templates.md](./k8s-production-templates.md)** — YAML production-ready (Deployment/PDB/HPA/NetworkPolicy/Quota/RBAC) + runbook từng bước (etcd/Velero restore, bảo trì node, rollback).

### 🎫 Chứng chỉ Kubernetes — CKA & CKS
- **[certs/k8s/](./certs/k8s/README.md)** — bộ tài liệu ôn thi bám sát curriculum chính chủ CNCF
  (`CKA v1.35` · `CKS v1.34`), cho người **đã làm K8s thật**:
  - [exam-day-playbook](./certs/k8s/exam-day-playbook.md) — chiến thuật phòng thi: check-in, alias/vim setup 60s, quản lý thời gian, copy-paste, 10 sai lầm mất điểm oan
  - [study-plan](./certs/k8s/study-plan.md) — lộ trình 12 tuần CKA → 8 tuần CKS, mốc kiểm tra theo tuần
  - [lab-2-cum-rke2-calico](./certs/k8s/lab-2-cum-rke2-calico.md) — **đã có cụm thật?** bản đồ domain↔cụm, bảng dịch RKE2 ↔ kubeadm, lộ trình tuần gắn sẵn tên cụm
  - [resources](./certs/k8s/resources.md) — nguồn học VN (devops.vn, Viblo, devopsviet) + quốc tế, mẹo săn voucher
  - **CKA**: [00-exam-guide](./certs/k8s/cka/00-exam-guide.md) · [cluster-architecture 25%](./certs/k8s/cka/01-cluster-architecture.md) · [workloads&scheduling 15%](./certs/k8s/cka/02-workloads-scheduling.md) · [services&networking 20%](./certs/k8s/cka/03-services-networking.md) · [storage 10%](./certs/k8s/cka/04-storage.md) · [troubleshooting 30%](./certs/k8s/cka/05-troubleshooting.md) · [30 bài luyện](./certs/k8s/cka/practice-questions.md)
  - **CKS**: [00-exam-guide](./certs/k8s/cks/00-exam-guide.md) · [cluster-setup](./certs/k8s/cks/01-cluster-setup.md) · [cluster-hardening](./certs/k8s/cks/02-cluster-hardening.md) · [system-hardening](./certs/k8s/cks/03-system-hardening.md) · [microservice-vuln](./certs/k8s/cks/04-minimize-microservice-vulnerabilities.md) · [supply-chain](./certs/k8s/cks/05-supply-chain-security.md) · [runtime-security](./certs/k8s/cks/06-monitoring-logging-runtime.md) · [25 bài luyện](./certs/k8s/cks/practice-questions.md)

### ☁️ Cloud — AWS (ôn cert)
- **[cloud/aws/](./cloud/aws/README.md)** — giáo trình AWS đầy đủ:
  - [00-fundamentals](./cloud/aws/00-fundamentals.md) · [01-networking-vpc](./cloud/aws/01-networking-vpc.md) (CIDR/VPC/subnet sâu) · [02-compute](./cloud/aws/02-compute.md) · [03-storage](./cloud/aws/03-storage.md) · [04-databases](./cloud/aws/04-databases.md) · [05-security-iam](./cloud/aws/05-security-iam.md) · [06-management-devops](./cloud/aws/06-management-devops.md) · [07-architecture-patterns](./cloud/aws/07-architecture-patterns.md)
  - [labs/cidr-subnetting-deep-dive](./cloud/aws/labs/cidr-subnetting-deep-dive.md) — lab tính CIDR có lời giải
  - [practice-questions](./cloud/aws/practice-questions.md) — câu hỏi kiểu đề thật
  - [cert-roadmap](./cloud/aws/cert-roadmap.md) — chiến lược từng cert

### 🛡️ Chuyên môn hoá (DevSecOps · SRE)
- **[tracks/](./tracks/README.md)** — 🎯 big picture + **lộ trình 12 tháng** (bắt đầu ở đây): trục chính **DevSecOps**, trục hai **SRE**, bối cảnh 2026 (supply chain, SLSA/SBOM, EU CRA, Luật BVDLCN VN, runtime-first).
  - Hướng nghề: [devsecops](./tracks/devsecops.md) · [sre](./tracks/sre.md)
  - Tham chiếu (không nằm trong lộ trình): [roadmap-ai-platform-engineer](./tracks/roadmap-ai-platform-engineer.md) · [mlops](./tracks/mlops.md) · [ml-serving-k8s](./tracks/ml-serving-k8s.md) · [llmops-end-to-end](./tracks/llmops-end-to-end.md) · [fine-tuning-lora](./tracks/fine-tuning-lora.md) · [vector-db-scale](./tracks/vector-db-scale.md) · [ml-concepts-explained](./tracks/ml-concepts-explained.md)

### 📈 Sự nghiệp & kỹ năng
- **[career-roadmap.md](./career-roadmap.md)** — Middle → Senior → Staff/Architect (IC track) → Leadership: skill matrix, checklist lên cấp, kế hoạch 12 tháng, ra quyết định kiến trúc.
- **[career-devsecops-finance.md](./career-devsecops-finance.md)** — 🔐 lộ trình chi tiết **Senior → Lead DevSecOps trong ngành tài chính**: bản đồ hệ thống ngân hàng, luồng tiền 7 chặng & rủi ro từng chặng, 20 khái niệm nghiệp vụ bắt buộc, bản đồ tuân thủ (TT 09/2020, TT 50/2024, QĐ 2345, Luật BVDLCN 2025, PCI DSS 4.0.1, SWIFT CSCF v2026, DORA), cách dịch điều khoản → control → bằng chứng tự động, kế hoạch theo quý, portfolio 12 hiện vật, câu hỏi phỏng vấn.
- **[devops-fast-track.md](./devops-fast-track.md)** — kế hoạch 90 ngày lên tay nhanh: mental models + 11 break-and-fix lab + capstone.

### 📝 Templates (luyện viết & trình sếp)
- **[templates/](./templates/README.md)** — [adr-template](./templates/adr-template.md) · [design-doc-template](./templates/design-doc-template.md) · [threat-model-template](./templates/threat-model-template.md) (STRIDE + ví dụ đã điền cho API chuyển tiền) · [risk-acceptance-template](./templates/risk-acceptance-template.md) (hồ sơ ngoại lệ có hạn) · [ví dụ ADR đã điền](./templates/examples/ADR-0001-transit-gateway-vs-peering.md).

---

## 🗺️ Lộ trình học đề xuất (theo mục tiêu)

```text
Nền DevOps vững:      cheatsheet -> k8s-operations-playbook -> k8s-production-templates
                      -> devops-fast-track (lab tự phá & sửa)

Chứng chỉ K8s:        certs/k8s/exam-day-playbook -> certs/k8s/study-plan
                      -> cka/00 -> 01..05 -> cka/practice-questions   (thi CKA)
                      -> cks/00 -> 01..06 -> cks/practice-questions   (thi CKS)

Kiến trúc Cloud/Cert: cloud/aws (00 -> 07) -> labs/cidr -> practice-questions -> cert-roadmap

Chuyên môn hoá:       tracks/README (lộ trình 12 tháng) -> devsecops -> sre
                      (pipeline security -> supply chain/SBOM+ký -> policy & runtime
                       -> cloud security & compliance -> SLO/incident)

Phát triển sự nghiệp: career-roadmap  (+ dùng templates/ viết ADR mỗi quyết định thật)
```

---

## 💡 Cách dùng repo này hiệu quả

1. **Tra khi cần** — dùng cheatsheet & playbook như tài liệu tham chiếu lúc làm việc.
2. **Học theo lộ trình** — chọn 1 mục tiêu, đi theo path ở trên, mỗi mục có phần "tóm tắt điểm bẫy".
3. **Viết lại** — mỗi quyết định kỹ thuật thật → 1 ADR (dùng template). Đây vừa là học, vừa là bằng chứng năng lực.
4. **Ghi chú của bạn** — gặp bẫy/bài học mới → thêm vào file tương ứng. Repo lớn lên cùng bạn.

> Bản thân repo này chính là một **body-of-work công khai** — vừa để học, vừa là visibility &
> minh chứng năng lực trên con đường lên Architect. Cứ tiếp tục bồi đắp.
