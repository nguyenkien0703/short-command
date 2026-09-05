# CKS — Cluster Hardening (15%)

**Nội dung curriculum v1.34:**
- Use Role Based Access Controls to minimize exposure
- Exercise caution in using service accounts e.g. **disable defaults**, minimize permissions
  on newly created ones
- **Restrict access to Kubernetes API**
- **Upgrade Kubernetes** to avoid vulnerabilities

---

## 1. RBAC theo tư duy bảo mật

Nền tảng RBAC đã có ở [CKA § 2](../cka/01-cluster-architecture.md#2-rbac--chắc-chắn-có-trong-đề).
CKS hỏi khác: **không phải "tạo role" mà là "role này quá rộng, thu hẹp lại"**.

### Audit RBAC — tìm quyền thừa
```bash
# Ai có quyền gì
k auth can-i --list --as=system:serviceaccount:dev:mysa -n dev
k auth can-i '*' '*' --as=jane
k auth can-i create pods --as=jane -n dev

# Tìm ClusterRoleBinding gắn với cluster-admin (nguy hiểm nhất)
k get clusterrolebinding -o json | \
  jq -r '.items[] | select(.roleRef.name=="cluster-admin") | .metadata.name + " -> " + (.subjects // [] | map(.kind+"/"+.name) | join(","))'

# Cách không cần jq
k get clusterrolebinding -o wide | grep cluster-admin

# Tìm Role có wildcard
k get role,clusterrole -A -o json | \
  jq -r '.items[] | select(.rules[]?.verbs[]? == "*") | .kind + "/" + .metadata.name'

# Xem chi tiết 1 role
k describe clusterrole edit
k get role <name> -n <ns> -o yaml
```

### Nguyên tắc least privilege
```yaml
# ❌ QUÁ RỘNG
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]

# ❌ Vẫn rộng
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["*"]

# ✅ Tối thiểu
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
  resourceNames: ["my-app"]            # giới hạn đúng object cần
```

### Verb nào nguy hiểm
| Verb / Resource | Rủi ro |
|---|---|
| `create pods` | Có thể tạo Pod privileged, mount hostPath `/` → chiếm node |
| `create pods/exec` | Vào shell mọi container |
| `get secrets` | Đọc mọi credential |
| `create serviceaccounts/token` | Sinh token cho SA khác → mạo danh |
| `escalate` (trên roles) | **Tự nâng quyền cho chính mình** |
| `bind` (trên rolebindings) | Gán role bất kỳ cho mình |
| `impersonate` | Giả danh user/group khác |
| `patch nodes` | Đổi label/taint, hút Pod về node bị chiếm |
| `create` trên `certificatesigningrequests/approval` | Tự cấp cert client → thành admin |

> 🔴 **Quyền `create pods` gần như tương đương root trên node** nếu không có
> Pod Security Standards chặn privileged. Xem [04](./04-minimize-microservice-vulnerabilities.md).

### Thu hẹp một ClusterRole quá rộng — quy trình
```bash
# 1. Xem role hiện tại
k get clusterrole <name> -o yaml > /tmp/role.yaml

# 2. Xem ai đang dùng
k get clusterrolebinding -o json | jq -r ".items[] | select(.roleRef.name==\"<name>\") | .metadata.name"

# 3. Sửa rules cho hẹp lại
k edit clusterrole <name>

# 4. Verify: quyền cần thiết còn, quyền thừa mất
k auth can-i get pods    --as=system:serviceaccount:dev:mysa   # yes
k auth can-i delete pods --as=system:serviceaccount:dev:mysa   # no
```

### Role vs ClusterRole — chọn cái nào
```text
Resource namespaced + chỉ 1 ns          → Role + RoleBinding
Resource namespaced + nhiều ns cụ thể   → ClusterRole + nhiều RoleBinding (1 mỗi ns)
Resource namespaced + MỌI ns            → ClusterRole + ClusterRoleBinding
Resource cluster-scoped (node, pv, ns)  → ClusterRole + ClusterRoleBinding  (bắt buộc)
```

---

## 2. ServiceAccount ⭐⭐ (trọng tâm CKS)

### Vấn đề với default ServiceAccount
Mọi Pod không khai `serviceAccountName` sẽ dùng SA `default` của namespace,
và **token của nó được mount tự động** vào `/var/run/secrets/kubernetes.io/serviceaccount/`.
Nếu container bị chiếm → kẻ tấn công có ngay token để gọi API.

```bash
# Xem token bên trong Pod
k exec <pod> -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
k exec <pod> -- cat /var/run/secrets/kubernetes.io/serviceaccount/namespace
k exec <pod> -- ls /var/run/secrets/kubernetes.io/serviceaccount/
# → ca.crt  namespace  token

# Thử gọi API từ trong Pod (mô phỏng tấn công)
k exec <pod> -- sh -c '
  TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
  curl -sk -H "Authorization: Bearer $TOKEN" \
    https://kubernetes.default.svc/api/v1/namespaces/default/pods'
```

### Cách 1 — Tắt automount ở cấp ServiceAccount ⭐
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: default              # hoặc SA bất kỳ
  namespace: dev
automountServiceAccountToken: false
```
```bash
k patch sa default -n dev -p '{"automountServiceAccountToken": false}'
```

### Cách 2 — Tắt ở cấp Pod (ưu tiên cao hơn SA)
```yaml
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: my-sa
  automountServiceAccountToken: false     # ← ghi đè setting của SA
  containers: [...]
```

> **Thứ tự ưu tiên: Pod > ServiceAccount.** Đề hay hỏi:
> *"SA có `automount: false` nhưng Pod có `automount: true` — token có được mount không?"*
> → **Có**, vì Pod thắng.

### Cách 3 — Tạo SA riêng với quyền tối thiểu
```bash
k create sa app-sa -n dev
k create role app-role --verb=get,list --resource=configmaps -n dev
k create rolebinding app-bind --role=app-role --serviceaccount=dev:app-sa -n dev

# Gán vào Deployment
k set serviceaccount deploy/web app-sa -n dev
# hoặc trong YAML: spec.template.spec.serviceAccountName: app-sa
```

### Token của ServiceAccount — thay đổi quan trọng
Từ K8s 1.24+, **tạo SA KHÔNG còn tự sinh Secret token vĩnh viễn**.
Token giờ là **bound token**: có thời hạn, gắn với Pod, tự xoay vòng.

```bash
# Tạo token tạm (dùng để test)
k create token app-sa -n dev
k create token app-sa -n dev --duration=1h
k create token app-sa -n dev --bound-object-kind=Pod --bound-object-name=mypod

# Nếu THỰC SỰ cần token vĩnh viễn (không khuyến khích)
cat <<'EOF' | k apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: app-sa-token
  namespace: dev
  annotations:
    kubernetes.io/service-account.name: app-sa
type: kubernetes.io/service-account-token
EOF
k get secret app-sa-token -n dev -o jsonpath='{.data.token}' | base64 -d
```

### Projected token — kiểm soát audience & thời hạn
```yaml
volumes:
- name: sa-token
  projected:
    sources:
    - serviceAccountToken:
        path: token
        expirationSeconds: 3600      # tối thiểu 600
        audience: vault              # token chỉ dùng được với audience này
```

### Checklist ServiceAccount an toàn
- [ ] `automountServiceAccountToken: false` cho SA `default` của **mọi** namespace
- [ ] Mỗi app có SA riêng, không dùng `default`
- [ ] SA chỉ được gán đúng quyền cần thiết
- [ ] Không có SA nào bind vào `cluster-admin`
- [ ] Không tạo Secret token vĩnh viễn trừ khi bắt buộc

```bash
# Áp dụng hàng loạt
for ns in $(k get ns -o jsonpath='{.items[*].metadata.name}'); do
  k patch sa default -n $ns -p '{"automountServiceAccountToken": false}'
done
```

---

## 3. Hạn chế truy cập Kubernetes API ⭐

### 3.1 Tắt anonymous access
```yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml
- --anonymous-auth=false
```
**Test:**
```bash
curl -k https://<cp-ip>:6443/api/v1/namespaces
# 401 Unauthorized ⇒ OK
# Trả về danh sách ⇒ LỖ HỔNG
```

> ⚠️ Tắt anonymous-auth có thể làm `/healthz`, `/livez`, `/readyz` probe fail.
> Nếu apiserver không lên lại, kiểm tra `livenessProbe` trong manifest.

### 3.2 Authorization mode
```yaml
- --authorization-mode=Node,RBAC       # ✅ ĐÚNG
# - --authorization-mode=AlwaysAllow   # ❌ tắt hết kiểm tra quyền
```

| Mode | Ý nghĩa |
|---|---|
| `Node` | Node authorizer — giới hạn kubelet chỉ đọc được resource liên quan Pod trên node đó |
| `RBAC` | Role-based access control |
| `ABAC` | Attribute-based, dựa file policy — cũ |
| `Webhook` | Ủy quyền cho service ngoài |
| `AlwaysAllow` | ❌ **Cho phép tất cả** — CIS FAIL |
| `AlwaysDeny` | Chặn tất cả |

### 3.3 Admission Controllers
```yaml
- --enable-admission-plugins=NodeRestriction,PodSecurity,ServiceAccount,NamespaceLifecycle
- --disable-admission-plugins=...
```

| Plugin | Tác dụng |
|---|---|
| **NodeRestriction** ⭐ | Kubelet chỉ sửa được Node/Pod **của chính nó**. Ngăn node bị chiếm sửa node khác. **CIS bắt buộc.** |
| **PodSecurity** | Thực thi Pod Security Standards (thay PodSecurityPolicy đã bị xóa) |
| **ServiceAccount** | Tự động gán SA |
| **AlwaysPullImages** | Ép pull image mỗi lần → ngăn Pod dùng image cached mà không có quyền pull |
| **ImagePolicyWebhook** | Gọi webhook ngoài duyệt image → [05](./05-supply-chain-security.md) |
| **ValidatingAdmissionWebhook** | OPA Gatekeeper / Kyverno cắm vào đây |
| **EventRateLimit** | Chống DoS bằng event |

**Kiểm tra plugin nào đang bật:**
```bash
grep enable-admission /etc/kubernetes/manifests/kube-apiserver.yaml
k get po -n kube-system kube-apiserver-<node> -o yaml | grep admission
```

### 3.4 Giới hạn network tới apiserver
```bash
# Chỉ bind vào IP nội bộ
- --bind-address=127.0.0.1        # (chỉ khi có LB phía trước)

# Firewall
iptables -A INPUT -p tcp --dport 6443 -s 10.0.0.0/8 -j ACCEPT
iptables -A INPUT -p tcp --dport 6443 -j DROP
ufw allow from 10.0.0.0/8 to any port 6443
ufw deny 6443
```

### 3.5 Kiểm soát người dùng
```bash
# Xem user hiện tại
k auth whoami
k config view --minify

# Tạo user bằng CSR (dạng bài hay ra)
openssl genrsa -out jane.key 2048
openssl req -new -key jane.key -out jane.csr -subj "/CN=jane/O=dev-team"

cat <<EOF | k apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata: {name: jane}
spec:
  request: $(cat jane.csr | base64 -w0)
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400
  usages: [client auth]
EOF

k get csr
k certificate approve jane
k certificate deny jane                 # ← đề hay bảo TỪ CHỐI một CSR đáng ngờ
k get csr jane -o jsonpath='{.status.certificate}' | base64 -d > jane.crt

# Thêm vào kubeconfig
k config set-credentials jane --client-key=jane.key --client-certificate=jane.crt --embed-certs
k config set-context jane-ctx --cluster=kubernetes --user=jane
```
> `CN` = username, `O` = group. Đây là cách K8s ánh xạ cert sang identity.

---

## 4. Upgrade Kubernetes để vá lỗ hổng

Quy trình chi tiết ở [CKA § 4](../cka/01-cluster-architecture.md#4-upgrade-cluster-gần-như-chắc-chắn-có-1-câu).
Góc nhìn CKS bổ sung:

```bash
# Kiểm tra version hiện tại
k version
k get nodes -o custom-columns='NAME:.metadata.name,VERSION:.status.nodeInfo.kubeletVersion'

# Xem CVE của version đang chạy
# → kubernetes.io/docs/reference/issues-security/official-cve-feed/
k get cm -n kube-system kubeadm-config -o yaml | grep kubernetesVersion
```

**Nguyên tắc bảo mật khi upgrade:**
- Không nhảy quá **1 minor version** (1.32 → 1.33 → 1.34).
- Control plane trước, worker sau. **Không bao giờ ngược lại.**
- kubelet có thể **thấp hơn** apiserver tối đa 3 minor version, nhưng **không được cao hơn**.
- Sau upgrade: chạy lại `kube-bench` — flag có thể bị reset về mặc định.
- Gia hạn chứng chỉ: `kubeadm certs check-expiration` → `kubeadm certs renew all`.

**Version skew policy (hay hỏi khái niệm):**
```text
kube-apiserver         : version cao nhất
kube-controller-manager: ≤ apiserver, cách tối đa 1 minor
kube-scheduler         : ≤ apiserver, cách tối đa 1 minor
kubelet                : ≤ apiserver, cách tối đa 3 minor
kube-proxy             : cùng version với kubelet trên node đó
kubectl                : ±1 minor so với apiserver
```

---

## 5. Dạng bài hay ra

| # | Đề bài | Hướng làm |
|---|---|---|
| 1 | Tắt automount token cho SA `default` ở ns `dev` và mọi Pod đang chạy | `k patch sa` + sửa Pod spec |
| 2 | ClusterRole `X` có quyền `*` — thu hẹp còn `get,list` trên `pods` | Mục 1 |
| 3 | Tìm và xóa ClusterRoleBinding gán `cluster-admin` cho SA không cần | `k get clusterrolebinding \| grep cluster-admin` |
| 4 | Tạo SA + Role tối thiểu cho app chỉ đọc ConfigMap | Mục 2 |
| 5 | Sửa apiserver: `anonymous-auth=false`, `authorization-mode=Node,RBAC` | Mục 3 |
| 6 | Bật admission plugin `NodeRestriction` | Sửa `--enable-admission-plugins` |
| 7 | Từ chối (deny) một CSR đáng ngờ | `k certificate deny <name>` |
| 8 | Approve CSR và tạo kubeconfig cho user mới | Mục 3.5 |
| 9 | Tạo token có thời hạn 2h cho SA | `k create token sa --duration=2h` |
| 10 | Kiểm tra Pod nào đang mount SA token và tắt đi | `k get po -o yaml \| grep -B5 serviceaccount` |

**Câu 10 — script hữu ích:**
```bash
k get po -A -o json | jq -r '
  .items[] |
  select(.spec.automountServiceAccountToken != false) |
  .metadata.namespace + "/" + .metadata.name + " -> " + .spec.serviceAccountName'
```

---

## 6. Bẫy tổng kết

1. **`automountServiceAccountToken` ở Pod thắng SA.** Tắt ở SA chưa đủ nếu Pod bật lại.
2. **Tắt automount không áp dụng cho Pod đang chạy** — phải tạo lại Pod/rollout restart.
3. **`k create sa` từ 1.24+ không sinh Secret token** — dùng `k create token`.
4. **`--anonymous-auth=false` có thể làm healthz probe fail** — kiểm tra apiserver lên lại được không.
5. **`AlwaysAllow` trong `authorization-mode` = vô hiệu hóa RBAC hoàn toàn.**
6. **`NodeRestriction` là admission plugin bắt buộc theo CIS.**
7. **`resourceNames` không dùng được với verb `list`, `watch`, `create`** (chỉ get/update/delete/patch).
8. **Verb `escalate` và `bind` cho phép tự nâng quyền** — không bao giờ cấp.
9. **Xóa ClusterRoleBinding hệ thống** (`system:*`) sẽ làm hỏng cluster. Đọc kỹ đề trước khi xóa.
10. **`k auth can-i --list --as=` là cách verify nhanh nhất** — dùng sau mỗi câu RBAC.
