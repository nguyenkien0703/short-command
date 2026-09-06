# CKS — System Hardening (10%)

> Domain nhẹ nhất về trọng số nhưng chứa **AppArmor và seccomp** — hai thứ DevOps đi làm
> gần như không bao giờ đụng tới. Đừng chủ quan.

**Nội dung curriculum v1.34:**
- Minimize host OS footprint (reduce attack surface)
- Using least-privilege identity and access management
- Minimize external access to the network
- Appropriately use kernel hardening tools such as **AppArmor, seccomp**

---

## 1. Giảm attack surface của host OS

### Tắt service không cần thiết
```bash
systemctl list-units --type=service --state=running
systemctl list-unit-files --state=enabled

# Tắt hẳn (đề hay bảo "disable service X permanently")
systemctl stop <service>
systemctl disable <service>
systemctl mask <service>              # ← mạnh nhất: không thể start kể cả thủ công

# Ví dụ hay ra đề
systemctl disable --now snapd
systemctl disable --now apache2
systemctl disable --now telnet.socket
```

### Gỡ package thừa
```bash
apt list --installed | grep -i <pkg>
apt-get remove --purge -y <pkg>
apt-get autoremove -y

dpkg -l | grep -E 'telnet|ftp|nfs'
```

### Tìm process đang mở port
```bash
ss -tulpn                             # mọi socket đang listen
ss -tulpn | grep LISTEN
netstat -tulpn
lsof -i :8080
lsof -i -P -n | grep LISTEN

# Truy ngược process → service
ps aux | grep <pid>
systemctl status <pid>                # systemd cho biết pid thuộc unit nào
ls -l /proc/<pid>/exe
cat /proc/<pid>/cmdline | tr '\0' ' '
```

> **Dạng đề kinh điển:** *"Trên node01 có một service đang mở port 1234 không được phép.
> Tìm và tắt nó vĩnh viễn."*
> ```bash
> ss -tulpn | grep 1234              # → pid
> systemctl status <pid>             # → tên unit
> systemctl stop <unit> && systemctl disable <unit>
> ss -tulpn | grep 1234              # verify: không còn gì
> ```

### Tìm binary có SUID (leo thang đặc quyền)
```bash
find / -perm -4000 -type f 2>/dev/null          # SUID
find / -perm -2000 -type f 2>/dev/null          # SGID
find / -perm -u=s -type f -exec ls -l {} \; 2>/dev/null

# Gỡ SUID
chmod u-s /path/to/binary
```

### Kiểm tra user & quyền
```bash
cat /etc/passwd | grep -v nologin | grep -v false      # user có shell
awk -F: '$3 == 0 {print $1}' /etc/passwd               # user có UID 0 (chỉ nên có root!)
cat /etc/sudoers /etc/sudoers.d/*
lastlog ; last                                          # ai đã đăng nhập

# Khóa user
usermod -L <user>            # khóa mật khẩu
usermod -s /sbin/nologin <user>
passwd -l <user>
userdel -r <user>            # xóa hẳn + home
```

### Kernel module
```bash
lsmod                                  # module đang nạp
modprobe -r <module>                   # gỡ

# Chặn nạp vĩnh viễn
echo "blacklist <module>" >> /etc/modprobe.d/blacklist.conf
echo "install <module> /bin/true" >> /etc/modprobe.d/disable.conf
```

---

## 2. Hạn chế truy cập mạng ngoài

```bash
# --- ufw ---
ufw status verbose
ufw default deny incoming
ufw default allow outgoing
ufw allow from 10.0.0.0/8 to any port 6443
ufw deny 8080
ufw enable

# --- iptables ---
iptables -L -n -v
iptables -A INPUT -p tcp --dport 22 -s 10.0.0.0/8 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP
iptables-save > /etc/iptables/rules.v4
```

> ⚠️ Cẩn thận: chặn nhầm port 6443 / 10250 / 2379 sẽ **làm hỏng cluster**.
> Trong phòng thi, chỉ chặn đúng port đề yêu cầu.

**Cổng K8s cần biết:**
| Port | Component | Chiều |
|---|---|---|
| 6443 | kube-apiserver | Mọi node + client → cp |
| 2379-2380 | etcd | cp → cp |
| 10250 | kubelet API | cp → node |
| 10256 | kube-proxy | health check |
| 30000-32767 | NodePort | ngoài → node |

