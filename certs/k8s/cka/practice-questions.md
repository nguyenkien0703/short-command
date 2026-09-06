# CKA — 30 bài luyện tập kiểu đề thật

> **Cách dùng:** bấm giờ **7 phút/câu**. Làm hết rồi mới xem lời giải.
> Mỗi câu ghi rõ trọng số như đề thật để bạn tập thói quen ưu tiên.
>
> Cần một cluster: `kind create cluster --config 3-node.yaml` hoặc kubeadm 2 node.
> Một số câu (upgrade, etcd, static pod) **bắt buộc** cluster kubeadm thật, kind không làm được.

---

## Đề bài

### Domain: Workloads & Scheduling

**Câu 1** *(4%)* — Tạo namespace `ecom`. Trong đó tạo Deployment `catalog` dùng image
`nginx:1.27`, 4 replica, container port 80, label `app=catalog,tier=frontend`.

**Câu 2** *(4%)* — Deployment `catalog` đang chạy `nginx:1.27`. Nâng lên `nginx:1.28`,
đợi rollout xong, sau đó rollback về version trước. Ghi số revision hiện tại vào `/opt/rev.txt`.

**Câu 3** *(5%)* — Tạo Pod `logger` trong ns `ecom` có **2 container**:
- `writer`: image `busybox:1.36`, chạy `sh -c 'while true; do date >> /data/out.log; sleep 5; done'`
- `reader`: image `busybox:1.36`, chạy `sh -c 'tail -f /data/out.log'`

Hai container chia sẻ volume `emptyDir` mount tại `/data`.

**Câu 4** *(4%)* — Tạo CronJob `cleanup` trong ns `ecom` chạy mỗi 3 phút, image `busybox:1.36`,
command `sh -c 'echo cleaning'`, giữ tối đa 2 job thành công và 1 job thất bại,
không cho chạy chồng lấn.

**Câu 5** *(4%)* — Tạo ConfigMap `app-cfg` trong ns `ecom` với `LOG_LEVEL=debug` và `MAX_CONN=100`.
Tạo Pod `cfg-app` (image `nginx`) nạp **toàn bộ** ConfigMap thành biến môi trường,
đồng thời mount nó thành file tại `/etc/appcfg`.

**Câu 6** *(5%)* — Node `worker-1` cần dành riêng cho workload GPU:
1. Gắn taint `gpu=true:NoSchedule`
2. Gắn label `hardware=gpu`
3. Tạo Pod `gpu-job` (image `nginx`) **chỉ** chạy được trên node đó

**Câu 7** *(4%)* — Deployment `api` trong ns `ecom` đang có 2 replica. Tạo HPA scale từ 2 đến 10
khi CPU vượt 65%. Nếu HPA báo `<unknown>`, sửa cho đúng.

**Câu 8** *(4%)* — Sửa Deployment `catalog` để có QoS class `Guaranteed`,
với cpu 200m và memory 256Mi.

---

### Domain: Services & Networking

**Câu 9** *(4%)* — Expose Deployment `catalog` (ns `ecom`) qua Service `catalog-svc` kiểu NodePort,
service port 80, target port 80, nodePort 30090.

**Câu 10** *(6%)* — Trong ns `ecom` có Service `broken-svc` và Deployment `broken-app`.
Service không trả về endpoint nào. Tìm nguyên nhân và sửa. **Không** được xóa/tạo lại Service.

**Câu 11** *(7%)* — Trong ns `ecom`, tạo NetworkPolicy `db-policy`:
- Áp lên Pod có label `role=db`
- **Chỉ** cho phép ingress từ Pod có label `role=api` **trong namespace có label `env=prod`**
- Chỉ cho phép cổng TCP 5432
- Chặn toàn bộ egress trừ DNS

**Câu 12** *(4%)* — Tạo Ingress `shop` trong ns `ecom` cho host `shop.example.com`:
- `/api` → service `api-svc` port 8080
- `/` → service `catalog-svc` port 80
- Dùng ingressClass `nginx`

**Câu 13** *(5%)* — Gateway API đã được cài. Tạo `HTTPRoute` tên `canary` trong ns `ecom`,
gắn vào Gateway `main-gw` ở ns `infra`, host `shop.example.com`, chia traffic:
90% tới `catalog-svc:80`, 10% tới `catalog-v2-svc:80`.

