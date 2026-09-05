# CKA — Cluster Architecture, Installation & Configuration (25%)

> Domain nặng thứ hai. Đây là chỗ DevOps đi làm hay **yếu nhất** vì công ty thường dùng
> EKS/GKE (không đụng control plane) — trong khi đề thi bắt bạn `kubeadm` bằng tay.

**Nội dung curriculum v1.35:**
- Manage role based access control (RBAC)
- Prepare underlying infrastructure for installing a Kubernetes cluster
- Create and manage Kubernetes clusters using kubeadm
- Manage the lifecycle of Kubernetes clusters
- Implement and configure a highly-available control plane
- Use **Helm** and **Kustomize** to install cluster components
- Understand extension interfaces (**CNI, CSI, CRI**, etc.)
- Understand **CRDs**, install and configure **operators**

---

## 1. Kiến trúc cluster — bản đồ trong đầu

```text
┌──────────────────── CONTROL PLANE NODE ────────────────────┐
│  /etc/kubernetes/manifests/   ← STATIC POD (kubelet tự chạy)│
│    ├── kube-apiserver.yaml      cổng vào duy nhất, :6443    │
│    ├── kube-controller-manager.yaml   vòng lặp reconcile     │
│    ├── kube-scheduler.yaml            gán Pod → Node         │
│    └── etcd.yaml                      key-value store, :2379 │
│                                                              │
│  /etc/kubernetes/                                            │
│    ├── admin.conf         kubeconfig của cluster-admin       │
│    ├── kubelet.conf                                          │
│    ├── controller-manager.conf / scheduler.conf              │
│    └── pki/               toàn bộ chứng chỉ (ca.crt, ...)    │
│                                                              │
│  kubelet (systemd) ── /var/lib/kubelet/config.yaml           │
│  kube-proxy (DaemonSet)                                      │
└──────────────────────────────────────────────────────────────┘
                              │
┌──────────────── WORKER NODE ─────────────────┐
│  kubelet (systemd)                            │
│  container runtime: containerd (:/run/...sock)│
│  kube-proxy (DaemonSet)                       │
│  CNI plugin (Calico/Flannel/Cilium)           │
└───────────────────────────────────────────────┘
```

**Đường dẫn phải thuộc lòng** (đề thi bắt sửa file trực tiếp):

| Đường dẫn | Là gì |
|---|---|
| `/etc/kubernetes/manifests/` | Static pod manifest của control plane |
| `/etc/kubernetes/pki/` | Chứng chỉ (ca.crt, apiserver.crt, etcd/) |
| `/etc/kubernetes/admin.conf` | kubeconfig admin |
| `/var/lib/kubelet/config.yaml` | Cấu hình kubelet |
| `/var/lib/etcd` | Data dir của etcd |
| `/etc/systemd/system/kubelet.service.d/` | Drop-in config kubelet |
| `/var/log/pods/`, `/var/log/containers/` | Log container trên node |

> 🔑 **Static pod**: kubelet đọc thư mục `manifests/` và tự chạy pod, **không qua scheduler**.
> Sửa file YAML ở đó → kubelet tự restart pod trong vài giây. Không có `kubectl apply` gì cả.
> Đây là cách duy nhất để đổi flag của apiserver/scheduler/controller-manager.

---

## 2. RBAC — chắc chắn có trong đề

### Mô hình
```text
Subject (User / Group / ServiceAccount)
        │
        │  RoleBinding  (trong 1 namespace)
        │  ClusterRoleBinding  (toàn cluster)
        ▼
Role (namespaced)  /  ClusterRole (cluster-wide)
        │
        ▼
Rules: apiGroups × resources × verbs
```

**4 tổ hợp cần phân biệt:**

| Binding | Role | Phạm vi hiệu lực |
|---|---|---|
| RoleBinding | Role | 1 namespace |
| RoleBinding | **ClusterRole** | 1 namespace ← *dùng lại ClusterRole cho từng ns, rất hay ra đề* |
| ClusterRoleBinding | ClusterRole | Toàn cluster |
| ClusterRoleBinding | Role | ❌ **KHÔNG hợp lệ** |