---

## 3. seccomp ⭐⭐ (chắc chắn có trong đề)

**seccomp** (secure computing mode) = lọc **syscall** mà container được phép gọi.
Ví dụ: chặn `mount`, `ptrace`, `reboot`.

### 3.1 Dùng profile mặc định — cách nhanh nhất
```yaml
apiVersion: v1
kind: Pod
metadata: {name: secure-pod}
spec:
  securityContext:                    # ← cấp POD: áp cho mọi container
    seccompProfile:
      type: RuntimeDefault            # profile mặc định của container runtime
  containers:
  - name: app
    image: nginx
    securityContext:                  # ← cấp CONTAINER: ghi đè
      seccompProfile:
        type: RuntimeDefault
```

| `type` | Ý nghĩa |
|---|---|
| `RuntimeDefault` | Profile mặc định của containerd/CRI-O (chặn ~44 syscall nguy hiểm) |
| `Localhost` | Profile custom, cần thêm `localhostProfile` |
| `Unconfined` | ❌ **Không lọc gì** — mặc định nếu không khai |

### 3.2 Custom profile
File phải đặt trong **`<kubelet-root-dir>/seccomp/`**, mặc định là
**`/var/lib/kubelet/seccomp/`**:

```bash
mkdir -p /var/lib/kubelet/seccomp/profiles
cat <<'EOF' > /var/lib/kubelet/seccomp/profiles/audit.json
{
  "defaultAction": "SCMP_ACT_LOG"
}
EOF

cat <<'EOF' > /var/lib/kubelet/seccomp/profiles/violation.json
{
  "defaultAction": "SCMP_ACT_ERRNO"
}
EOF

# Profile thực tế: cho phép danh sách, chặn phần còn lại
cat <<'EOF' > /var/lib/kubelet/seccomp/profiles/restricted.json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "architectures": ["SCMP_ARCH_X86_64", "SCMP_ARCH_X86", "SCMP_ARCH_X32"],
  "syscalls": [
    {
      "names": ["accept4","access","arch_prctl","bind","brk","close","connect",
                "execve","exit","exit_group","fstat","futex","getpid","listen",
                "mmap","mprotect","munmap","openat","read","rt_sigaction",
                "rt_sigprocmask","set_tid_address","socket","write"],
      "action": "SCMP_ACT_ALLOW"
    }
  ]
}
EOF
```

```yaml
# Dùng trong Pod
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/restricted.json     # ← đường dẫn TƯƠNG ĐỐI
                                                     #    so với /var/lib/kubelet/seccomp/
```

| `defaultAction` | Hành vi |
|---|---|
| `SCMP_ACT_ALLOW` | Cho phép |
| `SCMP_ACT_ERRNO` | Chặn, trả lỗi `EPERM` (mặc định để chặn) |
| `SCMP_ACT_LOG` | **Cho phép nhưng ghi log** — dùng để audit xem app cần syscall nào |
| `SCMP_ACT_KILL` | Giết process ngay |
| `SCMP_ACT_TRAP` | Gửi SIGSYS |

### 3.3 Verify
```bash
k exec <pod> -- grep Seccomp /proc/1/status
# Seccomp: 0 = disabled, 1 = strict, 2 = filtered  ← muốn thấy 2
# Seccomp_filters: 1

k get po <pod> -o jsonpath='{.spec.securityContext.seccompProfile}'
k describe po <pod> | grep -i seccomp
```

> 🔴 Bẫy:
> - Đường dẫn `localhostProfile` là **tương đối** so với `/var/lib/kubelet/seccomp/`,
>   KHÔNG phải đường dẫn tuyệt đối.
> - File profile phải nằm **trên node mà Pod sẽ chạy**. Nhiều node → copy đủ, hoặc dùng `nodeName`.
> - Không có file → Pod `CreateContainerError` với message
>   `cannot load seccomp profile ... no such file or directory`.

---

## 4. AppArmor ⭐⭐

**AppArmor** = Mandatory Access Control ở tầng Linux, giới hạn **file/network/capability**
mà process được đụng tới. (Ubuntu dùng AppArmor; RHEL dùng SELinux.)