**Câu 14** *(4%)* — Pod trong ns `ecom` không resolve được tên service. Chẩn đoán và sửa.
Ghi nguyên nhân vào `/opt/dns-issue.txt`.

---

### Domain: Storage

**Câu 15** *(4%)* — Tạo PersistentVolume `pv-logs`: 3Gi, accessMode `ReadWriteOnce`,
reclaimPolicy `Retain`, storageClassName `slow`, hostPath `/mnt/logs`.

**Câu 16** *(4%)* — Tạo PVC `pvc-logs` trong ns `ecom` xin 2Gi, storageClassName `slow`,
accessMode `ReadWriteOnce`. Tạo Pod `log-app` (image `nginx`) mount PVC đó vào `/var/log/app`.

**Câu 17** *(4%)* — PVC `stuck-pvc` trong ns `ecom` đang `Pending`. Tìm nguyên nhân và sửa.

**Câu 18** *(3%)* — Tạo StorageClass `local-fast` với provisioner `kubernetes.io/no-provisioner`,
`volumeBindingMode: WaitForFirstConsumer`, `allowVolumeExpansion: true`, reclaimPolicy `Delete`.
Đặt nó làm StorageClass mặc định.

---

### Domain: Cluster Architecture

**Câu 19** *(7%)* — Upgrade **chỉ** control-plane node từ version hiện tại lên minor version kế tiếp
(kubeadm, kubelet, kubectl). **Không** upgrade worker node.

**Câu 20** *(7%)* — Backup etcd ra `/opt/etcd-backup.db`. Sau đó restore cluster
từ snapshot `/data/etcd-snapshot-previous.db` vào data dir `/var/lib/etcd-restore`.

**Câu 21** *(5%)* — Tạo ServiceAccount `monitor` trong ns `ecom`. Tạo ClusterRole cho phép
`get`, `list`, `watch` trên `pods`, `nodes`, `services`. Bind để `monitor` có quyền đó
trên **toàn cluster**. Verify bằng `auth can-i`.

**Câu 22** *(4%)* — Node `worker-2` cần bảo trì. Drain nó an toàn (bỏ qua DaemonSet, xóa emptyDir data),
sau đó đưa trở lại cluster.

**Câu 23** *(5%)* — Dùng Helm cài chart `bitnami/nginx` thành release `web-helm`
vào namespace `helm-test` (tạo namespace nếu chưa có), với `replicaCount=2`.
Ghi version của chart đã cài vào `/opt/helm-version.txt`.

**Câu 24** *(5%)* — Trong thư mục `/opt/kustomize-app/` có `base/` và `overlays/prod/`.
Sửa `overlays/prod/kustomization.yaml` để: đặt namespace `prod`, thêm prefix `prod-`,
đổi image tag thành `1.28`, đặt replicas = 4. Apply nó.

**Câu 25** *(4%)* — Sửa kube-apiserver để cho phép NodePort trong dải `25000-32767`.
Đảm bảo apiserver chạy lại bình thường.

---

### Domain: Troubleshooting

**Câu 26** *(7%)* — Node `worker-1` đang `NotReady`. Tìm nguyên nhân, sửa,
và đảm bảo nó vẫn Ready sau khi node reboot.

**Câu 27** *(6%)* — Pod `crash-app` trong ns `ecom` đang `CrashLoopBackOff`.
Tìm nguyên nhân từ log và sửa.

**Câu 28** *(5%)* — Ghi tên của Pod đang dùng **nhiều CPU nhất** trong toàn cluster
vào `/opt/top-cpu-pod.txt` (chỉ tên pod, không có gì khác).

**Câu 29** *(5%)* — Tìm Pod nào trong ns `ecom` có log chứa chuỗi `ERROR_CODE_500`.
Ghi tên Pod vào `/opt/error-pod.txt` và toàn bộ dòng log lỗi vào `/opt/error-lines.txt`.

**Câu 30** *(6%)* — `kubectl` báo `connection refused`. Control plane có vấn đề. Chẩn đoán và sửa.