### Lệnh imperative (nhanh nhất — dùng trong phòng thi)

```bash
# Role: cho phép get/list/watch pods trong ns dev
k create role pod-reader --verb=get,list,watch --resource=pods -n dev

# Gán cho user
k create rolebinding pod-reader-bind --role=pod-reader --user=jane -n dev

# Gán cho ServiceAccount (chú ý cú pháp ns:name)
k create rolebinding sa-bind --role=pod-reader --serviceaccount=dev:mysa -n dev

# ClusterRole + ClusterRoleBinding
k create clusterrole deploy-viewer --verb=get,list --resource=deployments
k create clusterrolebinding dv-bind --clusterrole=deploy-viewer --group=devs

# ClusterRole giới hạn vào 1 resource cụ thể
k create clusterrole pod-x --verb=get --resource=pods --resource-name=nginx-1

# Role cho subresource (log, exec, portforward) — hay ra đề!
k create role log-reader --verb=get --resource=pods/log -n dev
```

### Kiểm tra quyền — `auth can-i` ⭐
```bash
k auth can-i create deployments --as=jane -n dev
k auth can-i get pods --as=system:serviceaccount:dev:mysa -n dev
k auth can-i --list --as=jane -n dev          # liệt kê TẤT CẢ quyền
```
> 💡 Sau khi làm câu RBAC, **luôn verify bằng `auth can-i --as=`**. Đây là 20 giây rẻ nhất trong bài thi.

### Bẫy RBAC hay gặp
- `apiGroups: [""]` là **core group** (pods, services, configmaps, secrets, nodes, pv, pvc).
  Deployment thuộc `apps`; Ingress/NetworkPolicy thuộc `networking.k8s.io`; Role/RoleBinding thuộc `rbac.authorization.k8s.io`.
  Tra nhanh: `k api-resources | grep -i deploy`.
- ServiceAccount user string đầy đủ: `system:serviceaccount:<ns>:<name>`.
- ClusterRole tạo bằng `k create clusterrole` **không** tự động có quyền trên namespaced resource
  ở mọi ns cho tới khi có ClusterRoleBinding.

---

## 3. kubeadm — dựng & join cluster

### Chuẩn bị hạ tầng (curriculum: "prepare underlying infrastructure")
```bash
# 1. Tắt swap (K8s yêu cầu)
swapoff -a
sed -i '/ swap / s/^/#/' /etc/fstab

# 2. Module kernel
cat <<EOF > /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
modprobe overlay && modprobe br_netfilter

# 3. Sysctl cho bridged traffic + ip forward
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sysctl --system

# 4. Container runtime (containerd) — bật SystemdCgroup
containerd config default > /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
systemctl restart containerd
```
> 🔴 **Bẫy kinh điển**: `SystemdCgroup = true`. Nếu quên, kubelet và containerd dùng
> cgroup driver khác nhau → node `NotReady`, Pod không lên. Đề troubleshooting rất hay dùng bẫy này.

### Init & join
```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=<IP>

mkdir -p $HOME/.kube && cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

# Cài CNI (bắt buộc, nếu không node mãi NotReady)
k apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml

# Lấy lại lệnh join khi token hết hạn (token sống 24h)
kubeadm token create --print-join-command
```

### Xem cấu hình cluster
```bash
kubeadm config print init-defaults
k get cm kubeadm-config -n kube-system -o yaml   # cấu hình cluster đang chạy
kubeadm certs check-expiration                   # hạn chứng chỉ
kubeadm certs renew all                          # gia hạn
```

---

## 4. Upgrade cluster ⭐⭐ (gần như chắc chắn có 1 câu)

**Nguyên tắc:** control plane trước → worker sau. Không nhảy quá 1 minor version (1.32 → 1.33 → 1.34).

