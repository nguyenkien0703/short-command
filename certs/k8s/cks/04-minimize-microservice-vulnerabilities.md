# CKS — Minimize Microservice Vulnerabilities (20%) 🔴

> Một trong 3 domain nặng nhất. Trọng tâm: **Pod Security Standards** và **securityContext**.

**Nội dung curriculum v1.34:**
- Use appropriate **pod security standards**
- Manage kubernetes **secrets**
- Understand and implement **isolation techniques** (multi-tenancy, sandboxed containers, etc.)
- Implement **Pod-to-Pod encryption** (Cilium, Istio)

---

## 1. Pod Security Standards & Pod Security Admission ⭐⭐⭐

PodSecurityPolicy (PSP) **đã bị xóa khỏi Kubernetes**. Thay thế bằng
**Pod Security Admission (PSA)** — admission controller có sẵn, cấu hình bằng **label trên namespace**.

### 3 mức Pod Security Standards
| Level | Cho phép | Dùng khi |
|---|---|---|
| `privileged` | **Mọi thứ** — không giới hạn | System workload, CNI, monitoring agent |
| `baseline` | Chặn các thứ nguy hiểm rõ ràng: `privileged`, hostNetwork/PID/IPC, hostPath, hostPort, thêm capability ngoài danh sách, đổi AppArmor/SELinux, sysctl không an toàn | Ứng dụng thông thường |
| `restricted` | Baseline **+** bắt buộc: `runAsNonRoot`, `allowPrivilegeEscalation: false`, `capabilities.drop: [ALL]`, `seccompProfile` = RuntimeDefault/Localhost, volume type hạn chế | Workload cần siết chặt |

### 3 chế độ (mode)
| Mode | Hành vi |
|---|---|
| `enforce` | **Từ chối** Pod vi phạm |
| `audit` | Cho tạo, nhưng ghi vào **audit log** |
| `warn` | Cho tạo, nhưng in **cảnh báo** cho user |

### Cấu hình — label trên namespace ⭐
```bash
# Cú pháp: pod-security.kubernetes.io/<MODE>: <LEVEL>
k label ns dev pod-security.kubernetes.io/enforce=restricted
k label ns dev pod-security.kubernetes.io/enforce-version=v1.34
k label ns dev pod-security.kubernetes.io/audit=restricted
k label ns dev pod-security.kubernetes.io/warn=restricted

# Ghi đè label đã có
k label ns dev pod-security.kubernetes.io/enforce=baseline --overwrite

# Xóa label
k label ns dev pod-security.kubernetes.io/enforce-

# Xem
k get ns dev --show-labels
k get ns -o custom-columns='NAME:.metadata.name,ENFORCE:.metadata.labels.pod-security\.kubernetes\.io/enforce'
```

**YAML tương đương:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: v1.34
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

> 💡 Chiến lược triển khai thật: bật `warn` + `audit` ở mức `restricted` trước để xem cái gì vi phạm,
> rồi mới bật `enforce`. Đề thi thường bảo bật thẳng `enforce`.

### Pod pass được `restricted` — mẫu THUỘC LÒNG ⭐
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: restricted-pod
  namespace: dev
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop: ["ALL"]
      runAsNonRoot: true
      seccompProfile:
        type: RuntimeDefault
```

**5 điều kiện bắt buộc của `restricted`:**
1. `runAsNonRoot: true`
2. `allowPrivilegeEscalation: false`
3. `capabilities.drop: ["ALL"]` (chỉ được `add: ["NET_BIND_SERVICE"]`)
4. `seccompProfile.type`: `RuntimeDefault` hoặc `Localhost`
5. Volume chỉ được: `configMap`, `secret`, `emptyDir`, `downwardAPI`, `projected`, `persistentVolumeClaim`, `ephemeral`

Cộng thêm mọi thứ `baseline` cấm: không `privileged`, không `hostNetwork/hostPID/hostIPC`,
không `hostPath`, không `hostPort`.

### Test xem Pod có pass không
```bash
k label ns test pod-security.kubernetes.io/enforce=restricted
k run test --image=nginx -n test
# Error from server (Forbidden): pods "test" is forbidden: violates PodSecurity
# "restricted:latest": allowPrivilegeEscalation != false, unrestricted capabilities,
# runAsNonRoot != true, seccompProfile

