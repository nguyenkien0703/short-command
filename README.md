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
| Vận hành / xử lý sự cố cluster K8s | [k8s-operations-playbook.md](./k8s-operations-playbook.md) |
| Học AWS để thi cert (SAA/SAP/SysOps/DevOps) | [cloud/aws/](./cloud/aws/README.md) |
| Định hình hướng đi AI (MLOps/SRE/DevSecOps) | [tracks/](./tracks/README.md) |
| Phát triển sự nghiệp lên Architect/Lead | [career-roadmap.md](./career-roadmap.md) |
| Luyện viết ADR / design doc (trình sếp) | [templates/](./templates/README.md) |

---

## 📂 Toàn bộ tài liệu theo nhóm

### ⚡ Tra cứu nhanh
- **[cheatsheet.md](./cheatsheet.md)** — 31 nhóm lệnh: Docker, Compose, Helm, kubectl, grep, troubleshooting Linux, Git, Database, SSH, HTTP/SSL, jq/yq, IaC, cloud CLI, monitoring, message queue, nginx, firewall, profiling, ArgoCD, CI/CD, secrets, systemd, container runtime, service mesh, k8s TUI...
- **[docker.txt](./docker.txt)** — bộ alias shell gốc.

### ☸️ Kubernetes — vận hành & production
- **[k8s-operations-playbook.md](./k8s-operations-playbook.md)** — playbook sự cố (Pod/Node/Network/Storage/Control-plane/Scaling) theo cấu trúc *triệu chứng → chẩn đoán → xử lý → root cause → phòng ngừa*; backup/DR; reliability; observability; security; lộ trình DevOps Lead.
- **[k8s-production-templates.md](./k8s-production-templates.md)** — YAML production-ready (Deployment/PDB/HPA/NetworkPolicy/Quota/RBAC) + runbook từng bước (etcd/Velero restore, bảo trì node, rollback).

### ☁️ Cloud — AWS (ôn cert)
- **[cloud/aws/](./cloud/aws/README.md)** — giáo trình AWS đầy đủ:
  - [00-fundamentals](./cloud/aws/00-fundamentals.md) · [01-networking-vpc](./cloud/aws/01-networking-vpc.md) (CIDR/VPC/subnet sâu) · [02-compute](./cloud/aws/02-compute.md) · [03-storage](./cloud/aws/03-storage.md) · [04-databases](./cloud/aws/04-databases.md) · [05-security-iam](./cloud/aws/05-security-iam.md) · [06-management-devops](./cloud/aws/06-management-devops.md) · [07-architecture-patterns](./cloud/aws/07-architecture-patterns.md)
  - [labs/cidr-subnetting-deep-dive](./cloud/aws/labs/cidr-subnetting-deep-dive.md) — lab tính CIDR có lời giải
  - [practice-questions](./cloud/aws/practice-questions.md) — câu hỏi kiểu đề thật
  - [cert-roadmap](./cloud/aws/cert-roadmap.md) — chiến lược từng cert

### 🤖 AI Platform Engineering (MLOps · SRE · DevSecOps)
- **[tracks/](./tracks/README.md)** — định hình & đào sâu hướng AI:
  - [roadmap-ai-platform-engineer](./tracks/roadmap-ai-platform-engineer.md) — 🎯 lộ trình phân lớp + LLMOps vs MLOps *(bắt đầu ở đây)*
  - Hướng nghề: [mlops](./tracks/mlops.md) · [sre](./tracks/sre.md) · [devsecops](./tracks/devsecops.md)
  - Kỹ thuật sâu: [ml-serving-k8s](./tracks/ml-serving-k8s.md) (KServe/GPU) · [llmops-end-to-end](./tracks/llmops-end-to-end.md) (RAG+vLLM+eval+guardrails) · [fine-tuning-lora](./tracks/fine-tuning-lora.md) · [vector-db-scale](./tracks/vector-db-scale.md)
  - Nền tảng: [ml-concepts-explained](./tracks/ml-concepts-explained.md) — phân biệt train/inference/fine-tune/RAG...

### 📈 Sự nghiệp & kỹ năng
- **[career-roadmap.md](./career-roadmap.md)** — Middle → Senior → Staff/Architect (IC track) → Leadership: skill matrix, checklist lên cấp, kế hoạch 12 tháng, ra quyết định kiến trúc.
- **[devops-fast-track.md](./devops-fast-track.md)** — kế hoạch 90 ngày lên tay nhanh: mental models + 11 break-and-fix lab + capstone.

### 📝 Templates (luyện viết & trình sếp)
- **[templates/](./templates/README.md)** — [adr-template](./templates/adr-template.md) · [design-doc-template](./templates/design-doc-template.md) · [ví dụ ADR đã điền](./templates/examples/ADR-0001-transit-gateway-vs-peering.md).

---

## 🗺️ Lộ trình học đề xuất (theo mục tiêu)

```text
Nền DevOps vững:      cheatsheet -> k8s-operations-playbook -> k8s-production-templates
                      -> devops-fast-track (lab tự phá & sửa)

Kiến trúc Cloud/Cert: cloud/aws (00 -> 07) -> labs/cidr -> practice-questions -> cert-roadmap

Chuyển hướng AI:      tracks/roadmap-ai-platform-engineer -> mlops -> ml-serving-k8s
                      -> llmops-end-to-end -> fine-tuning-lora -> vector-db-scale
                      (bổ sung: sre, devsecops)

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
