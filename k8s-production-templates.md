# Kubernetes Production Templates & Runbooks

> Bộ **manifest production-ready (copy-paste được)** và **runbook từng bước** để vận hành thực tế.
> Bổ trợ cho [`k8s-operations-playbook.md`](./k8s-operations-playbook.md) (phần lý thuyết & chẩn đoán).

## 📑 Mục lục

- [A. Deployment chuẩn production](#a-deployment-chuẩn-production)
- [B. PodDisruptionBudget](#b-poddisruptionbudget)
- [C. HorizontalPodAutoscaler](#c-horizontalpodautoscaler)
- [D. NetworkPolicy (default-deny + allow)](#d-networkpolicy-default-deny--allow)
- [E. ResourceQuota & LimitRange](#e-resourcequota--limitrange)
- [F. Pod Security & RBAC tối thiểu](#f-pod-security--rbac-tối-thiểu)
- [G. VolumeSnapshot (backup PVC)](#g-volumesnapshot-backup-pvc)
- [H. Runbook: Backup & Restore etcd](#h-runbook-backup--restore-etcd)
- [I. Runbook: Velero cài & khôi phục](#i-runbook-velero-cài--khôi-phục)
- [J. Runbook: Bảo trì / thay node](#j-runbook-bảo-trì--thay-node)
- [K. Runbook: Rollback deploy khẩn cấp](#k-runbook-rollback-deploy-khẩn-cấp)
- [L. CronJob backup database](#l-cronjob-backup-database)

---

## A. Deployment chuẩn production

Gộp gần như tất cả best practice reliability vào 1 template:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels: { app: myapp }
spec:
  replicas: 3
  revisionHistoryLimit: 5            # giữ 5 revision để rollback
  strategy:
    type: RollingUpdate
    rollingUpdate: { maxSurge: 1, maxUnavailable: 0 }   # 0 downtime
  selector: { matchLabels: { app: myapp } }
  template:
    metadata:
      labels: { app: myapp }
    spec:
      terminationGracePeriodSeconds: 30
      # Trải pod đều các node/zone -> mất 1 node không mất dịch vụ
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: kubernetes.io/hostname
          whenUnsatisfiable: ScheduleAnyway
          labelSelector: { matchLabels: { app: myapp } }
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
        seccompProfile: { type: RuntimeDefault }
      containers:
        - name: myapp
          image: registry.example.com/myapp@sha256:...   # pin bằng digest
          imagePullPolicy: IfNotPresent
          ports: [{ containerPort: 8080 }]
          # --- Tài nguyên: LUÔN đặt requests + limits ---
          resources:
            requests: { cpu: 100m, memory: 128Mi }
            limits:   { cpu: 500m, memory: 512Mi }
          # --- Probes: startup tách khỏi liveness, readiness riêng ---
          startupProbe:                     # cứu app boot chậm khỏi CrashLoop
            httpGet: { path: /health, port: 8080 }
            failureThreshold: 30
            periodSeconds: 5                # cho phép tới 150s để khởi động
          livenessProbe:                    # còn sống? không -> restart
            httpGet: { path: /health, port: 8080 }
            periodSeconds: 10
            failureThreshold: 3
          readinessProbe:                   # sẵn sàng nhận traffic?
            httpGet: { path: /ready, port: 8080 }
            periodSeconds: 5
            failureThreshold: 3
          # --- Graceful shutdown: cho LB rút endpoint trước khi tắt ---
          lifecycle:
            preStop: { exec: { command: ["sh","-c","sleep 10"] } }
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities: { drop: ["ALL"] }
          envFrom:
            - configMapRef: { name: myapp-config }
            - secretRef:    { name: myapp-secret }
```

---

## B. PodDisruptionBudget

Giữ tối thiểu pod khi node bị drain/upgrade (bắt buộc cho mọi service quan trọng):

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata: { name: myapp-pdb }
spec:
  minAvailable: 2                    # hoặc maxUnavailable: 1
  selector: { matchLabels: { app: myapp } }
```
```bash
kubectl get pdb                      # ALLOWED DISRUPTIONS cho biết còn drain được bao nhiêu
```

---

## C. HorizontalPodAutoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: { name: myapp-hpa }
spec:
  scaleTargetRef: { apiVersion: apps/v1, kind: Deployment, name: myapp }
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource: { name: cpu,    target: { type: Utilization, averageUtilization: 70 } }
    - type: Resource
      resource: { name: memory, target: { type: Utilization, averageUtilization: 80 } }
  behavior:                          # chống "flapping" (scale lên/xuống liên tục)
    scaleDown:
      stabilizationWindowSeconds: 300
      policies: [{ type: Percent, value: 50, periodSeconds: 60 }]
    scaleUp:
      stabilizationWindowSeconds: 0
      policies: [{ type: Percent, value: 100, periodSeconds: 30 }]
```
> ⚠️ HPA cần pod có `resources.requests` và cài **metrics-server**. Scale theo queue/Kafka lag → dùng **KEDA**.

---

## D. NetworkPolicy (default-deny + allow)

Chiến lược chuẩn: **chặn hết trước, rồi mở có chủ đích** (phân đoạn Đông-Tây).

```yaml
# 1. Default deny toàn bộ ingress+egress trong namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: default-deny-all, namespace: prod }
spec:
  podSelector: {}                    # áp cho mọi pod
  policyTypes: [Ingress, Egress]
---
# 2. Cho phép DNS (nếu không, mọi thứ vỡ vì không resolve được)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: allow-dns, namespace: prod }
spec:
  podSelector: {}
  policyTypes: [Egress]
  egress:
    - to: [{ namespaceSelector: {} }]
      ports:
        - { protocol: UDP, port: 53 }
        - { protocol: TCP, port: 53 }
---
# 3. Cho frontend gọi backend:8080
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: allow-frontend-to-backend, namespace: prod }
spec:
  podSelector: { matchLabels: { app: backend } }
  policyTypes: [Ingress]
  ingress:
    - from: [{ podSelector: { matchLabels: { app: frontend } } }]
      ports: [{ protocol: TCP, port: 8080 }]
```

---

## E. ResourceQuota & LimitRange

Chặn 1 namespace/pod ăn hết tài nguyên cluster:

```yaml
# Giới hạn tổng tài nguyên cả namespace
apiVersion: v1
kind: ResourceQuota
metadata: { name: prod-quota, namespace: prod }
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    pods: "100"
    persistentvolumeclaims: "20"
---
# Đặt request/limit MẶC ĐỊNH cho pod không khai báo (tránh pod "vô hạn")
apiVersion: v1
kind: LimitRange
metadata: { name: prod-limits, namespace: prod }
spec:
  limits:
    - type: Container
      default:        { cpu: 500m, memory: 512Mi }   # limit mặc định
      defaultRequest: { cpu: 100m, memory: 128Mi }   # request mặc định
      max:            { cpu: "4",  memory: 8Gi }      # trần cho phép
```

---

## F. Pod Security & RBAC tối thiểu

```bash
# Bật Pod Security Standards mức "restricted" cho namespace
kubectl label namespace prod \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/warn=restricted
```
```yaml
# ServiceAccount + Role tối thiểu (least-privilege) — chỉ đọc pod trong namespace
apiVersion: v1
kind: ServiceAccount
metadata: { name: myapp-sa, namespace: prod }
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata: { name: pod-reader, namespace: prod }
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata: { name: myapp-pod-reader, namespace: prod }
subjects: [{ kind: ServiceAccount, name: myapp-sa, namespace: prod }]
roleRef: { kind: Role, name: pod-reader, apiGroup: rbac.authorization.k8s.io }
```
```bash
kubectl auth can-i get pods \
  --as=system:serviceaccount:prod:myapp-sa -n prod   # Verify quyền
```

---

## G. VolumeSnapshot (backup PVC)

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata: { name: data-snap-2026-07-03, namespace: prod }
spec:
  volumeSnapshotClassName: csi-snapclass
  source: { persistentVolumeClaimName: data-pvc }
```
```bash
kubectl get volumesnapshot -n prod             # READYTOUSE=true là xong
# Khôi phục: tạo PVC mới với spec.dataSource trỏ tới VolumeSnapshot này
```

---

## H. Runbook: Backup & Restore etcd

### Backup (chạy trên control-plane node, nên đặt cron)
```bash
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%F-%H%M).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

ETCDCTL_API=3 etcdctl --write-out=table snapshot status /backup/etcd-*.db
aws s3 cp /backup/etcd-*.db s3://<bucket>/etcd/   # ĐẨY RA NGOÀI cluster
```

### Restore (khi mất/hỏng control plane) — TỪNG BƯỚC
```bash
# 1. Dừng static pod etcd & apiserver (di chuyển manifest ra ngoài)
mkdir -p /tmp/manifests-backup
mv /etc/kubernetes/manifests/etcd.yaml /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/manifests-backup/
# kubelet sẽ tự dừng 2 static pod này

# 2. Restore snapshot vào data-dir MỚI (không ghi đè cũ)
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-xxx.db \
  --data-dir /var/lib/etcd-restore

# 3. Trỏ etcd tới data-dir mới: sửa etcd.yaml
#    - hostPath volume "etcd-data" -> /var/lib/etcd-restore
#    (hoặc: mv /var/lib/etcd /var/lib/etcd-old && mv /var/lib/etcd-restore /var/lib/etcd)

# 4. Đưa static pod trở lại -> kubelet dựng lại etcd + apiserver
mv /tmp/manifests-backup/*.yaml /etc/kubernetes/manifests/

# 5. Verify
watch crictl ps            # đợi etcd + apiserver Running
kubectl get nodes          # phải trả về danh sách node
kubectl get pods -A        # trạng thái khôi phục
```
> ⚠️ HA (nhiều etcd member): restore trên 1 node với `--initial-cluster` đúng, rồi để member khác join lại. Test quy trình này ở staging trước khi cần dùng thật.

---

## I. Runbook: Velero cài & khôi phục

```bash
# 1. Cài Velero, trỏ backup location = S3 (ví dụ AWS)
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.9.0 \
  --bucket <bucket> \
  --backup-location-config region=<region> \
  --snapshot-location-config region=<region> \
  --secret-file ./credentials-velero

# 2. Backup thủ công + lịch tự động
velero backup create manual-$(date +%F) --snapshot-volumes --include-cluster-resources
velero schedule create daily --schedule="0 2 * * *" --snapshot-volumes --ttl 720h

# 3. Kiểm tra
velero backup get
velero backup describe manual-2026-07-03 --details
velero backup logs manual-2026-07-03

# 4. Khôi phục (toàn bộ / 1 namespace)
velero restore create --from-backup manual-2026-07-03
velero restore create --from-backup manual-2026-07-03 --include-namespaces prod
velero restore get; velero restore describe <name>

# 5. Test restore vào namespace khác (game day, không đụng prod)
velero restore create test-restore --from-backup manual-2026-07-03 \
  --namespace-mappings prod:prod-restore-test
```

---

## J. Runbook: Bảo trì / thay node

```bash
# 1. Chặn node nhận pod mới
kubectl cordon <node>

# 2. Đẩy pod hiện có sang node khác (PDB sẽ đảm bảo không rớt quá mức)
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data --timeout=300s
#    Nếu kẹt do PDB -> kiểm tra: kubectl get pdb ; tăng replica trước khi drain

# 3. Thực hiện bảo trì (patch OS, nâng kubelet, thay phần cứng...)

# 4. Trả node về cluster
kubectl uncordon <node>

# 5. Verify pod schedule lại & khỏe
kubectl get pods -o wide | grep <node>
kubectl get nodes

# --- Thay node hẳn (managed k8s) ---
# EKS/GKE/AKS: tạo node pool mới -> cordon+drain node cũ -> xoá node cũ
kubectl delete node <old-node>     # sau khi đã drain sạch
```

---

## K. Runbook: Rollback deploy khẩn cấp

```bash
# 1. Phát hiện deploy hỏng (5xx tăng, pod CrashLoop sau khi apply)
kubectl rollout status deploy/<name> --timeout=60s

# 2. ROLLBACK NGAY (cầm máu trước, điều tra sau)
kubectl rollout undo deploy/<name>                 # về revision trước
kubectl rollout undo deploy/<name> --to-revision=5 # về revision cụ thể
kubectl rollout history deploy/<name>              # xem các revision

# 3. Xác nhận hồi phục
kubectl rollout status deploy/<name>
kubectl get pods -l app=<name>

# 4. Với GitOps (ArgoCD): rollback = revert commit trong Git
git revert <bad-commit> && git push               # ArgoCD tự sync về trạng thái tốt
argocd app rollback <app> <revision>              # hoặc rollback trực tiếp
```
> 💡 Phòng ngừa: dùng **Argo Rollouts/Flagger** canary — tự động rollback khi metric (5xx/latency) vượt ngưỡng, không cần thao tác tay.

---

## L. CronJob backup database

Backup DB định kỳ ngay trong cluster, đẩy lên S3:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata: { name: pg-backup, namespace: prod }
spec:
  schedule: "0 2 * * *"              # 2h sáng mỗi ngày
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      backoffLimit: 2
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup
              image: postgres:16
              command:
                - /bin/sh
                - -c
                - |
                  set -e
                  FILE=db-$(date +%F-%H%M).sql.gz
                  pg_dump -h $PGHOST -U $PGUSER $PGDATABASE | gzip > /tmp/$FILE
                  aws s3 cp /tmp/$FILE s3://$BUCKET/db/$FILE
                  echo "Backup done: $FILE"
              envFrom:
                - secretRef: { name: pg-backup-secret }   # PGHOST/PGUSER/PGPASSWORD/BUCKET...
```
```bash
kubectl create job --from=cronjob/pg-backup manual-backup -n prod   # Chạy thử ngay
kubectl logs -n prod job/manual-backup                              # Xem kết quả
```

---

## ✅ Checklist "Production-ready" cho mỗi service

```text
[ ] resources.requests & limits đã đặt (không để trống)
[ ] readiness + liveness (+ startup nếu boot chậm) probe
[ ] replicas >= 2 + PodDisruptionBudget
[ ] topologySpread / anti-affinity (trải node/zone)
[ ] graceful shutdown (preStop + terminationGracePeriod)
[ ] securityContext: non-root, drop capabilities, readOnlyRootFilesystem
[ ] image pin bằng digest/tag bất biến (không :latest) + đã scan
[ ] HPA (nếu cần scale) + metrics-server sẵn sàng
[ ] NetworkPolicy (default-deny + allow có chủ đích)
[ ] config/secret tách khỏi image (ConfigMap/Secret/Vault)
[ ] metrics /metrics + dashboard + alert (5xx, latency, restart, RAM)
[ ] backup dữ liệu (PV/DB) + đã TEST restore
[ ] manifest nằm trong Git (GitOps) — không sửa tay trên prod
[ ] runbook cho alert của service này
```
