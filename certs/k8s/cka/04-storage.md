# CKA — Storage (10%)

> Domain nhẹ nhất nhưng **dễ ăn trọn điểm**. Thường 1–2 câu, làm trong 4–5 phút.
> Đừng bỏ qua chỉ vì trọng số thấp — 10% là ~6–7 điểm, gần bằng khoảng cách đậu/trượt.

**Nội dung curriculum v1.35:**
- Implement storage classes and dynamic volume provisioning
- Configure volume types, access modes and reclaim policies
- Manage persistent volumes and persistent volume claims

---

## 1. Bức tranh tổng thể

```text
   ┌──────────────────────────────────────────────────────────┐
   │  STATIC provisioning                                      │
   │  Admin tạo PV  ──── bind ────►  PVC (dev tạo) ──► Pod     │
   └──────────────────────────────────────────────────────────┘

   ┌──────────────────────────────────────────────────────────┐
   │  DYNAMIC provisioning  (phổ biến hơn)                     │
   │  PVC (có storageClassName)                                │
   │        │                                                   │
   │        ▼                                                   │
   │  StorageClass ──► provisioner (CSI driver) ──► tạo PV tự  │
   │        │                                                   │
   │        └──── bind ────► PVC ──► Pod                       │
   └──────────────────────────────────────────────────────────┘
```

| Khái niệm | Là gì | Phạm vi |
|---|---|---|
| **Volume** | Thư mục gắn vào container, vòng đời **theo Pod** | Pod |
| **PersistentVolume (PV)** | Miếng storage thật trong cluster | **Cluster-scoped** |
| **PersistentVolumeClaim (PVC)** | Yêu cầu xin storage của user | **Namespaced** |
| **StorageClass (SC)** | "Loại" storage + provisioner để cấp phát động | **Cluster-scoped** |

> 🔴 PV là **cluster-scoped** (không có namespace), PVC là **namespaced**.
> Câu hỏi khái niệm hay ra chỗ này.

---

## 2. Volume types — dùng trực tiếp trong Pod

```yaml
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: cache
      mountPath: /cache
    - name: cfg
      mountPath: /etc/nginx/conf.d
    - name: data
      mountPath: /data
      subPath: mydir              # chỉ mount 1 thư mục con
  volumes:
  # 1. emptyDir — thư mục rỗng, MẤT khi Pod bị xóa. Chia sẻ giữa các container.
  - name: cache
    emptyDir: {}
  - name: ram-cache
    emptyDir:
      medium: Memory              # tmpfs (RAM), tính vào memory limit
      sizeLimit: 500Mi

  # 2. hostPath — thư mục trên NODE. Nguy hiểm về bảo mật (CKS hỏi nhiều).
  - name: logs
    hostPath:
      path: /var/log
      type: Directory             # Directory | DirectoryOrCreate | File | FileOrCreate | Socket

  # 3. configMap / secret
  - name: cfg
    configMap:
      name: nginx-conf
      defaultMode: 0644
  - name: sec
    secret:
      secretName: db-secret

  # 4. persistentVolumeClaim — dùng nhiều nhất trong thực tế
  - name: data
    persistentVolumeClaim:
      claimName: my-pvc

  # 5. projected — gộp nhiều nguồn vào 1 volume
  - name: all-in-one
    projected:
      sources:
      - configMap: {name: cfg}
      - secret: {name: sec}
      - serviceAccountToken:
          path: token
          expirationSeconds: 3600
          audience: vault

  # 6. downwardAPI — đưa metadata của Pod thành file
  - name: podinfo
    downwardAPI:
      items:
      - path: labels
        fieldRef: {fieldPath: metadata.labels}
      - path: cpu_limit
        resourceFieldRef: {containerName: app, resource: limits.cpu}
```

| Type | Vòng đời | Dùng khi |
|---|---|---|
| `emptyDir` | Theo Pod (xóa Pod = mất data) | Cache, chia sẻ giữa container trong 1 Pod, scratch |
| `hostPath` | Theo node | Agent cần đọc file node (log, socket). **Rủi ro bảo mật** |
| `configMap`/`secret` | Theo object | Config, cert |
| `persistentVolumeClaim` | Độc lập với Pod | Data thật cần bền |

---

## 3. PersistentVolume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
  labels: {type: local}
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain    # Retain | Delete | Recycle(deprecated)
  storageClassName: manual                 # phải KHỚP với PVC
  volumeMode: Filesystem                   # Filesystem (mặc định) | Block
  hostPath:
    path: /mnt/data
  # nodeAffinity: ép PV chỉ dùng được ở 1 node (local volume)
  # nodeAffinity:
  #   required:
  #     nodeSelectorTerms:
  #     - matchExpressions:
  #       - {key: kubernetes.io/hostname, operator: In, values: [worker-1]}
