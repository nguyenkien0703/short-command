# CKS — 25 bài luyện tập kiểu đề thật

> **Cách dùng:** bấm giờ **7–8 phút/câu**. Làm hết rồi mới xem lời giải.
>
> Cần cluster **kubeadm thật** (nhiều câu đụng `/etc/kubernetes/manifests/`, kubelet config,
> AppArmor, Falco — kind/minikube không đủ). Gợi ý: 2 VM Ubuntu + kubeadm, hoặc Killercoda CKS playground.
>
> 🔴 **Luôn `cp` backup manifest control plane trước khi sửa.**

---

## Đề bài

### Cluster Setup

**Câu 1** *(6%)* — Chạy `kube-bench` trên control plane. Sửa **mọi** FAIL liên quan tới
`kube-apiserver` ở mục 1.2. Đảm bảo apiserver chạy lại bình thường.

**Câu 2** *(5%)* — Trên node `worker-1`, kubelet đang cho phép truy cập ẩn danh và mở
read-only port. Vá cả hai lỗ hổng. Chứng minh port 10255 đã đóng.

**Câu 3** *(6%)* — Trong ns `payment`, áp policy mạng:
- Mặc định chặn toàn bộ ingress và egress
- Cho phép mọi Pod trong ns gọi DNS
- Cho phép Pod `role=frontend` gọi tới Pod `role=backend` trên TCP 8080

**Câu 4** *(4%)* — Chặn mọi Pod trong ns `payment` truy cập cloud metadata endpoint
`169.254.169.254`, nhưng vẫn cho phép truy cập mọi IP khác.

**Câu 5** *(4%)* — Trong `/opt/binaries/` có 3 file (`kubectl`, `kubelet`, `kubeadm`) và file
`/opt/binaries/checksums.txt`. Xác định file nào đã bị chỉnh sửa và ghi tên nó vào `/opt/tampered.txt`.

---

### Cluster Hardening

**Câu 6** *(5%)* — ClusterRole `dev-access` đang có `verbs: ["*"]` trên `resources: ["*"]`.
Thu hẹp lại chỉ còn `get`, `list`, `watch` trên `pods` và `services`. Verify bằng `auth can-i`.

**Câu 7** *(5%)* — Tắt automount ServiceAccount token cho SA `default` ở **mọi** namespace
(trừ `kube-system`). Ngoài ra, sửa Pod `legacy-app` trong ns `payment` để nó cũng không mount token.

**Câu 8** *(5%)* — Tạo ServiceAccount `reader` trong ns `payment` chỉ được `get` và `list`
ConfigMap trong ns đó. Tạo Deployment `reader-app` (image `nginx`) dùng SA này.
Sinh một token thời hạn 30 phút cho SA và lưu vào `/opt/reader-token.txt`.

**Câu 9** *(4%)* — Có một CSR tên `suspicious-user` đang `Pending`. Nó xin quyền group
`system:masters`. Từ chối CSR này.

**Câu 10** *(4%)* — Bật admission plugin `NodeRestriction` trên kube-apiserver
và đảm bảo `--authorization-mode` là `Node,RBAC`.

---

### System Hardening

**Câu 11** *(5%)* — Trên `worker-1` có service lạ đang listen port `9999`.
Tìm và tắt vĩnh viễn (không thể start lại được).

**Câu 12** *(6%)* — Profile AppArmor `k8s-deny-write` có sẵn tại `/etc/apparmor.d/k8s-deny-write`
nhưng chưa được nạp. Nạp nó và tạo Pod `apparmor-app` (image `nginx`) sử dụng profile này.
Chứng minh profile đang có hiệu lực.

**Câu 13** *(5%)* — Tạo Pod `seccomp-app` (image `nginx`) trong ns `payment` dùng
seccomp profile `RuntimeDefault`. Sau đó tạo Pod thứ hai `seccomp-custom` dùng custom profile
`profiles/audit.json` (bạn phải tự tạo file với `defaultAction: SCMP_ACT_LOG`).

