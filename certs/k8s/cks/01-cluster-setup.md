# CKS — Cluster Setup (15%)

**Nội dung curriculum v1.34:**
- Use Network security policies to restrict cluster level access
- Use **CIS benchmark** to review the security configuration of Kubernetes components
  (etcd, kubelet, kubedns, kubeapi)
- Properly set up **Ingress objects with TLS**
- **Protect node metadata and endpoints**
- **Verify platform binaries** before deploying

---

## 1. NetworkPolicy ở tầng cluster

Nền tảng NetworkPolicy đã có ở [CKA § 4](../cka/03-services-networking.md#4-networkpolicy).
Phần này là góc nhìn **bảo mật**: mặc định K8s cho phép mọi Pod nói chuyện với mọi Pod —
đây là vi phạm zero-trust cơ bản nhất.

### Bước 1 — Default deny mọi namespace ⭐
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: <ns>
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
```
Áp cho **từng namespace** (NetworkPolicy là namespaced — không có bản cluster-wide trong K8s core).

```bash
# Áp nhanh cho mọi namespace
for ns in $(k get ns -o jsonpath='{.items[*].metadata.name}'); do
  k apply -n $ns -f default-deny.yaml
done
```

### Bước 2 — Mở DNS (bắt buộc)
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: allow-dns, namespace: <ns>}
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
```

### Bước 3 — Chặn Pod ra Internet nhưng cho phép nội bộ
```yaml
spec:
  podSelector: {}
  policyTypes: [Egress]
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 169.254.169.254/32      # ← chặn cloud metadata (mục 4)
        - 10.0.0.0/8              # chặn mạng nội bộ nếu muốn
  - to:                           # nhưng vẫn cho DNS
    - namespaceSelector:
        matchLabels: {kubernetes.io/metadata.name: kube-system}
    ports: [{protocol: UDP, port: 53}]
```

### Bước 4 — Cô lập namespace (multi-tenancy)
```yaml
# Chỉ cho phép traffic TRONG cùng namespace
spec:
  podSelector: {}
  policyTypes: [Ingress]
  ingress:
  - from:
    - podSelector: {}             # {} trong cùng ns = mọi Pod của ns này
```

> 🔴 **CKS luôn kiểm tra bạn có nhớ mở DNS không.** Deny egress mà quên port 53
> → app "hỏng" mà bạn tưởng do policy sai chỗ khác.

**Verify:**
```bash
k get netpol -A
k describe netpol <name> -n <ns>
k run test --image=busybox:1.28 --rm -it --restart=Never -n <ns> -- \
  sh -c 'nslookup kubernetes.default; wget -qO- --timeout=3 http://target-svc'
```

---

## 2. CIS Benchmark & kube-bench ⭐⭐

CIS Kubernetes Benchmark = bộ ~120 kiểm tra cấu hình an toàn. `kube-bench` là tool chạy nó.

### Chạy kube-bench
```bash
# Cài nhanh (đề thi thường đã cài sẵn hoặc cho file yaml)
kube-bench run --targets=master
kube-bench run --targets=node
kube-bench run --targets=etcd
kube-bench run --targets=policies
kube-bench run --targets=master,node,etcd

# Chỉ xem FAIL (quan trọng nhất trong phòng thi)
kube-bench run --targets=master | grep -A5 '\[FAIL\]'

# Chạy 1 check cụ thể
kube-bench run --targets=master --check 1.2.20

# Chạy dạng Pod (khi không cài trên node)
k apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job-master.yaml
k logs job/kube-bench-master
```

### Đọc output
```text
[FAIL] 1.2.20 Ensure that the --profiling argument is set to false (Automated)

== Remediations master ==
1.2.20 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the below parameter.
--profiling=false
```
> ⭐ **kube-bench nói luôn cách sửa** trong phần `Remediations`.
> Chiến thuật phòng thi: `kube-bench run --targets=master | grep -A10 FAIL`, đọc remediation, làm theo.

### Các FAIL hay ra đề nhất

**kube-apiserver** (`/etc/kubernetes/manifests/kube-apiserver.yaml`):
```yaml
spec:
  containers:
  - command:
    - kube-apiserver
    - --anonymous-auth=false                          # 1.2.1  chặn truy cập ẩn danh
    - --authorization-mode=Node,RBAC                  # 1.2.7  KHÔNG được có AlwaysAllow
    - --enable-admission-plugins=NodeRestriction,...  # 1.2.11
    - --profiling=false                               # 1.2.20
    - --audit-log-path=/var/log/kubernetes/audit.log  # 1.2.21
    - --audit-log-maxage=30                           # 1.2.22
    - --audit-log-maxbackup=10                        # 1.2.23
    - --audit-log-maxsize=100                         # 1.2.24
    - --kubelet-certificate-authority=/etc/kubernetes/pki/ca.crt
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --tls-cert-file=...
    - --tls-private-key-file=...
    - --insecure-port=0                               # (bản cũ; đã bỏ ở K8s mới)
    - --service-account-lookup=true
    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml
```

**kubelet** (`/var/lib/kubelet/config.yaml`):
```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
authentication:
  anonymous:
    enabled: false                 # 4.2.1  ← FAIL hay gặp nhất
  webhook:
    enabled: true                  # 4.2.2
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook                    # 4.2.2  KHÔNG được AlwaysAllow
readOnlyPort: 0                    # 4.2.4  ← tắt port 10255 (không xác thực!)
protectKernelDefaults: true        # 4.2.6
makeIPTablesUtilChains: true       # 4.2.7
tlsCertFile: ...
tlsPrivateKeyFile: ...
rotateCertificates: true           # 4.2.11
```
```bash
systemctl restart kubelet          # bắt buộc sau khi sửa
```

**etcd** (`/etc/kubernetes/manifests/etcd.yaml`):
```yaml
- --client-cert-auth=true          # 2.2
- --peer-client-cert-auth=true     # 2.5
- --auto-tls=false                 # 2.3
- --peer-auto-tls=false            # 2.6
- --cert-file=... --key-file=...
```

**Quyền file** (rất hay ra):
```bash
chmod 600 /etc/kubernetes/manifests/kube-apiserver.yaml
chmod 600 /etc/kubernetes/manifests/etcd.yaml
chmod 600 /etc/kubernetes/admin.conf
chmod 600 /var/lib/kubelet/config.yaml
chmod 700 /var/lib/etcd
chown root:root /etc/kubernetes/manifests/*
chown etcd:etcd /var/lib/etcd        # nếu etcd chạy user riêng
```

> 🔴 Sau khi sửa manifest control plane, **chờ kubelet restart pod**:
> `watch crictl ps | grep apiserver`. Sai YAML → apiserver không lên, `kubectl` chết.
> **Luôn `cp` backup trước khi sửa.**

---

## 3. Ingress với TLS

```bash
# 1. Tạo self-signed cert (đề thường cho sẵn hoặc bắt tự tạo)
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=app.example.com/O=app"

# 2. Tạo Secret type kubernetes.io/tls
k create secret tls app-tls --cert=tls.crt --key=tls.key -n dev

# Xem lại
k get secret app-tls -n dev -o yaml       # có 2 key: tls.crt, tls.key
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
  namespace: dev
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"          # ép HTTP → HTTPS
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"     # TLS tới cả backend
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.example.com                # ← PHẢI khớp với host ở rules
    secretName: app-tls              # ← Secret cùng namespace với Ingress
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service: {name: web-svc, port: {number: 80}}
```

**Verify:**
```bash
k get ingress -n dev
curl -kv https://app.example.com --resolve app.example.com:443:<INGRESS_IP>
openssl s_client -connect app.example.com:443 -servername app.example.com </dev/null 2>/dev/null | openssl x509 -noout -subject -dates
```

| Bẫy | Chi tiết |
|---|---|
| Secret phải **cùng namespace** với Ingress | Không cross-namespace được |
| Secret phải đúng type `kubernetes.io/tls` | Tạo bằng `k create secret tls`, không phải `generic` |
| `tls.hosts` phải khớp `rules.host` | Lệch → trả cert mặc định (fake certificate) |
| Chỉ có `tls:` mà không có `ssl-redirect` | HTTP vẫn vào được — CKS coi là chưa an toàn |

---

## 4. Bảo vệ node metadata & endpoints ⭐

### Cloud metadata endpoint — `169.254.169.254`
Trên AWS/GCP/Azure, mọi Pod theo mặc định gọi được endpoint này và
**lấy được IAM credential của node** → leo thang đặc quyền toàn bộ tài khoản cloud.
Đây là lỗ hổng CKS hỏi rất nhiều.

**Chặn bằng NetworkPolicy:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: deny-metadata, namespace: dev}
spec:
  podSelector: {}
  policyTypes: [Egress]
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 169.254.169.254/32       # ← điểm mấu chốt
```

**Chỉ cho phép 1 Pod cụ thể truy cập metadata:**
```yaml
# Policy 1: deny cho tất cả
spec:
  podSelector: {}
  policyTypes: [Egress]
  egress:
  - to: [{ipBlock: {cidr: 0.0.0.0/0, except: [169.254.169.254/32]}}]
