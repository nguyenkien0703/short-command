# CKS — Tổng quan đề thi (Curriculum v1.34)

> Certified Kubernetes Security Specialist. 100% thực hành. 120 phút. Đậu từ **67%**.
> Hiệu lực **2 năm**. Giá niêm yết **$445** (kèm 1 lần thi lại miễn phí).
> **Điều kiện bắt buộc: đã thi đậu CKA** (CKA không cần còn hiệu lực).

---

## 1. Curriculum chính chủ — 6 domain

Bản CNCF `CKS_Curriculum_v1.34`, nguyên văn kèm dịch:

### 🟠 15% — Cluster Setup
- Use Network security policies to restrict cluster level access
- Use CIS benchmark to review the security configuration of Kubernetes components
  (etcd, kubelet, kubedns, kubeapi)
- Properly set up Ingress objects with TLS
- Protect node metadata and endpoints
- Verify platform binaries before deploying

→ [01-cluster-setup.md](./01-cluster-setup.md)

### 🟠 15% — Cluster Hardening
- Use Role Based Access Controls to minimize exposure
- Exercise caution in using service accounts e.g. disable defaults, minimize permissions
  on newly created ones
- Restrict access to Kubernetes API
- Upgrade Kubernetes to avoid vulnerabilities

→ [02-cluster-hardening.md](./02-cluster-hardening.md)

### 🟡 10% — System Hardening
- Minimize host OS footprint (reduce attack surface)
- Using least-privilege identity and access management
- Minimize external access to the network
- Appropriately use kernel hardening tools such as **AppArmor, seccomp**

→ [03-system-hardening.md](./03-system-hardening.md)

### 🔴 20% — Minimize Microservice Vulnerabilities
- Use appropriate **pod security standards**
- Manage kubernetes secrets
- Understand and implement isolation techniques (multi-tenancy, **sandboxed containers**, etc.)
- Implement **Pod-to-Pod encryption** (Cilium, Istio)

→ [04-minimize-microservice-vulnerabilities.md](./04-minimize-microservice-vulnerabilities.md)

### 🔴 20% — Supply Chain Security
- Minimize base image footprint
- Understand your supply chain (e.g. **SBOM**, CI/CD, artifact repositories)
- Secure your supply chain (permitted registries, sign and validate artifacts, etc.)
- Perform static analysis of user workloads and container images (e.g. **Kubesec, KubeLinter**)

→ [05-supply-chain-security.md](./05-supply-chain-security.md)

### 🔴 20% — Monitoring, Logging and Runtime Security
- Perform behavioral analytics to detect malicious activities
- Detect threats within physical infrastructure, apps, networks, data, users and workloads
- Investigate and identify phases of attack and bad actors within the environment
- Use **Kubernetes audit logs** to monitor access

→ [06-monitoring-logging-runtime.md](./06-monitoring-logging-runtime.md)

---

## 2. ⚠️ Lưu ý về trọng số

Nhiều tài liệu (kể cả trang CNCF ở một số thời điểm) vẫn ghi **Cluster Setup 10% / System Hardening 15%**.
Bản curriculum PDF `v1.34` hiện hành ghi ngược lại: **Cluster Setup 15% / System Hardening 10%**.

Về mặt ôn thi, khác biệt này **không quan trọng** — cả hai đều là domain nhẹ.
Điều quan trọng là: **3 domain 20% (Microservice + Supply Chain + Runtime) = 60% đề.**
Đó là nơi phải đổ công sức.

```text
Minimize Microservice Vuln  20%  ████████
Supply Chain Security       20%  ████████     ← 60% tổng điểm
Monitoring/Logging/Runtime  20%  ████████
Cluster Setup               15%  ██████
Cluster Hardening           15%  ██████
System Hardening            10%  ████
```

---

## 3. Vì sao CKS khó hơn CKA