**Câu 14** *(4%)* — Pod `over-privileged` trong ns `payment` đang chạy với
`privileged: true`, `runAsUser: 0`, và `capabilities.add: ["SYS_ADMIN"]`.
Sửa nó về cấu hình least-privilege mà vẫn chạy được nginx.

---

### Minimize Microservice Vulnerabilities

**Câu 15** *(6%)* — Bật Pod Security Admission mức `restricted` (enforce + audit + warn)
cho ns `payment`. Sau đó sửa Deployment `catalog` trong ns đó để nó deploy được.

**Câu 16** *(7%)* — Bật encryption at rest cho Secret bằng provider `aescbc`.
File cấu hình đặt tại `/etc/kubernetes/enc/enc.yaml`. Sau đó mã hóa lại toàn bộ Secret đang có.
Chứng minh Secret trong etcd đã được mã hóa.

**Câu 17** *(4%)* — Tạo RuntimeClass `gvisor` với handler `runsc`, và Pod `sandboxed`
(image `nginx`) chạy trong runtime này.

**Câu 18** *(4%)* — Secret `db-cred` trong ns `payment` đang được inject qua biến môi trường.
Đổi sang mount dạng file tại `/etc/secrets` với `defaultMode: 0400`.

**Câu 19** *(5%)* — Tìm mọi Pod trong cluster đang chạy `privileged: true`
hoặc dùng `hostPath`, ghi danh sách `namespace/pod` vào `/opt/risky-pods.txt`.

---

### Supply Chain Security

**Câu 20** *(6%)* — Trong `/opt/images.txt` có 4 image. Dùng Trivy tìm image nào **không có**
CVE mức `CRITICAL`. Ghi tên image đó vào `/opt/safe-image.txt`, và sửa Deployment `web`
trong ns `payment` sang dùng image đó.

**Câu 21** *(6%)* — Bật `ImagePolicyWebhook` trên kube-apiserver:
- File config: `/etc/kubernetes/admission/admission-config.yaml`
- kubeconfig của webhook: `/etc/kubernetes/admission/kubeconf.yaml` (đã có sẵn)
- **Từ chối** Pod nếu webhook không phản hồi

**Câu 22** *(5%)* — Chạy `kubesec` trên `/opt/manifest.yaml`. Sửa manifest cho tới khi
không còn mục `critical` nào.

**Câu 23** *(4%)* — Sinh SBOM (định dạng CycloneDX) cho image `nginx:1.27`
và lưu vào `/opt/nginx-sbom.json`.

---

### Monitoring, Logging & Runtime Security

**Câu 24** *(7%)* — Falco đã cài trên `worker-1`. Sửa rule `Terminal shell in container`
để output đúng định dạng sau (giữ nguyên condition):
```
%evt.time,%container.id,%container.name,%user.name,%proc.cmdline
```
priority đổi thành `WARNING`. Sau đó exec vào một Pod và xác nhận alert đúng format.

**Câu 25** *(7%)* — Bật Kubernetes audit log trên kube-apiserver:
- Policy tại `/etc/kubernetes/audit/policy.yaml`
- Log tại `/var/log/kubernetes/audit/audit.log`
- `secrets` và `configmaps`: chỉ mức `Metadata`
- `pods`: mức `RequestResponse`
- Mọi thứ khác: `Metadata`
- Giữ log 20 ngày, tối đa 5 file backup, mỗi file 50MB

Sau đó tìm trong log xem user nào đã thực hiện `delete` trên resource `pods`.

---
---

## Lời giải

<details>
<summary><b>Câu 1 — kube-bench</b></summary>

```bash
kube-bench run --targets=master | grep -B2 -A10 '\[FAIL\].*1\.2'
```
Đọc phần `== Remediations master ==`, rồi:
```bash
cp /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/api.bak
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```
Các flag thường phải thêm/sửa:
```yaml
    - --anonymous-auth=false
    - --authorization-mode=Node,RBAC
    - --enable-admission-plugins=NodeRestriction
    - --profiling=false
    - --audit-log-path=/var/log/kubernetes/audit.log
    - --audit-log-maxage=30
    - --audit-log-maxbackup=10
    - --audit-log-maxsize=100
    - --service-account-lookup=true
    - --kubelet-certificate-authority=/etc/kubernetes/pki/ca.crt
```
```bash
watch crictl ps | grep apiserver
k get nodes
kube-bench run --targets=master | grep -c '\[FAIL\]'   # phải giảm
```
⚠️ Thêm `--audit-log-path` mà chưa có thư mục → tạo trước: `mkdir -p /var/log/kubernetes`.
Nếu apiserver không lên: `cp /tmp/api.bak /etc/kubernetes/manifests/kube-apiserver.yaml`.
</details>