---
---

## Lời giải

<details>
<summary><b>Câu 1 — Deployment cơ bản</b></summary>

```bash
k create ns ecom
k create deploy catalog --image=nginx:1.27 --replicas=4 --port=80 -n ecom \
  --dry-run=client -o yaml > c1.yaml
vim c1.yaml     # thêm labels vào spec.template.metadata.labels
k apply -f c1.yaml
```
Hoặc nhanh hơn:
```bash
k create deploy catalog --image=nginx:1.27 --replicas=4 --port=80 -n ecom
k label deploy catalog tier=frontend -n ecom
k patch deploy catalog -n ecom --type=merge \
  -p '{"spec":{"template":{"metadata":{"labels":{"app":"catalog","tier":"frontend"}}}}}'
```
**Verify:** `k get deploy,po -n ecom --show-labels`

⚠️ Nhớ label phải ở `spec.template.metadata.labels` (của Pod), không chỉ ở Deployment.
</details>

<details>
<summary><b>Câu 2 — Rolling update & rollback</b></summary>

```bash
k set image deploy/catalog nginx=nginx:1.28 -n ecom
k rollout status deploy/catalog -n ecom
k rollout history deploy/catalog -n ecom
k rollout undo deploy/catalog -n ecom
k rollout status deploy/catalog -n ecom

k rollout history deploy/catalog -n ecom | tail -2
# Ghi revision hiện tại
k get deploy catalog -n ecom -o jsonpath='{.metadata.annotations.deployment\.kubernetes\.io/revision}' > /opt/rev.txt
cat /opt/rev.txt
```
⚠️ Tên container trong `k set image` phải đúng (`k get deploy catalog -n ecom -o jsonpath='{.spec.template.spec.containers[*].name}'`).
</details>

<details>
<summary><b>Câu 3 — Multi-container + emptyDir</b></summary>

```yaml
apiVersion: v1
kind: Pod
metadata: {name: logger, namespace: ecom}
spec:
  containers:
  - name: writer
    image: busybox:1.36
    command: ["sh","-c","while true; do date >> /data/out.log; sleep 5; done"]
    volumeMounts: [{name: data, mountPath: /data}]
  - name: reader
    image: busybox:1.36
    command: ["sh","-c","tail -f /data/out.log"]
    volumeMounts: [{name: data, mountPath: /data}]
  volumes:
  - name: data
    emptyDir: {}
```
```bash
k apply -f logger.yaml
k logs logger -c reader -n ecom
```
💡 Mẹo sinh khung: `k run logger --image=busybox:1.36 -n ecom $do > logger.yaml` rồi sửa.
</details>

<details>
<summary><b>Câu 4 — CronJob</b></summary>

```bash
k create cronjob cleanup --image=busybox:1.36 --schedule="*/3 * * * *" -n ecom \
  --dry-run=client -o yaml -- sh -c 'echo cleaning' > cj.yaml
```
Sửa thêm vào `spec`:
```yaml
spec:
  schedule: "*/3 * * * *"
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 2
  failedJobsHistoryLimit: 1
```
```bash
k apply -f cj.yaml
k get cronjob -n ecom
```
</details>

<details>
<summary><b>Câu 5 — ConfigMap: env + volume</b></summary>

```bash
k create cm app-cfg --from-literal=LOG_LEVEL=debug --from-literal=MAX_CONN=100 -n ecom
```
```yaml
apiVersion: v1
kind: Pod
metadata: {name: cfg-app, namespace: ecom}
spec:
  containers:
  - name: nginx
    image: nginx
    envFrom:
    - configMapRef: {name: app-cfg}
    volumeMounts:
    - {name: cfg, mountPath: /etc/appcfg}
  volumes:
  - name: cfg
    configMap: {name: app-cfg}
```
**Verify:**
```bash
k exec cfg-app -n ecom -- env | grep -E 'LOG_LEVEL|MAX_CONN'
k exec cfg-app -n ecom -- ls /etc/appcfg
```
</details>

<details>
<summary><b>Câu 6 — Taint + label + nodeSelector + toleration</b></summary>