# → thông báo lỗi NÓI THẲNG cần sửa gì. Đọc kỹ nó.
```

> ⭐ **Mẹo phòng thi:** cứ apply Pod, đọc message `violates PodSecurity`, sửa đúng những gì nó liệt kê.
> Không cần nhớ hết — chỉ cần biết message ở đâu.

### PSA cấp cluster (AdmissionConfiguration)
```yaml
# /etc/kubernetes/psa/admission-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: PodSecurity
  configuration:
    apiVersion: pod-security.admission.config.k8s.io/v1
    kind: PodSecurityConfiguration
    defaults:
      enforce: "baseline"
      enforce-version: "latest"
      audit: "restricted"
      warn: "restricted"
    exemptions:
      usernames: []
      runtimeClasses: []
      namespaces: [kube-system]
```
```yaml
# kube-apiserver.yaml
- --admission-control-config-file=/etc/kubernetes/psa/admission-config.yaml
# + mount volume cho thư mục /etc/kubernetes/psa
```

---

## 2. securityContext — chi tiết

```yaml
apiVersion: v1
kind: Pod
spec:
  # ===== CẤP POD — áp cho mọi container =====
  securityContext:
    runAsUser: 1000                # UID chạy process
    runAsGroup: 3000               # GID chính
    runAsNonRoot: true             # từ chối chạy nếu image có USER root
    fsGroup: 2000                  # GID sở hữu volume được mount
    fsGroupChangePolicy: OnRootMismatch
    supplementalGroups: [4000]
    seccompProfile:
      type: RuntimeDefault
    appArmorProfile:
      type: RuntimeDefault
    sysctls:
    - name: net.core.somaxconn
      value: "1024"

  containers:
  - name: app
    image: nginx
    # ===== CẤP CONTAINER — ghi đè cấp Pod =====
    securityContext:
      privileged: false                  # true = gần như root trên host
      allowPrivilegeEscalation: false    # chặn setuid/setgid leo quyền
      readOnlyRootFilesystem: true       # / chỉ đọc
      runAsUser: 1000
      runAsNonRoot: true
      capabilities:
        drop: ["ALL"]
        add: ["NET_BIND_SERVICE"]
      procMount: Default
      seLinuxOptions:
        level: "s0:c123,c456"
```

| Field | Cấp Pod | Cấp Container | Ghi chú |
|---|:---:|:---:|---|
| `runAsUser`, `runAsGroup`, `runAsNonRoot` | ✅ | ✅ | Container thắng |
| `fsGroup`, `supplementalGroups`, `sysctls` | ✅ | ❌ | Chỉ ở Pod |
| `privileged`, `allowPrivilegeEscalation` | ❌ | ✅ | Chỉ ở container |
| `readOnlyRootFilesystem`, `capabilities` | ❌ | ✅ | Chỉ ở container |
| `seccompProfile`, `appArmorProfile` | ✅ | ✅ | |

### Những thứ CKS coi là "không an toàn" — phải nhận ra ngay
```yaml
# ❌ Cờ đỏ trong Pod spec
spec:
  hostNetwork: true          # Pod dùng network stack của node
  hostPID: true              # thấy mọi process của host
  hostIPC: true
  containers:
  - securityContext:
      privileged: true       # ← NGUY HIỂM NHẤT
      allowPrivilegeEscalation: true
      runAsUser: 0           # chạy root
      capabilities:
        add: ["SYS_ADMIN"]   # tương đương root
    ports:
    - hostPort: 80           # chiếm port trên node
  volumes:
  - hostPath:
      path: /                # ← mount toàn bộ filesystem node = chiếm node
```

**`readOnlyRootFilesystem: true` làm app hỏng?** Mount `emptyDir` vào chỗ cần ghi:
```yaml
    volumeMounts:
    - {name: tmp, mountPath: /tmp}
    - {name: cache, mountPath: /var/cache/nginx}
    - {name: run, mountPath: /var/run}
  volumes:
  - {name: tmp, emptyDir: {}}
  - {name: cache, emptyDir: {}}
  - {name: run, emptyDir: {}}
```

### Tìm Pod không an toàn trong cluster
```bash
# Pod privileged
k get po -A -o json | jq -r '.items[] |
  select(.spec.containers[].securityContext.privileged == true) |
  .metadata.namespace + "/" + .metadata.name'