<details>
<summary><b>Câu 2 — kubelet hardening</b></summary>

```bash
ssh worker-1 ; sudo -i
cp /var/lib/kubelet/config.yaml /tmp/kubelet.bak
vim /var/lib/kubelet/config.yaml
```
```yaml
authentication:
  anonymous:
    enabled: false          # ← sửa
  webhook:
    enabled: true
authorization:
  mode: Webhook             # ← không được AlwaysAllow
readOnlyPort: 0             # ← sửa (hoặc thêm nếu chưa có)
```
```bash
systemctl restart kubelet
systemctl status kubelet

# Chứng minh
curl -s http://localhost:10255/pods ; echo "exit=$?"     # phải connection refused
curl -sk https://localhost:10250/pods -o /dev/null -w '%{http_code}\n'   # 401
exit
k get nodes                 # worker-1 vẫn Ready
```
</details>

<details>
<summary><b>Câu 3 — NetworkPolicy 3 tầng</b></summary>

```yaml
# 1. Default deny
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: default-deny, namespace: payment}
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
---
# 2. Cho phép DNS
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: allow-dns, namespace: payment}
spec:
  podSelector: {}
  policyTypes: [Egress]
  egress:
  - to:
    - namespaceSelector:
        matchLabels: {kubernetes.io/metadata.name: kube-system}
      podSelector:
        matchLabels: {k8s-app: kube-dns}
    ports:
    - {protocol: UDP, port: 53}
    - {protocol: TCP, port: 53}
---
# 3. frontend -> backend:8080  (cần CẢ egress cho frontend LẪN ingress cho backend)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: fe-to-be, namespace: payment}
spec:
  podSelector:
    matchLabels: {role: frontend}
  policyTypes: [Egress]
  egress:
  - to:
    - podSelector:
        matchLabels: {role: backend}
    ports: [{protocol: TCP, port: 8080}]
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: be-from-fe, namespace: payment}
spec:
  podSelector:
    matchLabels: {role: backend}
  policyTypes: [Ingress]
  ingress:
  - from:
    - podSelector:
        matchLabels: {role: frontend}
    ports: [{protocol: TCP, port: 8080}]
```
🔴 **Đã deny cả 2 chiều thì phải mở CẢ egress (bên gọi) LẪN ingress (bên nhận).**
Nhiều người chỉ mở ingress rồi thắc mắc sao vẫn không thông.
</details>

<details>
<summary><b>Câu 4 — Chặn metadata endpoint</b></summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: deny-metadata, namespace: payment}
spec:
  podSelector: {}
  policyTypes: [Egress]
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 169.254.169.254/32
```
**Verify:**
```bash
k run t --image=busybox:1.28 --rm -it --restart=Never -n payment -- \
  wget -qO- --timeout=3 http://169.254.169.254/    # phải timeout
```
⚠️ `except` chỉ dùng được **bên trong `ipBlock`**.
</details>

<details>
<summary><b>Câu 5 — Verify binary checksum</b></summary>

```bash
cd /opt/binaries
cat checksums.txt
sha256sum -c checksums.txt
# kubectl: OK
# kubelet: FAILED          ← đây
# kubeadm: OK

sha256sum -c checksums.txt 2>/dev/null | grep FAILED | cut -d: -f1 > /opt/tampered.txt
cat /opt/tampered.txt
```
Nếu file checksum dùng sha512: `sha512sum -c checksums.txt`.
Nếu format khác chuẩn, so thủ công: `sha256sum kubelet` rồi đối chiếu.
</details>

<details>
<summary><b>Câu 6 — Thu hẹp ClusterRole</b></summary>

```bash
k get clusterrole dev-access -o yaml > /tmp/cr.bak
k edit clusterrole dev-access
```
```yaml
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
```
```bash
# Verify — tìm subject đang bind
k get clusterrolebinding -o wide | grep dev-access