```bash
k taint node worker-1 gpu=true:NoSchedule
k label node worker-1 hardware=gpu
```
```yaml
apiVersion: v1
kind: Pod
metadata: {name: gpu-job}
spec:
  nodeSelector:
    hardware: gpu              # ← ĐẢM BẢO lên đúng node
  tolerations:                 # ← ĐƯỢC PHÉP lên node có taint
  - {key: gpu, operator: Equal, value: "true", effect: NoSchedule}
  containers:
  - {name: app, image: nginx}
```
⚠️ **Cần CẢ HAI.** Toleration chỉ *cho phép*, nodeSelector mới *bắt buộc*.
**Verify:** `k get po gpu-job -o wide`
</details>

<details>
<summary><b>Câu 7 — HPA</b></summary>

```bash
k autoscale deploy api --min=2 --max=10 --cpu-percent=65 -n ecom
k get hpa -n ecom
```
Nếu thấy `TARGETS: <unknown>/65%`:
```bash
# Nguyên nhân 1: Deployment thiếu requests.cpu
k set resources deploy api --requests=cpu=100m -n ecom

# Nguyên nhân 2: metrics-server chưa chạy
k top po -n ecom
k get deploy metrics-server -n kube-system
```
**Verify:** `k get hpa api -n ecom` → phải thấy `0%/65%` chứ không phải `<unknown>`.
</details>

<details>
<summary><b>Câu 8 — QoS Guaranteed</b></summary>

```bash
k set resources deploy catalog -n ecom \
  --requests=cpu=200m,memory=256Mi --limits=cpu=200m,memory=256Mi
k rollout status deploy/catalog -n ecom
k get po -n ecom -l app=catalog -o jsonpath='{.items[0].status.qosClass}'
# → Guaranteed
```
⚠️ `Guaranteed` yêu cầu requests **==** limits cho **cả cpu lẫn memory**, ở **mọi** container.
</details>

<details>
<summary><b>Câu 9 — NodePort</b></summary>

```bash
k expose deploy catalog --name=catalog-svc --port=80 --target-port=80 \
  --type=NodePort -n ecom --dry-run=client -o yaml > svc.yaml
# thêm nodePort: 30090 vào ports
k apply -f svc.yaml
```
Hoặc:
```bash
k expose deploy catalog --name=catalog-svc --port=80 --target-port=80 --type=NodePort -n ecom
k patch svc catalog-svc -n ecom --type=json \
  -p '[{"op":"replace","path":"/spec/ports/0/nodePort","value":30090}]'
```
**Verify:** `k get svc catalog-svc -n ecom` + `curl <node-ip>:30090`
</details>

<details>
<summary><b>Câu 10 — Service không có endpoint</b></summary>

```bash
# 1. Xác nhận
k get ep broken-svc -n ecom          # ENDPOINTS: <none>

# 2. So selector với label Pod
k describe svc broken-svc -n ecom | grep Selector
k get po -n ecom --show-labels

# 3a. Nếu selector lệch → sửa selector của Service
k patch svc broken-svc -n ecom -p '{"spec":{"selector":{"app":"broken-app"}}}'

# 3b. Nếu Pod chưa Ready → readinessProbe fail
k get po -n ecom                     # READY 0/1?
k describe po <pod> -n ecom | grep -i readiness

# 3c. Nếu targetPort sai
k get po <pod> -n ecom -o jsonpath='{.spec.containers[*].ports[*].containerPort}'
k patch svc broken-svc -n ecom --type=json \
  -p '[{"op":"replace","path":"/spec/ports/0/targetPort","value":8080}]'

# 4. Verify
k get ep broken-svc -n ecom          # phải có IP
```
</details>

<details>
<summary><b>Câu 11 — NetworkPolicy (AND selector + DNS)</b></summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: db-policy, namespace: ecom}
spec:
  podSelector:
    matchLabels: {role: db}
  policyTypes: [Ingress, Egress]
  ingress:
  - from:
    - namespaceSelector:              # ← KHÔNG có dấu "-" trước podSelector
        matchLabels: {env: prod}      #    ⇒ AND: pod role=api TRONG ns env=prod
      podSelector:
        matchLabels: {role: api}
    ports:
    - {protocol: TCP, port: 5432}
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
🔴 **Điểm chết:** `podSelector` phải cùng một phần tử list với `namespaceSelector` (không có `-`).
Nếu để `- podSelector:` thành phần tử riêng → thành OR, sai đề.

