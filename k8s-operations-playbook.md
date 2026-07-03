# Kubernetes Operations Playbook — Vận hành, Sự cố & Backup

> Tài liệu vận hành cluster Kubernetes cho DevOps/SRE. Trọng tâm: **sự cố thực tế hay gặp**,
> cách **chẩn đoán → xử lý ngay → nguyên nhân gốc → giải pháp lâu dài (để không tái diễn)**.
> Mục tiêu: đủ để bạn **own toàn bộ vòng đời cluster** — xây dựng, triển khai, vận hành,
> và trở thành trụ cột/DevOps Lead.
>
> 🧩 **Đi kèm:** [`k8s-production-templates.md`](./k8s-production-templates.md) — YAML production-ready
> (copy-paste được) và runbook từng bước (etcd/Velero restore, bảo trì node, rollback khẩn cấp).

## 📑 Mục lục

- [0. Tư duy vận hành (Operating Mindset)](#0-tư-duy-vận-hành-operating-mindset)
- [1. Incident Playbook — Pod](#1-incident-playbook--pod)
- [2. Incident Playbook — Node](#2-incident-playbook--node)
- [3. Incident Playbook — Network & DNS](#3-incident-playbook--network--dns)
- [4. Incident Playbook — Storage](#4-incident-playbook--storage)
- [5. Incident Playbook — Control Plane & etcd](#5-incident-playbook--control-plane--etcd)
- [6. Incident Playbook — Scaling & Resource](#6-incident-playbook--scaling--resource)
- [7. Backup & Disaster Recovery (chuyên sâu)](#7-backup--disaster-recovery-chuyên-sâu)
- [8. Reliability Engineering (probe/PDB/SLO)](#8-reliability-engineering-probe--pdb--slo)
- [9. Observability (metrics/logs/traces)](#9-observability-metrics--logs--traces)
- [10. Security Hardening](#10-security-hardening)
- [11. Capacity Planning & Cost](#11-capacity-planning--cost)
- [12. Cluster Upgrade Strategy](#12-cluster-upgrade-strategy)
- [13. On-call & Incident Management](#13-on-call--incident-management)
- [14. Lộ trình trở thành DevOps Lead](#14-lộ-trình-trở-thành-devops-lead)

---

## 0. Tư duy vận hành (Operating Mindset)

Trước khi vào lệnh, đây là những nguyên tắc một DevOps Lead phải thấm:

1. **Cluster là cattle, không phải pet.** Mọi thứ phải dựng lại được từ code (IaC + GitOps). Không "sửa tay" trên production rồi quên.
2. **Mọi thay đổi qua Git.** `kubectl edit` trên prod là anti-pattern — nó sẽ bị GitOps ghi đè hoặc mất khi restore. Sửa ở Git → CI/CD/ArgoCD apply.
3. **Backup để NGOÀI cluster, và phải TEST restore.** Backup chưa test = không có backup.
4. **Quan sát được (observable) trước khi mở rộng.** Không có metrics/logs/alerts thì mọi sự cố đều là "mò kim đáy bể".
5. **Đặt giới hạn cho mọi thứ.** requests/limits, PDB, quota, network policy — thiếu chúng, 1 pod lỗi có thể kéo sập cả node/cluster.
6. **Blameless postmortem.** Sự cố là lỗi hệ thống, không phải lỗi người. Mỗi incident phải sinh ra 1 action item phòng ngừa.
7. **Tự động hóa cái lặp lại 3 lần.** Việc tay lần 3 là dấu hiệu cần script/operator/automation.

**Quy trình chuẩn khi có sự cố (mỗi mục dưới đây tuân theo):**
`Triệu chứng → Chẩn đoán → Mitigate (dừng chảy máu) → Root cause → Long-term fix`

**Lệnh "cầm máu" đầu tiên khi có sự cố bất kỳ:**
```bash
kubectl get pods -A | grep -vE 'Running|Completed'   # Cái gì đang không khỏe?
kubectl get nodes                                    # Node có Ready không?
kubectl get events -A --sort-by=.lastTimestamp | tail -40   # Chuyện gì vừa xảy ra?
kubectl top nodes; kubectl top pods -A --sort-by=memory      # Ai đang ngốn tài nguyên?
```

---

## 1. Incident Playbook — Pod

### 1.1 CrashLoopBackOff (pod crash liên tục)

**Triệu chứng:** Pod restart liên tục, cột STATUS = `CrashLoopBackOff`, RESTARTS tăng dần.

**Chẩn đoán:**
```bash
kubectl describe pod <pod>                     # Xem Events + Last State + Exit Code
kubectl logs <pod> --previous                  # Log của LẦN CHẠY TRƯỚC khi crash (quan trọng nhất)
kubectl logs <pod> -c <container>              # Nếu pod nhiều container
kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].lastState.terminated.exitCode}'
```

**Đọc Exit Code:**
| Exit Code | Ý nghĩa |
|-----------|---------|
| 0 | Thoát bình thường (nhưng process chính không nên thoát) |
| 1 | Lỗi ứng dụng chung (xem log app) |
| 137 | Bị SIGKILL — thường là **OOMKilled** (vượt memory limit) hoặc liveness probe giết |
| 139 | Segfault (SIGSEGV) |
| 143 | Bị SIGTERM (thường ổn khi shutdown) |

**Nguyên nhân gốc & xử lý thường gặp:**
- **App lỗi cấu hình / thiếu env / sai connection string** → sửa ConfigMap/Secret, kiểm tra dependency (DB, cache) có sẵn sàng chưa.
- **Liveness probe quá gắt** → app khởi động chậm nhưng probe kill sớm. Tăng `initialDelaySeconds`, hoặc dùng **`startupProbe`** cho app boot chậm.
- **OOMKilled (137)** → xem mục 1.2.
- **Thiếu file/quyền/volume** → kiểm tra mount, securityContext.

**Giải pháp lâu dài:**
- Thêm **`startupProbe`** cho app khởi động chậm để tách khỏi liveness.
- Validate config bằng CI (schema/lint) trước khi deploy.
- App phải **fail fast + log rõ ràng** lý do thoát; đặt `terminationMessagePolicy: FallbackToLogsOnError`.
- Đảm bảo dependency qua **init container** hoặc retry/backoff trong app, không giả định thứ tự khởi động.

### 1.2 OOMKilled (bị giết vì hết RAM)

**Triệu chứng:** RESTARTS tăng, `lastState.terminated.reason: OOMKilled`, exit 137.

**Chẩn đoán:**
```bash
kubectl describe pod <pod> | grep -A5 "Last State"     # Reason: OOMKilled
kubectl top pod <pod> --containers                     # RAM thực tế đang dùng
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].resources}'   # limit đang set
dmesg -T | grep -i "oom"                               # Trên node: OOM killer log
```

**Xử lý ngay:** Tăng `resources.limits.memory` cho container, hoặc giảm tải.

**Nguyên nhân gốc:**
- Memory limit đặt quá thấp so với thực tế.
- **Memory leak** trong app (RAM tăng đều theo thời gian).
- Xử lý batch/query lớn nạp hết vào RAM.
- JVM/Node không biết limit của container → cấp phát heap vượt limit.

**Giải pháp lâu dài:**
- Đặt `requests` = mức bình thường, `limits` = đỉnh hợp lý (theo dõi p95/p99 memory qua Prometheus rồi tinh chỉnh).
- Với **JVM**: dùng `-XX:MaxRAMPercentage=75` (Java tự nhận limit container). Với **Node.js**: `--max-old-space-size`.
- Bật alert khi `container_memory_working_set_bytes / limit > 0.9` để phát hiện trước khi bị kill.
- Dùng **VPA (Vertical Pod Autoscaler)** ở chế độ recommend để có số liệu right-sizing.
- Truy leak bằng profiler; đặt giới hạn kích thước batch/pagination trong app.

### 1.3 ImagePullBackOff / ErrImagePull

**Triệu chứng:** Pod `Pending`/`ImagePullBackOff`, không kéo được image.

**Chẩn đoán:**
```bash
kubectl describe pod <pod> | grep -A3 -i "failed"      # "not found" / "unauthorized" / "no such host"
```

**Nguyên nhân gốc & xử lý:**
- **Sai tên/tag image** → sửa manifest. Tránh dùng `:latest` (không xác định).
- **Registry private thiếu credential** → tạo `imagePullSecret`:
  ```bash
  kubectl create secret docker-registry regcred \
    --docker-server=<registry> --docker-username=<u> --docker-password=<p>
  # rồi tham chiếu trong pod spec: imagePullSecrets: [{name: regcred}]
  ```
- **Rate limit (Docker Hub)** → dùng registry riêng/mirror, hoặc pull-through cache.
- **Node không ra được internet** → kiểm tra NAT/firewall/proxy.

**Giải pháp lâu dài:**
- Pin image bằng **digest** (`@sha256:...`) hoặc tag bất biến (semver + git sha), không dùng `latest`.
- Chạy **registry riêng** (Harbor/ECR/GCR) + mirror để tránh rate limit và tăng tốc pull.
- Gắn imagePullSecret ở mức **ServiceAccount** để không phải lặp ở từng pod.
- **Scan image** (Trivy/Grype) và ký (cosign) trong CI trước khi cho deploy.

### 1.4 Pod Pending (không được schedule)

**Chẩn đoán:**
```bash
kubectl describe pod <pod> | grep -A10 Events    # "Insufficient cpu/memory" / "node(s) didn't match" / "unbound PVC"
```

**Nguyên nhân gốc & xử lý:**
- **Thiếu tài nguyên** (Insufficient cpu/memory) → thêm node / bật Cluster Autoscaler / giảm requests.
- **Không có node khớp** (nodeSelector/affinity/taint) → sửa affinity hoặc thêm toleration.
- **PVC chưa bound** → xem mục 4.
- **Vượt ResourceQuota namespace** → tăng quota hoặc dọn resource.

**Giải pháp lâu dài:**
- Bật **Cluster Autoscaler** / Karpenter để tự thêm node khi thiếu.
- Đặt `requests` sát thực tế (đặt quá cao gây lãng phí + khó schedule).
- Dùng **PriorityClass** cho workload quan trọng để không bị "kẹt hàng".

### 1.5 Pod Evicted

**Triệu chứng:** Pod status `Evicted`, node bị `DiskPressure`/`MemoryPressure`.

**Xử lý ngay:**
```bash
kubectl get pods -A | grep Evicted
kubectl delete pod --field-selector status.phase=Failed -A   # Dọn pod Evicted
kubectl describe node <node> | grep -i pressure               # Node đang thiếu gì
```

**Nguyên nhân gốc:** Node hết RAM/disk → kubelet evict pod để tự cứu.

**Giải pháp lâu dài:**
- Đặt `requests`/`limits` đúng để scheduler không nhồi quá tải node.
- Dọn log/image trên node (`kubelet --image-gc`), theo dõi disk node.
- Cấu hình **eviction thresholds** hợp lý; tách workload nặng disk ra node riêng.
- Alert sớm khi node RAM/disk > 80%.

### 1.6 Readiness probe fail (pod chạy nhưng không nhận traffic)

**Triệu chứng:** Pod `Running` nhưng `READY 0/1`, không có trong endpoints của Service.

**Chẩn đoán:**
```bash
kubectl describe pod <pod> | grep -i readiness       # Readiness probe failed
kubectl get endpoints <svc>                          # Pod có trong endpoint không
kubectl exec <pod> -- curl -s localhost:<port>/health # Thử probe thủ công
```

**Giải pháp lâu dài:** Readiness endpoint phải kiểm tra **dependency thật** (DB/cache reachable) chứ không chỉ trả 200 vô điều kiện; tách readiness (nhận traffic) khỏi liveness (còn sống).

---

## 2. Incident Playbook — Node

### 2.1 Node NotReady

**Chẩn đoán:**
```bash
kubectl get nodes
kubectl describe node <node> | grep -A15 Conditions   # Ready/DiskPressure/MemoryPressure/PIDPressure
# SSH vào node:
systemctl status kubelet                              # kubelet còn sống?
journalctl -u kubelet -f                              # Log kubelet
systemctl status containerd                           # container runtime
df -h; free -h                                        # Disk/RAM node
```

**Nguyên nhân gốc & xử lý:**
- **kubelet chết** → `systemctl restart kubelet`.
- **Runtime (containerd/docker) chết** → restart runtime.
- **Hết disk** (thường `/var/lib` đầy do log/image) → dọn, mở rộng disk.
- **Mất kết nối tới API server / cert hết hạn** → kiểm tra network, cert kubelet.

**Giải pháp lâu dài:**
- Monitor node (node-exporter) + alert `NodeNotReady`, disk/RAM > 80%.
- Log rotation (journald `SystemMaxUse`), image GC.
- Nhiều node + PDB để mất 1 node không sập dịch vụ.
- Node dùng chung template (IaC) để nhất quán, thay vì sửa tay.

### 2.2 Disk Pressure trên node

```bash
kubectl describe node <node> | grep -i disk
# Trên node: cái gì ăn disk?
du -sh /var/lib/containerd/* 2>/dev/null | sort -rh | head
du -sh /var/log/* | sort -rh | head
crictl rmi --prune                                   # Dọn image không dùng
journalctl --vacuum-size=500M                        # Dọn journald
```

**Long-term:** đặt `--image-gc-high-threshold`/`low-threshold` cho kubelet, tách disk cho `/var/lib/containerd`, alert disk sớm.

---

## 3. Incident Playbook — Network & DNS

### 3.1 DNS trong cluster hỏng (CoreDNS)

**Triệu chứng:** App báo "could not resolve host", "getaddrinfo ENOTFOUND", kết nối service bằng tên fail.

**Chẩn đoán:**
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns   # CoreDNS có Running không
kubectl logs -n kube-system -l k8s-app=kube-dns       # Log CoreDNS
# Test resolve từ 1 pod:
kubectl run tmp --rm -it --image=busybox:1.28 -- nslookup kubernetes.default
kubectl exec <pod> -- cat /etc/resolv.conf            # Cấu hình DNS của pod
```

**Nguyên nhân gốc & xử lý:**
- **CoreDNS quá tải / thiếu replica** → scale CoreDNS, thêm resources.
- **CoreDNS crash / config sai** → kiểm tra ConfigMap `coredns`.
- **NetworkPolicy chặn cổng 53** → cho phép egress tới kube-dns.
- **Node mất route tới CoreDNS** → kiểm tra CNI.

**Giải pháp lâu dài:**
- Bật **NodeLocal DNSCache** (giảm tải CoreDNS, giảm latency, tránh lỗi conntrack UDP).
- Scale CoreDNS theo tải + đặt PDB.
- Giảm phụ thuộc DNS: dùng đúng FQDN, cân nhắc `dnsConfig ndots`.

### 3.2 Service không kết nối được

**Chẩn đoán từng lớp (Pod → Service → Endpoint → NetworkPolicy):**
```bash
kubectl get endpoints <svc>                    # Service có trỏ tới pod nào không (rỗng = readiness fail/selector sai)
kubectl get svc <svc> -o wide                  # Selector, port, type đúng chưa
kubectl exec <pod-A> -- curl -s <svc>:<port>   # Từ pod khác gọi thử
kubectl get networkpolicy -A                   # Có policy nào chặn không
```

**Nguyên nhân hay gặp:** selector Service không khớp label pod; pod chưa Ready (không vào endpoint); NetworkPolicy chặn; sai port/targetPort.

**Long-term:** chuẩn hóa label; NetworkPolicy default-deny + cho phép có chủ đích; test connectivity trong CI/e2e.

### 3.3 Ingress 502 / 504

| Lỗi | Nguyên nhân | Xử lý |
|-----|-------------|-------|
| **502 Bad Gateway** | Backend pod chết / sai service port / pod chưa Ready | Kiểm tra `kubectl get endpoints`, readiness, service port |
| **504 Gateway Timeout** | Backend xử lý chậm hơn timeout | Tăng `proxy-read-timeout` annotation; tối ưu app |
| **413** | Body quá lớn | Tăng `proxy-body-size` |
| **404** | Sai host/path rule | Kiểm tra Ingress rule + IngressClass |

```bash
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx   # Log ingress controller
kubectl describe ingress <name>               # Rule, backend, TLS
```

**Long-term:** health check đúng, timeout hợp lý, có nhiều replica backend + PDB, monitor tỷ lệ 5xx qua Prometheus.

---

## 4. Incident Playbook — Storage

### 4.1 PVC Pending (không bound được volume)

```bash
kubectl describe pvc <pvc>                     # "no persistent volumes available" / "waiting for provisioner"
kubectl get storageclass                       # Có StorageClass default không
kubectl get pv                                 # PV có sẵn không
kubectl logs -n kube-system <csi-provisioner>  # Log CSI provisioner
```

**Nguyên nhân:** không có StorageClass default; CSI driver lỗi; hết quota volume ở cloud; access mode không khớp (RWO vs RWX).

**Long-term:** đặt StorageClass default, monitor CSI, chọn access mode đúng nhu cầu (RWX cần driver hỗ trợ như EFS/NFS).

### 4.2 Volume mount fail / Multi-Attach

**Triệu chứng:** `Multi-Attach error` — PV RWO đang gắn ở node khác (thường khi pod chuyển node mà pod cũ chưa detach).

**Xử lý:** đảm bảo pod cũ đã bị xóa hẳn; với RWO không chạy 2 replica cùng ghi; dùng StatefulSet cho stateful; cân nhắc RWX nếu cần chia sẻ.

### 4.3 Disk trong pod đầy

```bash
kubectl exec <pod> -- df -h                    # Filesystem nào đầy
```
**Long-term:** đặt `sizeLimit` cho emptyDir, xoay log ứng dụng, dùng PVC đủ lớn + alert dung lượng.

---

## 5. Incident Playbook — Control Plane & etcd

### 5.1 API Server không phản hồi

**Triệu chứng:** `kubectl` treo/timeout, "connection refused" tới :6443.

**Chẩn đoán (trên control-plane node):**
```bash
crictl ps | grep apiserver                     # Container apiserver còn chạy?
crictl logs <apiserver-id>                     # Log
journalctl -u kubelet | tail -50               # kubelet có dựng static pod không
curl -k https://localhost:6443/healthz         # Health API
systemctl status etcd                          # apiserver phụ thuộc etcd
```

**Nguyên nhân hay gặp:** etcd hỏng/chậm; cert hết hạn; hết RAM/disk control plane; quá tải (list-all pods liên tục).

**Long-term:** HA control plane (≥3 node), monitor apiserver latency/etcd, đặt `--request-timeout`, tránh client list toàn bộ resource liên tục (dùng watch/informer).

### 5.2 Cert control plane hết hạn (cluster "chết" đột ngột)

```bash
kubeadm certs check-expiration                 # Xem hạn tất cả cert
kubeadm certs renew all                         # Gia hạn
systemctl restart kubelet                       # Restart để nạp cert mới
# Cập nhật lại admin.conf nếu cần
```
**Long-term:** alert cert < 30 ngày; nâng cấp cluster định kỳ (kubeadm tự gia hạn khi upgrade); tự động hóa renew.

### 5.3 etcd chậm / phình to / mất quorum

```bash
ETCDCTL_API=3 etcdctl endpoint status --write-out=table   # DB size, leader, term
ETCDCTL_API=3 etcdctl endpoint health                     # Từng member khỏe không
ETCDCTL_API=3 etcdctl member list                         # Đủ member/quorum?
ETCDCTL_API=3 etcdctl defrag                               # Nén DB khi phình
ETCDCTL_API=3 etcdctl alarm list                          # Có NOSPACE alarm không
ETCDCTL_API=3 etcdctl alarm disarm                        # Gỡ alarm sau khi defrag
```

**Nguyên nhân phình to:** quá nhiều event/secret/configmap, không nén định kỳ, ổ chậm (etcd cần disk nhanh — SSD/NVMe).

**Long-term:** etcd trên **SSD riêng, độ trễ thấp**; số member lẻ (3/5) để có quorum; bật auto-compaction; defrag định kỳ; **snapshot etcd định kỳ ra ngoài cluster** (xem mục 7); giới hạn TTL event.

---

## 6. Incident Playbook — Scaling & Resource

### 6.1 HPA không scale

```bash
kubectl get hpa                                # TARGETS hiển thị <unknown>?
kubectl describe hpa <name>                    # Lý do
kubectl top pods                               # metrics-server hoạt động?
kubectl get apiservice v1beta1.metrics.k8s.io  # metrics API available?
```

**Nguyên nhân:** thiếu **metrics-server**; pod không đặt `resources.requests` (HPA cần requests để tính %); ngưỡng sai; đụng maxReplicas.

**Long-term:** luôn đặt requests; cài metrics-server; dùng **KEDA** cho scale theo custom/queue metric (Kafka lag, độ dài queue…); kết hợp Cluster Autoscaler để có node cho pod mới.

### 6.2 Noisy Neighbor (1 pod ăn hết tài nguyên node)

**Xử lý & long-term:** đặt `limits` cho mọi pod; dùng **LimitRange** đặt default cho namespace; **ResourceQuota** giới hạn namespace; tách workload nhạy cảm bằng node pool + taint/toleration; QoS class `Guaranteed` cho workload quan trọng (requests = limits).

### 6.3 Rollout kẹt

```bash
kubectl rollout status deploy/<name>           # Kẹt ở đâu
kubectl rollout history deploy/<name>          # Lịch sử
kubectl rollout undo deploy/<name>             # Rollback ngay khi deploy hỏng
kubectl rollout undo deploy/<name> --to-revision=3
```
**Long-term:** dùng `maxSurge/maxUnavailable` hợp lý; readiness probe chuẩn (rollout chờ Ready thật); progressive delivery (Argo Rollouts/Flagger canary) để tự rollback khi metric xấu.

---

## 7. Backup & Disaster Recovery (chuyên sâu)

> **3 lớp phải backup:** (A) **etcd** = trạng thái cluster, (B) **Manifest/YAML** = cấu hình,
> (C) **Persistent Volume** = dữ liệu. Mất bất kỳ lớp nào đều có thể mất production.

### A. etcd — trạng thái toàn cluster (ưu tiên #1)
```bash
# Snapshot (đặt vào cron mỗi 15–30 phút với cluster quan trọng)
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%F-%H%M).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-*.db --write-out=table
aws s3 cp /backup/etcd-*.db s3://<bucket>/etcd/    # BẮT BUỘC đẩy ra ngoài cluster
find /backup -name 'etcd-*.db' -mtime +14 -delete # Retention 14 ngày

# Restore khi mất control plane:
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-xxx.db --data-dir /var/lib/etcd-new
# -> trỏ static pod etcd vào /var/lib/etcd-new, restart kubelet, verify kubectl get nodes
```

### B. Manifest — cấu hình resource
```bash
# Cách thủ công (dump toàn bộ):
for ns in $(kubectl get ns -o jsonpath='{.items[*].metadata.name}'); do
  kubectl get all,cm,secret,ingress,pvc,networkpolicy -n $ns -o yaml > backup-$ns.yaml
done
kubectl get pv,clusterrole,clusterrolebinding,storageclass,crd -o yaml > backup-cluster.yaml
# Làm sạch để commit Git (bỏ status/uid/resourceVersion):
kubectl get deploy <name> -o yaml | kubectl neat > clean.yaml
```
**Best practice (khuyên dùng): GitOps chính là backup manifest.** Toàn bộ YAML để trong Git,
deploy qua **ArgoCD/Flux**. Dựng lại cluster = trỏ ArgoCD vào repo → tự sync lại tất cả.
Bổ sung **Velero** để backup cả manifest + PV theo lịch.

### C. Persistent Volume — dữ liệu
```bash
# Velero (khuyên dùng cho production, tích hợp cloud snapshot):
velero install ...                                  # Cài, trỏ backup location = S3/GCS
velero backup create full-$(date +%F) --snapshot-volumes --include-cluster-resources
velero schedule create daily --schedule="0 2 * * *" --snapshot-volumes  # Tự động 2h sáng
velero backup get; velero backup describe <name>; velero backup logs <name>
velero restore create --from-backup full-2026-07-03 # Khôi phục cả resource + PV

# Backup DB ở tầng ứng dụng (logic backup, an toàn nhất cho DB):
kubectl exec -n prod <pg-pod> -- pg_dump -U user db | gzip > db-$(date +%F).sql.gz
aws s3 cp db-*.sql.gz s3://<bucket>/db/

# VolumeSnapshot (CSI):
kubectl get volumesnapshot -A
```

### DR Runbook (khi mất cluster)
```text
1. Dựng lại control plane (kubeadm init / IaC / Terraform).
2. Khôi phục etcd từ snapshot mới nhất  ->  kubectl get nodes phải trả về.
3. Nếu KHÔNG có etcd backup nhưng có manifest trong Git:
   -> dựng cluster mới, cài ArgoCD, sync toàn bộ từ Git.
4. Khôi phục PV: velero restore / restore DB dump.
5. Kiểm tra ingress, cert (cert-manager), DNS, external LB.
6. Smoke test end-to-end trước khi mở traffic.

Nguyên tắc vàng:
 - Backup để NGOÀI cluster (S3/GCS), mã hóa.
 - Định nghĩa RPO (mất tối đa bao nhiêu dữ liệu) & RTO (khôi phục trong bao lâu) -> chọn tần suất backup.
 - Quy tắc 3-2-1: 3 bản sao, 2 loại lưu trữ, 1 offsite.
 - TEST RESTORE định kỳ (game day) — backup chưa test = không có backup.
```

---

## 8. Reliability Engineering (probe / PDB / SLO)

Những cấu hình biến "chạy được" thành "chạy đáng tin cậy":

```yaml
# 1. Probes — startup tách khỏi liveness (cứu app boot chậm khỏi CrashLoop)
startupProbe:   { httpGet: {path: /health, port: 8080}, failureThreshold: 30, periodSeconds: 5 }
livenessProbe:  { httpGet: {path: /health, port: 8080}, periodSeconds: 10 }   # còn sống?
readinessProbe: { httpGet: {path: /ready,  port: 8080}, periodSeconds: 5 }    # nhận traffic?

# 2. Resources — luôn đặt requests & limits
resources:
  requests: { cpu: 100m, memory: 128Mi }   # để scheduler & HPA tính
  limits:   { cpu: 500m, memory: 512Mi }   # trần cứng

# 3. Graceful shutdown — không rớt request khi rollout
terminationGracePeriodSeconds: 30
lifecycle: { preStop: { exec: { command: ["sleep","10"] } } }  # cho LB rút endpoint trước
```

```yaml
# 4. PodDisruptionBudget — giữ tối thiểu pod khi drain/upgrade node
apiVersion: policy/v1
kind: PodDisruptionBudget
spec: { minAvailable: 2, selector: { matchLabels: { app: myapp } } }
```

```bash
# 5. Chống dồn 1 node — topologySpreadConstraints / podAntiAffinity
#    -> replica trải đều các node/zone, mất 1 node không mất dịch vụ
```

**SLO/SLI/Error Budget (tư duy SRE):**
- **SLI** = chỉ số đo (tỷ lệ request 2xx, latency p99…).
- **SLO** = mục tiêu (vd 99.9% request thành công/tháng).
- **Error budget** = phần được phép lỗi (0.1%). Hết budget → freeze feature, tập trung ổn định.
- Alert dựa trên **burn rate** của error budget, không alert theo từng lỗi lẻ (giảm nhiễu).

---

## 9. Observability (metrics / logs / traces)

**3 trụ cột:**
```text
Metrics  -> Prometheus + Grafana + Alertmanager   (điều gì đang xảy ra, xu hướng)
Logs     -> Loki / ELK (Elasticsearch+Kibana)     (chi tiết điều gì đã xảy ra)
Traces   -> Tempo / Jaeger (OpenTelemetry)         (request đi qua đâu, chậm ở đâu)
```

```bash
# Prometheus queries cốt lõi để vận hành:
# up == 0                                          -> target chết
# rate(http_requests_total{code=~"5.."}[5m])       -> tỷ lệ lỗi 5xx
# histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))  -> p99 latency
# sum(container_memory_working_set_bytes) by (pod) / sum(kube_pod_container_resource_limits{resource="memory"}) by (pod)  -> % RAM so với limit
# kube_pod_container_status_restarts_total          -> pod restart nhiều
# kube_node_status_condition{condition="Ready",status="true"}  -> node Ready
```

**Alert tối thiểu phải có:** NodeNotReady, PodCrashLooping, high 5xx rate, high latency, PVC gần đầy, cert sắp hết hạn, etcd DB size cao, memory/cpu gần limit, HPA đụng maxReplicas.

**Long-term:** chuẩn hóa **structured logging (JSON)** có trace_id; dùng **kube-prometheus-stack** (Helm) để có sẵn dashboard + alert; gắn **OpenTelemetry** để trace xuyên service.

---

## 10. Security Hardening

```bash
# RBAC — least privilege
kubectl auth can-i --list                      # Quyền của service account hiện tại
kubectl auth can-i create pods --as system:serviceaccount:ns:sa
# Quét cấu hình & image
trivy image <image>                            # Scan lỗ hổng image
trivy k8s cluster --report summary             # Scan cả cluster
kube-bench run                                 # CIS Kubernetes Benchmark
kubectl get networkpolicy -A                   # Có phân đoạn mạng không
```

**Checklist bảo mật cốt lõi:**
- **RBAC least-privilege** — không dùng `cluster-admin` bừa bãi; mỗi app 1 ServiceAccount tối thiểu quyền.
- **NetworkPolicy default-deny** — mặc định chặn, chỉ mở luồng cần thiết (phân đoạn Đông-Tây).
- **Pod Security Standards (restricted)** — cấm privileged, hostPath, chạy root; đặt `runAsNonRoot`, `readOnlyRootFilesystem`, drop capabilities.
- **Secrets** — mã hóa etcd at-rest, dùng External Secrets/Vault, Sealed Secrets/SOPS (không commit secret thô vào Git).
- **Image** — scan (Trivy) + ký (cosign) + admission control (Kyverno/OPA Gatekeeper) chỉ cho image tin cậy.
- **Cập nhật & vá** — theo dõi CVE, nâng cấp cluster/node định kỳ.
- **Audit log** — bật API server audit để truy vết "ai làm gì".

---

## 11. Capacity Planning & Cost

```bash
kubectl top nodes                              # Mức dùng thực tế
kubectl describe node <node> | grep -A5 Allocated   # Requests đã đặt chỗ vs capacity
# Công cụ:
# kubectl resource-capacity (krew)             -> so requests/limits/usage
# kube-capacity                                -> báo cáo dung lượng
# Goldilocks (VPA)                             -> gợi ý right-sizing
# OpenCost / Kubecost                          -> chi phí theo namespace/team
```

**Nguyên tắc:**
- **Requests đặt chỗ**, không phải mức dùng thực → đặt quá cao = trả tiền cho tài nguyên rảnh. Right-size định kỳ bằng số liệu p95/p99.
- **Bin-packing**: dùng Cluster Autoscaler/Karpenter để gom pod, giảm node thừa; scale-down node rảnh.
- **Spot/Preemptible** cho workload chịu gián đoạn; on-demand cho stateful.
- **Chargeback**: gắn label team/cost-center để quy chi phí theo team → tạo động lực tối ưu.

---

## 12. Cluster Upgrade Strategy

```text
Nguyên tắc:
 - K8s hỗ trợ skew: chỉ nâng LẦN LƯỢT từng minor (1.28 -> 1.29 -> 1.30), không nhảy cóc.
 - Thứ tự: control plane trước -> node pool sau. kubelet không được mới hơn apiserver.
 - Kiểm tra API deprecation TRƯỚC (kubectl deprecations / pluto / kubent).
```
```bash
# Kiểm tra API sắp bị bỏ trước khi nâng:
kubent                                         # (kube-no-trouble) quét manifest dùng API deprecated
kubeadm upgrade plan                           # Xem đường nâng cấp khả dĩ
# Quy trình node (rolling, không downtime):
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data   # rút pod
#   nâng kubeadm/kubelet trên node...
kubectl uncordon <node>                        # trả node về cluster
```

**Long-term:** luôn **backup etcd trước khi nâng**; nâng trên môi trường staging trước; dùng **blue-green node pool** (dựng node pool mới version mới, chuyển workload, xoá pool cũ) với managed k8s (EKS/GKE/AKS); giữ cluster **không quá 1-2 version so với latest** để còn được support.

---

## 13. On-call & Incident Management

**Quy trình xử lý sự cố (một Lead phải thiết lập cho team):**
```text
1. DETECT   — alert tự động (không đợi user báo). Alert phải actionable, có severity.
2. TRIAGE   — đánh giá mức độ ảnh hưởng (SEV1..SEV4), ai bị ảnh hưởng.
3. MITIGATE — CẦM MÁU trước, tìm nguyên nhân sau (rollback, scale, failover, tắt feature).
4. RESOLVE  — khắc phục, xác nhận hồi phục qua metrics.
5. POSTMORTEM (blameless) — timeline, root cause, action items phòng ngừa, có người & deadline.
```

**Công cụ/kỹ năng cần chuẩn hóa:**
- **Runbook** cho mỗi alert (alert → link runbook → các bước xử lý). Giảm phụ thuộc "người biết".
- **Severity & escalation policy** rõ ràng (PagerDuty/Opsgenie).
- **Status page** + kênh liên lạc sự cố (war room).
- **Error budget policy**: hết budget thì ưu tiên reliability hơn feature.
- **Game day / Chaos engineering** (kill pod/node ngẫu nhiên — Chaos Mesh/Litmus) để kiểm chứng khả năng chịu lỗi & DR.

---

## 14. Lộ trình trở thành DevOps Lead

Để **own toàn bộ vòng đời k8s** và trở thành trụ cột, xây năng lực theo 4 tầng:

**Tầng 1 — Nền tảng (phải chắc):**
- Linux (process, network, systemd, filesystem), networking (TCP/IP, DNS, TLS, HTTP), container (image, runtime).
- Kubernetes core objects & vòng đời (Pod, Deployment, StatefulSet, Service, Ingress, PV/PVC, RBAC).
- Git + một ngôn ngữ script (Bash/Python/Go).

**Tầng 2 — Vận hành (nội dung tài liệu này):**
- Debug thành thạo mọi loại sự cố Pod/Node/Network/Storage/Control plane.
- Backup/DR: etcd, manifest (GitOps), PV (Velero) — và **đã test restore**.
- Observability: Prometheus/Grafana/Loki, viết alert & dashboard.
- Reliability: probes, PDB, resource, SLO/error budget.

**Tầng 3 — Xây dựng & Tự động hóa (Platform):**
- **IaC**: Terraform/Pulumi dựng cluster & hạ tầng từ code.
- **GitOps**: ArgoCD/Flux — mọi thứ khai báo, tự động đồng bộ, có audit.
- **CI/CD**: build → scan → sign → deploy an toàn, progressive delivery (canary/blue-green).
- **Helm/Kustomize** hóa manifest; viết **Operator/Controller** cho tự động hóa nâng cao.
- **Policy as Code**: Kyverno/OPA; **Secrets**: Vault/External Secrets.

**Tầng 4 — Lãnh đạo & Chiến lược:**
- Thiết kế **platform** để dev tự phục vụ (golden path, template, self-service) — giảm phụ thuộc vào ops.
- Chuẩn hóa **runbook, on-call, postmortem** cho team; đào tạo/nâng đội.
- Cân đối **reliability ↔ tốc độ ↔ chi phí** (SLO, error budget, FinOps).
- Kiến trúc **multi-cluster / multi-region / DR**, capacity & cost planning dài hạn.
- Ra quyết định công nghệ có lý do (trade-off), viết ADR (Architecture Decision Record).

**Chứng chỉ tham khảo (không bắt buộc nhưng hệ thống hóa kiến thức):** CKA → CKAD → CKS (bảo mật) → Terraform Associate.

**Thói quen của một Lead giỏi:**
- Ưu tiên **phòng ngừa** hơn chữa cháy — mỗi incident đẻ ra 1 cải tiến hệ thống.
- **Tài liệu hóa** mọi thứ (như file này) — kiến thức trong đầu 1 người là rủi ro (bus factor).
- **Tự động hóa** việc lặp lại; đo lường trước khi tối ưu.
- Giữ hệ thống **đơn giản nhất có thể** — phức tạp là kẻ thù của độ tin cậy.