k auth can-i list pods    --as=<user>     # yes
k auth can-i get services --as=<user>     # yes
k auth can-i delete pods  --as=<user>     # no
k auth can-i create pods  --as=<user>     # no
k auth can-i --list --as=<user>
```
</details>

<details>
<summary><b>Câu 7 — Tắt automount SA token</b></summary>

```bash
# Mọi namespace trừ kube-system
for ns in $(k get ns -o jsonpath='{.items[*].metadata.name}'); do
  [ "$ns" = "kube-system" ] && continue
  k patch sa default -n $ns -p '{"automountServiceAccountToken": false}'
done

# Verify
k get sa default -A -o custom-columns='NS:.metadata.namespace,AUTOMOUNT:.automountServiceAccountToken'

# Pod legacy-app: sửa spec (Pod không patch field này được → tạo lại)
k get po legacy-app -n payment -o yaml > /tmp/legacy.yaml
vim /tmp/legacy.yaml
#   spec:
#     automountServiceAccountToken: false
#   (xóa metadata.uid, resourceVersion, creationTimestamp, status)
k delete po legacy-app -n payment
k apply -f /tmp/legacy.yaml

# Verify
k exec legacy-app -n payment -- ls /var/run/secrets/kubernetes.io/serviceaccount 2>&1
# → No such file or directory
```
🔴 **Setting ở Pod thắng ở SA.** Tắt ở SA không ảnh hưởng Pod đã khai `automount: true`.
</details>

<details>
<summary><b>Câu 8 — SA least privilege + token</b></summary>

```bash
k create sa reader -n payment
k create role cm-reader --verb=get,list --resource=configmaps -n payment
k create rolebinding cm-reader-bind --role=cm-reader --serviceaccount=payment:reader -n payment

k create deploy reader-app --image=nginx -n payment
k set serviceaccount deploy/reader-app reader -n payment

k create token reader -n payment --duration=30m > /opt/reader-token.txt

# Verify
k auth can-i get configmaps --as=system:serviceaccount:payment:reader -n payment   # yes
k auth can-i get secrets    --as=system:serviceaccount:payment:reader -n payment   # no
k get deploy reader-app -n payment -o jsonpath='{.spec.template.spec.serviceAccountName}'
```
</details>

<details>
<summary><b>Câu 9 — Deny CSR</b></summary>

```bash
k get csr
k get csr suspicious-user -o yaml
# Kiểm tra: request base64 -> openssl để xem CN/O
k get csr suspicious-user -o jsonpath='{.spec.request}' | base64 -d | openssl req -noout -text | grep Subject

k certificate deny suspicious-user

k get csr suspicious-user
# CONDITION: Denied
```
⚠️ `deny` chứ không phải `delete`. Đề muốn thấy trạng thái `Denied`.
</details>

<details>
<summary><b>Câu 10 — NodeRestriction</b></summary>

```bash
cp /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/api.bak
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```
```yaml
    - --enable-admission-plugins=NodeRestriction
    - --authorization-mode=Node,RBAC
```
Nếu đã có `--enable-admission-plugins` với plugin khác thì **thêm vào cuối, phân cách bằng dấu phẩy**:
```yaml
    - --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,PodSecurity
```
```bash
watch crictl ps | grep apiserver
k get nodes
grep -E 'admission-plugins|authorization-mode' /etc/kubernetes/manifests/kube-apiserver.yaml
```
</details>

<details>
<summary><b>Câu 11 — Tắt service lạ</b></summary>

```bash
ssh worker-1 ; sudo -i

ss -tulpn | grep 9999
# users:(("suspicious",pid=3421,fd=3))

systemctl status 3421              # systemd cho biết unit nào
# hoặc
ps -p 3421 -o pid,ppid,cmd
ls -l /proc/3421/exe
systemctl list-units --type=service --state=running | grep -i suspicious

systemctl stop <unit>
systemctl disable <unit>
systemctl mask <unit>              # ← "không thể start lại được"