```

### Access Modes ⭐ (chắc chắn hỏi)
| Mode | Viết tắt | Ý nghĩa |
|---|---|---|
| `ReadWriteOnce` | **RWO** | Mount đọc-ghi bởi **1 node** (nhiều Pod trên cùng node đó vẫn OK) |
| `ReadOnlyMany` | **ROX** | Mount chỉ-đọc bởi **nhiều node** |
| `ReadWriteMany` | **RWX** | Mount đọc-ghi bởi **nhiều node** (cần NFS/CephFS/EFS — EBS không hỗ trợ) |
| `ReadWriteOncePod` | **RWOP** | Chỉ **đúng 1 Pod** duy nhất trong toàn cluster |

> 🔴 `ReadWriteOnce` = **một NODE**, không phải một Pod. Nhầm chỗ này rất phổ biến.

### Reclaim Policy ⭐
| Policy | Khi xóa PVC thì PV... |
|---|---|
| `Retain` | Giữ nguyên data, PV chuyển sang trạng thái **`Released`** — phải xóa/dọn tay mới dùng lại |
| `Delete` | Xóa luôn PV **và** storage thật (mặc định của dynamic provisioning) |
| `Recycle` | Đã deprecated — xóa sạch nội dung rồi cho dùng lại |

> 🔴 PV `Released` **không tự bind lại được** với PVC mới, dù cùng spec.
> Phải xóa `spec.claimRef` trong PV (hoặc xóa PV rồi tạo lại).
> ```bash
> k patch pv pv-data -p '{"spec":{"claimRef": null}}'
> ```

### Trạng thái PV
```text
Available ──► Bound ──► Released ──► Failed
   ▲             │
   └─────────────┘  (chỉ khi xóa claimRef thủ công)
```

---

## 4. PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
  namespace: dev
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 3Gi
  storageClassName: manual        # "" = tắt dynamic, chỉ bind PV có SC rỗng
  # selector:                     # (tuỳ chọn) chọn PV theo label
  #   matchLabels: {type: local}
```

### Điều kiện để PVC bind được với PV
Cả **4** phải thỏa:
1. `storageClassName` **khớp nhau** (kể cả cùng rỗng)
2. `accessModes` PVC ⊆ accessModes PV
3. `capacity` PV **≥** `requests.storage` PVC
4. PV đang ở trạng thái `Available` (và `volumeMode` khớp)

> PV **5Gi** có thể bind cho PVC xin **3Gi** — Pod nhận nguyên 5Gi.
> Ngược lại thì không.

```bash
k get pv,pvc -A
k describe pvc my-pvc -n dev        # Events nói rõ vì sao chưa bind
```

| PVC `Pending` — nguyên nhân | Cách tìm |
|---|---|
| Không có PV nào khớp | `k get pv` — xem SC/accessModes/size |
| Sai `storageClassName` | So sánh PVC ↔ PV ↔ `k get sc` |
| SC không tồn tại / không có provisioner | `k get sc`, `k get po -n kube-system \| grep provisioner` |
| SC dùng `WaitForFirstConsumer` | **Bình thường!** PVC chỉ bind khi có Pod dùng nó |
| PV đang `Released` | Xóa `claimRef` |

---

## 5. StorageClass & Dynamic Provisioning ⭐

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"   # SC mặc định
provisioner: kubernetes.io/no-provisioner    # hoặc ebs.csi.aws.com, rancher.io/local-path...
parameters:
  type: gp3
  fsType: ext4
reclaimPolicy: Delete                        # PV sinh ra sẽ có policy này
allowVolumeExpansion: true                   # cho phép resize PVC
volumeBindingMode: WaitForFirstConsumer      # Immediate | WaitForFirstConsumer
```

| `volumeBindingMode` | Hành vi |
|---|---|
| `Immediate` (mặc định) | Tạo PV & bind ngay khi PVC được tạo |
| `WaitForFirstConsumer` | **Chờ có Pod dùng PVC** mới tạo PV → PV được đặt đúng zone/node của Pod |

> 🔴 Thấy PVC `Pending` với message `waiting for first consumer to be created before binding`
> → **KHÔNG phải lỗi**. Tạo Pod dùng PVC đó là nó bind.

```bash
k get sc
k get sc -o custom-columns='NAME:.metadata.name,PROVISIONER:.provisioner,DEFAULT:.metadata.annotations.storageclass\.kubernetes\.io/is-default-class'
k describe sc fast

# Đặt SC mặc định
k patch sc fast -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
# Bỏ mặc định cho SC cũ
k patch sc standard -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
```

**SC đặc biệt — không dùng dynamic:**
```yaml
storageClassName: ""     # trong PVC: chỉ bind với PV có storageClassName rỗng, KHÔNG dùng SC mặc định
```
Bỏ hẳn field `storageClassName` → dùng **SC mặc định** (nếu có).

---

## 6. Resize (mở rộng) PVC

```bash
# Điều kiện: SC có allowVolumeExpansion: true
k edit pvc my-pvc -n dev
#   spec.resources.requests.storage: 3Gi → 10Gi