**Verify:**
```bash
k run t --image=busybox:1.28 --rm -it --restart=Never -n ecom -l role=api -- \
  nc -zv -w3 <db-pod-ip> 5432
```
</details>

<details>
<summary><b>Câu 12 — Ingress</b></summary>

```bash
k create ingress shop -n ecom --class=nginx \
  --rule="shop.example.com/api*=api-svc:8080" \
  --rule="shop.example.com/*=catalog-svc:80"

k get ingress shop -n ecom
k describe ingress shop -n ecom
```
⚠️ Dấu `*` ở cuối path trong `--rule` sinh ra `pathType: Prefix`.
Không có `*` → `pathType: Exact`.
</details>

<details>
<summary><b>Câu 13 — HTTPRoute traffic splitting</b></summary>

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata: {name: canary, namespace: ecom}
spec:
  parentRefs:
  - name: main-gw
    namespace: infra
  hostnames: ["shop.example.com"]
  rules:
  - backendRefs:
    - {name: catalog-svc, port: 80, weight: 90}
    - {name: catalog-v2-svc, port: 80, weight: 10}
```
```bash
k apply -f canary.yaml
k describe httproute canary -n ecom | grep -A10 Status
# Conditions phải có Accepted=True, ResolvedRefs=True
```
⚠️ HTTPRoute ở ns khác Gateway → Gateway phải có `allowedRoutes.namespaces.from: All`
(hoặc Selector khớp). Nếu `ResolvedRefs=False`, kiểm tra chỗ này.
</details>

<details>
<summary><b>Câu 14 — DNS không hoạt động</b></summary>

```bash
# 1. Test
k run t --image=busybox:1.28 --rm -it --restart=Never -n ecom -- nslookup kubernetes.default

# 2. CoreDNS còn sống không?
k get po -n kube-system -l k8s-app=kube-dns
k get deploy coredns -n kube-system            # replicas = 0?
k logs -n kube-system -l k8s-app=kube-dns

# 3. Service kube-dns
k get svc kube-dns -n kube-system
k get ep kube-dns -n kube-system

# 4. Sửa (tùy nguyên nhân)
k scale deploy coredns -n kube-system --replicas=2       # nếu bị scale về 0
k rollout restart deploy coredns -n kube-system          # nếu Corefile vừa sửa
k edit cm coredns -n kube-system                         # nếu Corefile hỏng

echo "CoreDNS deployment scaled to 0 replicas" > /opt/dns-issue.txt
```
</details>

<details>
<summary><b>Câu 15 — PersistentVolume</b></summary>

```yaml
apiVersion: v1
kind: PersistentVolume
metadata: {name: pv-logs}
spec:
  capacity: {storage: 3Gi}
  accessModes: [ReadWriteOnce]
  persistentVolumeReclaimPolicy: Retain
  storageClassName: slow
  hostPath: {path: /mnt/logs, type: DirectoryOrCreate}
```
**Verify:** `k get pv pv-logs` → STATUS `Available`
</details>

<details>
<summary><b>Câu 16 — PVC + Pod</b></summary>

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata: {name: pvc-logs, namespace: ecom}
spec:
  accessModes: [ReadWriteOnce]
  storageClassName: slow
  resources: {requests: {storage: 2Gi}}
---
apiVersion: v1
kind: Pod
metadata: {name: log-app, namespace: ecom}
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts: [{name: logs, mountPath: /var/log/app}]
  volumes:
  - name: logs
    persistentVolumeClaim: {claimName: pvc-logs}
```
**Verify:** `k get pvc -n ecom` → `Bound` tới `pv-logs`
</details>

<details>
<summary><b>Câu 17 — PVC Pending</b></summary>