# Verify
ss -tulpn | grep 9999              # không còn gì
systemctl start <unit>             # Failed to start: Unit is masked.
```
</details>

<details>
<summary><b>Câu 12 — AppArmor</b></summary>

```bash
ssh worker-1 ; sudo -i

apparmor_parser -q /etc/apparmor.d/k8s-deny-write
aa-status | grep k8s-deny-write
exit
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: apparmor-app
spec:
  nodeName: worker-1              # ← đảm bảo chạy đúng node đã nạp profile
  securityContext:
    appArmorProfile:
      type: Localhost
      localhostProfile: k8s-deny-write     # ← TÊN profile, không phải đường dẫn
  containers:
  - name: nginx
    image: nginx
```
Cách annotation (bản cũ, vẫn hay gặp trong đề):
```yaml
metadata:
  annotations:
    container.apparmor.security.beta.kubernetes.io/nginx: localhost/k8s-deny-write
```
**Chứng minh:**
```bash
k exec apparmor-app -- cat /proc/1/attr/current
# → k8s-deny-write (enforce)

k exec apparmor-app -- touch /tmp/x
# → touch: cannot touch '/tmp/x': Permission denied
```
🔴 Chưa nạp profile → Pod `Blocked` / `CreateContainerError`.
</details>

<details>
<summary><b>Câu 13 — seccomp</b></summary>

```yaml
# Pod 1 — RuntimeDefault
apiVersion: v1
kind: Pod
metadata: {name: seccomp-app, namespace: payment}
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  containers:
  - {name: nginx, image: nginx}
```
```bash
# Pod 2 — tạo custom profile TRÊN NODE trước
ssh worker-1 ; sudo -i
mkdir -p /var/lib/kubelet/seccomp/profiles
cat <<'EOF' > /var/lib/kubelet/seccomp/profiles/audit.json
{
  "defaultAction": "SCMP_ACT_LOG"
}
EOF
exit
```
```yaml
apiVersion: v1
kind: Pod
metadata: {name: seccomp-custom, namespace: payment}
spec:
  nodeName: worker-1
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/audit.json    # ← TƯƠNG ĐỐI với /var/lib/kubelet/seccomp/
  containers:
  - {name: nginx, image: nginx}
```
**Verify:**
```bash
k exec seccomp-app -n payment -- grep Seccomp /proc/1/status
# Seccomp: 2   (filtered)
```
</details>

<details>
<summary><b>Câu 14 — Least privilege container</b></summary>

```bash
k get po over-privileged -n payment -o yaml > /tmp/op.yaml
vim /tmp/op.yaml
```
```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 101              # nginx user trong image nginx official
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: nginx
    image: nginxinc/nginx-unprivileged   # image nginx chạy được non-root
    securityContext:
      privileged: false
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
    volumeMounts:                        # cần vì readOnlyRootFilesystem
    - {name: tmp, mountPath: /tmp}
    - {name: cache, mountPath: /var/cache/nginx}
    - {name: run, mountPath: /var/run}
  volumes:
  - {name: tmp, emptyDir: {}}
  - {name: cache, emptyDir: {}}
  - {name: run, emptyDir: {}}
```
```bash
k delete po over-privileged -n payment
k apply -f /tmp/op.yaml
k get po over-privileged -n payment      # Running
```
💡 nginx official cần bind port 80 (< 1024) → hoặc dùng `nginxinc/nginx-unprivileged`
(listen 8080), hoặc `add: ["NET_BIND_SERVICE"]`. Nếu đề chỉ nói "vẫn chạy được",
giữ image gốc + `add: ["NET_BIND_SERVICE"]`, `runAsUser: 0` bỏ, `runAsNonRoot` không đặt.
</details>

<details>
<summary><b>Câu 15 — Pod Security Admission</b></summary>

```bash
k label ns payment \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/warn=restricted --overwrite