### Control plane node
```bash
# --- 1. Nâng gói kubeadm ---
apt-get update
apt-cache madison kubeadm | head              # tìm version chính xác
apt-mark unhold kubeadm
apt-get install -y kubeadm=1.34.1-1.1
apt-mark hold kubeadm

# --- 2. Xem kế hoạch & áp dụng ---
kubeadm upgrade plan
kubeadm upgrade apply v1.34.1                 # ← control plane ĐẦU TIÊN

# --- 3. Drain node ---
k drain <cp-node> --ignore-daemonsets

# --- 4. Nâng kubelet + kubectl ---
apt-mark unhold kubelet kubectl
apt-get install -y kubelet=1.34.1-1.1 kubectl=1.34.1-1.1
apt-mark hold kubelet kubectl
systemctl daemon-reload && systemctl restart kubelet

# --- 5. Trả node về ---
k uncordon <cp-node>
```

### Worker node
```bash
# Trên control plane:
k drain <worker> --ignore-daemonsets --delete-emptydir-data

# SSH vào worker:
apt-mark unhold kubeadm && apt-get install -y kubeadm=1.34.1-1.1 && apt-mark hold kubeadm
kubeadm upgrade node                          # ← KHÁC control plane: "node", không phải "apply"
apt-mark unhold kubelet kubectl
apt-get install -y kubelet=1.34.1-1.1 kubectl=1.34.1-1.1
apt-mark hold kubelet kubectl
systemctl daemon-reload && systemctl restart kubelet
exit

# Về control plane:
k uncordon <worker>
```

| Bẫy | Chi tiết |
|---|---|
| `kubeadm upgrade apply` vs `kubeadm upgrade node` | `apply` chỉ dùng cho **control plane đầu tiên**. Mọi node khác dùng `node`. |
| Quên `apt-mark unhold` | `apt-get install` sẽ không nâng version |
| Quên `--ignore-daemonsets` | drain fail vì DaemonSet pod không evict được |
| Có emptyDir | phải thêm `--delete-emptydir-data` |
| Quên `uncordon` | Node vẫn `SchedulingDisabled` → mất điểm |
| Quên `systemctl daemon-reload` | kubelet không nhận cấu hình mới |

> ⚠️ Đề có thể nói *"upgrade only the control plane node, do not upgrade worker"*. **Đọc kỹ.**

---

## 5. etcd — Backup & Restore ⭐⭐

```bash
# --- BACKUP ---
ETCDCTL_API=3 etcdctl snapshot save /opt/backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

ETCDCTL_API=3 etcdctl snapshot status /opt/backup.db --write-out=table
```

```bash
# --- RESTORE ---
# 1. Restore vào thư mục MỚI (không ghi đè /var/lib/etcd)
ETCDCTL_API=3 etcdctl snapshot restore /opt/backup.db \
  --data-dir=/var/lib/etcd-restore

# 2. Trỏ static pod etcd sang data dir mới
vim /etc/kubernetes/manifests/etcd.yaml
#    volumes:
#    - hostPath:
#        path: /var/lib/etcd-restore      ← đổi dòng này (volume tên etcd-data)
#        type: DirectoryOrCreate

# 3. kubelet tự restart etcd. Chờ:
watch crictl ps | grep etcd
# hoặc: k get po -n kube-system
```

**Ghi nhớ 3 cờ cert**: `--cacert` / `--cert` / `--key` đều nằm trong `/etc/kubernetes/pki/etcd/`.
Nếu quên đường dẫn: `cat /etc/kubernetes/manifests/etcd.yaml | grep file` — mọi cờ có sẵn ở đó.

| Bẫy | Chi tiết |
|---|---|
| Restore **không** cần `--endpoints`/cert | Restore đọc file local, không nói chuyện với etcd đang chạy |
| Ghi đè `/var/lib/etcd` | Dễ hỏng. Restore ra dir mới rồi sửa manifest — an toàn hơn |
| Sửa `hostPath.path` nhưng quên `mountPath` | `mountPath` **giữ nguyên** `/var/lib/etcd` — chỉ đổi `hostPath` |
| Không chờ apiserver lên lại | `kubectl` sẽ báo connection refused vài chục giây — bình thường, đợi |