---
# Policy 2: allow riêng cho Pod có label
spec:
  podSelector:
    matchLabels: {role: metadata-accessor}
  policyTypes: [Egress]
  egress:
  - to: [{ipBlock: {cidr: 169.254.169.254/32}}]
```

### kubelet read-only port `10255`
Port này **không xác thực** — ai gọi được là đọc được toàn bộ thông tin Pod trên node.
```bash
curl http://<node-ip>:10255/pods          # nếu trả về JSON ⇒ LỖ HỔNG
```
**Tắt:**
```yaml
# /var/lib/kubelet/config.yaml
readOnlyPort: 0
```
```bash
systemctl restart kubelet
curl http://<node-ip>:10255/pods          # phải connection refused
```

### kubelet API port `10250`
Port này **có** xác thực nhưng phải cấu hình đúng:
```yaml
authentication:
  anonymous:
    enabled: false          # nếu true → ai cũng exec vào container được!
  webhook:
    enabled: true
authorization:
  mode: Webhook             # KHÔNG được AlwaysAllow
```
**Test lỗ hổng:**
```bash
curl -k https://<node-ip>:10250/pods                  # 401 = an toàn, 200 = LỖ HỔNG
curl -k https://<node-ip>:10250/runningpods/
```

### etcd port `2379`
```bash
# Chỉ được truy cập từ localhost / control plane, có client cert
curl -k https://<node-ip>:2379/version                # phải fail nếu không có cert
ss -tulpn | grep 2379                                 # nên bind 127.0.0.1
```

### Tổng kết cổng cần khóa
| Port | Component | Rủi ro nếu mở |
|---|---|---|
| `169.254.169.254` | Cloud metadata | Lấy IAM credential |
| `10250` | kubelet API | Exec vào mọi container |
| `10255` | kubelet read-only | Đọc thông tin Pod, env, không cần auth |
| `2379/2380` | etcd | Đọc **mọi Secret** dưới dạng plaintext |
| `6443` | apiserver | Cần, nhưng phải `anonymous-auth=false` |
| `10256` | kube-proxy healthz | Thông tin nội bộ |

---

## 5. Verify platform binaries ⭐

Trước khi cài `kubectl`/`kubeadm`/`kubelet`, phải kiểm tra checksum để chắc binary không bị chèn mã độc.

```bash
# 1. Tải binary + file checksum chính chủ
VER=v1.34.1
curl -LO "https://dl.k8s.io/release/${VER}/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/${VER}/bin/linux/amd64/kubectl.sha256"