k get ns payment --show-labels
```
Deployment `catalog` sẽ bị chặn. Xem lý do:
```bash
k rollout restart deploy catalog -n payment
k get events -n payment | grep -i forbidden
k describe rs -n payment | grep -A5 -i 'violates PodSecurity'
```
Sửa:
```bash
k patch deploy catalog -n payment --type=merge -p '
spec:
  template:
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: nginx
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop: ["ALL"]
'
k rollout status deploy catalog -n payment
k get po -n payment
```
⚠️ `patch --type=merge` trên list `containers` sẽ **thay thế cả list** — an toàn hơn là
`k edit deploy catalog -n payment`.
</details>

<details>
<summary><b>Câu 16 — Encryption at rest</b></summary>

```bash
mkdir -p /etc/kubernetes/enc
head -c 32 /dev/urandom | base64        # copy chuỗi này

cat <<EOF > /etc/kubernetes/enc/enc.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources:
  - secrets
  providers:
  - aescbc:
      keys:
      - name: key1
        secret: <PASTE_BASE64_KEY>
  - identity: {}
EOF
chmod 600 /etc/kubernetes/enc/enc.yaml

cp /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/api.bak
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```
```yaml
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
```bash
watch crictl ps | grep apiserver
k get nodes

# Mã hóa lại Secret đã có
k get secrets -A -o json | k replace -f -

# CHỨNG MINH
ETCDCTL_API=3 etcdctl get /registry/secrets/default/<secret-name> \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key | hexdump -C | head
# → phải thấy "k8s:enc:aescbc:v1:key1" và data lộn xộn
```
🔴 **Quên mount volume = apiserver không lên.** Đây là lỗi phổ biến nhất của câu này.
🔴 `identity` phải nằm **sau** `aescbc`, nếu đặt trước thì không mã hóa gì.
</details>

<details>
<summary><b>Câu 17 — RuntimeClass gVisor</b></summary>

```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata: {name: gvisor}
handler: runsc
---
apiVersion: v1
kind: Pod
metadata: {name: sandboxed}
spec:
  runtimeClassName: gvisor
  containers:
  - {name: nginx, image: nginx}
```
```bash
k apply -f gvisor.yaml
k get runtimeclass
k get po sandboxed

# Verify
k exec sandboxed -- dmesg | head -3        # → "Starting gVisor..."
k exec sandboxed -- uname -r                # kernel khác host
```
⚠️ Nếu `CreateContainerError`: handler `runsc` chưa khai trong
`/etc/containerd/config.toml`. Đề CKS thường cài sẵn.
</details>

<details>
<summary><b>Câu 18 — Secret env → file</b></summary>

```bash
k get po -n payment -o yaml | grep -B5 db-cred      # tìm Pod đang dùng
k get deploy -n payment -o yaml > /tmp/d.yaml
k edit deploy <name> -n payment
```
Xóa phần `env`/`envFrom` tham chiếu secret, thêm:
```yaml
    volumeMounts:
    - name: secrets
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secrets
    secret:
      secretName: db-cred
      defaultMode: 0400
```
```bash
k rollout status deploy <name> -n payment
k exec deploy/<name> -n payment -- ls -l /etc/secrets
k exec deploy/<name> -n payment -- env | grep -i db     # không còn secret trong env
```
</details>

<details>
<summary><b>Câu 19 — Tìm Pod rủi ro</b></summary>

```bash
{
  # privileged
  k get po -A -o json | jq -r '.items[] |
    select(.spec.containers[]?.securityContext?.privileged == true) |
    .metadata.namespace + "/" + .metadata.name'
  # hostPath
  k get po -A -o json | jq -r '.items[] |
    select(.spec.volumes[]?.hostPath != null) |
    .metadata.namespace + "/" + .metadata.name'
} | sort -u > /opt/risky-pods.txt

cat /opt/risky-pods.txt
```
Không có `jq`? Dùng jsonpath + grep:
```bash
k get po -A -o jsonpath='{range .items[*]}{.metadata.namespace}{"/"}{.metadata.name}{" "}{.spec.containers[*].securityContext.privileged}{" "}{.spec.volumes[*].hostPath.path}{"\n"}{end}' \
  | grep -E 'true|/' | awk '{print $1}' | sort -u > /opt/risky-pods.txt
```
</details>

<details>
<summary><b>Câu 20 — Trivy scan</b></summary>

