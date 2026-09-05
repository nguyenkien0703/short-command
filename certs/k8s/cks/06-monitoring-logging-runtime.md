# CKS — Monitoring, Logging and Runtime Security (20%) 🔴

**Nội dung curriculum v1.34:**
- Perform **behavioral analytics** to detect malicious activities
- Detect threats within physical infrastructure, apps, networks, data, users and workloads
- Investigate and identify **phases of attack** and bad actors within the environment
- Use **Kubernetes audit logs** to monitor access

---

## 1. Falco ⭐⭐ (tool trung tâm của domain này)

**Falco** = runtime security engine. Nó theo dõi **syscall** ở tầng kernel (qua eBPF hoặc kernel module)
và bắn alert khi thấy hành vi khớp rule — ví dụ: có người `exec` vào container,
ghi file vào `/etc`, mở shell, đọc file nhạy cảm.

### Vận hành
```bash
systemctl status falco
systemctl restart falco
journalctl -u falco -f
journalctl -u falco | grep Warning

# Chạy trực tiếp (debug)
falco
falco -r /etc/falco/falco_rules.local.yaml
falco -L                                  # liệt kê mọi rule đang nạp
falco --list                              # liệt kê field có thể dùng
falco -V /etc/falco/falco_rules.local.yaml  # validate cú pháp rule

# Nếu chạy dạng DaemonSet
k logs -n falco -l app.kubernetes.io/name=falco -f
```

### Cấu trúc file
| File | Vai trò |
|---|---|
| `/etc/falco/falco.yaml` | Config chính: output, log level, driver |
| `/etc/falco/falco_rules.yaml` | Rule mặc định — **KHÔNG sửa file này** |
| `/etc/falco/falco_rules.local.yaml` | ⭐ **Rule tùy chỉnh / override** — sửa ở đây |
| `/etc/falco/k8s_audit_rules.yaml` | Rule cho K8s audit log |
| `/etc/falco/rules.d/` | Thư mục rule bổ sung |

> 🔴 Đề CKS **luôn** bảo sửa/thêm rule ở `falco_rules.local.yaml`, không phải file gốc.
> File local được nạp **sau** nên override được rule cùng tên.

### Đọc & viết rule
```yaml
- rule: Terminal shell in container
  desc: A shell was used as the entrypoint/exec target in a container
  condition: >
    spawned_process and container
    and shell_procs and proc.tty != 0
    and container_entrypoint
  output: >
    A shell was spawned in a container
    (user=%user.name user_loginuid=%user.loginuid %container.info
    shell=%proc.name parent=%proc.pname cmdline=%proc.cmdline
    terminal=%proc.tty container_id=%container.id image=%container.image.repository)
  priority: NOTICE
  tags: [container, shell, mitre_execution]
```

**5 thành phần của một rule:**
| Field | Ý nghĩa |
|---|---|
| `rule` | Tên rule |
| `desc` | Mô tả |
| `condition` | Điều kiện khớp (dùng field + macro + list) |
| `output` | Nội dung alert — dùng `%field` để chèn giá trị |
| `priority` | `EMERGENCY > ALERT > CRITICAL > ERROR > WARNING > NOTICE > INFORMATIONAL > DEBUG` |

**Field hay dùng — nhớ nhóm:**
```text
proc.name        tên process           proc.cmdline   dòng lệnh đầy đủ
proc.pname       process cha           proc.tty       terminal
fd.name          tên file/socket       fd.sip/fd.sport IP/port
user.name        user                  user.uid
evt.type         loại syscall (open, execve...)      evt.time
container.id     ID container          container.name
container.image.repository             container.image.tag
k8s.ns.name      namespace             k8s.pod.name
```

### Dạng bài kinh điển: sửa `output` của rule ⭐
> *"Sửa rule `Terminal shell in container` để output có thêm timestamp, tên container,
> tên pod, namespace, tên process — theo đúng định dạng:
> `time,container-id,container-name,user-name`"*