```bash
k describe pvc stuck-pvc -n ecom | tail -15
```
Đối chiếu 4 điều kiện bind:
```bash
k get pv
k get sc
```
| Message | Sửa |
|---|---|
| `no persistent volumes available for this claim` | Tạo PV khớp, hoặc sửa `storageClassName` của PVC |
| `storageclass.storage.k8s.io "x" not found` | Tạo SC hoặc đổi tên trong PVC |
| `waiting for first consumer` | **Không phải lỗi** — tạo Pod dùng PVC |
| PV đang `Released` | `k patch pv <n> -p '{"spec":{"claimRef":null}}'` |

PVC không sửa được `storageClassName` khi đã tạo → phải xóa và tạo lại:
```bash
k get pvc stuck-pvc -n ecom -o yaml > /tmp/pvc.yaml
k delete pvc stuck-pvc -n ecom
vim /tmp/pvc.yaml     # sửa storageClassName, xóa metadata thừa
k apply -f /tmp/pvc.yaml
```
</details>

<details>
<summary><b>Câu 18 — StorageClass mặc định</b></summary>

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-fast
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
reclaimPolicy: Delete
```
```bash
k apply -f sc.yaml
# Bỏ default của SC cũ (nếu có)
k get sc
k patch sc <sc-cũ> -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
k get sc      # chỉ 1 cái có (default)
```
</details>

<details>
<summary><b>Câu 19 — Upgrade control plane</b></summary>

```bash
# Trên control plane node
k get nodes                       # ghi lại version hiện tại, vd v1.33.x

apt-get update
apt-cache madison kubeadm | head -5
apt-mark unhold kubeadm
apt-get install -y kubeadm=1.34.1-1.1
apt-mark hold kubeadm
kubeadm version

kubeadm upgrade plan
kubeadm upgrade apply v1.34.1 -y

k drain <cp-node> --ignore-daemonsets

apt-mark unhold kubelet kubectl
apt-get install -y kubelet=1.34.1-1.1 kubectl=1.34.1-1.1
apt-mark hold kubelet kubectl
systemctl daemon-reload
systemctl restart kubelet

k uncordon <cp-node>
k get nodes                       # cp mới, worker giữ nguyên version cũ
```
🔴 **Đừng chạm vào worker.** Đề nói rõ "do not upgrade worker".
</details>

<details>
<summary><b>Câu 20 — etcd backup & restore</b></summary>

```bash
# --- BACKUP ---
ETCDCTL_API=3 etcdctl snapshot save /opt/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

ETCDCTL_API=3 etcdctl snapshot status /opt/etcd-backup.db --write-out=table

# --- RESTORE ---
ETCDCTL_API=3 etcdctl snapshot restore /data/etcd-snapshot-previous.db \
  --data-dir=/var/lib/etcd-restore

vim /etc/kubernetes/manifests/etcd.yaml
#   volumes:
#   - name: etcd-data
#     hostPath:
#       path: /var/lib/etcd-restore        ← chỉ đổi dòng này
#       type: DirectoryOrCreate
#   (mountPath GIỮ NGUYÊN /var/lib/etcd)

watch crictl ps | grep etcd
k get nodes                       # apiserver mất ~1 phút để lên lại
```
💡 Quên cờ cert? `grep 'file' /etc/kubernetes/manifests/etcd.yaml` có sẵn đủ đường dẫn.
</details>

<details>
<summary><b>Câu 21 — RBAC ClusterRole</b></summary>

```bash
k create sa monitor -n ecom
k create clusterrole monitor-role --verb=get,list,watch --resource=pods,nodes,services
k create clusterrolebinding monitor-bind \
  --clusterrole=monitor-role --serviceaccount=ecom:monitor

# Verify
k auth can-i list pods --as=system:serviceaccount:ecom:monitor -A          # yes
k auth can-i get nodes --as=system:serviceaccount:ecom:monitor             # yes
k auth can-i delete pods --as=system:serviceaccount:ecom:monitor           # no
k auth can-i --list --as=system:serviceaccount:ecom:monitor
```
⚠️ `ClusterRoleBinding` (không phải RoleBinding) mới cho quyền toàn cluster.
</details>

<details>
<summary><b>Câu 22 — Drain & uncordon</b></summary>

```bash
k drain worker-2 --ignore-daemonsets --delete-emptydir-data
k get nodes                     # worker-2: Ready,SchedulingDisabled
k get po -A -o wide | grep worker-2      # chỉ còn DaemonSet pod