```bash
cat /opt/images.txt

for img in $(cat /opt/images.txt); do
  n=$(trivy image --severity CRITICAL --quiet --format json "$img" 2>/dev/null \
      | jq '[.Results[]?.Vulnerabilities // [] | length] | add // 0')
  echo "$img -> CRITICAL=$n"
done
```
Cách đơn giản hơn (không cần jq):
```bash
for img in $(cat /opt/images.txt); do
  if trivy image --exit-code 1 --severity CRITICAL --quiet "$img" >/dev/null 2>&1; then
    echo "SACH: $img"
  else
    echo "CO CRITICAL: $img"
  fi
done
```
```bash
echo "<image-sach>" > /opt/safe-image.txt
k set image deploy/web nginx=<image-sach> -n payment
k rollout status deploy/web -n payment
```
⚠️ Không internet → `trivy image --skip-db-update --offline-scan`.
⚠️ `--severity HIGH,CRITICAL` viết liền, **không có space** sau dấu phẩy.
</details>

<details>
<summary><b>Câu 21 — ImagePolicyWebhook</b></summary>

```bash
mkdir -p /etc/kubernetes/admission
cat <<'EOF' > /etc/kubernetes/admission/admission-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: ImagePolicyWebhook
  configuration:
    imagePolicy:
      kubeConfigFile: /etc/kubernetes/admission/kubeconf.yaml
      allowTTL: 50
      denyTTL: 50
      retryBackoff: 500
      defaultAllow: false          # ← "TỪ CHỐI nếu webhook không phản hồi"
EOF

cp /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/api.bak
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```
```yaml
    - --enable-admission-plugins=NodeRestriction,ImagePolicyWebhook
    - --admission-control-config-file=/etc/kubernetes/admission/admission-config.yaml
    volumeMounts:
    - name: admission
      mountPath: /etc/kubernetes/admission
      readOnly: true
  volumes:
  - name: admission
    hostPath:
      path: /etc/kubernetes/admission
      type: DirectoryOrCreate
```
```bash
watch crictl ps | grep apiserver
k get nodes
k run test --image=nginx        # nếu webhook chết → bị từ chối (đúng ý đồ)
```
🔴 **3 việc bắt buộc:** (1) tạo file config, (2) thêm `ImagePolicyWebhook` vào
`--enable-admission-plugins`, (3) mount volume. Thiếu bất kỳ cái nào là fail.
</details>

<details>
<summary><b>Câu 22 — kubesec</b></summary>

```bash
kubesec scan /opt/manifest.yaml
# hoặc
docker run -i kubesec/kubesec:v2 scan /dev/stdin < /opt/manifest.yaml
```
Sửa manifest theo các mục `critical`/`advise`:
```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
  serviceAccountName: my-sa
  containers:
  - name: app
    image: nginx
    securityContext:
      privileged: false                  # bỏ privileged
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      runAsNonRoot: true
      runAsUser: 1000
      capabilities:
        drop: ["ALL"]
    resources:
      limits: {cpu: 500m, memory: 512Mi}
      requests: {cpu: 100m, memory: 128Mi}
```
Cũng bỏ nếu có: `hostNetwork`, `hostPID`, `hostIPC`, `hostPath` volume.
```bash
kubesec scan /opt/manifest.yaml | jq '.[0].score'    # score > 0, critical rỗng
```
</details>

<details>
<summary><b>Câu 23 — SBOM</b></summary>

```bash
trivy image --format cyclonedx -o /opt/nginx-sbom.json nginx:1.27
jq '.bomFormat, .specVersion, (.components | length)' /opt/nginx-sbom.json

# Dùng SBOM để quét CVE
trivy sbom /opt/nginx-sbom.json
```
SPDX nếu đề yêu cầu: `trivy image --format spdx-json -o /opt/nginx-sbom.json nginx:1.27`
</details>

<details>
<summary><b>Câu 24 — Falco rule</b></summary>

