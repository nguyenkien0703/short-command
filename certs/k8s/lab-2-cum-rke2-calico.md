# Lộ trình ôn CKA & CKS bám vào 2 cụm thật (RKE2+Cilium eBPF · Calico)

> Dành cho người **đã có cụm production/lab thật**, không dựng lab từ đầu.
> File này thay thế phần "môi trường lab" trong [study-plan.md](./study-plan.md) —
> lộ trình tuần vẫn giữ nguyên, chỉ đổi **luyện ở đâu và luyện thế nào**.

---

## 0. TL;DR — kết luận trước, giải thích sau

| | Phủ được | Ghi chú |
|---|:---:|---|
| **CKA** với 2 cụm hiện có | **~65%** | Thiếu hẳn domain *Cluster Architecture 25%* (kubeadm) + phần control-plane của *Troubleshooting* |
| **CKS** với 2 cụm hiện có | **~70%** | Thiếu các câu sửa flag apiserver theo kiểu kubeadm (audit, encryption, ImagePolicyWebhook) |

**⚠️ Vấn đề lớn nhất: RKE2 không phải kubeadm.**
Nó không chỉ *thiếu*, mà **dạy bạn phản xạ sai** cho ~30% số câu của cả hai kỳ thi.

Trong đề thi bạn sẽ `vim /etc/kubernetes/manifests/kube-apiserver.yaml`.
Trên RKE2, file tương ứng nằm ở `/var/lib/rancher/rke2/agent/pod-manifests/` và **được sinh tự động** —
sửa tay thì rke2-server ghi đè lại. Cách đúng của RKE2 là sửa `/etc/rancher/rke2/config.yaml`.
Hai quy trình **hoàn toàn khác nhau**.

### Việc phải làm ngay
```text
1. Chạy script recon ở §1 trên CẢ HAI cụm → biết chính xác mình đang có gì
2. Nếu cụm Calico là kubeadm  → nó thành "cụm thi", RKE2 thành "cụm tooling". Đủ dùng.
   Nếu cụm Calico KHÔNG phải kubeadm → dựng thêm 1 cụm kubeadm nháp 2 VM (§7). Bắt buộc.
3. Theo lộ trình §8 (CKA) và §9 (CKS) — mỗi tuần đã ghi rõ luyện ở cụm nào
```

**Đừng đụng vào cụm production.** Nếu 2 cụm này đang chạy thật, mọi bài lab phá hoại
(kill kubelet, sửa flag apiserver, drain node, bật default-deny) chỉ làm trên **namespace riêng**
hoặc trên cụm nháp. Xem §10 về ranh giới an toàn.

---

## 1. Bước 0 — Recon 2 cụm (chạy trước khi làm gì khác)

Kế hoạch phía dưới phụ thuộc vào một vài dữ kiện mà chỉ bạn kiểm tra được.
Chạy script này trên **từng cụm** (SSH vào 1 control-plane node), lưu output lại.

```bash
#!/usr/bin/env bash
# recon.sh — chạy bằng root trên control-plane node
echo "================= CLUSTER RECON ================="

echo "--- 1. Bản phân phối K8s ---"
which kubeadm       && echo ">> CO kubeadm  (cụm chuẩn đề thi)" || echo ">> KHONG co kubeadm"
which rke2          && echo ">> RKE2"
which k3s           && echo ">> k3s"
ls /etc/kubernetes/manifests/          2>/dev/null && echo ">> manifest kubeadm CHUAN"
ls /var/lib/rancher/rke2/agent/pod-manifests/ 2>/dev/null && echo ">> manifest RKE2 (sinh tu dong!)"

echo "--- 2. Version ---"
kubectl version -o yaml 2>/dev/null | grep -E 'gitVersion' | head -2
kubectl get nodes -o custom-columns='NAME:.metadata.name,VER:.status.nodeInfo.kubeletVersion,OS:.status.nodeInfo.osImage,RUNTIME:.status.nodeInfo.containerRuntimeVersion'

echo "--- 3. Control plane chạy kiểu gì ---"
kubectl get po -n kube-system -o wide | grep -E 'apiserver|etcd|scheduler|controller' || echo "(khong thay static pod - co the la RKE2/k3s)"
systemctl list-units --type=service --state=running | grep -E 'kubelet|rke2|k3s|containerd'

echo "--- 4. CNI ---"
kubectl get po -n kube-system -o wide | grep -Ei 'cilium|calico|flannel|canal|weave'
ls /etc/cni/net.d/ 2>/dev/null

echo "--- 5. kube-proxy còn không? (Cilium có thể đã thay thế) ---"
kubectl get ds -n kube-system | grep -i proxy || echo ">> KHONG CO kube-proxy => Cilium kubeProxyReplacement"

echo "--- 6. Đường dẫn quan trọng ---"
echo "kubelet root-dir:"; ps aux | grep -o '\-\-root-dir=[^ ]*' | head -1
echo "kubelet config:";   ps aux | grep kubelet | grep -o '\-\-config=[^ ]*' | head -1
echo "containerd sock:";  ls /run/containerd/containerd.sock /run/k3s/containerd/containerd.sock 2>/dev/null
echo "PKI:";              ls -d /etc/kubernetes/pki /var/lib/rancher/rke2/server/tls 2>/dev/null
echo "kubeconfig:";       ls /etc/kubernetes/admin.conf /etc/rancher/rke2/rke2.yaml 2>/dev/null

echo "--- 7. Tính năng bảo mật đang bật ---"
kubectl get ns -o custom-columns='NS:.metadata.name,PSA-ENFORCE:.metadata.labels.pod-security\.kubernetes\.io/enforce'
kubectl get netpol -A 2>/dev/null | head
kubectl api-resources | grep -Ei 'gateway|cilium|calico|kyverno|constraint' | head -20

echo "--- 8. Tool đã có ---"
for t in trivy falco kube-bench kubesec helm cilium calicoctl etcdctl crictl jq; do
  printf '%-12s: %s\n' "$t" "$(command -v $t || echo '-- CHUA CO --')"
done

echo "--- 9. Có metrics-server không (cần cho HPA/k top) ---"
kubectl top nodes >/dev/null 2>&1 && echo ">> metrics-server OK" || echo ">> THIEU metrics-server"
echo "================================================="
```

