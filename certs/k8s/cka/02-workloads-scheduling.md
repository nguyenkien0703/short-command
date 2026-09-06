# CKA — Workloads & Scheduling (15%)

> Domain **dễ ăn điểm nhất**. Làm nhanh, gọn, hầu hết bằng lệnh imperative.
> Mục tiêu: mỗi câu domain này xong trong **3–4 phút**, để dành thời gian cho Troubleshooting.

**Nội dung curriculum v1.35:**
- Understand application deployments and how to perform rolling update and rollbacks
- Use ConfigMaps and Secrets to configure applications
- **Configure workload autoscaling** ← mới 2025
- Understand the primitives used to create robust, self-healing, application deployments
- Configure Pod admission and scheduling (limits, node affinity, etc.)

---

## 1. Sinh YAML nhanh — bộ lệnh xương sống ⭐

```bash
export do="--dry-run=client -o yaml"

# Pod
k run nginx --image=nginx $do > pod.yaml
k run nginx --image=nginx --port=80 --labels=app=web,tier=fe $do
k run busybox --image=busybox --restart=Never -- sleep 3600
k run tmp --image=busybox --rm -it --restart=Never -- sh     # pod tạm, tự xóa
k run nginx --image=nginx --env=KEY=value --env=K2=v2 $do

# Deployment
k create deploy web --image=nginx --replicas=3 $do > deploy.yaml
k create deploy web --image=nginx --port=80 --replicas=3 $do

# Job / CronJob
k create job pi --image=perl -- perl -Mbignum=bpi -wle 'print bpi(200)' $do
k create cronjob backup --image=busybox --schedule="*/5 * * * *" -- /bin/sh -c 'date' $do

# Service
k expose deploy web --port=80 --target-port=8080 --name=web-svc $do
k create svc clusterip web --tcp=80:8080 $do

# ConfigMap / Secret
k create cm app-cfg --from-literal=LOG=debug --from-literal=ENV=prod $do
k create cm app-cfg --from-file=./config.properties $do
k create cm app-cfg --from-env-file=./app.env $do
k create secret generic db --from-literal=user=admin --from-literal=pass=s3cr3t $do
k create secret docker-registry regcred --docker-server=x --docker-username=y --docker-password=z

# ServiceAccount
k create sa deploy-bot $do

# Namespace / Quota / LimitRange
k create ns dev $do
k create quota compute --hard=cpu=2,memory=4Gi,pods=10 -n dev $do
```

> 💡 `k run` chỉ tạo **Pod**. Muốn Deployment thì `k create deploy`.
> Trước K8s 1.18 `k run` tạo deployment — nhiều tài liệu cũ vẫn viết sai, cẩn thận.

---

## 2. Deployment — rolling update & rollback ⭐

```bash
# Đổi image (cách nhanh nhất)
k set image deploy/web nginx=nginx:1.27 --record
k set image deploy/web nginx=nginx:1.27          # --record đã deprecated nhưng vẫn dùng được

# Scale
k scale deploy web --replicas=5

# Theo dõi rollout
k rollout status deploy/web
k rollout history deploy/web
k rollout history deploy/web --revision=2         # xem chi tiết 1 revision

# Rollback
k rollout undo deploy/web                          # về revision trước
k rollout undo deploy/web --to-revision=2

# Tạm dừng / tiếp tục (gom nhiều thay đổi thành 1 rollout)
k rollout pause deploy/web
k set resources deploy/web -c=nginx --limits=cpu=200m,memory=512Mi
k rollout resume deploy/web

# Restart (rolling, không đổi spec)
k rollout restart deploy/web
```

### Chiến lược rollout
```yaml
spec:
  strategy:
    type: RollingUpdate          # hoặc Recreate (giết hết rồi tạo mới)
    rollingUpdate:
      maxSurge: 25%              # tạo thêm tối đa bao nhiêu Pod so với replicas
      maxUnavailable: 25%        # cho phép thiếu tối đa bao nhiêu Pod
  minReadySeconds: 10            # Pod phải Ready bao lâu mới tính là available
  revisionHistoryLimit: 10       # giữ bao nhiêu ReplicaSet cũ để rollback
  progressDeadlineSeconds: 600   # quá hạn → rollout bị đánh dấu Failed
```