# ... bảo trì ...

k uncordon worker-2
k get nodes                     # Ready
```
Nếu drain bị treo vì Pod "bare" (không thuộc controller): thêm `--force`
(⚠️ Pod đó sẽ mất luôn, không được tạo lại).
</details>

<details>
<summary><b>Câu 23 — Helm</b></summary>

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm install web-helm bitnami/nginx \
  -n helm-test --create-namespace \
  --set replicaCount=2

helm list -n helm-test
k get po -n helm-test

# Ghi version chart
helm list -n helm-test -o json | jq -r '.[0].chart' > /opt/helm-version.txt
# hoặc không có jq:
helm list -n helm-test --no-headers | awk '{print $9}' > /opt/helm-version.txt
cat /opt/helm-version.txt
```
⚠️ `-n` **không** tự tạo namespace — cần `--create-namespace`.
</details>

<details>
<summary><b>Câu 24 — Kustomize</b></summary>

```yaml
# /opt/kustomize-app/overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: prod
namePrefix: prod-

resources:
  - ../../base

images:
  - name: nginx
    newTag: "1.28"

replicas:
  - name: <tên-deployment-trong-base>
    count: 4
```
```bash
cd /opt/kustomize-app
kubectl kustomize overlays/prod          # xem trước, KHÔNG apply
k create ns prod
kubectl apply -k overlays/prod
k get all -n prod
```
🔴 Docs Kustomize bị chặn trong phòng thi — phải thuộc các field trên.
⚠️ `replicas[].name` là tên resource **trong base** (trước khi thêm prefix).
</details>

<details>
<summary><b>Câu 25 — Sửa flag apiserver</b></summary>

```bash
cp /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/apiserver.bak   # LUÔN backup

vim /etc/kubernetes/manifests/kube-apiserver.yaml
# thêm vào phần command:
#     - --service-node-port-range=25000-32767

watch crictl ps | grep apiserver         # chờ container mới lên
k get nodes                              # apiserver sống lại

# Verify
k create svc nodeport test --tcp=80:80 --node-port=25001
k get svc test
k delete svc test
```
Nếu apiserver không lên: `crictl ps -a | grep apiserver` → `crictl logs <id>`,
hoặc `cat /var/log/pods/kube-system_kube-apiserver-*/kube-apiserver/*.log | tail -30`.
Cứu: `cp /tmp/apiserver.bak /etc/kubernetes/manifests/kube-apiserver.yaml`.
</details>

<details>
<summary><b>Câu 26 — Node NotReady</b></summary>

```bash
k describe node worker-1 | grep -A15 Conditions

ssh worker-1
sudo -i
systemctl status kubelet
journalctl -u kubelet -n 50 --no-pager
```
Xử lý theo nguyên nhân:
```bash
# a) kubelet không chạy
systemctl start kubelet && systemctl enable kubelet     # ← enable cho survive reboot

# b) swap bật
swapoff -a && sed -i '/ swap / s/^/#/' /etc/fstab

# c) cgroup driver lệch
grep cgroupDriver /var/lib/kubelet/config.yaml          # phải systemd
grep SystemdCgroup /etc/containerd/config.toml          # phải true
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
systemctl restart containerd kubelet

# d) sai endpoint apiserver
grep server /etc/kubernetes/kubelet.conf                # port phải 6443

# e) containerd chết
systemctl start containerd && systemctl enable containerd

# f) đầy đĩa
df -h && crictl rmi --prune

exit
k get nodes                                             # Ready
```
🔴 Đề nói **"sau khi reboot vẫn Ready"** ⇒ bắt buộc `systemctl enable`, không chỉ `start`.
</details>

<details>
<summary><b>Câu 27 — CrashLoopBackOff</b></summary>

```bash
k describe po crash-app -n ecom | tail -25
k logs crash-app -n ecom --previous
```
| Phát hiện | Sửa |
|---|---|
| `Exit Code: 137` / `Reason: OOMKilled` | Tăng `limits.memory` |
| `Liveness probe failed` | Tăng `initialDelaySeconds`, sửa port/path probe |
| `Exit Code: 127` | Sai `command` |
| `Exit Code: 0` (container xong rồi thoát) | Thêm `command: ["sleep","3600"]` |
| `CreateContainerConfigError` | Thiếu ConfigMap/Secret → tạo nó |
| Log có stack trace app | Sửa env/config theo lỗi |