```bash
# 1. Tìm rule trong file gốc
grep -n -A15 "Terminal shell in container" /etc/falco/falco_rules.yaml

# 2. Copy sang file local và sửa
vim /etc/falco/falco_rules.local.yaml
```
```yaml
- rule: Terminal shell in container
  desc: A shell was used as the entrypoint/exec target in a container
  condition: >
    spawned_process and container
    and shell_procs and proc.tty != 0
    and container_entrypoint
  output: "%evt.time,%container.id,%container.name,%user.name"
  priority: WARNING
```
```bash
# 3. Restart & verify
systemctl restart falco
journalctl -u falco -f
# Tab khác: k exec -it <pod> -- sh   → phải thấy alert đúng format
```

> 🔴 Bẫy:
> - Chỉ ghi đè **`output`** (và `priority` nếu đề bảo), giữ nguyên `condition` — đổi condition
>   là rule không bắt được nữa.
> - Rule override phải có **tên trùng chính xác** với rule gốc.
> - `%evt.time` khác `%evt.time.iso8601` — đọc kỹ đề yêu cầu format nào.
> - Sau khi sửa **phải `systemctl restart falco`**.

### Tìm Pod có hành vi bất thường bằng Falco
> *"Dùng Falco tìm Pod nào đang spawn shell, ghi tên Pod ra `/opt/answer.txt`, rồi xóa Pod đó."*
```bash
journalctl -u falco -n 200 | grep -i "shell"
# hoặc
cat /var/log/syslog | grep falco | grep -i shell

# Từ output lấy container.id → truy ngược ra Pod
crictl ps -a | grep <container-id>
k get po -A -o wide | grep <node>
```

### Thêm rule mới
```yaml
# /etc/falco/falco_rules.local.yaml
- rule: Detect cat on shadow file
  desc: Ai đó đọc /etc/shadow trong container
  condition: >
    open_read and container
    and fd.name = /etc/shadow
  output: "Shadow file read (user=%user.name container=%container.name file=%fd.name)"
  priority: CRITICAL
  tags: [filesystem, mitre_credential_access]

- rule: Write below etc in container
  desc: Ghi file vào /etc bên trong container
  condition: >
    open_write and container
    and fd.name startswith /etc
  output: "File written below /etc (user=%user.name file=%fd.name container=%container.name)"
  priority: ERROR
```

**Macro & list (tái sử dụng):**
```yaml
- list: allowed_images
  items: [nginx, redis, postgres]

- macro: my_container
  condition: container.image.repository in (allowed_images)

- rule: Unexpected image
  condition: spawned_process and container and not my_container
  output: "Container không nằm trong allowlist (image=%container.image.repository)"
  priority: WARNING
```

---

## 2. Kubernetes Audit Log ⭐⭐⭐ (dạng bài chắc chắn có)

Audit log ghi lại **mọi request** tới kube-apiserver: ai gọi, gọi gì, lúc nào, kết quả ra sao.

### 4 giai đoạn (stage) của một request
```text
RequestReceived ──► ResponseStarted ──► ResponseComplete
                                    └─► Panic (nếu lỗi)
```

### 4 mức log (level)
| Level | Ghi gì |
|---|---|
| `None` | Không ghi gì |
| `Metadata` | Ai, gì, khi nào, verb, URI — **không** ghi body |
| `Request` | Metadata + **request body** |
| `RequestResponse` | Metadata + request body + **response body** — chi tiết nhất, nặng nhất |

> Rule được duyệt **từ trên xuống, dừng ở rule khớp đầu tiên**. Thứ tự rất quan trọng.