# 2. Cách 1 — so tự động
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
# → "kubectl: OK"    ⇒ hợp lệ
# → "kubectl: FAILED" ⇒ binary đã bị thay đổi, KHÔNG cài

# 3. Cách 2 — so thủ công
sha256sum kubectl
cat kubectl.sha256
# hai chuỗi phải giống hệt nhau
```

**Kiểm tra binary đang chạy trên node:**
```bash
which kubelet
sha512sum /usr/bin/kubelet
sha256sum $(which kubectl)

# So với bản chính thức
curl -sL "https://dl.k8s.io/$(kubectl version --client -o json | jq -r .clientVersion.gitVersion)/bin/linux/amd64/kubectl.sha256"
```

**Kiểm tra image trong Pod:**
```bash
# Digest thật của image đang chạy
k get po <pod> -o jsonpath='{.status.containerStatuses[*].imageID}'
crictl inspecti <image> | grep -i digest
```

> Dạng đề: *"Có 3 file binary trong `/opt/`. Xác định file nào đã bị sửa đổi so với checksum
> trong `/opt/checksums.txt` và xóa nó đi."*
> ```bash
> cd /opt && sha256sum -c checksums.txt
> ```

---

## 6. Dạng bài hay ra

| # | Đề bài | Hướng làm |
|---|---|---|
| 1 | Chạy kube-bench, sửa mọi FAIL của kube-apiserver | Mục 2 — đọc `Remediations` |
| 2 | Tắt anonymous auth cho kubelet trên node01 | `/var/lib/kubelet/config.yaml` + restart |
| 3 | Tắt `readOnlyPort` của kubelet | `readOnlyPort: 0` |
| 4 | Chặn mọi Pod trong ns `dev` truy cập `169.254.169.254` | Mục 4, NetworkPolicy `except` |
| 5 | Tạo Ingress TLS cho `app.example.com` với secret có sẵn | Mục 3 |
| 6 | Áp default-deny cho ns `prod`, chỉ mở DNS + 1 service | Mục 1 |
| 7 | Đổi `--authorization-mode` từ `AlwaysAllow` sang `Node,RBAC` | Sửa manifest apiserver |
| 8 | Sửa quyền file manifest về 600, owner root:root | `chmod`/`chown` |
| 9 | Kiểm tra binary `kubectl` trong `/opt` có khớp checksum không | `sha256sum --check` |
| 10 | Bật `--profiling=false` và audit log cho apiserver | Sửa manifest + [06](./06-monitoring-logging-runtime.md) |

---

## 7. Bẫy tổng kết

1. **Sửa manifest control plane phải backup trước** (`cp x.yaml /tmp/`) và **chờ restart**.
2. **Sửa `/var/lib/kubelet/config.yaml` phải `systemctl restart kubelet`** — không tự reload.
3. **Deny egress mà quên DNS** → hỏng mọi thứ.
4. **NetworkPolicy `except` chỉ dùng được bên trong `ipBlock`**, không dùng với podSelector.
5. **Secret TLS phải cùng namespace với Ingress** và đúng type `kubernetes.io/tls`.
6. **`tls.hosts` phải khớp `rules.host`** — lệch thì trả fake certificate.
7. **kube-bench có sẵn phần Remediation** — đừng tự nghĩ, đọc nó.
8. **`readOnlyPort: 0`** ≠ xóa dòng đó (mặc định là 10255 ở một số bản) — đặt rõ `0`.
9. **`--anonymous-auth=false` cho cả apiserver LẪN kubelet** — hai chỗ khác nhau.
10. **NetworkPolicy chỉ hoạt động nếu CNI hỗ trợ** (Calico/Cilium).
