# CKA — Troubleshooting (30%) 🔴

> **Domain nặng nhất của CKA.** Gần 1/3 số điểm. Cũng là domain khó "học vẹt" nhất —
> phải có phản xạ chẩn đoán, không phải thuộc lệnh.
>
> Tin tốt: bạn làm K8s production nhiều → đây là sân nhà. Việc cần làm là **hệ thống hóa**
> phản xạ đó thành quy trình chạy được trong 5 phút dưới áp lực.

**Nội dung curriculum v1.35:**
- Troubleshoot clusters and nodes
- Troubleshoot cluster components
- Monitor cluster and application resource usage
- Manage and evaluate container output streams
- Troubleshoot services and networking

> 📖 Bổ trợ rất tốt: [k8s-operations-playbook.md](../../../k8s-operations-playbook.md) trong repo này
> (playbook sự cố thật theo cấu trúc *triệu chứng → chẩn đoán → xử lý → root cause → phòng ngừa*).

---

## 1. Cây quyết định — bắt đầu từ đâu

```text
                     ┌─────────────────────┐
                     │  kubectl có chạy?   │
                     └──────────┬──────────┘
                 KHÔNG ─────────┴───────── CÓ
                   │                        │
                   ▼                        ▼
        ┌──────────────────────┐   ┌──────────────────┐
        │ apiserver/etcd chết  │   │  k get nodes     │
        │ → SSH vào cp node    │   └────────┬─────────┘
        │ → crictl ps -a       │   NotReady─┴─Ready
        │ → /var/log/pods/     │       │         │
        │ → systemctl kubelet  │       ▼         ▼
        └──────────────────────┘  ┌────────┐ ┌──────────────┐
                                  │ MỤC 3  │ │  k get po -A │
                                  │ NODE   │ └──────┬───────┘
                                  └────────┘        │
                                          ┌─────────┴──────────┐
                                          ▼                    ▼
                                    Pod lỗi              Pod OK nhưng
                                    (MỤC 4)              không truy cập được
                                                         (MỤC 6)
```

---

## 2. Bộ lệnh mở màn — gõ trong 30 giây đầu

```bash
# Toàn cảnh
k get nodes -o wide
k get po -A -o wide | grep -vE 'Running|Completed'    # chỉ hiện thứ đang lỗi
k get events -A --sort-by=.lastTimestamp | tail -30   # ← VÀNG. Đọc cái này trước.

# Zoom vào 1 đối tượng
k describe po <pod> -n <ns>          # xem phần Events ở cuối
k logs <pod> -n <ns>
k logs <pod> -n <ns> --previous      # ← log của lần chạy TRƯỚC (khi CrashLoopBackOff)
k logs <pod> -c <container> -n <ns>  # multi-container
k logs -l app=web -n <ns> --tail=50 --all-containers

# Control plane
k get po -n kube-system
k get componentstatuses               # deprecated nhưng vẫn dùng nhanh được

# Tài nguyên
k top nodes
k top pods -A --sort-by=cpu
k top pods -A --sort-by=memory
```

> ⭐ **`k get events --sort-by=.lastTimestamp` là lệnh giá trị nhất trong troubleshooting.**
> Nó nói thẳng nguyên nhân trong 80% trường hợp. Đừng bỏ qua.
>
> Lọc theo object cụ thể:
> ```bash
> k get events -n dev --field-selector involvedObject.name=<pod>
> k events -n dev --for pod/<pod>          # cú pháp mới, gọn hơn
> ```

---

## 3. Node NotReady ⭐⭐ (dạng bài kinh điển)

### Quy trình
```bash
# 1. Xem lý do
k get nodes
k describe node <node> | grep -A15 Conditions
k describe node <node> | grep -A10 Events

# 2. SSH vào node
ssh <node>
sudo -i

# 3. kubelet — thủ phạm số 1
systemctl status kubelet
journalctl -u kubelet -n 100 --no-pager
journalctl -u kubelet -f                 # theo dõi real-time

# 4. Container runtime
systemctl status containerd
crictl ps
crictl info | head -30

# 5. Sửa xong
systemctl daemon-reload
systemctl restart kubelet
systemctl enable kubelet                 # nhớ enable nếu đề bảo "survive reboot"
```