### Đọc kết quả
| Nếu thấy | Nghĩa là | Hành động |
|---|---|---|
| Có `kubeadm` + `/etc/kubernetes/manifests/` | ✅ Cụm chuẩn đề thi | Đây là **cụm chính** để luyện CKA domain 1 + CKS manifest tasks |
| Chỉ có `rke2` | ⚠️ Lệch đề thi ở phần control plane | Dùng cho workload/tooling; **cần cụm kubeadm nháp** |
| Không có DaemonSet `kube-proxy` | Cilium `kubeProxyReplacement` đang bật | `iptables-save \| grep KUBE-SERVICES` sẽ rỗng — xem §5 |
| `metrics-server` thiếu | `k top` và HPA không chạy | Cài trước khi luyện HPA (CKA 15%) |

> 💡 Lưu output recon vào `certs/k8s/my-clusters.md` (file cá nhân, có thể gitignore)
> để lần sau tra nhanh đường dẫn thay vì mò lại.

---

## 2. Bản đồ — domain nào luyện ở cụm nào

### CKA
| Domain | % | RKE2+Cilium | Calico | Cần kubeadm nháp |
|---|:--:|:--:|:--:|:--:|
| Workloads & Scheduling | 15 | ✅ | ✅ | – |
| Storage | 10 | ✅ | ✅ | – |
| Services & Networking | 20 | 🟡 (netpol ✅, kube-proxy ✗) | ✅ | – |
| **Cluster Architecture** | **25** | ❌ **lệch hoàn toàn** | ✅ nếu kubeadm | **BẮT BUỘC** nếu Calico không phải kubeadm |
| Troubleshooting | 30 | 🟡 (app ✅, control-plane ✗) | ✅ nếu kubeadm | 🟡 phần control-plane |

### CKS
| Domain | % | RKE2+Cilium | Calico | Cần kubeadm nháp |
|---|:--:|:--:|:--:|:--:|
| Cluster Setup | 15 | ✅ (kube-bench có profile RKE2 riêng!) | ✅ | – |
| Cluster Hardening | 15 | 🟡 (RBAC/SA ✅, sửa apiserver ✗) | ✅ | 🟡 |
| System Hardening | 10 | ✅ (AppArmor/seccomp ở tầng node) | ✅ | – |
| Minimize Microservice Vuln | 20 | ✅✅ (Cilium WireGuard = mTLS ngon) | ✅ | 🟡 encryption-at-rest |
| Supply Chain | 20 | ✅ (Trivy/kubesec/Kyverno) | ✅ | 🟡 ImagePolicyWebhook |
| Monitoring/Logging/Runtime | 20 | ✅ (Falco + Hubble) | ✅ | 🟡 audit log |

**Đọc bảng:** 🟡 = luyện được **khái niệm** ở cụm đó, nhưng **thao tác trong đề thi khác** →
phải drill lại trên cụm kubeadm.

---

## 3. Bảng dịch RKE2 ↔ kubeadm ⭐ (phần quan trọng nhất file này)

Học thuộc **cột giữa** (kubeadm) để đi thi. Cột phải chỉ để bạn làm việc hằng ngày không bị lẫn.