---

## 6. HA Control Plane

```text
STACKED etcd (mặc định kubeadm)      │  EXTERNAL etcd
                                      │
 ┌────────┐ ┌────────┐ ┌────────┐    │  ┌────────┐ ┌────────┐ ┌────────┐
 │ cp1    │ │ cp2    │ │ cp3    │    │  │ cp1    │ │ cp2    │ │ cp3    │
 │ api    │ │ api    │ │ api    │    │  │ api    │ │ api    │ │ api    │
 │ etcd   │ │ etcd   │ │ etcd   │    │  └───┬────┘ └───┬────┘ └───┬────┘
 └────────┘ └────────┘ └────────┘    │      └──────────┼──────────┘
     ▲          ▲          ▲          │            ┌────┴────┐
     └──────────┴──────────┘          │            │ etcd    │ (cụm riêng 3 node)
        Load Balancer :6443           │            └─────────┘
                                      │
 Ít máy hơn, dễ dựng                  │  Tách bạch, etcd hỏng không kéo cp chết
 etcd chết theo control plane         │  Tốn gấp đôi máy
```

- **Luôn dùng số lẻ** node etcd (3, 5) — quorum = `(n/2)+1`. 3 node chịu được 1 hỏng; 4 node **cũng chỉ** chịu 1 hỏng.
- Join thêm control plane:
  ```bash
  kubeadm init --control-plane-endpoint "LB_IP:6443" --upload-certs   # trên cp1
  kubeadm join LB_IP:6443 --token ... --control-plane --certificate-key ...
  ```
- `--certificate-key` sống **2 giờ**. Lấy lại: `kubeadm init phase upload-certs --upload-certs`.

**Kiểm tra sức khỏe etcd cluster:**
```bash
ETCDCTL_API=3 etcdctl member list --write-out=table \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

---

## 7. Quản lý vòng đời node

```bash
k cordon <node>        # không nhận Pod mới, Pod cũ ở nguyên
k drain <node> --ignore-daemonsets --delete-emptydir-data --force
k uncordon <node>      # cho nhận Pod trở lại
k delete node <node>   # gỡ khỏi cluster (chạy kubeadm reset trên node đó trước)
```

| Cờ của `drain` | Khi nào cần |
|---|---|
| `--ignore-daemonsets` | Hầu như **luôn** (kube-proxy, CNI là DaemonSet) |
| `--delete-emptydir-data` | Khi có Pod dùng `emptyDir` |
| `--force` | Khi có Pod "bare" (không thuộc controller nào) — Pod này sẽ **mất luôn** |
| `--timeout=5m` | Tránh treo vô hạn vì PodDisruptionBudget |

---

## 8. Helm ⭐ (mới trong CKA 2025)

```bash
# Repo
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx
helm search repo nginx --versions           # xem mọi version của chart

# Cài
helm install my-release bitnami/nginx -n web --create-namespace
helm install my-release bitnami/nginx --set replicaCount=3
helm install my-release bitnami/nginx -f values.yaml

# Xem trước KHÔNG cài (rất hay dùng để đọc/sửa)
helm template my-release bitnami/nginx > out.yaml
helm install my-release bitnami/nginx --dry-run --debug

# Quản lý
helm list -A                      # -A = mọi namespace
helm list -n web
helm status my-release -n web
helm get values my-release -n web           # values đang dùng
helm get manifest my-release -n web         # YAML đã render
helm show values bitnami/nginx              # toàn bộ values mặc định

# Nâng cấp / hạ cấp
helm upgrade my-release bitnami/nginx --set replicaCount=5 -n web
helm upgrade --install my-release bitnami/nginx -n web   # idempotent
helm history my-release -n web
helm rollback my-release 1 -n web