# hoặc
k patch pvc my-pvc -n dev -p '{"spec":{"resources":{"requests":{"storage":"10Gi"}}}}'

k get pvc my-pvc -n dev -o jsonpath='{.status.capacity.storage}'
```
- Chỉ **tăng**, không giảm được.
- Một số driver cần **restart Pod** để filesystem nhận size mới.
- `k describe pvc` sẽ có condition `FileSystemResizePending`.

---

## 7. Lab đầy đủ — làm 1 lần là nhớ

```bash
# ---------- 1. PV ----------
cat <<'EOF' | k apply -f -
apiVersion: v1
kind: PersistentVolume
metadata: {name: pv-manual}
spec:
  capacity: {storage: 2Gi}
  accessModes: [ReadWriteOnce]
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath: {path: /mnt/data, type: DirectoryOrCreate}
EOF

# ---------- 2. PVC ----------
cat <<'EOF' | k apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata: {name: pvc-manual, namespace: default}
spec:
  accessModes: [ReadWriteOnce]
  storageClassName: manual
  resources: {requests: {storage: 1Gi}}
EOF

k get pv,pvc          # PV phải chuyển Available → Bound

# ---------- 3. Pod dùng PVC ----------
cat <<'EOF' | k apply -f -
apiVersion: v1
kind: Pod
metadata: {name: app}
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts: [{name: data, mountPath: /usr/share/nginx/html}]
  volumes:
  - name: data
    persistentVolumeClaim: {claimName: pvc-manual}
EOF

# ---------- 4. Verify ----------
k exec app -- sh -c 'echo hello > /usr/share/nginx/html/index.html'
k exec app -- cat /usr/share/nginx/html/index.html
k delete po app && k apply -f -  <<< "..."   # tạo lại → data vẫn còn (đó là ý nghĩa của PV)
```

---

## 8. Dạng bài hay ra

| # | Đề bài | Hướng làm |
|---|---|---|
| 1 | Tạo PV 2Gi, hostPath `/mnt/data`, RWO, reclaim `Retain` | Mục 3 |
| 2 | Tạo PVC 1Gi bind vào PV trên, rồi cho Pod dùng | Mục 4 + 7 |
| 3 | PVC đang `Pending` — tìm và sửa | Mục 4, so 4 điều kiện bind |
| 4 | Mở rộng PVC từ 1Gi lên 5Gi | Mục 6 — kiểm tra `allowVolumeExpansion` trước |
| 5 | Đặt StorageClass `fast` làm mặc định, bỏ mặc định của cái cũ | `k patch sc` |
| 6 | Tạo Pod 2 container chia sẻ `emptyDir`, container A ghi, B đọc | Mục 2 |
| 7 | Đổi reclaimPolicy của PV từ Delete sang Retain | `k patch pv <n> -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'` |
| 8 | Tạo SC dùng `WaitForFirstConsumer` | Mục 5 |
| 9 | Liệt kê PV sắp xếp theo capacity ghi ra file | `k get pv --sort-by=.spec.capacity.storage` |
| 10 | Pod không mount được volume — chẩn đoán | `k describe po` → Events (FailedMount / FailedAttachVolume) |

**Câu 9 hay ra ở dạng jsonpath:**
```bash
k get pv --sort-by=.spec.capacity.storage
k get pv --sort-by=.spec.capacity.storage -o custom-columns=NAME:.metadata.name,CAP:.spec.capacity.storage
k get pv -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.capacity.storage}{"\n"}{end}'
```

---

## 9. Bẫy tổng kết

1. **`ReadWriteOnce` = 1 NODE**, không phải 1 Pod. (`ReadWriteOncePod` mới là 1 Pod.)
2. **PV cluster-scoped, PVC namespaced.** Đừng thêm `-n` cho PV.
3. **`storageClassName` phải khớp chính xác** giữa PV và PVC — kể cả khi cả hai đều rỗng.
4. **PV lớn hơn PVC thì bind được**; nhỏ hơn thì không.
5. **PV `Released` không tự tái sử dụng** — phải xóa `claimRef`.
6. **`WaitForFirstConsumer` → PVC Pending là bình thường**, chờ Pod.
7. **`emptyDir` mất khi Pod bị xóa** (không phải khi container restart).
8. **`hostPath` chỉ dùng được ở cluster 1 node** hoặc kèm `nodeAffinity` — Pod chuyển node là mất data.
9. **Resize chỉ tăng, cần `allowVolumeExpansion: true`.**
10. **Bỏ `storageClassName` ≠ đặt `storageClassName: ""`.** Cái đầu dùng SC mặc định; cái sau tắt hẳn dynamic.
11. **Xóa PVC khi Pod còn dùng** → PVC kẹt ở `Terminating` (do finalizer `kubernetes.io/pvc-protection`).
    Xóa Pod trước.