```bash
# Ví dụ sửa OOM
k set resources po/crash-app --limits=memory=512Mi -n ecom     # Pod không patch được → tạo lại
k get po crash-app -n ecom -o yaml > /tmp/p.yaml
# sửa /tmp/p.yaml, xóa status/metadata thừa
k delete po crash-app -n ecom && k apply -f /tmp/p.yaml

k get po crash-app -n ecom -w      # Running, RESTARTS ngừng tăng
```
</details>

<details>
<summary><b>Câu 28 — Pod dùng nhiều CPU nhất</b></summary>

```bash
k top pod -A --sort-by=cpu
k top pod -A --sort-by=cpu --no-headers | head -1 | awk '{print $2}' > /opt/top-cpu-pod.txt
cat /opt/top-cpu-pod.txt
```
⚠️ Cột 1 là namespace, cột 2 là tên pod (khi dùng `-A`). Đề chỉ hỏi **tên pod** → `$2`.
Nếu `k top` lỗi → kiểm tra metrics-server.
</details>

<details>
<summary><b>Câu 29 — Tìm Pod theo nội dung log</b></summary>

```bash
for p in $(k get po -n ecom -o jsonpath='{.items[*].metadata.name}'); do
  if k logs $p -n ecom --all-containers 2>/dev/null | grep -q 'ERROR_CODE_500'; then
    echo $p > /opt/error-pod.txt
    k logs $p -n ecom --all-containers | grep 'ERROR_CODE_500' > /opt/error-lines.txt
  fi
done

cat /opt/error-pod.txt
cat /opt/error-lines.txt
```
Nếu Pod cùng label thì nhanh hơn:
```bash
k logs -l app=x -n ecom --all-containers --prefix | grep ERROR_CODE_500
```
</details>

<details>
<summary><b>Câu 30 — Control plane chết</b></summary>

```bash
ssh <control-plane-node>
sudo -i

# 1. Container control plane nào chết?
crictl ps -a | grep -E 'apiserver|etcd|scheduler|controller'

# 2. Log
crictl logs <apiserver-container-id> 2>&1 | tail -40
cat /var/log/pods/kube-system_kube-apiserver-*/kube-apiserver/*.log | tail -40
journalctl -u kubelet -n 60 --no-pager | grep -i apiserver

# 3. Nguyên nhân hay gặp
```
| Log nói gì | Sửa |
|---|---|
| `unknown flag: --xxx` | Xóa flag sai trong `kube-apiserver.yaml` |
| `error loading ... no such file` | Sai đường dẫn cert / thiếu volume mount |
| `dial tcp 127.0.0.1:2379: connect: connection refused` | etcd chết → sửa `etcd.yaml` trước |
| YAML parse error | Sai indent trong manifest |
| `bind: address already in use` | Port bị chiếm |

```bash
# 4. Sửa & chờ
cp /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/bak
vim /etc/kubernetes/manifests/kube-apiserver.yaml
watch crictl ps | grep apiserver

# 5. Nếu kubelet không nhận file, ép đọc lại:
mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/ && sleep 20
mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/

exit
k get nodes
k get po -n kube-system
```
</details>

---

## Tự chấm

| Số câu đúng | Đánh giá |
|---|---|
| 27–30 | Sẵn sàng thi. Chuyển sang killer.sh. |
| 22–26 | Gần được. Ôn lại đúng những domain sai. |
| 16–21 | Cần thêm 2–3 tuần lab. |
| < 16 | Quay lại học lý thuyết từng domain trước khi luyện đề. |

**Nếu sai nhiều ở:**
- Câu 19, 20, 25, 26, 30 → yếu **Cluster Architecture + Troubleshooting** (55% đề!). Ưu tiên số 1.
- Câu 11, 13 → yếu **NetworkPolicy / Gateway API**. Luyện viết tay.
- Câu 23, 24 → yếu **Helm / Kustomize** (phần mới 2025).