| | CKA | CKS |
|---|---|---|
| Phạm vi | Trong K8s | K8s **+ Linux security + supply chain** |
| Tool bên ngoài | Gần như không | **Falco, AppArmor, seccomp, gVisor, Trivy, kube-bench, Kubesec, OPA/Kyverno, Cilium** |
| Kiểu câu hỏi | "Tạo X" | "Tìm chỗ không an toàn và sửa" |
| Docs hỗ trợ | kubernetes.io đủ dùng | Phải nhớ cú pháp tool ngoài |
| Đọc đề | Rõ ràng | Mơ hồ hơn, phải tự suy ra "cái gì đang sai" |

> Người đã thi nhận xét: CKS là **"khó nhất nhưng thú vị nhất"** trong bộ chứng chỉ K8s.
> Điểm mấu chốt: bạn phải **tự dựng kịch bản tấn công** để hiểu, chứ không chỉ học lệnh.

---

## 4. Tài liệu được phép mở trong phòng thi ⭐

Ngoài `kubernetes.io/docs` và `kubernetes.io/blog`, CKS cho phép thêm (danh sách này
thay đổi theo đợt — **luôn đọc lại Candidate Handbook trước ngày thi**):

| Domain được phép | Dùng cho |
|---|---|
| `kubernetes.io/docs`, `kubernetes.io/blog` | Mọi thứ K8s |
| `github.com/kubernetes` (một số repo) | Manifest mẫu |
| **Trivy** docs (`aquasecurity.github.io/trivy` / `trivy.dev`) | Scan image |
| **Falco** docs (`falco.org/docs`) | Runtime security, rule |
| **AppArmor** docs (`gitlab.com/apparmor/apparmor/-/wikis`) | Profile |
| **etcd** docs (`etcd.io/docs`) | Encryption, backup |
| **Cilium** docs (`docs.cilium.io`) | Pod-to-Pod encryption, NetworkPolicy nâng cao |

> ❌ **Không** được mở: Stack Overflow, blog cá nhân, GitHub tùy ý, Google search kết quả ngoài whitelist.
> Vào nhầm domain có thể bị proctor cảnh cáo hoặc hủy bài.