# Gỡ
helm uninstall my-release -n web
```

**Dạng bài hay ra:**
- "Cài chart X vào namespace Y với replicaCount=3" → `helm install ... -n Y --set replicaCount=3`
- "Tìm version chart mới nhất của X" → `helm search repo X --versions`
- "Chart này sẽ tạo ra những resource nào?" → `helm template`
- "Nâng cấp release lên version mới" → `helm upgrade --version x.y.z`

---

## 9. Kustomize ⭐ (mới, và **docs bị chặn** → phải thuộc)

Cấu trúc chuẩn:
```text
base/
  ├── kustomization.yaml
  ├── deployment.yaml
  └── service.yaml
overlays/prod/
  ├── kustomization.yaml
  └── replica-patch.yaml
```

**`base/kustomization.yaml`:**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
```

**`overlays/prod/kustomization.yaml` — thuộc lòng các field này:**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: production            # ép namespace cho mọi resource
namePrefix: prod-                # thêm tiền tố tên
nameSuffix: -v2
resources:
  - ../../base

commonLabels:                    # gắn label cho mọi resource
  env: prod
commonAnnotations:
  owner: platform-team

images:                          # đổi image mà không sửa deployment.yaml
  - name: nginx
    newName: nginx
    newTag: "1.27"

replicas:                        # đổi số replica
  - name: my-app
    count: 5

configMapGenerator:
  - name: app-config
    literals:
      - LOG_LEVEL=debug
    files:
      - app.properties
secretGenerator:
  - name: app-secret
    literals:
      - password=s3cr3t

patches:                         # patch tuỳ ý (strategic merge hoặc JSON6902)
  - path: replica-patch.yaml
    target:
      kind: Deployment
      name: my-app
```

**Lệnh:**
```bash
kubectl kustomize overlays/prod            # render ra stdout (KHÔNG apply)
kubectl apply -k overlays/prod             # render + apply
kubectl delete -k overlays/prod
kubectl diff -k overlays/prod
```

> 🔴 `kubectl.docs.kubernetes.io` (docs Kustomize) **không nằm trong danh sách được phép**.
> Hãy gõ tay file `kustomization.yaml` ở trên ít nhất 10 lần trước khi thi.
> Mẹo cứu nguy: `kubectl kustomize --help` trong terminal vẫn dùng được.

---

## 10. CRD & Operator (mới)

```bash
# Xem CRD đã cài
k get crd
k get crd <name> -o yaml
k api-resources --api-group=<group>       # resource mà CRD tạo ra

# Xem schema/field của custom resource
k explain <kind> --recursive
k explain <kind>.spec.<field>

# Custom resource cụ thể
k get <plural-name> -A
k describe <kind> <name> -n <ns>
```

**CRD tối thiểu (đọc hiểu là đủ, hiếm khi phải viết từ đầu):**
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: backups.stable.example.com          # PHẢI là <plural>.<group>
spec:
  group: stable.example.com
  scope: Namespaced                          # hoặc Cluster
  names:
    plural: backups
    singular: backup
    kind: Backup
    shortNames: [bk]
  versions:
    - name: v1
      served: true
      storage: true                          # đúng 1 version có storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                schedule: {type: string}
```

**Operator** = CRD + Controller (Deployment chạy vòng lặp reconcile).
Cài operator trong đề thường là: `k apply -f <operator-bundle.yaml>` hoặc `helm install`.
Sau đó verify: `k get crd`, `k get po -n <operator-ns>`.

---

## 11. Extension Interfaces — CNI / CSI / CRI (mới)

```text
┌─────────────────────────────────────────────────────────────┐
│  kubelet                                                     │
│    ├── CRI  → container runtime  (containerd / CRI-O)        │
│    │         socket: /run/containerd/containerd.sock         │
│    ├── CNI  → mạng cho Pod       (Calico / Flannel / Cilium) │
│    │         config: /etc/cni/net.d/  binary: /opt/cni/bin/  │
│    └── CSI  → storage            (ebs.csi.aws.com, ...)      │
│              driver chạy dạng DaemonSet + Deployment          │
└─────────────────────────────────────────────────────────────┘
```