# Pod dùng hostPath
k get po -A -o json | jq -r '.items[] |
  select(.spec.volumes[]?.hostPath) |
  .metadata.namespace + "/" + .metadata.name'

# Pod hostNetwork
k get po -A -o json | jq -r '.items[] |
  select(.spec.hostNetwork == true) |
  .metadata.namespace + "/" + .metadata.name'

# Pod chạy root
k get po -A -o json | jq -r '.items[] |
  select(.spec.securityContext.runAsNonRoot != true) |
  .metadata.namespace + "/" + .metadata.name'
```

---

## 3. Quản lý Secret ⭐

### Vấn đề cơ bản
```bash
# Secret KHÔNG mã hóa — chỉ base64
k get secret db -o jsonpath='{.data.password}' | base64 -d

# Trong etcd cũng là plaintext (nếu chưa bật encryption)
ETCDCTL_API=3 etcdctl get /registry/secrets/default/db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key | hexdump -C | head
# → đọc thấy chữ rõ ⇒ CHƯA mã hóa
```

### Encryption at Rest ⭐⭐ (dạng bài rất hay ra)

**Bước 1 — tạo key và file cấu hình:**
```bash
head -c 32 /dev/urandom | base64        # sinh key 32 byte

mkdir -p /etc/kubernetes/enc
cat <<EOF > /etc/kubernetes/enc/enc.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources:
  - secrets
  - configmaps
  providers:
  - aescbc:                             # provider ĐẦU TIÊN dùng để MÃ HÓA
      keys:
      - name: key1
        secret: <BASE64_KEY_32_BYTES>
  - identity: {}                        # để đọc được secret CŨ chưa mã hóa
EOF
chmod 600 /etc/kubernetes/enc/enc.yaml
```

**Bước 2 — cắm vào apiserver:**
```yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml
spec:
  containers:
  - command:
    - kube-apiserver
    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml
    volumeMounts:
    - name: enc
      mountPath: /etc/kubernetes/enc
      readOnly: true
  volumes:
  - name: enc
    hostPath:
      path: /etc/kubernetes/enc
      type: DirectoryOrCreate
```
> 🔴 **Quên mount volume là lỗi số 1** — apiserver sẽ không tìm thấy file và không khởi động được.

**Bước 3 — mã hóa lại các Secret đã có:**
```bash
k get secrets -A -o json | k replace -f -
# hoặc 1 namespace
k get secrets -n default -o json | k replace -f -
```

**Bước 4 — verify:**
```bash
ETCDCTL_API=3 etcdctl get /registry/secrets/default/db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key | hexdump -C | head
# → phải thấy "k8s:enc:aescbc:v1:key1" và data lộn xộn
```

**Provider — so sánh:**
| Provider | Bảo mật | Tốc độ | Ghi chú |
|---|---|---|---|
| `identity` | ❌ Không mã hóa | Nhanh nhất | Mặc định |
| `aescbc` | Tốt | Nhanh | Hay dùng trong đề |
| `secretbox` | Tốt | Nhanh | XSalsa20+Poly1305 |
| `aesgcm` | Tốt nhưng phải xoay key thường xuyên | Nhanh nhất | |
| `kms` | **Tốt nhất** | Chậm hơn | Key ở KMS ngoài (AWS/Vault) |

> Thứ tự provider quan trọng: **cái đầu tiên dùng để mã hóa**, các cái sau chỉ để **giải mã**.
> Muốn *gỡ* mã hóa: đặt `identity` lên đầu rồi `k get secrets -A -o json | k replace -f -`.

### Best practice về Secret
- Không đưa secret vào biến môi trường (`env`) — dễ lộ qua `k describe po`, crash dump, log.
  **Mount dạng file** an toàn hơn.
- `defaultMode: 0400` cho volume secret.
- RBAC: hạn chế `get secrets` triệt để.
- `immutable: true` cho secret không đổi → giảm tải apiserver, chống sửa nhầm.
- Dùng external secret store (Vault, AWS Secrets Manager) cho production.

```yaml
apiVersion: v1
kind: Secret
metadata: {name: db}
type: Opaque
immutable: true
stringData:                    # ← stringData: viết chữ rõ, K8s tự base64
  password: s3cr3t
