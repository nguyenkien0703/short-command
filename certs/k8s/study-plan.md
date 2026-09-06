# Lộ trình ôn CKA → CKS

> Kế hoạch cho người **đã làm K8s production** (không phải người mới học K8s).
> Tổng: **~12 tuần CKA + 8 tuần CKS**, khoảng 1–2h/ngày.
> Nếu bạn có nhiều thời gian hơn thì rút ngắn được; ít hơn thì giãn ra, đừng cắt bớt lab.

---

## Nguyên tắc xuyên suốt

1. **Gõ tay, không đọc suông.** Xem video 1 giờ ≈ lab 15 phút về giá trị. Ưu tiên lab.
2. **Bấm giờ từ sớm.** Đề CKA/CKS không khó về kiến thức, khó về **tốc độ**.
   Từ tuần thứ 3 trở đi, mọi lab đều bấm giờ.
3. **Một cluster kubeadm thật.** kind/minikube không làm được: upgrade, etcd restore,
   static pod, kubelet config, AppArmor. Dựng 2 VM Ubuntu (2 vCPU/4GB mỗi máy) là đủ.

   > ⚠️ **Đã có sẵn cụm RKE2 / k3s / EKS / GKE?** Chúng **không thay được** cụm kubeadm cho
   > phần control plane — RKE2 còn dạy phản xạ *sai* (sửa `config.yaml` thay vì sửa manifest).
   > Nếu bạn đang có cụm thật và muốn lộ trình bám vào chúng:
   > 👉 **[lab-2-cum-rke2-calico.md](./lab-2-cum-rke2-calico.md)** — bản đồ domain nào luyện ở cụm nào,
   > bảng dịch RKE2 ↔ kubeadm, và lộ trình tuần đã gắn sẵn tên cụm.
4. **Ghi lại mọi lỗi bạn mắc** vào file domain tương ứng trong repo này. Đọc lại trước ngày thi.
5. **Săn voucher.** Cyber Monday (thứ Hai cuối tháng 11) giảm ~30–50%. Mua trước, thi sau —
   voucher thường có hạn 12 tháng.

---

## PHẦN 1 — CKA (12 tuần)

### Giai đoạn A: Nền + domain dễ (tuần 1–3)