| Việc | **kubeadm (ĐỀ THI)** | RKE2 |
|---|---|---|
| Static pod manifest | `/etc/kubernetes/manifests/` — **sửa tay, kubelet tự apply** | `/var/lib/rancher/rke2/agent/pod-manifests/` — **SINH TỰ ĐỘNG, sửa tay bị ghi đè** |
| Đổi flag apiserver | `vim kube-apiserver.yaml` → thêm dòng `- --flag=x` | `/etc/rancher/rke2/config.yaml` → `kube-apiserver-arg:` → `systemctl restart rke2-server` |
| Đổi flag kubelet | `/var/lib/kubelet/config.yaml` → `systemctl restart kubelet` | `/etc/rancher/rke2/config.yaml` → `kubelet-arg:` → restart rke2 |
| kubeconfig admin | `/etc/kubernetes/admin.conf` | `/etc/rancher/rke2/rke2.yaml` |
| PKI / cert | `/etc/kubernetes/pki/` | `/var/lib/rancher/rke2/server/tls/` |
| etcd data dir | `/var/lib/etcd` | `/var/lib/rancher/rke2/server/db/etcd` |
| etcd backup | `etcdctl snapshot save` + 3 cờ cert | `rke2 etcd-snapshot save --name X` |
| etcd restore | restore ra dir mới + sửa `hostPath` trong `etcd.yaml` | `rke2 server --cluster-reset --cluster-reset-restore-path=...` |
| Upgrade | `kubeadm upgrade apply` / `kubeadm upgrade node` | Đổi version rồi restart, hoặc `system-upgrade-controller` |
| Service systemd | `kubelet.service` | `rke2-server.service` / `rke2-agent.service` |
| containerd socket | `/run/containerd/containerd.sock` | `/run/k3s/containerd/containerd.sock` |
| `crictl` | có sẵn trong PATH | `/var/lib/rancher/rke2/bin/crictl` + phải set endpoint |
| Addon / manifest tự apply | (không có) | `/var/lib/rancher/rke2/server/manifests/` |
| Pod CIDR mặc định | tùy `--pod-network-cidr` | `10.42.0.0/16` |
| Service CIDR mặc định | `10.96.0.0/12` | `10.43.0.0/16` |
| Bật CIS hardening | sửa từng flag theo kube-bench | `profile: cis` trong `config.yaml` (làm hộ gần hết) |
| Encryption at rest | tự viết `EncryptionConfiguration` + mount volume | `secrets-encryption: true` → `rke2 secrets-encrypt status` |
| Audit log | tự viết policy + 5 flag + 2 volume mount | `audit-policy-file:` trong `config.yaml` |

### `crictl` trên RKE2 — set 1 lần cho đỡ khổ
```bash
export PATH=$PATH:/var/lib/rancher/rke2/bin
export CRI_CONFIG_FILE=/var/lib/rancher/rke2/agent/etc/crictl.yaml
# hoặc
crictl --runtime-endpoint unix:///run/k3s/containerd/containerd.sock ps
```

### Ví dụ cụ thể: "bật audit log" — 2 cách hoàn toàn khác nhau

**kubeadm (cách phải thuộc để thi):**
```bash
mkdir -p /etc/kubernetes/audit /var/log/kubernetes/audit
vim /etc/kubernetes/audit/policy.yaml           # viết Policy
cp /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/bak
vim /etc/kubernetes/manifests/kube-apiserver.yaml
#   + 5 flag --audit-*
#   + 2 volumeMounts, 2 volumes  ← quên là apiserver chết
watch crictl ps | grep apiserver
```

**RKE2 (cách của cụm bạn):**
```yaml
# /etc/rancher/rke2/config.yaml
audit-policy-file: /etc/rancher/rke2/audit-policy.yaml
kube-apiserver-arg:
  - "audit-log-path=/var/lib/rancher/rke2/server/logs/audit.log"
  - "audit-log-maxage=30"
```
```bash
systemctl restart rke2-server
```

> 🔴 **Nhìn thấy sự khác biệt chưa?** Ở RKE2 bạn không bao giờ phải mount volume vào apiserver —
> mà **quên mount volume chính là bẫy số 1 của câu audit log trong CKS**.
> Làm 50 lần trên RKE2 cũng không luyện được phản xạ đó.

---

## 4. 5 nơi RKE2 dạy sai phản xạ (đọc kỹ, đây là chỗ mất điểm)

| # | Trên RKE2 bạn quen | Trong phòng thi phải làm | Hậu quả nếu nhầm |
|---|---|---|---|
| 1 | Sửa `config.yaml` rồi restart service | Sửa YAML manifest, kubelet tự apply | Loay hoay tìm `config.yaml` không tồn tại → mất cả câu |
| 2 | Không bao giờ mount volume cho apiserver | **Phải** thêm `volumeMounts` + `volumes` | apiserver không lên → hỏng luôn các câu sau |
| 3 | `rke2 etcd-snapshot save` một dòng | `etcdctl` + `--cacert/--cert/--key` | Không nhớ 3 cờ cert → mất câu etcd (hay ra) |
| 4 | Upgrade = đổi version + restart | `kubeadm upgrade apply` rồi `upgrade node`, kèm `apt-mark unhold` | Mất trọn câu upgrade (7%) |
| 5 | `profile: cis` bật hardening hộ | Sửa **từng flag** theo remediation của kube-bench | Không biết flag nào ở đâu |