### 4.1 Quản lý profile trên node
```bash
# Kiểm tra AppArmor có bật không
systemctl status apparmor
cat /sys/module/apparmor/parameters/enabled       # → Y

# Xem profile đang có
aa-status
apparmor_parser -Q /etc/apparmor.d/<profile>      # kiểm tra cú pháp (không nạp)

# Nạp / gỡ profile
apparmor_parser -q /etc/apparmor.d/k8s-deny-write        # nạp (enforce)
apparmor_parser -r /etc/apparmor.d/k8s-deny-write        # nạp lại
apparmor_parser -R /etc/apparmor.d/k8s-deny-write        # gỡ

# Đổi mode
aa-enforce /etc/apparmor.d/k8s-deny-write         # bắt buộc
aa-complain /etc/apparmor.d/k8s-deny-write        # chỉ log, không chặn
aa-disable  /etc/apparmor.d/k8s-deny-write
```

### 4.2 Viết profile
```bash
cat <<'EOF' > /etc/apparmor.d/k8s-deny-write
#include <tunables/global>

profile k8s-deny-write flags=(attach_disconnected) {
  #include <abstractions/base>

  file,                      # cho phép mọi thao tác file...
  deny /** w,                # ...trừ GHI (deny thắng allow)
  deny /proc/** w,

  network,                   # cho phép network
  # deny network,            # chặn hết network

  capability,
  deny capability sys_admin,
}
EOF

apparmor_parser -q /etc/apparmor.d/k8s-deny-write
aa-status | grep k8s-deny-write
```

**Cú pháp hay dùng:**
```text
file,                        # allow mọi file
/etc/passwd r,               # allow đọc 1 file
deny /etc/shadow rw,         # deny đọc/ghi
deny /** w,                  # deny ghi mọi nơi
deny /bin/** x,              # deny thực thi
network,                     # allow network
deny network inet,           # deny IPv4
capability,                  # allow mọi capability
deny capability net_raw,     # deny 1 capability
```

### 4.3 Gắn profile vào Pod

**Cách mới (K8s 1.30+) — dùng field trong `securityContext`:**
```yaml
apiVersion: v1
kind: Pod
metadata: {name: apparmor-pod}
spec:
  securityContext:
    appArmorProfile:
      type: Localhost                   # RuntimeDefault | Localhost | Unconfined
      localhostProfile: k8s-deny-write  # ← TÊN profile, không phải đường dẫn file
  containers:
  - name: app
    image: nginx
```

**Cách cũ (annotation) — vẫn còn trong nhiều tài liệu và đề thi:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: apparmor-pod
  annotations:
    container.apparmor.security.beta.kubernetes.io/<container-name>: localhost/k8s-deny-write
    # hoặc: runtime/default  |  unconfined
spec:
  containers:
  - name: <container-name>              # ← tên phải KHỚP với annotation
    image: nginx
```

> 🔴 Bẫy annotation: phần sau dấu `:` phải là **`localhost/<tên-profile>`**,
> không phải đường dẫn file. Và `<container-name>` trong key annotation
> phải trùng chính xác tên container.

### 4.4 Verify
```bash
k exec <pod> -- cat /proc/1/attr/current
# → k8s-deny-write (enforce)     ⇒ đang áp dụng
# → unconfined                    ⇒ CHƯA áp dụng

# Test thực tế
k exec <pod> -- touch /tmp/test
# → touch: cannot touch '/tmp/test': Permission denied  ⇒ profile hoạt động