**Chiến lược:** không phụ thuộc vào docs ngoài. Danh sách trên chỉ để **tra cú pháp chi tiết**,
còn cấu trúc lệnh phải thuộc. Xem
[checklist thuộc lòng CKS](../exam-day-playbook.md#7-checklist-thuộc-lòng-trước-khi-thi).

---

## 5. Cấu trúc đề thi thực tế

| Hạng mục | Con số |
|---|---|
| Thời lượng | 120 phút |
| Số câu | **~15–17** |
| Điểm đậu | **67/100** |
| Thời gian/câu | ~7–8 phút |
| K8s version | Bám release mới (~v1.34) |
| Cluster | Nhiều cluster, mỗi câu 1 context |

**Kiểu đề CKS đặc trưng:**
```text
Task weight: 6%

Context: A CIS benchmark tool was run against the kubeadm-created cluster
and found multiple issues that must be immediately addressed.

Task: Fix all issues via configuration and restart the affected components
to ensure the new settings take effect.
  - Fix all issues identified by the kube-bench tool related to the kube-apiserver
  - Fix all issues identified by the kube-bench tool related to the kubelet
```
→ Đề **không nói rõ phải sửa gì**. Bạn phải chạy `kube-bench`, đọc `[FAIL]`, tự suy ra.

---

## 6. Bộ công cụ CKS — phải biết dùng

| Tool | Dùng để | Domain |
|---|---|---|
| **kube-bench** | Kiểm tra CIS Benchmark | Cluster Setup |
| **kubesec** | Static analysis manifest K8s | Supply Chain |
| **KubeLinter** | Lint manifest tìm cấu hình không an toàn | Supply Chain |
| **Trivy** | Scan image tìm CVE, tạo SBOM | Supply Chain |
| **Falco** | Runtime security, phát hiện hành vi bất thường | Runtime |
| **AppArmor** | MAC profile giới hạn syscall/file access | System Hardening |
| **seccomp** | Lọc syscall | System Hardening |
| **gVisor (runsc) / Kata** | Sandbox container | Microservice Vuln |
| **OPA Gatekeeper / Kyverno** | Policy admission | Cluster Hardening |
| **Cilium / Istio** | mTLS Pod-to-Pod | Microservice Vuln |
| **audit policy** | Log mọi request tới API | Runtime |
| **etcd encryption** | Mã hóa Secret at rest | Microservice Vuln |
| **cosign / sigstore** | Ký & verify image | Supply Chain |

> Không cần cài thành thạo từng cái. Đề thi **cài sẵn** tool, bạn chỉ cần biết
> **chạy lệnh, đọc output, sửa cấu hình**.

---

## 7. Lộ trình ôn CKS (8 tuần)

| Tuần | Nội dung | Lab |
|---|---|---|
| 1 | Cluster Setup: NetworkPolicy nâng cao, kube-bench, Ingress TLS, node metadata | Chạy kube-bench, sửa 10 FAIL |
| 2 | Cluster Hardening: RBAC sâu, SA token, API access, anonymous auth | Tự tạo SA có quyền tối thiểu |
| 3 | System Hardening: AppArmor, seccomp, giảm OS footprint | Viết 1 profile AppArmor + 1 seccomp |
| 4 | Microservice Vuln (1): Pod Security Admission, securityContext | Ép ns `restricted`, sửa Pod cho pass |
| 5 | Microservice Vuln (2): Secret, etcd encryption, gVisor, mTLS | Bật encryption at rest, chạy Pod với runsc |
| 6 | Supply Chain: Trivy, Kubesec, ImagePolicyWebhook, SBOM, ký image | Scan 5 image, chặn registry lạ |
| 7 | Runtime: Falco rule, audit policy, phases of attack | Bật audit log, viết 1 Falco rule |
| 8 | Luyện đề: KodeKloud CKS challenges, killer.sh ×2 | Bấm giờ như thi thật |

---

## 8. Tiêu chí "sẵn sàng đi thi"

- [ ] KodeKloud CKS mock ×3, mỗi bài ≥ 90%
- [ ] KodeKloud **CKS Challenges** (miễn phí) — làm hết, đây là phần sát đề nhất
- [ ] Killercoda CKS scenarios — làm mỗi lab **2–3 lần**
- [ ] killer.sh session 1 ≥ 60%, session 2 ≥ **90%**
- [ ] [practice-questions.md](./practice-questions.md) trong repo này, bấm giờ
- [ ] Bật audit log trên apiserver trong **< 6 phút** không tra docs
- [ ] Viết NetworkPolicy default-deny + mở DNS trong **< 3 phút**
- [ ] Sửa Pod cho pass Pod Security Standard `restricted` trong **< 4 phút**
- [ ] Chạy kube-bench, đọc và sửa 1 FAIL trong **< 5 phút**

---

## 9. Bạn làm K8s nhiều — điểm mù thường gặp ở CKS

| Điểm mù | Vì sao |
|---|---|
| **AppArmor / seccomp** | Ở công ty hầu như không ai bật thủ công |
| **gVisor / Kata** | Rất ít production dùng |
| **ImagePolicyWebhook** | Thường thay bằng Kyverno/OPA hoặc scan ở CI |
| **etcd encryption at rest** | Managed K8s làm hộ, không thấy config |
| **Audit policy YAML** | EKS/GKE bật bằng 1 checkbox |
| **kube-bench** | Ít khi tự chạy |
| **Falco rule syntax** | Thường chỉ đọc alert, không viết rule |
| **Static analysis (kubesec)** | Bị thay bằng tool CI khác |

→ **Đây là danh sách ưu tiên của bạn.** RBAC, NetworkPolicy, securityContext thì bạn đã quen —
chỉ cần luyện tốc độ.
