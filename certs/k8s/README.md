# Kubernetes Certifications — CKA & CKS

> Bộ tài liệu ôn thi **CKA** (Certified Kubernetes Administrator) và **CKS** (Certified Kubernetes
> Security Specialist) — viết cho người **đã làm K8s thật**, muốn lấy chứng chỉ để chứng minh năng lực.
>
> Bám sát **curriculum chính chủ CNCF**: `CKA_Curriculum_v1.35` và `CKS_Curriculum_v1.34`.
> Mỗi domain: concept → lệnh/YAML thật → dạng bài hay ra → bẫy thường gặp.

---

## 📚 Cấu trúc tài liệu

### Chung
| File | Nội dung |
|------|----------|
| [exam-day-playbook.md](./exam-day-playbook.md) | **Chiến thuật phòng thi**: check-in, môi trường PSI, alias/vim/tmux setup, quản lý thời gian, copy-paste, tra docs |
| [study-plan.md](./study-plan.md) | Lộ trình 12 tuần CKA → 8 tuần CKS, lịch học theo tuần, mốc kiểm tra |
| [resources.md](./resources.md) | Nguồn học: cộng đồng VN (devops.vn, Viblo) + quốc tế (KodeKloud, Killercoda, killer.sh), mẹo săn voucher |

### CKA — 5 domain
| File | Domain | Trọng số |
|------|--------|:--------:|
| [cka/00-exam-guide.md](./cka/00-exam-guide.md) | Tổng quan đề thi, thay đổi 2025 (Helm/Kustomize/Gateway API/CRD) | — |
| [cka/01-cluster-architecture.md](./cka/01-cluster-architecture.md) | Cluster Architecture, Installation & Configuration | **25%** |
| [cka/02-workloads-scheduling.md](./cka/02-workloads-scheduling.md) | Workloads & Scheduling | **15%** |
| [cka/03-services-networking.md](./cka/03-services-networking.md) | Services & Networking | **20%** |
| [cka/04-storage.md](./cka/04-storage.md) | Storage | **10%** |
| [cka/05-troubleshooting.md](./cka/05-troubleshooting.md) | Troubleshooting — **domain nặng nhất** | **30%** |
| [cka/practice-questions.md](./cka/practice-questions.md) | 30 bài tập kiểu đề thật + lời giải | — |

### CKS — 6 domain
| File | Domain | Trọng số |
|------|--------|:--------:|
| [cks/00-exam-guide.md](./cks/00-exam-guide.md) | Tổng quan đề thi, docs được phép mở, tool cần thuộc | — |
| [cks/01-cluster-setup.md](./cks/01-cluster-setup.md) | Cluster Setup (NetworkPolicy, CIS/kube-bench, Ingress TLS, node metadata) | **15%** |
| [cks/02-cluster-hardening.md](./cks/02-cluster-hardening.md) | Cluster Hardening (RBAC, ServiceAccount, API access, upgrade) | **15%** |
| [cks/03-system-hardening.md](./cks/03-system-hardening.md) | System Hardening (OS footprint, AppArmor, seccomp, immutability) | **10%** |
| [cks/04-minimize-microservice-vulnerabilities.md](./cks/04-minimize-microservice-vulnerabilities.md) | Pod Security Standards, Secrets, sandbox (gVisor/Kata), mTLS | **20%** |
| [cks/05-supply-chain-security.md](./cks/05-supply-chain-security.md) | Base image, SBOM, signing, ImagePolicyWebhook, Trivy/Kubesec | **20%** |
| [cks/06-monitoring-logging-runtime.md](./cks/06-monitoring-logging-runtime.md) | Falco, audit log, behavioral analytics, phases of attack | **20%** |
| [cks/practice-questions.md](./cks/practice-questions.md) | 25 bài tập kiểu đề thật + lời giải | — |

---

## 🎫 So sánh nhanh CKA vs CKS

| | **CKA** | **CKS** |
|---|---|---|
| Tên đầy đủ | Certified Kubernetes Administrator | Certified Kubernetes Security Specialist |
| Curriculum hiện hành | v1.35 | v1.34 |
| Điều kiện | Không | **Phải đã thi đậu CKA** (CKA không cần còn hiệu lực) |
| Thời lượng | 120 phút | 120 phút |
| Số câu | ~15–20 câu | ~15–17 câu |
| Điểm đậu | **66%** | **67%** |
| Giá niêm yết | $445 (kèm 1 lần thi lại free) | $445 (kèm 1 lần thi lại free) |
| Hiệu lực | **2 năm** | **2 năm** |
| Hình thức | 100% thực hành trên terminal, remote desktop, có proctor | Giống CKA |
| Trọng tâm | Vận hành cluster: kubeadm, HA, troubleshoot | Bảo mật: hardening, supply chain, runtime |
| Docs được mở | `kubernetes.io/docs`, `kubernetes.io/blog` (+ subdomain) | Như CKA + **Trivy, Falco, AppArmor, etcd, Cilium** docs |