```

```bash
# Đọc secret nhanh
k get secret db -o jsonpath='{.data.password}' | base64 -d
k get secret db -o go-template='{{.data.password | base64decode}}'

# Xem toàn bộ key
k describe secret db
k get secret db -o json | jq -r '.data | map_values(@base64d)'
```

---

## 4. Isolation & Sandboxed Containers ⭐

### 4.1 Container runtime sandbox — gVisor & Kata

```text
BÌNH THƯỜNG (runc)                SANDBOX (gVisor / Kata)
┌──────────────┐                  ┌──────────────┐
│  Container   │                  │  Container   │
├──────────────┤                  ├──────────────┤
│              │                  │ gVisor Sentry│ ← user-space kernel
│              │                  │ (chặn syscall)│
├──────────────┤                  ├──────────────┤
│ Host Kernel  │ ← syscall trực   │ Host Kernel  │ ← rất ít syscall lọt xuống
└──────────────┘   tiếp = rủi ro  └──────────────┘
```

| | gVisor (`runsc`) | Kata Containers |
|---|---|---|
| Cơ chế | User-space kernel chặn syscall | **Micro-VM** thật cho mỗi Pod |
| Cách ly | Mạnh | Mạnh nhất |
| Overhead | Nhẹ hơn VM | Nặng hơn (boot VM) |
| Tương thích | Một số syscall không hỗ trợ | Gần như hoàn toàn |

**Cấu hình — RuntimeClass:**
```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc            # ← tên handler khai trong containerd config
---
apiVersion: v1
kind: Pod
metadata: {name: sandboxed}
spec:
  runtimeClassName: gvisor      # ← trỏ tới RuntimeClass
  containers:
  - name: app
    image: nginx
```

**Phía containerd (`/etc/containerd/config.toml`):**
```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runsc]
  runtime_type = "io.containerd.runsc.v1"
```
```bash
systemctl restart containerd
```

**Verify:**
```bash
k get runtimeclass
k exec sandboxed -- dmesg | head
# gVisor sẽ hiện: "Starting gVisor..."
k exec sandboxed -- uname -a
# kernel version khác host ⇒ đang trong sandbox
```

> 🔴 Bẫy: RuntimeClass `handler` phải khớp tên runtime trong `containerd config.toml`.
> Không khớp → Pod `CreateContainerError`.

### 4.2 Multi-tenancy — cô lập bằng namespace
```bash
# 1. Namespace riêng cho mỗi tenant
k create ns tenant-a

# 2. ResourceQuota — chặn 1 tenant ăn hết tài nguyên
k create quota tq --hard=cpu=4,memory=8Gi,pods=20 -n tenant-a

# 3. LimitRange — mặc định cho từng Pod
# (xem CKA § 6)

# 4. NetworkPolicy — chặn cross-namespace
# (xem 01-cluster-setup § 1)

# 5. RBAC — mỗi tenant chỉ thấy ns của mình
k create role tenant-admin --verb='*' --resource='*' -n tenant-a
k create rolebinding ta --role=tenant-admin --user=alice -n tenant-a

# 6. Pod Security Admission
k label ns tenant-a pod-security.kubernetes.io/enforce=restricted
```

**Node isolation** — tách tenant sang node riêng:
```bash
k taint node node-tenant-a tenant=a:NoSchedule
k label node node-tenant-a tenant=a
```
```yaml
spec:
  nodeSelector: {tenant: a}
  tolerations:
  - {key: tenant, operator: Equal, value: a, effect: NoSchedule}
```

---

## 5. Pod-to-Pod Encryption (mTLS)

### 5.1 Cilium — mã hóa ở tầng CNI
```bash
# WireGuard (đơn giản hơn IPsec)
cilium install --set encryption.enabled=true --set encryption.type=wireguard

# Kiểm tra
cilium status
kubectl -n kube-system exec ds/cilium -- cilium-dbg status | grep Encryption
# → Encryption: Wireguard   [cilium_wg0 (Pubkey: ..., Port: 51871, Peers: 2)]

kubectl -n kube-system exec ds/cilium -- cilium-dbg encrypt status
```
```yaml
# Qua Helm values
encryption:
  enabled: true
  type: wireguard       # hoặc ipsec
  nodeEncryption: true