k describe po <pod> | grep -i apparmor
```

> 🔴 Bẫy lớn nhất: **profile phải đã được nạp trên node** trước khi tạo Pod.
> Chưa nạp → Pod kẹt ở `Blocked`/`CreateContainerError` với message
> `AppArmor profile is not loaded`. Nếu cluster nhiều node, nạp trên node đúng
> hoặc dùng `nodeName`/`nodeSelector`.

### seccomp vs AppArmor — phân biệt
| | seccomp | AppArmor |
|---|---|---|
| Lọc gì | **Syscall** | **File, network, capability** |
| Định dạng | JSON | Cú pháp riêng (text) |
| Đặt ở đâu | `/var/lib/kubelet/seccomp/` | `/etc/apparmor.d/` |
| Nạp thế nào | Không cần nạp, kubelet đọc file | `apparmor_parser -q` |
| Field trong Pod | `securityContext.seccompProfile` | `securityContext.appArmorProfile` (hoặc annotation) |
| Verify | `grep Seccomp /proc/1/status` | `cat /proc/1/attr/current` |

---

## 5. Least-privilege ở tầng container

Chi tiết `securityContext` ở [04](./04-minimize-microservice-vulnerabilities.md#2-securitycontext).
Tóm tắt cho domain này:

```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: app
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false     # chặn setuid leo quyền
      privileged: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
        add: ["NET_BIND_SERVICE"]         # chỉ thêm cái thực sự cần
```

**Linux capabilities nguy hiểm — phải drop:**
| Capability | Cho phép làm gì |
|---|---|
| `SYS_ADMIN` | Gần như root — mount, namespace, nhiều thứ |
| `SYS_PTRACE` | Debug/inject vào process khác |
| `SYS_MODULE` | Nạp kernel module → chiếm host |
| `NET_ADMIN` | Sửa cấu hình mạng, iptables |
| `NET_RAW` | Raw socket → ARP spoofing, sniff |
| `DAC_OVERRIDE` | Bỏ qua kiểm tra quyền file |
| `SETUID`/`SETGID` | Đổi UID/GID |
| `CHOWN` | Đổi chủ sở hữu file |

```bash
# Xem capability của container
k exec <pod> -- cat /proc/1/status | grep Cap
k exec <pod> -- capsh --print                 # nếu có capsh
```

---

## 6. Dạng bài hay ra

| # | Đề bài | Hướng làm |
|---|---|---|
| 1 | Tìm và tắt vĩnh viễn service đang mở port X trên node01 | `ss -tulpn` → `systemctl disable --now` |
| 2 | Nạp AppArmor profile có sẵn ở `/etc/apparmor.d/X`, gắn vào Pod | `apparmor_parser -q` + annotation/field |
| 3 | Tạo Pod dùng seccomp `RuntimeDefault` | Mục 3.1 |
| 4 | Tạo Pod dùng seccomp custom profile `violation.json` | Mục 3.2, `type: Localhost` |
| 5 | Pod đang `privileged: true` — sửa cho an toàn | drop ALL capabilities, `privileged: false` |
| 6 | Sửa Pod: chạy user 1000, root filesystem read-only, không leo quyền | Mục 5 |
| 7 | Gỡ package `X` khỏi node và chặn nạp kernel module `Y` | `apt purge` + `/etc/modprobe.d/` |
| 8 | Chặn mọi kết nối tới port 8080 trên node, giữ nguyên 6443 | `ufw`/`iptables` |
| 9 | Tìm binary có SUID bất thường trong `/usr/bin` | `find -perm -4000` |
| 10 | Pod bị `Blocked` vì AppArmor — chẩn đoán | Profile chưa nạp trên node đó |

---

## 7. Bẫy tổng kết

1. **AppArmor: `localhostProfile` là TÊN profile**, không phải đường dẫn file.
2. **seccomp: `localhostProfile` là đường dẫn TƯƠNG ĐỐI** so với `/var/lib/kubelet/seccomp/`.
3. **AppArmor profile phải nạp TRƯỚC** khi tạo Pod, và nạp **trên đúng node**.
4. **Annotation AppArmor phải có tên container trong key** — sai tên thì không áp dụng, và **không báo lỗi**.
5. **`aa-status` để verify đã nạp; `/proc/1/attr/current` để verify Pod đang dùng.**
6. **`systemctl disable` chưa đủ nếu đề nói "cannot be started"** → dùng `mask`.
7. **Không khai seccomp = `Unconfined`** — nghĩa là không lọc gì. CKS coi là chưa an toàn.
8. **`drop: ["ALL"]` rồi mới `add`** — thứ tự này quan trọng về mặt ý nghĩa.
9. **`readOnlyRootFilesystem: true` làm nhiều app hỏng** — thường phải mount `emptyDir` vào `/tmp`.
10. **Đừng chặn nhầm port cluster** (6443/10250/2379) khi làm firewall.