> ⚠️ Giá và chính sách thay đổi theo đợt (Cyber Monday ~30–50% off). Luôn kiểm tra lại
> [training.linuxfoundation.org](https://training.linuxfoundation.org/) trước khi mua.
> Trước đây chứng chỉ có hiệu lực 3 năm; các kỳ thi mới đã rút xuống **2 năm**.

---

## 🗺️ Lộ trình chứng chỉ Kubernetes (bức tranh lớn)

```text
                        ┌──────────────────────────┐
                        │  KCNA (associate, lý thuyết) │  ← tùy chọn, dễ, lấy đà
                        └────────────┬─────────────┘
                                     │
                     ┌───────────────┴───────────────┐
                     ▼                               ▼
          ┌────────────────────┐          ┌────────────────────┐
          │  CKA  (vận hành)   │          │  CKAD (dev/app)     │  ← tùy chọn
          │  ★ mục tiêu 1      │          └────────────────────┘
          └─────────┬──────────┘
                    │ (bắt buộc)
                    ▼
          ┌────────────────────┐          ┌────────────────────┐
          │  CKS  (bảo mật)    │◄─────────│ KCSA (security lý  │  ← tùy chọn, bổ trợ
          │  ★ mục tiêu 2      │          │ thuyết)             │
          └────────────────────┘          └────────────────────┘

Đủ 5 cái (KCNA + KCSA + CKA + CKAD + CKS) → danh hiệu "Kubestronaut".
```

**Bạn đang làm K8s nhiều → đi thẳng CKA → CKS.** KCNA/CKAD chỉ nên lấy nếu muốn gom Kubestronaut,
vì kiến thức của chúng nằm gọn trong CKA rồi.

---

## ⏱️ Ước lượng thời gian (người đã làm K8s production)

| Giai đoạn | Thời gian | Ghi chú |
|---|---|---|
| CKA — ôn concept + lab | 3–4 tuần (1–2h/ngày) | Bạn đã có nền vận hành → chủ yếu học phần *kubeadm, HA, Helm/Kustomize/Gateway API* |
| CKA — luyện đề | 1–2 tuần | KodeKloud mock ×3, Killercoda, killer.sh ×2 |
| **Nghỉ / thi CKA** | | |
| CKS — ôn concept + lab | 4–6 tuần | Nhiều tool mới: Falco, AppArmor, seccomp, gVisor, Trivy, OPA/Kyverno |
| CKS — luyện đề | 2 tuần | KodeKloud CKS challenges, killer.sh ×2 |

> Kinh nghiệm cộng đồng VN: người có lab thật ~1 tháng cho CKA; người mới cần 2–3 tháng.
> CKS khó hơn CKA rõ rệt vì **tool bên ngoài K8s** (Linux security) chiếm tỉ trọng lớn.

---

## 💡 Cách dùng bộ tài liệu này

1. **Đọc [exam-day-playbook.md](./exam-day-playbook.md) TRƯỚC** — biết luật chơi rồi mới học.
   Rất nhiều người mất 30–50 phút vì check-in, không phải vì không biết kiến thức.
2. **Học theo domain, ưu tiên theo trọng số.** CKA: Troubleshooting 30% → học kỹ nhất.
   CKS: 3 domain 20% (microservice/supply chain/runtime) = 60% đề.
3. **Gõ tay mọi lệnh trong file** trên cluster thật (kind/minikube/kubeadm). Đọc không ăn thua —
   đề thi tính bằng phút, phải thành *muscle memory*.
4. **Làm [practice-questions](./cka/practice-questions.md) như thi thật** — bấm giờ, không mở đáp án.
5. **Ghi chú điểm sai** ngay vào file tương ứng. Repo lớn lên cùng bạn.

---

## 🔗 Liên quan trong repo

- [../../k8s-operations-playbook.md](../../k8s-operations-playbook.md) — playbook sự cố K8s thật
  (rất hợp để luyện domain **Troubleshooting 30%** của CKA)
- [../../k8s-production-templates.md](../../k8s-production-templates.md) — YAML production-ready
  (PDB/HPA/NetworkPolicy/RBAC — trùng nhiều với đề CKA/CKS)
- [../../cheatsheet.md](../../cheatsheet.md) — tra nhanh lệnh kubectl/helm
- [../../tracks/devsecops.md](../../tracks/devsecops.md) — nền DevSecOps, bổ trợ tốt cho CKS
- [../../cloud/aws/](../../cloud/aws/README.md) — cert AWS (song song nếu muốn)