| Interface | Vai trò | Kiểm tra |
|---|---|---|
| **CRI** | kubelet ↔ runtime | `crictl info`, `crictl ps`, `systemctl status containerd` |
| **CNI** | cấp IP & routing cho Pod | `ls /etc/cni/net.d/`, `ls /opt/cni/bin/`, `k get po -n kube-system \| grep -E 'calico\|flannel\|cilium'` |
| **CSI** | mount volume | `k get csidrivers`, `k get csinodes`, `k get sc` |

**`crictl` — thay `docker` trên node** (rất hay dùng khi apiserver chết):
```bash
crictl ps                       # container đang chạy
crictl ps -a                    # cả container đã chết
crictl pods                     # pod sandbox
crictl logs <container-id>
crictl inspect <container-id>
crictl images
crictl rmi --prune
```
> Config nếu `crictl` báo lỗi endpoint:
> `crictl --runtime-endpoint unix:///run/containerd/containerd.sock ps`
> hoặc ghi vào `/etc/crictl.yaml`.

**Không có CNI thì sao?** → Node `NotReady` với message
`network plugin is not ready: cni config uninitialized`. Đây là câu troubleshooting kinh điển.

---

## 12. Dạng bài hay ra trong domain này

| # | Đề bài | Hướng làm |
|---|---|---|
| 1 | Upgrade control plane từ 1.33 → 1.34, **không** upgrade worker | Mục 4, chỉ phần control plane |
| 2 | Backup etcd ra `/opt/etcd-backup.db`, rồi restore từ `/data/old.db` | Mục 5 |
| 3 | Tạo SA `deploy-bot` + ClusterRole cho phép list deployment mọi ns, bind lại | Mục 2 |
| 4 | Node `NotReady` — tìm và sửa | kubelet status → cgroup driver / CNI / cert |
| 5 | Cài chart `nginx` bằng Helm vào ns `web`, replicaCount=3 | Mục 8 |
| 6 | Sửa `kustomization.yaml` để đổi image tag và số replica | Mục 9 |
| 7 | Join thêm worker vào cluster | `kubeadm token create --print-join-command` |
| 8 | Đổi `--service-node-port-range` của apiserver | Sửa `/etc/kubernetes/manifests/kube-apiserver.yaml`, chờ restart |
| 9 | Tìm CRD nào cung cấp resource `X` và liệt kê instance | `k get crd`, `k api-resources` |
| 10 | Gia hạn chứng chỉ sắp hết hạn | `kubeadm certs check-expiration` → `renew all` |

---

## 13. Bẫy tổng kết

1. **Sửa static pod xong phải CHỜ** — `watch k get po -n kube-system`. Sai YAML → apiserver không lên,
   `kubectl` chết. Cứu: xem log `crictl ps -a` + `crictl logs`, hoặc `/var/log/pods/`.
2. **`kubeadm upgrade apply` chỉ 1 lần, trên cp đầu tiên.** Còn lại `kubeadm upgrade node`.
3. **`apt-mark unhold` trước khi install**, `hold` lại sau.
4. **etcd restore không cần cert**, backup thì cần.
5. **`--pod-network-cidr` phải khớp với CNI** (Flannel mặc định `10.244.0.0/16`).
6. **RBAC: `apiGroups: [""]` là core group** — deployment KHÔNG ở đây.
7. **Helm `-n` không tự tạo namespace** — cần `--create-namespace`.
8. **Kustomize docs bị chặn** — thuộc lòng.
9. **Sau `drain` phải `uncordon`**, nếu không node vẫn `SchedulingDisabled`.
10. **`kubeadm join` token hết hạn sau 24h** — tạo lại bằng `kubeadm token create --print-join-command`.