### Bước 1 — viết audit policy
```yaml
# /etc/kubernetes/audit/policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
- "RequestReceived"                 # bỏ stage này cho gọn log
rules:
# 1. Không log request đọc thông thường (giảm nhiễu)
- level: None
  verbs: ["get", "list", "watch"]
  resources:
  - group: ""
    resources: ["events"]

# 2. Secret & ConfigMap: chỉ ghi metadata (KHÔNG ghi body — tránh lộ secret vào log!)
- level: Metadata
  resources:
  - group: ""
    resources: ["secrets", "configmaps"]
  - group: "authentication.k8s.io"
    resources: ["tokenreviews"]

# 3. Pod: ghi đầy đủ request+response
- level: RequestResponse
  resources:
  - group: ""
    resources: ["pods"]

# 4. Chỉ theo dõi 1 namespace
- level: Request
  namespaces: ["prod"]

# 5. Theo dõi 1 user cụ thể
- level: RequestResponse
  users: ["jane"]
  # userGroups: ["system:serviceaccounts"]

# 6. Bỏ qua request của system component
- level: None
  users: ["system:kube-proxy"]
  verbs: ["watch"]

# 7. Mặc định cho mọi thứ còn lại
- level: Metadata
```

### Bước 2 — bật trên apiserver
```yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml
spec:
  containers:
  - command:
    - kube-apiserver
    - --audit-policy-file=/etc/kubernetes/audit/policy.yaml
    - --audit-log-path=/var/log/kubernetes/audit/audit.log
    - --audit-log-maxage=30           # giữ log bao nhiêu ngày
    - --audit-log-maxbackup=10        # giữ bao nhiêu file cũ
    - --audit-log-maxsize=100         # MB, xoay file khi vượt
    # - --audit-log-format=json       # json (mặc định) | legacy

    volumeMounts:
    - name: audit-policy
      mountPath: /etc/kubernetes/audit
      readOnly: true
    - name: audit-log
      mountPath: /var/log/kubernetes/audit
      readOnly: false                 # ← phải ghi được!

  volumes:
  - name: audit-policy
    hostPath:
      path: /etc/kubernetes/audit
      type: DirectoryOrCreate
  - name: audit-log
    hostPath:
      path: /var/log/kubernetes/audit
      type: DirectoryOrCreate
```

> 🔴 **3 lỗi làm apiserver không lên lại:**
> 1. Quên mount volume cho **cả hai** thư mục (policy và log).
> 2. `readOnly: true` cho volume log → apiserver không ghi được.
> 3. Thư mục chưa tồn tại → dùng `type: DirectoryOrCreate`, hoặc `mkdir -p` trước.

### Bước 3 — verify
```bash
watch crictl ps | grep apiserver
ls -l /var/log/kubernetes/audit/
tail -f /var/log/kubernetes/audit/audit.log

# Đọc đẹp
tail -1 /var/log/kubernetes/audit/audit.log | jq
```

### Truy vấn audit log — dạng bài điều tra ⭐
```bash
L=/var/log/kubernetes/audit/audit.log

# Ai đã xóa Pod?
cat $L | jq 'select(.verb=="delete" and .objectRef.resource=="pods")
             | {user:.user.username, pod:.objectRef.name, ns:.objectRef.namespace, time:.requestReceivedTimestamp}'

# Ai đọc Secret?
cat $L | jq 'select(.objectRef.resource=="secrets" and .verb=="get")
             | {user:.user.username, secret:.objectRef.name, ns:.objectRef.namespace}'

# Mọi hành động của 1 user
cat $L | jq 'select(.user.username=="jane") | {verb, resource:.objectRef.resource, time:.requestReceivedTimestamp}'

# Request bị từ chối (403)
cat $L | jq 'select(.responseStatus.code==403) | {user:.user.username, verb, uri:.requestURI}'

# Truy cập từ IP lạ
cat $L | jq 'select(.sourceIPs[0] != "10.0.0.1") | {ip:.sourceIPs[0], user:.user.username, verb}'

# Thống kê ai gọi nhiều nhất
cat $L | jq -r '.user.username' | sort | uniq -c | sort -rn | head

# Không có jq? Dùng grep
grep '"verb":"delete"' $L | grep '"resource":"secrets"'
```