| Muốn | Đặt |
|---|---|
| Zero-downtime tuyệt đối | `maxUnavailable: 0`, `maxSurge: 1` (cần dư tài nguyên) |
| Không tăng tài nguyên | `maxSurge: 0`, `maxUnavailable: 1` (chấp nhận giảm capacity) |
| Thay toàn bộ 1 lần (app không chạy song song 2 version được) | `type: Recreate` |

> 🔴 Bẫy: `k rollout undo` **không quay lại được** nếu `revisionHistoryLimit: 0`.
> Và ReplicaSet cũ bị xóa thì mất luôn lịch sử.

---

## 3. Các loại workload

| Kind | Dùng khi | Đặc điểm quan trọng |
|---|---|---|
| **Deployment** | App stateless | Quản lý ReplicaSet, rolling update, rollback |
| **ReplicaSet** | Hiếm khi tạo tay | Deployment quản lý hộ |
| **StatefulSet** | DB, app cần identity ổn định | Tên `web-0,1,2` cố định; PVC riêng mỗi Pod qua `volumeClaimTemplates`; tạo/xóa **tuần tự**; cần **headless Service** |
| **DaemonSet** | Agent trên mọi node (log, CNI, monitoring) | 1 Pod/node; tự chạy trên node mới; **tự tolerate** một số taint |
| **Job** | Chạy 1 lần rồi xong | `completions`, `parallelism`, `backoffLimit`, `restartPolicy: Never/OnFailure` |
| **CronJob** | Chạy theo lịch | `schedule` cron, `concurrencyPolicy`, `successfulJobsHistoryLimit` |
| **Static Pod** | Control plane | kubelet tự chạy từ `/etc/kubernetes/manifests/`, không qua scheduler, tên có hậu tố `-<nodename>` |

### StatefulSet — điểm hay ra đề
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: web-headless        # ← BẮT BUỘC, trỏ tới headless svc (clusterIP: None)
  replicas: 3
  selector:
    matchLabels: {app: web}
  template:
    metadata:
      labels: {app: web}
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: data
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:            # ← mỗi Pod 1 PVC riêng: data-web-0, data-web-1...
  - metadata: {name: data}
    spec:
      accessModes: ["ReadWriteOnce"]
      resources: {requests: {storage: 1Gi}}
```
> DNS của StatefulSet: `web-0.web-headless.<ns>.svc.cluster.local`
> Xóa StatefulSet **không xóa PVC** — phải xóa tay.

### Job
```yaml
spec:
  completions: 5        # cần 5 Pod hoàn thành
  parallelism: 2        # chạy tối đa 2 Pod cùng lúc
  backoffLimit: 4       # thử lại tối đa 4 lần trước khi Failed
  activeDeadlineSeconds: 100
  ttlSecondsAfterFinished: 300   # tự xóa Job sau khi xong
  template:
    spec:
      restartPolicy: Never       # BẮT BUỘC: Never hoặc OnFailure
```

### CronJob
```yaml
spec:
  schedule: "*/5 * * * *"
  concurrencyPolicy: Forbid      # Allow (mặc định) | Forbid | Replace
  startingDeadlineSeconds: 60
  suspend: false                 # true = tạm dừng
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
```
```text
Cron format:  ┌ phút (0-59)
              │ ┌ giờ (0-23)
              │ │ ┌ ngày trong tháng (1-31)
              │ │ │ ┌ tháng (1-12)
              │ │ │ │ ┌ thứ (0-6, 0=CN)
              * * * * *
```

---

## 4. ConfigMap & Secret

### 3 cách đưa vào Pod
```yaml
spec:
  containers:
  - name: app
    image: nginx

    # (1) Từng biến một
    env:
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef: {name: app-cfg, key: LOG}
    - name: DB_PASS
      valueFrom:
        secretKeyRef: {name: db, key: pass}

    # (2) Nạp TOÀN BỘ cm/secret thành env
    envFrom:
    - configMapRef: {name: app-cfg}
    - secretRef: {name: db}

    # (3) Mount thành file (mỗi key = 1 file)
    volumeMounts:
    - name: cfg
      mountPath: /etc/config
      readOnly: true
  volumes:
  - name: cfg
    configMap:
      name: app-cfg
      items:                          # chỉ lấy 1 số key (tuỳ chọn)
      - key: app.properties
        path: application.properties
  - name: sec
    secret:
      secretName: db
      defaultMode: 0400
