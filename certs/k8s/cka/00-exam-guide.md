# CKA — Tổng quan đề thi (Curriculum v1.35)

> Certified Kubernetes Administrator. 100% thực hành. 120 phút. Đậu từ **66%**.
> Hiệu lực **2 năm**. Giá niêm yết **$445** (kèm 1 lần thi lại miễn phí).

---

## 1. Curriculum chính chủ — 5 domain

Đây là bản CNCF `CKA_Curriculum_v1.35`, **nguyên văn** (dịch sang tiếng Việt kèm bản gốc):

### 🔴 30% — Troubleshooting (nặng nhất)
- Troubleshoot clusters and nodes — *xử lý sự cố cluster và node*
- Troubleshoot cluster components — *xử lý sự cố thành phần control plane*
- Monitor cluster and application resource usage — *theo dõi tài nguyên*
- Manage and evaluate container output streams — *quản lý & đọc log container*
- Troubleshoot services and networking — *xử lý sự cố service/mạng*

→ [05-troubleshooting.md](./05-troubleshooting.md)

### 🟠 25% — Cluster Architecture, Installation and Configuration
- Manage role based access control (RBAC)
- Prepare underlying infrastructure for installing a Kubernetes cluster
- Create and manage Kubernetes clusters using **kubeadm**
- Manage the lifecycle of Kubernetes clusters
- Implement and configure a highly-available control plane
- **Use Helm and Kustomize to install cluster components** ← mới 2025
- **Understand extension interfaces (CNI, CSI, CRI, etc.)** ← mới 2025
- **Understand CRDs, install and configure operators** ← mới 2025

→ [01-cluster-architecture.md](./01-cluster-architecture.md)

### 🟡 20% — Services and Networking
- Understand connectivity between Pods
- Define and enforce Network Policies
- Use ClusterIP, NodePort, LoadBalancer service types and endpoints
- **Use the Gateway API to manage Ingress traffic** ← mới 2025
- Know how to use Ingress controllers and Ingress resources
- Understand and use CoreDNS

→ [03-services-networking.md](./03-services-networking.md)

### 🟢 15% — Workloads and Scheduling
- Understand application deployments and how to perform rolling update and rollbacks
- Use ConfigMaps and Secrets to configure applications
- **Configure workload autoscaling** ← mới 2025 (HPA)
- Understand the primitives used to create robust, self-healing, application deployments
- Configure Pod admission and scheduling (limits, node affinity, etc.)

→ [02-workloads-scheduling.md](./02-workloads-scheduling.md)

### 🔵 10% — Storage
- Implement storage classes and dynamic volume provisioning
- Configure volume types, access modes and reclaim policies
- Manage persistent volumes and persistent volume claims

→ [04-storage.md](./04-storage.md)

---

## 2. Thay đổi lớn của CKA từ 2025 ⚠️

Nếu bạn học theo tài liệu/khóa học cũ (trước 02/2025) sẽ **thiếu hẳn** các mục sau:

| Chủ đề mới | Vì sao quan trọng | Học ở đâu |
|---|---|---|
| **Helm** | Đề yêu cầu cài component cluster bằng Helm (`repo add`, `install`, `upgrade`, `-f values.yaml`, `template`) | [01](./01-cluster-architecture.md#8-helm) |
| **Kustomize** | ⚠️ **Docs Kustomize KHÔNG được mở trong phòng thi** → phải thuộc cú pháp | [01](./01-cluster-architecture.md#9-kustomize) |
| **Gateway API** | Thay thế dần Ingress. Phải biết `GatewayClass` / `Gateway` / `HTTPRoute` | [03](./03-services-networking.md#6-gateway-api) |
| **CRD & Operator** | Cài/cấu hình operator, đọc CRD | [01](./01-cluster-architecture.md#10-crd--operator) |
| **CNI / CSI / CRI** | Hiểu extension interface, cài CNI plugin, debug runtime | [01](./01-cluster-architecture.md#11-extension-interfaces) |
| **Workload autoscaling** | HPA (và biết `metrics-server` là điều kiện cần) | [02](./02-workloads-scheduling.md#7-autoscaling--hpa) |

**Ngược lại — đã bị giảm/bỏ nhấn mạnh:** ETCD backup vẫn còn nhưng ít hơn;
phần "Application Lifecycle" cũ giờ gộp vào Workloads.

> 💡 Nhiều khóa Udemy/KodeKloud đã cập nhật. Kiểm tra khóa bạn học **có section
> Helm/Kustomize/Gateway API không** — nếu không thì đó là bản cũ.

---

## 3. Cấu trúc đề thi thực tế

| Hạng mục | Con số |
|---|---|
| Thời lượng | 120 phút |
| Số câu | ~15–20 (thường 17) |
| Điểm đậu | 66/100 |
| Số cluster | 5–8 cluster khác nhau, mỗi câu 1 context |
| K8s version | Cập nhật theo quý, bám release mới nhất (~v1.34/v1.35) |
| Base OS | Ubuntu |
| Cài bằng | kubeadm |
| Docs được mở | `kubernetes.io/docs`, `kubernetes.io/blog` và subdomain |

**Mỗi câu hiện:**
```text
Task weight: 4%
Context: You have been asked to ...
kubectl config use-context k8s-c1-s   ← [nút Copy]
```

---

## 4. Chiến lược ôn theo trọng số

```text
Phân bổ thời gian ôn hợp lý (không phải chia đều!):

Troubleshooting  30%  ████████████  ← đầu tư nhiều nhất, cũng khó "học vẹt" nhất
Cluster Arch     25%  ██████████    ← kubeadm/HA/RBAC/Helm/Kustomize: học thuộc quy trình
Services&Net     20%  ████████      ← NetworkPolicy + Gateway API là điểm mới
Workloads        15%  ██████        ← dễ ăn điểm nhất, làm nhanh, gọn
Storage          10%  ████          ← ít câu nhưng dễ, không được bỏ
```

**Thứ tự học đề xuất:**
1. **Workloads + Storage trước** (15%+10% = 25% đề, dễ nhất) → lấy tự tin, quen `$do`.
2. **Services & Networking** — luyện viết NetworkPolicy tay, học Gateway API.
3. **Cluster Architecture** — kubeadm upgrade, etcd backup, HA, RBAC, Helm, Kustomize.
4. **Troubleshooting cuối cùng** — vì nó *dùng lại* toàn bộ 4 domain trên.
   Đây là lúc [k8s-operations-playbook.md](../../../k8s-operations-playbook.md) trong repo phát huy tác dụng.

---

## 5. Bạn đã làm K8s nhiều — nên tập trung vào đâu?

Người làm K8s production thường **đã mạnh**: workload, service, troubleshoot app, log, RBAC cơ bản.
Và thường **yếu** đúng những chỗ đề thi hay hỏi:

| Điểm mù thường gặp của DevOps đi làm | Vì sao |
|---|---|
| **kubeadm init/join/upgrade** | Ở công ty thường dùng managed K8s (EKS/GKE) hoặc cluster đã có sẵn |
| **etcd snapshot/restore bằng tay** | Thường có backup tự động, không tự gõ |
| **HA control plane (stacked vs external etcd)** | Ít khi tự dựng |
| **Static pod & `/etc/kubernetes/manifests/`** | Managed K8s giấu control plane đi |
| **Kustomize cú pháp thuần** | Hay dùng Helm hoặc ArgoCD generate hộ |
| **Gateway API** | Còn mới, đa số công ty vẫn Ingress-nginx |
| **Tốc độ gõ** | Ở công ty có thời gian; phòng thi 7 phút/câu |

→ **Đây chính là danh sách ưu tiên của bạn.** Đừng phí thời gian ôn lại `kubectl create deployment`.

---

## 6. Tiêu chí "sẵn sàng đi thi"

- [ ] Làm hết 3 mock test KodeKloud, mỗi bài **≥ 90%** trong ≤ 60 phút
- [ ] killer.sh session 1 đạt **≥ 60%** (killer.sh khó hơn đề thật nhiều — đừng hoảng)
- [ ] killer.sh session 2 đạt **≥ 85%**, hiểu rõ *tại sao* từng câu
- [ ] Làm [practice-questions.md](./practice-questions.md) trong repo này, bấm giờ
- [ ] Gõ được toàn bộ [checklist thuộc lòng](../exam-day-playbook.md#7-checklist-thuộc-lòng-trước-khi-thi) không tra docs
- [ ] Backup + restore etcd trong **< 5 phút** không nhìn tài liệu
- [ ] Upgrade cluster kubeadm 1 control-plane + 1 worker trong **< 12 phút**

> killer.sh cho **2 session, mỗi session 36 giờ** truy cập. Đừng phí:
> làm session 1 nghiêm túc như thi thật, đọc kỹ solution, luyện lại; ~1 tuần sau làm session 2.