**Cấu trúc 1 dòng audit log:**
```json
{
  "kind": "Event",
  "level": "RequestResponse",
  "auditID": "...",
  "stage": "ResponseComplete",
  "requestURI": "/api/v1/namespaces/default/pods/nginx",
  "verb": "delete",
  "user": {"username": "jane", "groups": ["dev-team"]},
  "sourceIPs": ["10.0.1.5"],
  "objectRef": {"resource": "pods", "namespace": "default", "name": "nginx"},
  "responseStatus": {"code": 200},
  "requestReceivedTimestamp": "2026-01-15T10:23:45.123Z"
}
```

---

## 3. Behavioral analytics — phát hiện hành vi bất thường

**Ý tưởng:** thay vì tìm signature đã biết, ta dựng **baseline** hành vi bình thường
rồi cảnh báo khi có sai lệch.

| Hành vi bất thường | Vì sao đáng ngờ | Bắt bằng |
|---|---|---|
| Shell spawn trong container production | Container không nên có ai vào | Falco `Terminal shell in container` |
| Ghi file vào `/etc`, `/bin` | Container nên immutable | Falco `Write below etc` |
| Đọc `/etc/shadow`, `/root/.ssh/` | Thu thập credential | Falco rule custom |
| Container gọi ra IP lạ | C2 / exfiltration | Falco `fd.sip`, NetworkPolicy |
| Process lạ chạy trong container | Payload được thả vào | Falco `spawned_process` |
| Pod mount `hostPath: /` | Chuẩn bị escape | PSA / Kyverno |
| `kubectl exec` nhiều bất thường | Recon | Audit log |
| SA token dùng từ IP ngoài cluster | Token bị đánh cắp | Audit log `sourceIPs` |
| Tạo ClusterRoleBinding tới `cluster-admin` | Leo thang đặc quyền | Audit log |

### Immutability của container
Container **không nên thay đổi** sau khi khởi chạy. Thực thi bằng:
```yaml
securityContext:
  readOnlyRootFilesystem: true       # ← cốt lõi
  allowPrivilegeEscalation: false
  runAsNonRoot: true
  capabilities: {drop: ["ALL"]}
```
Cộng với: image không có shell (distroless), Falco giám sát ghi file.

---

## 4. Phases of Attack — MITRE ATT&CK cho Kubernetes

```text
1. Initial Access      → image độc hại, credential lộ, apiserver hở
2. Execution           → exec vào container, chạy process lạ
3. Persistence         → tạo CronJob/DaemonSet ẩn, sửa manifest static pod
4. Privilege Escalation→ privileged pod, hostPath, SA quyền cao, capability SYS_ADMIN
5. Defense Evasion     → xóa log, tắt Falco, chạy trong ns hệ thống
6. Credential Access   → đọc Secret, SA token, /etc/shadow, cloud metadata
7. Discovery           → liệt kê pod/service/secret, quét mạng nội bộ
8. Lateral Movement    → dùng SA token gọi API, đi sang Pod khác
9. Collection          → thu thập dữ liệu
10. Exfiltration       → gửi data ra ngoài
11. Impact             → xóa resource, crypto mining, ransomware
```

**Bản đồ phòng thủ theo phase:**
| Phase | Phòng thủ |
|---|---|
| Initial Access | Scan image (Trivy), permitted registry, `anonymous-auth=false` |
| Execution | PSA `restricted`, seccomp, distroless (không có shell), Falco |
| Persistence | Audit log, RBAC hạn chế `create`, GitOps drift detection |
| Priv Esc | `allowPrivilegeEscalation: false`, drop caps, NodeRestriction, RBAC tối thiểu |
| Defense Evasion | Log gửi ra ngoài node ngay, audit log bất biến |
| Credential Access | Encryption at rest, tắt automount SA token, chặn metadata endpoint |
| Discovery | RBAC least privilege, NetworkPolicy |
| Lateral Movement | NetworkPolicy default-deny, mTLS, namespace isolation |
| Exfiltration | NetworkPolicy egress, giám sát DNS |
| Impact | Backup, RBAC, ResourceQuota |