### Bảng chẩn đoán theo Condition
| Condition | Nghĩa là | Xử lý |
|---|---|---|
| `Ready=False`, message `kubelet stopped posting node status` | kubelet chết/không chạy | `systemctl start kubelet`, đọc journalctl |
| `Ready=False`, `network plugin is not ready: cni config uninitialized` | Chưa cài CNI | Cài Calico/Flannel; kiểm tra `/etc/cni/net.d/` |
| `MemoryPressure=True` | Hết RAM | Dọn Pod, tăng RAM |
| `DiskPressure=True` | Hết đĩa | `df -h`, `crictl rmi --prune`, dọn `/var/log` |
| `PIDPressure=True` | Hết process ID | Giảm số Pod |
| `NetworkUnavailable=True` | Route mạng lỗi | Kiểm tra CNI |

### 8 nguyên nhân kubelet chết — kiểm tra theo thứ tự này
```bash
# 1. Service không chạy / không enable
systemctl is-active kubelet ; systemctl is-enabled kubelet

# 2. Config sai đường dẫn
cat /var/lib/kubelet/config.yaml
cat /etc/kubernetes/kubelet.conf | grep server        # ← IP/port apiserver ĐÚNG chưa?
ls /etc/systemd/system/kubelet.service.d/

# 3. Swap đang bật (K8s không cho)
free -h ; swapoff -a

# 4. cgroup driver lệch giữa kubelet và containerd
grep cgroupDriver /var/lib/kubelet/config.yaml        # phải là systemd
grep SystemdCgroup /etc/containerd/config.toml        # phải là true

# 5. Chứng chỉ hết hạn
openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -noout -dates
kubeadm certs check-expiration

# 6. Đầy đĩa
df -h /var/lib

# 7. containerd chết
systemctl status containerd

# 8. Sai port / firewall
ss -tulpn | grep -E '6443|10250'
```

> 🔴 Bẫy hay ra nhất: **`/etc/kubernetes/kubelet.conf` bị sửa sai port** (vd `:6444` thay vì `:6443`),
> hoặc **kubelet bị `systemctl stop` + `disable`**. Sửa xong nhớ cả `enable` lẫn `start`.

---

## 4. Pod lỗi — bảng tra theo trạng thái ⭐⭐

```bash
k get po <pod> -n <ns> -o wide
k describe po <pod> -n <ns>          # ← 90% câu trả lời nằm ở phần Events
k logs <pod> -n <ns> --previous
```