```

| Điểm | Chi tiết |
|---|---|
| Secret **không mã hóa**, chỉ base64 | `echo -n 'pass' \| base64` / `base64 -d` để giải |
| Đọc secret nhanh | `k get secret db -o jsonpath='{.data.pass}' \| base64 -d` |
| ConfigMap mount dạng volume **tự cập nhật** khi cm đổi | (~1 phút, và chỉ khi không dùng `subPath`) |
| ConfigMap dùng qua `env` **KHÔNG** tự cập nhật | Phải restart Pod |
| `subPath` phá vỡ auto-update | Nhớ khi đề hỏi "vì sao config không đổi" |
| Secret type | `Opaque` (mặc định), `kubernetes.io/dockerconfigjson`, `kubernetes.io/tls`, `kubernetes.io/service-account-token` |

---

## 5. Self-healing primitives (probes)

```yaml
containers:
- name: app
  image: myapp

  livenessProbe:                    # thất bại → RESTART container
    httpGet: {path: /healthz, port: 8080}
    initialDelaySeconds: 15
    periodSeconds: 10
    timeoutSeconds: 1
    failureThreshold: 3

  readinessProbe:                   # thất bại → GỠ khỏi Service endpoints (không restart)
    httpGet: {path: /ready, port: 8080}
    periodSeconds: 5

  startupProbe:                     # app khởi động chậm; liveness/readiness bị hoãn tới khi cái này pass
    httpGet: {path: /healthz, port: 8080}
    failureThreshold: 30
    periodSeconds: 10               # → cho phép tối đa 300s để khởi động
```

3 kiểu probe: `httpGet` · `tcpSocket` · `exec` (`command: ["cat","/tmp/healthy"]`).

| Probe | Fail thì sao | Dùng để |
|---|---|---|
| **liveness** | Restart container | App treo/deadlock |
| **readiness** | Xóa khỏi endpoints, Pod vẫn chạy | App chưa sẵn sàng nhận traffic (đang warm-up, mất kết nối DB) |
| **startup** | Restart container | Bảo vệ app khởi động chậm khỏi bị liveness giết oan |

> 🔴 Bẫy phổ biến: liveness quá gắt (`initialDelaySeconds` nhỏ) → Pod `CrashLoopBackOff`
> dù app hoàn toàn khỏe. Đây là dạng câu troubleshooting rất hay ra.

**Restart policy** (chỉ có ở cấp Pod): `Always` (mặc định, Deployment) · `OnFailure` · `Never` (Job).

---

## 6. Resources — requests & limits

```yaml
resources:
  requests:            # dùng để SCHEDULE (chọn node)
    cpu: "100m"        # 100 milli = 0.1 core
    memory: "128Mi"
  limits:              # trần khi CHẠY
    cpu: "500m"        # vượt → bị throttle (chậm lại)
    memory: "512Mi"    # vượt → bị OOMKilled (giết)
```

**QoS class** (đề hay hỏi, xem bằng `k describe po | grep QoS`):

| QoS | Điều kiện | Bị evict |
|---|---|---|
| `Guaranteed` | requests **==** limits cho **mọi** container, cả cpu lẫn memory | Cuối cùng |
| `Burstable` | Có requests nhưng không bằng limits | Giữa |
| `BestEffort` | Không đặt requests lẫn limits | **Đầu tiên** |

### LimitRange & ResourceQuota
```yaml
# LimitRange: mặc định + trần cho TỪNG Pod/Container trong ns
apiVersion: v1
kind: LimitRange
metadata: {name: mem-limit, namespace: dev}
spec:
  limits:
  - type: Container
    default:        {cpu: 500m, memory: 512Mi}   # limits mặc định
    defaultRequest: {cpu: 100m, memory: 128Mi}   # requests mặc định
    max:            {cpu: "2",  memory: 2Gi}
    min:            {cpu: 50m,  memory: 64Mi}
---
# ResourceQuota: trần TỔNG của cả namespace
apiVersion: v1
kind: ResourceQuota
metadata: {name: compute, namespace: dev}
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "20"
    persistentvolumeclaims: "5"
    count/deployments.apps: "10"
```
> 🔴 Khi ns có **ResourceQuota đặt `requests.cpu`**, mọi Pod tạo mới **bắt buộc** phải khai requests,
> nếu không sẽ bị từ chối: `Failed quota: must specify requests.cpu`.
> LimitRange giải quyết bằng cách gán giá trị mặc định.

```bash
k describe quota -n dev            # xem đã dùng bao nhiêu / trần bao nhiêu
k describe limitrange -n dev
```

---

## 7. Autoscaling — HPA ⭐ (mới trong CKA 2025)

```bash
# Điều kiện: metrics-server phải chạy
k top nodes && k top pods          # nếu lỗi → metrics-server chưa có/chưa sẵn sàng