```

### 5.2 Istio — mTLS ở tầng service mesh
```yaml
# Bật STRICT mTLS cho toàn mesh
apiVersion: security.istio.io/v1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system      # ns này = áp cho toàn mesh
spec:
  mtls:
    mode: STRICT               # STRICT | PERMISSIVE | DISABLE
---
# Chỉ 1 namespace
apiVersion: security.istio.io/v1
kind: PeerAuthentication
metadata: {name: default, namespace: prod}
spec:
  mtls: {mode: STRICT}
---
# AuthorizationPolicy — ai được gọi ai
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata: {name: api-policy, namespace: prod}
spec:
  selector:
    matchLabels: {app: api}
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/prod/sa/frontend"]
    to:
    - operation:
        methods: ["GET", "POST"]
        paths: ["/api/*"]
```

| Mode | Ý nghĩa |
|---|---|
| `STRICT` | **Chỉ** chấp nhận mTLS |
| `PERMISSIVE` | Chấp nhận cả mTLS lẫn plaintext (dùng khi đang migrate) |
| `DISABLE` | Tắt mTLS |

```bash
# Bật sidecar injection
k label ns prod istio-injection=enabled
k rollout restart deploy -n prod

# Verify
istioctl proxy-status
istioctl x describe pod <pod> -n prod
```

> Trong CKS, phần mTLS thường chỉ hỏi ở mức **cấu hình cơ bản** và **hiểu khái niệm** —
> không bắt dựng Istio từ đầu.

---

## 6. Dạng bài hay ra

| # | Đề bài | Hướng làm |
|---|---|---|
| 1 | Bật PSA `restricted` (enforce) cho ns `prod` | `k label ns` |
| 2 | Pod bị từ chối bởi PSA — sửa cho pass `restricted` | Mục 1, đọc message lỗi |
| 3 | Sửa Pod: bỏ privileged, drop ALL caps, runAsNonRoot | Mục 2 |
| 4 | Bật encryption at rest cho Secret bằng aescbc | Mục 3 — nhớ mount volume |
| 5 | Mã hóa lại toàn bộ Secret đang có | `k get secrets -A -o json \| k replace -f -` |
| 6 | Tạo RuntimeClass `gvisor` và Pod dùng nó | Mục 4.1 |
| 7 | Tìm mọi Pod đang chạy privileged trong cluster, ghi ra file | jq ở mục 2 |
| 8 | Đổi Secret từ `env` sang mount file, `defaultMode: 0400` | Mục 3 |
| 9 | Cô lập tenant: ns + quota + netpol + RBAC + PSA | Mục 4.2 |
| 10 | Bật STRICT mTLS cho namespace | Mục 5.2 |
| 11 | Sửa Pod có `readOnlyRootFilesystem: true` mà vẫn chạy được nginx | Mount emptyDir vào /tmp, /var/cache, /var/run |
| 12 | Đọc giá trị secret và ghi (đã decode) ra file | `base64 -d` |

---

## 7. Bẫy tổng kết

1. **PSP đã bị XÓA** khỏi K8s. Đừng viết PodSecurityPolicy — dùng PSA (label namespace).
2. **Label PSA đặt trên NAMESPACE, không phải Pod.**
3. **`restricted` cần đủ 5 điều kiện** — thiếu 1 là fail. Message lỗi liệt kê đủ, hãy đọc.
4. **Encryption at rest: quên mount volume `/etc/kubernetes/enc` → apiserver không lên.**
5. **`identity` phải nằm SAU provider mã hóa**, nếu đặt trước thì không mã hóa gì cả.
6. **Bật encryption không tự mã hóa Secret cũ** — phải `k replace`.
7. **Secret là base64, KHÔNG phải mã hóa.** Câu khái niệm hay hỏi.
8. **`stringData` viết chữ rõ; `data` phải base64.** Đừng lẫn.
9. **RuntimeClass `handler` phải khớp containerd config.**
10. **`securityContext` cấp container thắng cấp Pod.**
11. **`privileged: true` + `hostPath: /` = chiếm được node.** Nhận ra ngay khi thấy.
12. **`fsGroup` chỉ có ở cấp Pod**, `capabilities` chỉ có ở cấp container.