**Cách bù:** mọi bài lab thuộc 5 nhóm trên → làm trên **cụm kubeadm** (§7), tối thiểu **5 lần mỗi bài**
cho tới khi thành phản xạ. RKE2 chỉ dùng để hiểu *tại sao*, không dùng để luyện *thao tác*.

---

## 5. Cụm RKE2 + Cilium eBPF — khai thác thế nào

### 5.1 Điểm lệch cần biết

**kube-proxy có thể đã bị thay thế.** Kiểm tra:
```bash
kubectl get ds -n kube-system | grep -i proxy
kubectl -n kube-system exec ds/cilium -- cilium-dbg status | grep -i KubeProxyReplacement
```
Nếu là `True`:
```bash
iptables-save | grep KUBE-SERVICES        # → RỖNG
```
→ Quy trình debug Service trong [CKA §8](./cka/03-services-networking.md#8-quy-trình-debug-mạng--theo-thứ-tự)
ở bước "kiểm tra iptables" **không áp dụng**. Đề thi dùng kube-proxy chuẩn, nên bước này
phải luyện ở cụm Calico.

Ngược lại, Cilium cho bạn công cụ debug tốt hơn nhiều — dùng để **hiểu bản chất**:
```bash
cilium status --wait
cilium config view
kubectl -n kube-system exec ds/cilium -- cilium-dbg service list      # thay iptables
kubectl -n kube-system exec ds/cilium -- cilium-dbg endpoint list
kubectl -n kube-system exec ds/cilium -- cilium-dbg monitor --type drop   # xem gói bị chặn
cilium connectivity test                                              # test toàn diện
```

### 5.2 NetworkPolicy — luyện ĐÚNG loại

Cilium hỗ trợ **cả hai**:
- `networking.k8s.io/v1` NetworkPolicy ← **CHỈ CÁI NÀY CÓ TRONG ĐỀ THI**
- `cilium.io/v2` CiliumNetworkPolicy / CiliumClusterwideNetworkPolicy ← không thi

```bash
# Luyện đúng: chỉ dùng loại chuẩn
kubectl get netpol -A                    # chuẩn
kubectl get cnp,ccnp -A                  # Cilium CRD — biết là được, đừng luyện
```

**Lợi thế thật của Cilium:** xem policy có ăn không, và gói bị drop vì rule nào:
```bash
# Sau khi apply default-deny, xem gói bị chặn real-time
kubectl -n kube-system exec ds/cilium -- cilium-dbg monitor --type drop

# Hubble (nếu bật) — quan sát luồng theo pod
hubble observe --pod dev/my-pod --last 50
hubble observe --verdict DROPPED --last 50
hubble observe --namespace dev --protocol tcp --port 5432
```
> ⭐ Đây là cách học NetworkPolicy nhanh nhất mà cụm Calico không cho bạn được.
> Viết policy → xem `DROPPED` → hiểu ngay mình sai chỗ nào.

### 5.3 Cilium = món quà cho CKS "Pod-to-Pod encryption"

Curriculum CKS ghi rõ: *"Implement Pod-to-Pod encryption (**Cilium**, Istio)"*.
Cụm của bạn làm được ngay:
```bash
# Kiểm tra đang bật chưa
kubectl -n kube-system exec ds/cilium -- cilium-dbg status | grep -i encryption
kubectl -n kube-system exec ds/cilium -- cilium-dbg encrypt status

# Bật WireGuard (trên RKE2, sửa qua HelmChartConfig)
cat <<'EOF' > /var/lib/rancher/rke2/server/manifests/rke2-cilium-config.yaml
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: rke2-cilium
  namespace: kube-system
spec:
  valuesContent: |-
    encryption:
      enabled: true
      type: wireguard
      nodeEncryption: true
EOF
# RKE2 tự apply file trong server/manifests/

# Verify sau vài phút
kubectl -n kube-system exec ds/cilium -- cilium-dbg status | grep Encryption
# → Encryption: Wireguard [cilium_wg0 (Pubkey: ..., Port: 51871, Peers: N)]
```
⚠️ Bật encryption ảnh hưởng toàn cụm. Nếu là production, chỉ đọc `encrypt status`, đừng bật.

### 5.4 kube-bench có profile riêng cho RKE2
```bash
ls /etc/kube-bench/cfg/                          # xem profile có sẵn
kube-bench run --benchmark rke2-cis-1.24         # thay số theo profile thực tế
kube-bench run --targets master,node,etcd,policies
```
→ Luyện được kỹ năng **đọc `[FAIL]` + remediation** (đúng thứ CKS hỏi),
nhưng phần **sửa** thì theo cách RKE2. Cặp đôi: đọc ở RKE2, sửa ở kubeadm.

---

## 6. Cụm Calico — dùng làm gì

### Nếu là kubeadm → đây là **cụm thi chính** của bạn
Toàn bộ CKA domain 1 (25%), troubleshooting control-plane, và mọi câu CKS sửa manifest
đều luyện ở đây. Chỉ cần đảm bảo có quyền `root` trên control-plane node và **được phép phá**.

### Điểm mạnh cho việc ôn
| Việc | Vì sao Calico hợp |
|---|---|
| NetworkPolicy chuẩn | Calico enforce đúng spec `networking.k8s.io/v1` |
| Debug Service qua iptables | Có kube-proxy thật → `iptables-save \| grep KUBE-SERVICES` chạy như đề thi |
| CNI troubleshooting | `/etc/cni/net.d/`, `/opt/cni/bin/` đúng chuẩn |

```bash
# Nhận diện nhanh
kubectl get po -n kube-system -l k8s-app=calico-node
kubectl get ippool -o wide 2>/dev/null || calicoctl get ippool -o wide
kubectl get felixconfiguration default -o yaml | grep -i policy

# CRD của Calico — biết để KHÔNG nhầm với đề thi
kubectl api-resources | grep -i projectcalico
# GlobalNetworkPolicy, NetworkSet... → KHÔNG có trong CKA/CKS
```

### Bài lab đặc sản: so sánh 2 CNI cùng 1 policy
Apply **cùng một** NetworkPolicy chuẩn lên cả 2 cụm, quan sát khác biệt:
```bash
# Cùng file này, apply cả 2 nơi
cat <<'EOF' > /tmp/deny.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: default-deny, namespace: lab-cert}
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
EOF

kubectl --context=calico apply -f /tmp/deny.yaml
kubectl --context=rke2   apply -f /tmp/deny.yaml

# Quan sát cách mỗi bên enforce
# Calico:
kubectl --context=calico exec -n kube-system ds/calico-node -- iptables-save | grep cali | head
# Cilium:
kubectl --context=rke2 -n kube-system exec ds/cilium -- cilium-dbg monitor --type drop
```
> Hiểu được rằng **spec giống nhau, cơ chế khác nhau** là kiến thức Senior thật —
> và cũng là cái đề thi ngầm kiểm tra khi hỏi "NetworkPolicy không có tác dụng, vì sao?".

---

## 7. Cụm kubeadm nháp — dựng trong 20 phút

**Chỉ bỏ qua bước này nếu recon xác nhận cụm Calico là kubeadm và bạn được phép phá nó.**

Yêu cầu tối thiểu: **2 VM Ubuntu 22.04/24.04**, mỗi máy 2 vCPU / 4GB RAM / 20GB disk.
(1 control-plane + 1 worker là đủ cho **mọi** câu CKA/CKS.)

```bash
# ===== CHẠY TRÊN CẢ 2 MÁY =====
swapoff -a && sed -i '/ swap / s/^/#/' /etc/fstab

cat <<EOF > /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
modprobe overlay && modprobe br_netfilter

cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sysctl --system

# containerd
apt-get update && apt-get install -y containerd apt-transport-https ca-certificates curl gpg
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
systemctl restart containerd && systemctl enable containerd

# repo K8s — ĐỔI v1.34 theo version đề thi hiện hành
K8S_VER=v1.34
curl -fsSL https://pkgs.k8s.io/core:/stable:/${K8S_VER}/deb/Release.key \
  | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/${K8S_VER}/deb/ /" \
  > /etc/apt/sources.list.d/kubernetes.list
apt-get update && apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl

# ===== CHỈ TRÊN CONTROL-PLANE =====
kubeadm init --pod-network-cidr=192.168.0.0/16
mkdir -p $HOME/.kube && cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml
kubeadm token create --print-join-command     # copy lệnh này

# ===== TRÊN WORKER =====
kubeadm join <IP>:6443 --token ... --discovery-token-ca-cert-hash sha256:...
```

> 💡 **Cố tình cài version thấp hơn 1 minor** (vd `v1.33`) để có sẵn bài **upgrade** —
> câu 7% gần như chắc chắn ra trong CKA.

**Chụp snapshot VM sau khi dựng xong.** Mỗi lần phá nát thì restore lại trong 30 giây,
thay vì dựng lại 20 phút. Đây là mẹo tiết kiệm thời gian lớn nhất khi luyện troubleshooting.

**Thay thế miễn phí nếu không có VM:** [Killercoda CKA playground](https://killercoda.com/killer-shell-cka)
và [CKS playground](https://killercoda.com/killer-shell-cks) — cụm kubeadm thật, session 1 giờ,
đủ để làm 1–2 bài lab mỗi lần.

---

## 8. Lộ trình CKA 12 tuần — bám vào cụm

Ký hiệu: **[R]** = cụm RKE2+Cilium · **[C]** = cụm Calico · **[K]** = cụm kubeadm nháp

| Tuần | Domain | Cụm | Bài lab bắt buộc |
|:--:|---|:--:|---|
| **1** | Recon + setup | R, C | Chạy `recon.sh` cả 2 cụm, lưu kết quả.<br>Tạo ns `lab-cert` ở mỗi cụm.<br>Setup alias/vimrc (§10.2).<br>Dựng cụm **[K]** nếu cần. Snapshot VM. |
| **2** | Workloads & Scheduling 15% | R hoặc C | Sinh mọi resource bằng `$do` ×20 lượt.<br>Taint/toleration/affinity trên node thật.<br>Cài metrics-server nếu thiếu → HPA. |
| **3** | Storage 10% | C (và R) | PV/PVC/SC. **[R]** có sẵn `local-path` provisioner → so sánh `WaitForFirstConsumer` vs `Immediate`.<br>Tạo PVC Pending có chủ đích rồi sửa. |
| **4** | Services & Networking (Service/DNS/Ingress) | **C** ⭐ | Debug endpoints rỗng ×5 kịch bản.<br>`iptables-save \| grep KUBE-SERVICES` — **phải làm ở [C]** vì [R] không có.<br>CoreDNS troubleshooting. |
| **5** | NetworkPolicy + Gateway API | **R** ⭐ rồi **C** | Viết 10 policy chuẩn ở **[R]**, dùng `cilium-dbg monitor --type drop` để thấy ngay sai ở đâu.<br>Apply lại đúng 10 policy đó ở **[C]** để chắc không phụ thuộc Cilium.<br>Cài Gateway API + HTTPRoute (1 cụm là đủ). |
| **6** | **Cluster Architecture** — kubeadm/etcd | **[K]** 🔴 | `kubeadm init`/`join` lại từ đầu ×2.<br>etcd backup+restore ×5 (bấm giờ < 5').<br>Restore VM snapshot giữa các lần. |
| **7** | **Cluster Architecture** — upgrade/HA/RBAC | **[K]** 🔴 | Upgrade cp+worker ×3 (< 12').<br>RBAC ở cả 3 cụm (`auth can-i --as=`).<br>Helm + Kustomize: làm ở **[R]** hoặc **[C]** đều được. |
| **8** | Troubleshooting — node & pod | **[K]** + R/C | 15 lab tự phá ([CKA §11](./cka/05-troubleshooting.md#11-lab-tự-tạo-lỗi--cách-luyện-hiệu-quả-nhất)).<br>Lab 1–3, 5, 13 (control-plane) **chỉ làm ở [K]**.<br>Lab 7–12, 15 (pod/app) làm ở **[R]/[C]** trong ns `lab-cert`. |
| **9** | Troubleshooting — mạng & control plane | **[K]** 🔴 | Phá apiserver ×5 kiểu khác nhau, sửa bằng `crictl` + `/var/log/pods/`.<br>Mỗi lần < 8'. Restore snapshot giữa các lần. |
| **10** | Luyện đề vòng 1 | K + mock | KodeKloud mock ×3.<br>Killercoda CKA scenarios. |
| **11** | Luyện đề vòng 2 | K + mock | Làm lại 3 mock.<br>[30 bài trong repo](./cka/practice-questions.md), bấm giờ 120'. |
| **12** | Chốt hạ | killer.sh | Session 1 → đọc solution → luyện lại → Session 2. |

### Bảng phân bổ nhanh
```text
Tuần 1-5  : dùng cụm SẴN CÓ [R]/[C]  → 45% đề (workload, storage, network)
Tuần 6-9  : dùng cụm kubeadm [K]     → 55% đề (cluster arch + troubleshooting)  ← QUAN TRỌNG NHẤT
Tuần 10-12: luyện đề
```

---

## 9. Lộ trình CKS 8 tuần — bám vào cụm

| Tuần | Domain | Cụm | Bài lab bắt buộc |
|:--:|---|:--:|---|
| **1** | Cluster Setup 15% | R ⭐ + C | `kube-bench` ở **cả hai** — [R] dùng profile `rke2-cis-*`, [C] dùng profile chuẩn.<br>Đọc 10 `[FAIL]`, hiểu remediation.<br>**Sửa** thì làm ở **[K]** (kiểu manifest).<br>Chặn `169.254.169.254` bằng netpol ở [R].<br>Ingress TLS. |
| **2** | Cluster Hardening 15% | R, C (RBAC) + **[K]** (apiserver) | Audit RBAC toàn cụm thật: tìm ClusterRoleBinding `cluster-admin` thừa.<br>Tắt `automountServiceAccountToken` cho SA `default` — làm ở ns `lab-cert` trước.<br>Sửa `--anonymous-auth`, `--authorization-mode`, `NodeRestriction` → **[K]**. |
| **3** | System Hardening 10% | R hoặc C (node thật) | AppArmor: viết + `apparmor_parser -q` + gắn vào Pod ×3.<br>seccomp: `RuntimeDefault` + custom profile.<br>⚠️ Đường dẫn seccomp trên RKE2 khác — xác minh bằng `ps aux \| grep root-dir`.<br>Tìm & tắt service lạ, SUID. |
| **4** | Microservice Vuln (1) — PSA | R + C | Bật PSA `restricted` cho ns `lab-cert` ở **cả hai** cụm.<br>Sửa 5 Pod cho pass.<br>Đọc kỹ message `violates PodSecurity`. |
| **5** | Microservice Vuln (2) — Secret/mTLS/sandbox | **R** ⭐ | **Cilium WireGuard** — bài đặc sản của cụm bạn (§5.3).<br>`cilium-dbg encrypt status`.<br>Encryption at rest: hiểu ở [R] (`rke2 secrets-encrypt status`), **drill ở [K]** (viết `EncryptionConfiguration` + mount volume).<br>RuntimeClass gVisor. |
| **6** | Supply Chain 20% | R, C (tooling) + **[K]** | Trivy scan image thật trong cụm bạn — có CVE thật, thú vị hơn lab giả.<br>`kubesec scan` các manifest production của bạn.<br>Kyverno policy chặn registry lạ → [R]/[C].<br>ImagePolicyWebhook → **[K]** (cần sửa apiserver + mount volume). |
| **7** | Runtime 20% | R ⭐ + **[K]** | Falco cài ở [R]/[C], sửa rule + restart ×5.<br>**Hubble** (Cilium) — quan sát luồng, bổ trợ behavioral analytics.<br>Audit log: hiểu ở [R] (`config.yaml`), **drill ở [K]** (policy + 5 flag + 2 volume). |
| **8** | Luyện đề | K + mock | KodeKloud CKS challenges.<br>[25 bài trong repo](./cks/practice-questions.md).<br>killer.sh ×2. |

### Quy tắc vàng cho CKS với cụm RKE2
```text
Cái gì là WORKLOAD (netpol, PSA, securityContext, Trivy, Kyverno, Falco, AppArmor, seccomp)
   → luyện thoải mái trên [R]/[C], giống hệt đề thi

Cái gì là CONTROL PLANE (audit, encryption-at-rest, ImagePolicyWebhook, admission plugin,
   anonymous-auth, authorization-mode)
   → HIỂU ở [R], nhưng THAO TÁC phải drill ở [K]
```

---

## 10. Ranh giới an toàn — đừng phá cụm thật

### 10.1 Quy tắc
| Mức | Được làm ở đâu |
|---|---|
| Tạo/xóa resource trong **ns riêng** (`lab-cert`) | ✅ Cả [R] và [C], kể cả production |
| NetworkPolicy trong ns `lab-cert` | ✅ An toàn (policy là namespaced) |
| PSA label trên ns `lab-cert` | ✅ An toàn |
| Cài Falco / Trivy / Kyverno | 🟡 Chỉ nếu là cụm lab. Kyverno `Enforce` có thể chặn deploy thật |
| `k drain` node | ❌ Chỉ [K] |
| Sửa manifest / config control plane | ❌ Chỉ [K] |
| `systemctl stop kubelet` | ❌ Chỉ [K] |
| Bật Cilium encryption toàn cụm | ❌ Chỉ [K] hoặc cụm lab |
| etcd restore | ❌ Chỉ [K] |

```bash
# Tạo sân chơi riêng, có quota để không ăn hết tài nguyên cụm
kubectl create ns lab-cert
kubectl create quota lab-quota -n lab-cert \
  --hard=cpu=2,memory=4Gi,pods=20,persistentvolumeclaims=5
kubectl label ns lab-cert purpose=cert-practice

# Dọn sạch sau mỗi buổi
kubectl delete all --all -n lab-cert
kubectl delete netpol --all -n lab-cert
```

### 10.2 Quản lý nhiều context — thành phản xạ trước khi thi

Đề thi **mỗi câu một context**. Bạn có sẵn 3 cụm → tận dụng để luyện đúng thói quen đó.

```bash
# Gộp kubeconfig
KUBECONFIG=~/.kube/rke2.yaml:~/.kube/calico.yaml:~/.kube/kubeadm.yaml \
  kubectl config view --flatten > ~/.kube/config

kubectl config get-contexts
kubectl config rename-context <cũ> rke2
kubectl config rename-context <cũ> calico
kubectl config rename-context <cũ> kubeadm

# LUYỆN: chạy lệnh này TRƯỚC MỖI BÀI LAB, như trong phòng thi
kubectl config use-context kubeadm
```

> ⭐ Thói quen "đổi context trước mỗi câu" là thứ **cứu bạn khỏi mất điểm oan số 1** trong đề thi.
> Có 3 cụm là cơ hội hiếm để luyện nó thành phản xạ. Đừng dùng `kubectx` — phòng thi không có.

**Alias + vimrc — set trên cả 3 cụm:**
```bash
cat <<'EOF' >> ~/.bashrc
alias k=kubectl
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"
source <(kubectl completion bash)
complete -o default -F __start_kubectl k
EOF

cat <<'EOF' > ~/.vimrc
set expandtab
set tabstop=2
set shiftwidth=2
set number
set ai
EOF
```

---

## 11. Bài lab đặc sản — chỉ cụm của bạn mới làm được

Những bài này **không có trong khóa học nào**, vì cần đúng combo 2 CNI như bạn đang có.
Làm xong thì hiểu sâu hơn hẳn người chỉ học trên kind.

| # | Bài | Cụm | Học được gì |
|---|---|---|---|
| 1 | Apply **cùng 1** NetworkPolicy chuẩn lên cả 2 cụm, so cách enforce | R + C | Spec chuẩn ≠ cách hiện thực. Trả lời được "vì sao netpol không ăn?" |
| 2 | Debug Service: `iptables` ở [C] vs `cilium-dbg service list` ở [R] | R + C | Bản chất Service là **rule NAT**, không phải process |
| 3 | Viết netpol sai → xem `cilium-dbg monitor --type drop` chỉ ra chỗ sai | R | Học netpol nhanh gấp 3 lần |
| 4 | Bật Cilium WireGuard → `tcpdump` giữa 2 node xem gói đã mã hóa | R | CKS "Pod-to-Pod encryption" — trải nghiệm thật |
| 5 | `kube-bench` profile RKE2 vs profile kubeadm — so danh sách FAIL | R + K | Hiểu vì sao distro hardened sẵn khác cụm vanilla |
| 6 | `rke2 secrets-encrypt status` vs tự viết `EncryptionConfiguration` ở [K] | R + K | Hiểu cái RKE2 làm hộ là cái gì |
| 7 | Trivy scan **toàn bộ image production** trong cụm bạn | R hoặc C | CVE thật, không phải image mẫu |
| 8 | `kubesec scan` các manifest thật của công ty | – | Tìm ra chỗ đội mình đang làm chưa an toàn |
| 9 | Hubble observe luồng traffic thật → viết netpol vừa đủ | R | Cách làm zero-trust đúng bài |
| 10 | So `/etc/kubernetes/manifests/` [K] với `pod-manifests/` [R] | R + K | Nhìn tận mắt điều §3 và §4 nói |

**Bài 8 có giá trị ngoài kỳ thi:** kết quả `kubesec` / `trivy` trên hệ thống thật là
nguyên liệu tốt để viết một **ADR** (dùng [templates/adr-template.md](../../templates/adr-template.md))
hoặc đề xuất cải tiến bảo mật với sếp — đúng triết lý body-of-work của repo này.

---

## 12. Checklist trước khi đặt lịch thi

### Đã làm được trên cụm kubeadm [K] (không tra docs, bấm giờ)
- [ ] etcd backup + restore < 5 phút
- [ ] Upgrade control-plane + worker < 12 phút
- [ ] Sửa flag apiserver + chờ restart + verify < 4 phút
- [ ] Bật audit log (policy + 5 flag + **2 volume mount**) < 6 phút
- [ ] Bật encryption at rest (+ mount volume) < 6 phút
- [ ] Bật ImagePolicyWebhook (config + plugin + mount) < 6 phút
- [ ] Sửa node NotReady (5 nguyên nhân khác nhau) < 5 phút/lần
- [ ] Sửa apiserver chết bằng `crictl` + `/var/log/pods/` < 8 phút

### Đã làm được trên cụm sẵn có [R]/[C]
- [ ] Viết NetworkPolicy AND-selector không tra docs < 3 phút
- [ ] Default-deny + mở DNS < 3 phút
- [ ] Sửa Pod pass PSA `restricted` < 4 phút
- [ ] AppArmor: nạp profile + gắn Pod + verify < 4 phút
- [ ] seccomp custom profile < 3 phút
- [ ] Falco: sửa rule + restart + thấy alert đúng format < 5 phút
- [ ] Trivy scan 4 image, chọn cái sạch < 4 phút
- [ ] `k top` / HPA hoạt động (metrics-server đã cài)

### Thói quen
- [ ] **Luôn** `kubectl config use-context` trước mỗi bài — thành phản xạ
- [ ] **Luôn** `cp` backup manifest trước khi sửa
- [ ] **Luôn** verify sau mỗi bài (`k get`, `k describe`, `auth can-i`)
- [ ] Alias `$do`/`$now` + vimrc gõ được từ trí nhớ trong 45 giây

---

## 13. Tóm tắt một câu

> **Hai cụm của bạn là tài sản lớn cho ~65% đề thi — đặc biệt cụm Cilium là món quà cho
> phần NetworkPolicy và Pod-to-Pod encryption của CKS.
> Nhưng chúng KHÔNG thay được một cụm kubeadm nháp cho phần control plane,
> và RKE2 còn chủ động dạy sai phản xạ ở đó. Dựng thêm 2 VM, snapshot lại, và drill
> phần manifest cho tới khi thành máy.**

---

## Liên quan

- [study-plan.md](./study-plan.md) — lộ trình gốc (file này thay phần "môi trường lab")
- [exam-day-playbook.md](./exam-day-playbook.md) — chiến thuật phòng thi
- [cka/practice-questions.md](./cka/practice-questions.md) · [cks/practice-questions.md](./cks/practice-questions.md)
- [../../k8s-operations-playbook.md](../../k8s-operations-playbook.md) — playbook sự cố thật, hợp với tuần 8–9