```bash
ssh worker-1 ; sudo -i

# 1. Lấy rule gốc để copy condition
grep -n -A20 "rule: Terminal shell in container" /etc/falco/falco_rules.yaml

# 2. Override trong file LOCAL
vim /etc/falco/falco_rules.local.yaml
```
```yaml
- rule: Terminal shell in container
  desc: A shell was used as the entrypoint/exec target in a container
  condition: >
    spawned_process and container
    and shell_procs and proc.tty != 0
    and container_entrypoint
  output: "%evt.time,%container.id,%container.name,%user.name,%proc.cmdline"
  priority: WARNING
```
```bash
# 3. Validate & restart
falco -V /etc/falco/falco_rules.local.yaml
systemctl restart falco
systemctl status falco

# 4. Theo dõi
journalctl -u falco -f
```
Tab khác:
```bash
k exec -it <pod> -- sh
```
Quay lại tab log → phải thấy dòng đúng format.
🔴 Sửa `falco_rules.local.yaml`, **KHÔNG** sửa `falco_rules.yaml`. Tên rule phải trùng chính xác.
</details>

<details>
<summary><b>Câu 25 — Audit log</b></summary>

```bash
mkdir -p /etc/kubernetes/audit /var/log/kubernetes/audit

cat <<'EOF' > /etc/kubernetes/audit/policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
- "RequestReceived"
rules:
- level: Metadata
  resources:
  - group: ""
    resources: ["secrets", "configmaps"]
- level: RequestResponse
  resources:
  - group: ""
    resources: ["pods"]
- level: Metadata
EOF

cp /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/api.bak
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```
```yaml
    - --audit-policy-file=/etc/kubernetes/audit/policy.yaml
    - --audit-log-path=/var/log/kubernetes/audit/audit.log
    - --audit-log-maxage=20
    - --audit-log-maxbackup=5
    - --audit-log-maxsize=50
    volumeMounts:
    - name: audit-policy
      mountPath: /etc/kubernetes/audit
      readOnly: true
    - name: audit-log
      mountPath: /var/log/kubernetes/audit
      readOnly: false                       # ← PHẢI ghi được
  volumes:
  - name: audit-policy
    hostPath: {path: /etc/kubernetes/audit, type: DirectoryOrCreate}
  - name: audit-log
    hostPath: {path: /var/log/kubernetes/audit, type: DirectoryOrCreate}
```
```bash
watch crictl ps | grep apiserver
ls -l /var/log/kubernetes/audit/audit.log
tail -1 /var/log/kubernetes/audit/audit.log | jq

# Tìm ai delete pod
cat /var/log/kubernetes/audit/audit.log | jq -r '
  select(.verb=="delete" and .objectRef.resource=="pods")
  | .user.username + " deleted " + .objectRef.namespace + "/" + (.objectRef.name // "?")'

# Không có jq
grep '"verb":"delete"' /var/log/kubernetes/audit/audit.log | grep '"resource":"pods"'
```
🔴 **Thứ tự rule quan trọng** — rule khớp đầu tiên thắng.
🔴 Mount **cả hai** volume, log volume phải `readOnly: false`.
🔴 `Metadata` cho secrets (không dùng `RequestResponse` — sẽ ghi giá trị secret vào log!).
</details>

---

## Tự chấm

| Số câu đúng | Đánh giá |
|---|---|
| 22–25 | Sẵn sàng thi. Chuyển sang killer.sh. |
| 18–21 | Gần được. Ôn đúng domain sai. |
| 13–17 | Cần thêm 3–4 tuần lab, đặc biệt phần tool ngoài K8s. |
| < 13 | Học lại lý thuyết từng domain trước khi luyện đề. |

**Nếu sai nhiều ở:**
- Câu 1, 10, 16, 21, 25 → yếu **sửa manifest apiserver**. Đây là kỹ năng cốt lõi của CKS,
  luyện tới khi làm được trong 5 phút, không tra docs.
- Câu 12, 13 → yếu **AppArmor/seccomp**. Nhớ: AppArmor cần nạp trước; seccomp đường dẫn tương đối.
- Câu 20, 22, 23 → yếu **supply chain tooling**. Chạy Trivy/kubesec 20 lần cho quen cú pháp.
- Câu 24 → yếu **Falco**. Nhớ file `local.yaml` + restart.