# Tạo HPA nhanh
k autoscale deploy web --min=2 --max=10 --cpu-percent=70

k get hpa
k get hpa web -o yaml
k describe hpa web                 # xem lý do scale / lỗi "unable to get metrics"
```

**YAML (autoscaling/v2 — dạng đầy đủ):**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: {name: web}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70      # % so với requests.cpu
  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 500Mi
  behavior:                          # tinh chỉnh tốc độ scale
    scaleDown:
      stabilizationWindowSeconds: 300
```

> 🔴 **HPA theo `Utilization` cần Pod có `resources.requests.cpu`**.
> Không có requests → HPA báo `<unknown>/70%` và không scale. Đây là bẫy số 1 của HPA.

**Các loại autoscaler (biết để phân biệt):**
| | Scale gì |
|---|---|
| **HPA** | Số lượng Pod (ngang) |
| **VPA** | requests/limits của Pod (dọc) — không có sẵn trong K8s core |
| **Cluster Autoscaler** | Số lượng Node |

---

## 8. Scheduling — đưa Pod về đúng node ⭐

### 8.1 nodeSelector — đơn giản nhất
```bash
k label node worker-1 disktype=ssd
k get nodes --show-labels
k label node worker-1 disktype-          # xóa label (dấu trừ ở cuối)
```
```yaml
spec:
  nodeSelector:
    disktype: ssd
```

### 8.2 nodeName — bỏ qua scheduler hoàn toàn
```yaml
spec:
  nodeName: worker-2
```
> Dùng khi đề nói "schedule Pod này lên node X **mà không dùng scheduler**".

### 8.3 Node Affinity — linh hoạt hơn nodeSelector
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:      # BẮT BUỘC (hard)
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In                # In | NotIn | Exists | DoesNotExist | Gt | Lt
            values: ["ssd", "nvme"]
      preferredDuringSchedulingIgnoredDuringExecution:     # ƯU TIÊN (soft)
      - weight: 50
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values: ["us-east-1a"]
```

| Tên dài khó nhớ — giải mã | |
|---|---|
| `requiredDuringScheduling` | Bắt buộc thỏa mãn mới được schedule |
| `preferredDuringScheduling` | Cố gắng, không thỏa vẫn schedule |
| `IgnoredDuringExecution` | Đang chạy mà node đổi label thì **không** đuổi Pod đi |

### 8.4 Taint & Toleration — node "đẩy" Pod ra
```bash
k taint node worker-1 key=value:NoSchedule
k taint node worker-1 key=value:NoSchedule-        # xóa taint (dấu trừ)
k describe node worker-1 | grep -i taint
```
```yaml
spec:
  tolerations:
  - key: "key"
    operator: "Equal"        # Equal | Exists
    value: "value"
    effect: "NoSchedule"     # NoSchedule | PreferNoSchedule | NoExecute
  - operator: "Exists"       # tolerate MỌI taint
  - key: "node.kubernetes.io/not-ready"
    operator: "Exists"
    effect: "NoExecute"
    tolerationSeconds: 300   # ở lại 300s rồi mới bị đuổi
```

| Effect | Ý nghĩa |
|---|---|
| `NoSchedule` | Pod mới không được lên; Pod cũ ở nguyên |
| `PreferNoSchedule` | Cố tránh, không bắt buộc |
| `NoExecute` | Pod mới không lên **và** Pod cũ **bị đuổi** ngay |

> **Taint/Toleration vs Node Affinity** — câu hỏi khái niệm hay ra:
> - Taint = **node từ chối Pod** (trừ Pod có toleration).
> - Affinity = **Pod chọn node**.
> - Toleration **không** đảm bảo Pod lên đúng node đó — chỉ cho phép. Muốn ép thì kết hợp với affinity/nodeSelector.
> - Control plane mặc định có taint `node-role.kubernetes.io/control-plane:NoSchedule`.

### 8.5 Pod Affinity / Anti-Affinity
```yaml
affinity:
  podAntiAffinity:                                       # trải Pod ra các node khác nhau
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels: {app: web}
      topologyKey: kubernetes.io/hostname                # "cùng node" = cùng giá trị key này
  podAffinity:                                           # đặt gần Pod khác
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        labelSelector:
          matchLabels: {app: cache}
        topologyKey: topology.kubernetes.io/zone