> Đề CKS có thể hỏi dạng: *"Đọc Falco log/audit log, xác định Pod nào bị chiếm và xử lý."*
> → Tìm alert → truy container.id → ra Pod → `k delete po` hoặc cô lập bằng NetworkPolicy.

---

## 5. Log & monitoring hạ tầng

```bash
# Log của container
k logs <pod> -n <ns> --previous
crictl logs <container-id>
ls /var/log/pods/ /var/log/containers/

# Log hệ thống trên node
journalctl -u kubelet -f
journalctl -u containerd
journalctl -u falco
cat /var/log/syslog | grep -i falco
cat /var/log/auth.log                       # đăng nhập SSH

# Ai đang login
who ; w ; last ; lastlog

# Process bất thường
ps auxf
ps -eo pid,ppid,user,cmd --sort=-%cpu | head
top -c

# Kết nối mạng
ss -tunap
ss -tunap | grep ESTAB
lsof -i -P -n

# File thay đổi gần đây (dấu hiệu bị chiếm)
find /etc -mmin -60 -type f 2>/dev/null
find / -newermt "1 hour ago" -type f 2>/dev/null | grep -v '/proc\|/sys'
```

---

## 6. Dạng bài hay ra

| # | Đề bài | Hướng làm |
|---|---|---|
| 1 | Sửa Falco rule `Terminal shell in container` để output theo format cho trước | Mục 1 — `falco_rules.local.yaml` + restart |
| 2 | Dùng Falco tìm Pod đang spawn shell, ghi tên ra file, xóa Pod | Mục 1 |
| 3 | Viết Falco rule phát hiện đọc `/etc/shadow` | Mục 1 |
| 4 | Bật audit log với policy: Metadata cho secrets, RequestResponse cho pods | Mục 2 |
| 5 | Sửa audit policy: chỉ log 1 namespace, giữ 15 ngày | `namespaces:` + `--audit-log-maxage=15` |
| 6 | Đọc audit log tìm ai xóa Deployment X | `jq select(.verb=="delete")` |
| 7 | Tìm request nào bị 403 và của user nào | `jq select(.responseStatus.code==403)` |
| 8 | Falco không chạy — sửa | `systemctl status falco`, `falco -V` validate rule |
| 9 | Sửa Pod cho immutable (readOnlyRootFilesystem) | Mục 3 |
| 10 | Đổi Falco priority của 1 rule thành CRITICAL | Override trong file local |

---

## 7. Bẫy tổng kết

1. **Sửa Falco ở `/etc/falco/falco_rules.local.yaml`**, không sửa `falco_rules.yaml`.
2. **`systemctl restart falco` sau khi sửa rule** — không tự reload.
3. **Rule override phải trùng TÊN chính xác** với rule gốc.
4. **Chỉ đổi `output`, giữ nguyên `condition`** trừ khi đề bảo khác.
5. **Audit: quên mount volume → apiserver chết.** Cần mount **cả 2** (policy + log dir).
6. **Volume log phải `readOnly: false`.**
7. **Audit rule duyệt từ trên xuống, dừng ở cái khớp đầu tiên** — thứ tự quyết định.
8. **`level: Metadata` cho Secret**, đừng dùng `RequestResponse` — sẽ ghi cả giá trị secret vào log.
9. **`omitStages: ["RequestReceived"]`** giảm một nửa dung lượng log.
10. **`--audit-log-maxage/maxbackup/maxsize` là 3 cờ CIS bắt buộc.**
11. **`type: DirectoryOrCreate`** cho hostPath, nếu không thư mục chưa có là apiserver fail.
12. **Backup manifest trước khi sửa.** Luôn luôn.