### 4.1 `Pending`
| Message trong Events | Nguyên nhân | Sửa |
|---|---|---|
| `0/3 nodes are available: 3 Insufficient cpu/memory` | Không đủ tài nguyên | Giảm `requests`, scale node, xóa Pod khác |
| `had taint {...}, that the pod didn't tolerate` | Thiếu toleration | Thêm toleration / gỡ taint |
| `didn't match Pod's node affinity/selector` | Sai `nodeSelector`/affinity | `k get nodes --show-labels` |
| `pod has unbound immediate PersistentVolumeClaims` | PVC chưa bind | [Storage](./04-storage.md#4-persistentvolumeclaim) |
| `waiting for first consumer` | SC `WaitForFirstConsumer` | **Bình thường**, không phải lỗi |
| `Failed quota: ...` | ResourceQuota chặn | Khai requests/limits |
| **Không có Event nào** | **Scheduler chết** | `k get po -n kube-system \| grep scheduler` |

### 4.2 `ImagePullBackOff` / `ErrImagePull`
```bash
k describe po <pod> | grep -A5 Events
```
| Message | Nguyên nhân |
|---|---|
| `not found` / `manifest unknown` | **Sai tên image hoặc sai tag** (phổ biến nhất) |
| `unauthorized` / `authentication required` | Registry private, thiếu `imagePullSecrets` |
| `dial tcp: i/o timeout` | Node không ra được internet / proxy |
| `no such host` | DNS trên node hỏng |

```bash
# Sửa nhanh
k set image po/<pod> <container>=nginx:1.27          # Pod thường không sửa được image → phải tạo lại
k set image deploy/<deploy> <container>=nginx:1.27   # Deployment thì OK

# Registry private
k create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=user --docker-password=pass
# rồi thêm vào Pod spec:
#   imagePullSecrets:
#   - name: regcred

# Kiểm tra image có trên node không
crictl images | grep nginx
crictl pull nginx:1.27
```

### 4.3 `CrashLoopBackOff` ⭐
```bash
k logs <pod> --previous              # ← LỆNH QUAN TRỌNG NHẤT ở đây
k describe po <pod> | grep -A5 'Last State'
```
| Nguyên nhân | Dấu hiệu | Sửa |
|---|---|---|
| App lỗi khi khởi động | Log có stack trace / error | Sửa config, env, cmd |
| **livenessProbe quá gắt** | `Last State: Terminated, Reason: Error`, `Liveness probe failed` trong Events | Tăng `initialDelaySeconds`, dùng `startupProbe` |
| **OOMKilled** | `Last State: Terminated, Reason: OOMKilled`, `Exit Code: 137` | Tăng `limits.memory` |
| Container chạy xong rồi thoát | `Exit Code: 0`, image kiểu `busybox` không có command dài | Thêm `command: ["sleep","3600"]` |
| Thiếu ConfigMap/Secret | Events: `CreateContainerConfigError` | Tạo cm/secret |
| Sai command/args | `Exit Code: 127` (command not found) | Sửa `command` |

**Bảng exit code:**
| Code | Nghĩa |
|---|---|
| `0` | Thoát bình thường (nhưng container thì không nên thoát) |
| `1` | Lỗi ứng dụng chung |
| `126` | Command không executable |
| `127` | Command not found |
| `137` | SIGKILL — **OOMKilled** hoặc bị `kubectl delete --force` |
| `143` | SIGTERM — bị dừng có trật tự |

### 4.4 `CreateContainerConfigError`
→ Thiếu **ConfigMap** hoặc **Secret** mà Pod tham chiếu.
```bash
k describe po <pod> | grep -i 'configmap\|secret'
k get cm,secret -n <ns>
```

### 4.5 `CreateContainerError`
→ Sai `command`, sai `securityContext`, hoặc mount xung đột.

### 4.6 `Init:0/1`, `Init:Error`, `Init:CrashLoopBackOff`
```bash
k logs <pod> -c <init-container-name>
k describe po <pod> | grep -A10 'Init Containers'
```
Init container phải **chạy xong (exit 0)** thì container chính mới bắt đầu.

### 4.7 `Terminating` mãi không xóa
```bash
k delete po <pod> --force --grace-period=0        # = $now
k get po <pod> -o jsonpath='{.metadata.finalizers}'
k patch po <pod> -p '{"metadata":{"finalizers":null}}'
```
Thường do: node chết, finalizer, hoặc `preStop` hook treo.

### 4.8 `Running` nhưng `READY 0/1`
→ **readinessProbe fail**. Pod chạy nhưng không nhận traffic.
```bash
k describe po <pod> | grep -i readiness
k exec <pod> -- curl -s localhost:8080/healthz
```

### 4.9 `Evicted`
→ Node hết tài nguyên (DiskPressure/MemoryPressure). Pod `BestEffort` bị đuổi trước.
```bash
k get po -A | grep Evicted
k delete po -A --field-selector status.phase=Failed
```

---

## 5. Control plane component chết ⭐⭐

**Đặc điểm:** `kubectl` báo `The connection to the server ... was refused`.
→ Không dùng được `kubectl`, phải làm việc trên node bằng `crictl` + đọc file.

```bash
ssh <control-plane-node>
sudo -i

# 1. Container nào chết?
crictl ps -a | grep -E 'apiserver|etcd|scheduler|controller'

# 2. Log của container đã chết
crictl logs <container-id>
crictl logs --tail=50 <container-id>

# 3. Log static pod trên đĩa (đọc được cả khi apiserver chết)
ls /var/log/pods/
cat /var/log/pods/kube-system_kube-apiserver-*/kube-apiserver/*.log | tail -50

# 4. kubelet log — nói vì sao nó không chạy được static pod
journalctl -u kubelet -n 100 --no-pager | grep -i apiserver

# 5. Kiểm tra manifest có lỗi cú pháp không
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

### Quy trình sửa manifest an toàn
```bash
# 1. Backup TRƯỚC KHI SỬA (kỷ luật này cứu bạn)
cp /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/apiserver.bak

# 2. Sửa
vim /etc/kubernetes/manifests/kube-apiserver.yaml

# 3. kubelet tự phát hiện và restart — CHỜ 30-60 giây
watch crictl ps | grep apiserver

# 4. Verify
k get nodes
```

> 🔴 **Mẹo cứu nguy khi apiserver không lên lại được:**
> Di chuyển file ra ngoài rồi đưa lại → ép kubelet đọc lại:
> ```bash
> mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/
> sleep 20
> mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
> ```

### Lỗi hay gặp theo component

| Component | Triệu chứng | Nguyên nhân hay gặp |
|---|---|---|
| **kube-apiserver** | `kubectl` connection refused | Sai flag trong manifest; sai đường dẫn cert; etcd không lên; port bị chiếm; sai indent YAML |
| **etcd** | apiserver báo `etcdserver: request timed out` | Sai `--data-dir`; đầy đĩa; mất quorum; cert hỏng |
| **kube-scheduler** | Pod `Pending` mãi, **không có Event** | Container chết; sai `--kubeconfig` |
| **kube-controller-manager** | Deployment không tạo ReplicaSet; Node không bị đánh dấu NotReady khi chết | Container chết; sai cert |
| **kube-proxy** | Service không hoạt động, Pod-to-Pod vẫn OK | DaemonSet lỗi; sai ConfigMap |
| **CoreDNS** | Resolve tên fail | Pod Pending (thiếu node); Corefile sai |

**Test nhanh scheduler:**
```bash
k run test --image=nginx
k get po test -o wide            # có được gán node không?
# Không có node + không Event ⇒ scheduler chết
```

**Test nhanh controller-manager:**
```bash
k create deploy test --image=nginx
k get rs                          # có ReplicaSet sinh ra không?
# Không có ⇒ controller-manager chết
```

---

## 6. Troubleshoot Service & Networking

Xem chi tiết ở [03-services-networking.md § 8](./03-services-networking.md#8-quy-trình-debug-mạng--theo-thứ-tự).
Tóm tắt quy trình 9 bước:

```bash
# 1-2. Pod Ready? Service có endpoint?
k get po -o wide -n <ns>
k get ep <svc> -n <ns>              # RỖNG = 90% vấn đề nằm ở đây

# 3. Label khớp?
k get po --show-labels -n <ns>
k describe svc <svc> -n <ns> | grep Selector

# 4-7. Test tầng tầng lớp lớp
k run tmp --image=nicolaka/netshoot --rm -it --restart=Never -n <ns> -- bash
  curl -v <pod-ip>:<container-port>      # tầng Pod
  curl -v <cluster-ip>:<svc-port>        # tầng Service (kube-proxy)
  curl -v <svc-name>.<ns>                # tầng DNS
  nslookup <svc-name>.<ns>

# 8. NetworkPolicy?
k get netpol -n <ns>

# 9. Từ ngoài vào
k port-forward svc/<svc> 8080:80 -n <ns>     # bypass toàn bộ ingress
curl localhost:8080
```

**Ma trận chẩn đoán:**
| Pod IP | ClusterIP | DNS name | Kết luận |
|:---:|:---:|:---:|---|
| ❌ | ❌ | ❌ | App không listen, hoặc CNI hỏng |
| ✅ | ❌ | ❌ | kube-proxy / iptables / endpoints |
| ✅ | ✅ | ❌ | **CoreDNS** |
| ✅ | ✅ | ✅ | Vấn đề ở tầng ngoài: Ingress / NodePort / firewall |

---

## 7. Monitoring — `k top` & metrics-server

```bash
k top nodes
k top pods -A
k top pods -n <ns> --sort-by=memory
k top pods --containers -n <ns>        # tách theo container
k top node <node>
```

**`k top` báo lỗi `Metrics API not available`:**
```bash
k get deploy metrics-server -n kube-system
k get apiservice v1beta1.metrics.k8s.io
k logs -n kube-system deploy/metrics-server
```
Nguyên nhân hay gặp: metrics-server chưa cài, hoặc lỗi TLS với kubelet →
cần cờ `--kubelet-insecure-tls` trong lab.

**Không có metrics-server thì xem tài nguyên bằng cách nào?**
```bash
k describe node <node> | grep -A10 'Allocated resources'
k describe node <node> | grep -A20 'Non-terminated Pods'
```

---

## 8. Container output streams (log)

```bash
# Cơ bản
k logs <pod> -n <ns>
k logs <pod> -c <container>
k logs <pod> --previous                    # lần chạy trước (CrashLoopBackOff)
k logs <pod> --tail=100
k logs <pod> --since=1h
k logs <pod> --since-time=2026-01-01T00:00:00Z
k logs <pod> -f                            # follow
k logs <pod> --timestamps

# Theo label / nhiều Pod
k logs -l app=web --all-containers --tail=50
k logs deploy/web --all-containers

# Ghi ra file (đề rất hay yêu cầu)
k logs <pod> -n <ns> > /opt/output.log
k logs <pod> | grep -i error > /opt/errors.txt

# Trên node — khi kubectl không dùng được
crictl logs <container-id>
ls /var/log/pods/<ns>_<pod>_<uid>/<container>/
ls /var/log/containers/
journalctl -u kubelet
```

**Dạng bài hay ra:**
> *"Tìm Pod nào ghi log chứa chuỗi `file-not-found` và ghi tên Pod vào `/opt/answer.txt`"*
```bash
for p in $(k get po -n <ns> -o name); do
  k logs -n <ns> $p 2>/dev/null | grep -q 'file-not-found' && echo $p
done
# hoặc nhanh hơn nếu Pod cùng label:
k logs -l app=x -n <ns> --all-containers --prefix | grep file-not-found
```

---

## 9. `kubectl debug` — công cụ mạnh, ít người biết

```bash
# 1. Thêm container debug vào Pod đang chạy (ephemeral container)
k debug -it <pod> --image=busybox --target=<container> -- sh

# 2. Copy Pod ra bản debug (đổi image, giữ nguyên config)
k debug <pod> -it --image=ubuntu --share-processes --copy-to=<pod>-debug

# 3. Debug NODE — mở shell trên host với /host mount
k debug node/<node> -it --image=busybox
  chroot /host
  systemctl status kubelet
```
> ⭐ `k debug node/<node>` cực hữu ích khi **không SSH được vào node**.

---

## 10. JSONPath & custom-columns — hay ra ở dạng "ghi ra file"

```bash
# Lấy 1 field
k get po <pod> -o jsonpath='{.status.podIP}'
k get svc <svc> -o jsonpath='{.spec.clusterIP}'
k get node <n> -o jsonpath='{.status.nodeInfo.kubeletVersion}'

# Duyệt danh sách
k get po -A -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.metadata.name}{"\n"}{end}'
k get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