```

### 8.6 Topology Spread Constraints
```yaml
spec:
  topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule       # hoặc ScheduleAnyway
    labelSelector:
      matchLabels: {app: web}
```

### 8.7 PriorityClass & Preemption
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata: {name: high-priority}
value: 1000000
globalDefault: false
preemptionPolicy: PreemptLowerPriority     # hoặc Never
description: "Cho workload quan trọng"
```
Dùng: `spec.priorityClassName: high-priority`.

### 8.8 Multiple schedulers
```yaml
spec:
  schedulerName: my-custom-scheduler
```

---

## 9. Debug scheduling — Pod `Pending`

```bash
k get po -o wide
k describe po <pod> | tail -25          # phần Events nói rõ lý do
```

| Message trong Events | Nguyên nhân | Sửa |
|---|---|---|
| `0/3 nodes are available: 3 Insufficient cpu` | Không đủ tài nguyên | Giảm requests / scale node |
| `... had taint {key: value}, that the pod didn't tolerate` | Thiếu toleration | Thêm toleration hoặc gỡ taint |
| `... didn't match Pod's node affinity/selector` | Sai nodeSelector/affinity | Kiểm tra label node |
| `... had volume node affinity conflict` | PV bị ràng buộc vào node/zone khác | Sửa PV/SC |
| `pod has unbound immediate PersistentVolumeClaims` | PVC chưa bind | Xem [04-storage](./04-storage.md) |
| `Failed quota: ...` | ResourceQuota chặn | Khai requests/limits hoặc nới quota |
| Pending mà **không có Event nào** | Scheduler chết | `k get po -n kube-system \| grep scheduler` |

---

## 10. Dạng bài hay ra

| # | Đề bài | Hướng làm |
|---|---|---|
| 1 | Tạo Deployment 3 replica, image nginx:1.26, expose ClusterIP port 80 | `k create deploy` + `k expose` |
| 2 | Rolling update lên nginx:1.27, rồi rollback | `k set image` → `k rollout undo` |
| 3 | Tạo Pod có 2 container chia sẻ `emptyDir` | `$do` rồi thêm container thứ 2 |
| 4 | Tạo CronJob chạy mỗi 2 phút, giữ 3 job thành công | `k create cronjob` + sửa `successfulJobsHistoryLimit` |
| 5 | Tạo ConfigMap từ file, mount vào `/etc/config` | `k create cm --from-file` + volumes |
| 6 | Pod chỉ được chạy trên node có label `gpu=true` | `nodeSelector` |
| 7 | Taint node, tạo Pod tolerate được taint đó | `k taint` + `tolerations` |
| 8 | Tạo HPA min=2 max=8 khi CPU > 60% | `k autoscale` — nhớ kiểm tra Deployment có `requests.cpu` |
| 9 | Sửa Deployment để đạt QoS `Guaranteed` | requests == limits ở mọi container |
| 10 | Pod Pending — tìm nguyên nhân & sửa | Mục 9 |
| 11 | Scale StatefulSet, kiểm tra thứ tự Pod | `k scale sts` + `k get po -w` |
| 12 | Thêm readinessProbe HTTP `/healthz` port 8080 | Mục 5 |

---

## 11. Bẫy tổng kết

1. **`k run` = Pod**, `k create deploy` = Deployment. Không nhầm.
2. **Quên `-n <ns>`** — lỗi số 1 khiến mất điểm oan.
3. **`restartPolicy` của Job phải là `Never`/`OnFailure`**, không được `Always`.
4. **HPA cần `requests.cpu`** — không có thì `<unknown>/70%`.
5. **`k expose` lấy selector từ resource gốc**, không phải từ label bạn tự đặt.
6. **ConfigMap qua `env` không auto-reload**; qua volume thì có (trừ khi dùng `subPath`).
7. **Secret chỉ base64, không mã hóa** — hỏi khái niệm hay ra ở CKS.
8. **liveness fail = restart, readiness fail = gỡ khỏi endpoint.** Đừng lẫn.
9. **StatefulSet cần `serviceName`** trỏ tới headless Service (`clusterIP: None`).
10. **Xóa StatefulSet không xóa PVC.**
11. **Taint có dấu `-` ở cuối để xóa**: `k taint node n1 key=value:NoSchedule-`.
12. **`--record` đã deprecated** nhưng vẫn chạy; đừng hoảng khi thấy warning.