| Tuần | Nội dung | Tài liệu | Lab bắt buộc |
|---|---|---|---|
| **1** | Đọc [exam-day-playbook](./exam-day-playbook.md) + [cka/00-exam-guide](./cka/00-exam-guide.md).<br>Dựng cluster kubeadm 1cp+2worker từ đầu. Setup alias/vim. | Playbook, [01 §3](./cka/01-cluster-architecture.md#3-kubeadm--dựng--join-cluster) | `kubeadm init` + join + cài CNI, làm **2 lần** |
| **2** | **Workloads & Scheduling (15%)** | [cka/02](./cka/02-workloads-scheduling.md) | Sinh mọi resource bằng `$do`; taint/toleration; affinity; HPA |
| **3** | **Storage (10%)** + ôn lại Workloads | [cka/04](./cka/04-storage.md) | PV/PVC/SC full lab; sửa PVC Pending; resize |

**Mốc kiểm tra tuần 3:** làm câu 1–8, 15–18 của [practice-questions](./cka/practice-questions.md)
trong **50 phút**, đúng ≥ 10/12.

### Giai đoạn B: Domain nặng (tuần 4–7)

| Tuần | Nội dung | Tài liệu | Lab bắt buộc |
|---|---|---|---|
| **4** | **Services & Networking (20%)** — phần Service, DNS, Ingress | [cka/03 §1-5](./cka/03-services-networking.md) | Debug endpoints rỗng ×5 kịch bản; CoreDNS |
| **5** | **NetworkPolicy + Gateway API** (phần mới 2025) | [cka/03 §4,6](./cka/03-services-networking.md#4-networkpolicy-phải-viết-được-từ-đầu) | Viết 10 NetworkPolicy **không tra docs**; cài Gateway API + HTTPRoute |
| **6** | **Cluster Architecture (25%)** — kubeadm, upgrade, etcd, HA | [cka/01 §3-7](./cka/01-cluster-architecture.md) | Upgrade cluster ×3; etcd backup/restore ×5 |
| **7** | **Helm + Kustomize + CRD/Operator** (phần mới 2025) | [cka/01 §8-11](./cka/01-cluster-architecture.md#8-helm) | Cài 3 chart bằng Helm; viết `kustomization.yaml` **không tra docs** ×10 |

**Mốc kiểm tra tuần 7:** etcd restore trong **< 5 phút**, upgrade cp+worker trong **< 12 phút**,
viết NetworkPolicy AND-selector trong **< 3 phút** — tất cả không tra docs.

### Giai đoạn C: Troubleshooting (tuần 8–9)

| Tuần | Nội dung | Tài liệu | Lab bắt buộc |
|---|---|---|---|
| **8** | **Troubleshooting (30%)** — node, pod, control plane | [cka/05](./cka/05-troubleshooting.md) | **Toàn bộ 15 lab tự phá** ở [§11](./cka/05-troubleshooting.md#11-lab-tự-tạo-lỗi--cách-luyện-hiệu-quả-nhất) |
| **9** | Troubleshooting nâng cao: networking, jsonpath, log | [cka/05 §6-10](./cka/05-troubleshooting.md), [k8s-operations-playbook](../../k8s-operations-playbook.md) | Lặp lại 15 lab, mỗi cái **< 5 phút** |

**Mốc kiểm tra tuần 9:** bạn bè/AI tự phá cluster của bạn, bạn sửa được trong 8 phút mà
không biết trước lỗi gì.

### Giai đoạn D: Luyện đề (tuần 10–12)

| Tuần | Nội dung | Mục tiêu |
|---|---|---|
| **10** | KodeKloud mock 1,2,3 — lần 1. Đọc kỹ solution.<br>Killercoda CKA scenarios (miễn phí) | Mock ≥ 75% |
| **11** | Làm lại 3 mock (lần 2, 3).<br>[practice-questions](./cka/practice-questions.md) toàn bộ 30 câu, bấm giờ 120 phút | Mock ≥ 90%, practice ≥ 25/30 |
| **12** | **killer.sh session 1** (nghiêm túc như thi thật) → đọc toàn bộ solution → luyện lại chỗ sai.<br>Cuối tuần: **killer.sh session 2** | Session 1 ≥ 60%, session 2 ≥ 85% |

> 💡 killer.sh **khó hơn đề thật rõ rệt**. Được 60% ở session 1 là bình thường, đừng hoảng.
> Người thi thật đạt 93/100 kể rằng đề thật "ngắn gọn hơn killer.sh nhiều".

**→ ĐẶT LỊCH THI cuối tuần 12.**

---

## PHẦN 2 — CKS (8 tuần, sau khi đậu CKA)

> Nghỉ 1–2 tuần sau CKA rồi bắt đầu. Đừng thi liền — CKS cần đầu óc tỉnh táo.

| Tuần | Nội dung | Tài liệu | Lab bắt buộc |
|---|---|---|---|
| **1** | Đọc [cks/00-exam-guide](./cks/00-exam-guide.md).<br>**Cluster Setup (15%)**: NetworkPolicy nâng cao, kube-bench, Ingress TLS, node metadata, verify binary | [cks/01](./cks/01-cluster-setup.md) | Chạy kube-bench, sửa **10 FAIL**; tắt readOnlyPort; chặn metadata |
| **2** | **Cluster Hardening (15%)**: RBAC audit, SA token, API access, CSR | [cks/02](./cks/02-cluster-hardening.md) | Tắt automount toàn cluster; thu hẹp 3 ClusterRole; approve/deny CSR |
| **3** | **System Hardening (10%)**: OS footprint, AppArmor, seccomp, capabilities | [cks/03](./cks/03-system-hardening.md) | Viết + nạp 2 AppArmor profile; 2 seccomp profile; tìm & tắt service lạ |
| **4** | **Microservice Vuln (20%) phần 1**: Pod Security Admission, securityContext | [cks/04 §1-2](./cks/04-minimize-microservice-vulnerabilities.md) | Ép ns `restricted`, sửa 5 Pod cho pass |
| **5** | **Microservice Vuln (20%) phần 2**: Secret, encryption at rest, gVisor, mTLS | [cks/04 §3-5](./cks/04-minimize-microservice-vulnerabilities.md#3-quản-lý-secret) | Bật encryption at rest **×3 lần**; RuntimeClass gVisor |
| **6** | **Supply Chain (20%)**: Trivy, kubesec, SBOM, ImagePolicyWebhook, Kyverno | [cks/05](./cks/05-supply-chain-security.md) | Scan 10 image; bật ImagePolicyWebhook ×3; sửa Dockerfile |
| **7** | **Runtime (20%)**: Falco rule, audit log, phases of attack | [cks/06](./cks/06-monitoring-logging-runtime.md) | Sửa Falco rule ×5; bật audit log ×3; truy vấn audit bằng jq |
| **8** | **Luyện đề**: KodeKloud CKS challenges + mock ×3.<br>[cks/practice-questions](./cks/practice-questions.md) bấm giờ.<br>**killer.sh ×2** | Mock ≥ 90%, killer.sh session 2 ≥ 90% |

**→ ĐẶT LỊCH THI cuối tuần 8.**

---

## Kỹ năng phải thành phản xạ (đo bằng đồng hồ)

### CKA
| Việc | Thời gian mục tiêu |
|---|---|
| Setup alias + vimrc đầu giờ | < 45 giây |
| Tạo Deployment + Service + expose | < 90 giây |
| etcd backup | < 90 giây |
| etcd restore (sửa manifest + chờ) | < 5 phút |
| Upgrade control plane | < 8 phút |
| Viết NetworkPolicy AND-selector | < 3 phút |
| Viết `kustomization.yaml` đủ 5 field | < 2 phút |
| Chẩn đoán node NotReady | < 5 phút |
| RBAC (role+binding+verify) | < 2 phút |

### CKS
| Việc | Thời gian mục tiêu |
|---|---|
| Sửa flag apiserver + chờ restart | < 4 phút |
| Bật audit log (policy + 5 flag + 2 volume) | < 6 phút |
| Bật encryption at rest | < 6 phút |
| NetworkPolicy default-deny + DNS | < 3 phút |
| Sửa Pod cho pass PSS `restricted` | < 4 phút |
| Nạp AppArmor profile + gắn vào Pod | < 4 phút |
| Sửa Falco rule + restart + verify | < 5 phút |
| kube-bench → đọc FAIL → sửa 1 mục | < 5 phút |
| Trivy scan 4 image, chọn cái sạch | < 4 phút |

> Nếu chưa đạt mốc nào → **đó chính là bài tập tuần này của bạn.**

---

## Lịch tuần mẫu (1.5h/ngày)

```text
Thứ 2  : Đọc lý thuyết domain của tuần (45') + lab theo tài liệu (45')
Thứ 3  : Lab thực hành (90') — gõ lại mọi lệnh trong file, không copy-paste
Thứ 4  : Lab bấm giờ (60') + ghi chú lỗi vào repo (30')
Thứ 5  : Killercoda scenario cùng chủ đề (90')
Thứ 6  : Tự phá cluster & sửa (90')
Thứ 7  : Mini-mock 10 câu bấm giờ 60' + review (30')
CN     : NGHỈ, hoặc đọc lại ghi chú lỗi trong tuần (30')
```

---

## Checklist trước ngày thi (làm T-3 ngày)

### Kỹ thuật
- [ ] Chạy PSI system check thành công
- [ ] Test webcam rời, mạng LAN
- [ ] Đăng nhập được portal Linux Foundation, thấy lịch thi
- [ ] Tên trên CCCD/hộ chiếu **khớp chính xác** tên đăng ký

### Phòng
- [ ] Dọn bàn trống hoàn toàn
- [ ] Gỡ tranh/poster trên tường
- [ ] Kiểm tra **không có camera an ninh** trong phòng
- [ ] Ổ điện hoạt động, có chỗ cắm sạc
- [ ] Chốt cửa, báo người nhà không vào

### Kiến thức
- [ ] Đọc lại [checklist thuộc lòng](./exam-day-playbook.md#7-checklist-thuộc-lòng-trước-khi-thi)
- [ ] Đọc lại ghi chú lỗi của chính mình
- [ ] Đọc lại phần "Bẫy tổng kết" ở cuối mỗi file domain
- [ ] Gõ lại khối setup 60 giây (alias + vimrc) từ trí nhớ

### Ngày thi
- [ ] Vào sớm **30 phút**
- [ ] Uống nước, đi WC trước
- [ ] Chuẩn bị tinh thần check-in có thể mất 30–50 phút

---

## Sau khi đậu

1. **Cập nhật LinkedIn + CV** — badge Credly có link verify công khai.
2. **Viết lại hành trình** — 1 bài blog/Viblo. Vừa củng cố kiến thức, vừa là body-of-work.
   Đây đúng triết lý của repo này.
3. **Đặt nhắc lịch gia hạn** — chứng chỉ hết hạn sau **2 năm**.
4. **Bước tiếp theo** (nếu muốn):
   - Gom nốt KCNA + KCSA + CKAD → danh hiệu **Kubestronaut**
   - Hoặc rẽ sang [cloud/aws](../../cloud/aws/README.md) — AWS SAA/DOP
   - Hoặc đào sâu [tracks/](../../tracks/README.md) — MLOps/SRE/DevSecOps