# custom-columns (dễ đọc hơn jsonpath)
k get po -A -o custom-columns='NS:.metadata.namespace,NAME:.metadata.name,IMAGE:.spec.containers[*].image'
k get node -o custom-columns='NAME:.metadata.name,VERSION:.status.nodeInfo.kubeletVersion,OS:.status.nodeInfo.osImage'

# Sắp xếp
k get po -A --sort-by=.metadata.creationTimestamp
k get node --sort-by=.status.capacity.cpu
k get events --sort-by=.lastTimestamp

# Lọc bằng field-selector
k get po -A --field-selector status.phase=Running
k get po -A --field-selector spec.nodeName=<node>
k get events --field-selector type=Warning
```

**Dạng bài kinh điển:**
> *"Ghi tên node có nhiều CPU nhất vào `/opt/high-cpu.txt`"*
```bash
k get node --sort-by=.status.capacity.cpu -o jsonpath='{.items[-1].metadata.name}' > /opt/high-cpu.txt
```

---

## 11. Lab tự tạo lỗi — cách luyện hiệu quả nhất

Dựng cluster kubeadm 2 node, rồi **tự phá và tự sửa**. Mỗi lỗi làm 3 lần cho thành phản xạ:

| # | Cách phá | Triệu chứng bạn phải nhận ra |
|---|---|---|
| 1 | `systemctl stop kubelet` trên worker | Node NotReady |
| 2 | Sửa port trong `/etc/kubernetes/kubelet.conf` thành 6444 | Node NotReady, kubelet log báo connection refused |
| 3 | Đổi `SystemdCgroup = false` trong containerd | Node NotReady sau restart |
| 4 | Thêm flag rác vào `kube-apiserver.yaml` | `kubectl` connection refused |
| 5 | Đổi `--data-dir` của etcd sang đường dẫn không tồn tại | apiserver chết theo |
| 6 | Xóa `/etc/cni/net.d/*` | Node NotReady, `cni config uninitialized` |
| 7 | Đổi image Deployment thành `nginx:doesnotexist` | ImagePullBackOff |
| 8 | Đặt `limits.memory: 10Mi` cho app nặng | OOMKilled, exit 137 |
| 9 | Đặt `livenessProbe` port sai | CrashLoopBackOff |
| 10 | Đổi `selector` của Service | Endpoints rỗng |
| 11 | Scale CoreDNS về 0 | DNS fail, ClusterIP vẫn OK |
| 12 | Apply NetworkPolicy default-deny egress | App không gọi được gì, kể cả DNS |
| 13 | `k delete po -n kube-system <scheduler>` rồi sửa manifest sai | Pod mãi Pending không Event |
| 14 | Taint node `key=v:NoExecute` | Pod bị đuổi hết |
| 15 | Tạo PVC với SC không tồn tại | PVC Pending |

> 📖 Repo này có sẵn [devops-fast-track.md](../../../devops-fast-track.md) với 11 lab break-and-fix —
> dùng chung mục đích.

---

## 12. Dạng bài hay ra

| # | Đề bài | Hướng làm |
|---|---|---|
| 1 | Node `node01` NotReady — tìm và sửa, đảm bảo sống sau reboot | Mục 3; nhớ `systemctl enable` |
| 2 | Cluster không dùng được `kubectl` — sửa | Mục 5; `crictl` + `/var/log/pods/` |
| 3 | Pod `web` CrashLoopBackOff — tìm nguyên nhân, ghi vào file | `k logs --previous` |
| 4 | Deployment scale nhưng Pod không lên | Quota? taint? scheduler? |
| 5 | Service không truy cập được | Mục 6, kiểm tra endpoints trước |
| 6 | Ghi log của container `c1` trong Pod `p1` chứa "ERROR" ra `/opt/err.txt` | `k logs -c c1 \| grep ERROR >` |
| 7 | Tìm node dùng nhiều CPU nhất, ghi tên ra file | `k top node --sort-by=cpu` hoặc jsonpath |
| 8 | Pod Pending — chẩn đoán và sửa | Mục 4.1 |
| 9 | kube-proxy không chạy trên 1 node | `k get ds kube-proxy -n kube-system -o wide` |
| 10 | CoreDNS không resolve — sửa | Pod CoreDNS? svc kube-dns? Corefile? |
| 11 | Sửa `kubelet` để Pod trên node có thể chạy static pod từ `/etc/kubernetes/manifests` | `staticPodPath` trong `/var/lib/kubelet/config.yaml` |
| 12 | Đếm số Pod ở trạng thái Running trong mọi ns, ghi ra file | `k get po -A --field-selector status.phase=Running --no-headers \| wc -l` |

---

## 13. Bẫy & mẹo tổng kết

1. **`k describe` trước `k logs`.** Events nói nhiều hơn log ở giai đoạn đầu.
2. **`--previous` là chìa khóa của CrashLoopBackOff.** Log hiện tại thường rỗng.
3. **`k get events --sort-by=.lastTimestamp`** — lệnh sinh lời nhất.
4. **Backup file trước khi sửa manifest control plane.** `cp x.yaml /tmp/`.
5. **Sửa static pod phải CHỜ.** Đừng kết luận "hỏng" sau 5 giây.
6. **`crictl` thay `kubectl` khi apiserver chết.** Nhớ `crictl ps -a` (có `-a` mới thấy container đã chết).
7. **Endpoints rỗng** = manh mối vàng cho mọi sự cố Service.
8. **Đề bảo "make sure it survives reboot"** ⇒ `systemctl enable`, không chỉ `start`.
9. **Đề bảo ghi kết quả ra file** ⇒ ghi **đúng đường dẫn, đúng định dạng**. Đừng thêm header thừa.
   Kiểm tra lại bằng `cat`.
10. **Exit 137 = OOMKilled**, exit 127 = command not found. Thuộc 2 cái này.
11. **Không sa lầy.** Domain này dễ cuốn. Quá 8 phút chưa ra → flag, quay lại sau.
12. **Sau khi sửa, luôn verify**: `k get nodes`, `k get po -A | grep -v Running`.
