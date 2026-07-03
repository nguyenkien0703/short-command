# Cheatsheet: Docker, Docker Compose, Helm, kubectl & DevOps

Tổng hợp các câu lệnh hay dùng. Ghi chú bằng tiếng Việt để dễ tra cứu.

## 📑 Mục lục

- [🐳 Docker](#-docker)
- [🐙 Docker Compose](#-docker-compose)
- [⎈ Helm](#-helm)
- [☸️ kubectl](#️-kubectl)
- [🔍 grep & Xử lý text](#-grep--xử-lý-text-log-filtering)
- [🚑 Troubleshooting sự cố (Linux)](#-troubleshooting-sự-cố-linux)
- [🌿 Git](#-git---troubleshooting--lệnh-hay-dùng)
- [🗄️ Database](#️-database)
- [📁 File, Quyền & User](#-file-quyền--user-linux)
- [🔐 SSH & Truyền file](#-ssh--truyền-file-transfer)
- [🌐 HTTP, API & SSL/Certificate](#-http-api--sslcertificate)
- [🧰 JSON/YAML & Môi trường](#-công-cụ-xử-lý-jsonyaml--môi-trường)
- [🏗️ IaC & CI/CD (Terraform/Ansible)](#️-iac--cicd-terraform--ansible)
- [📦 Package Managers](#-package-managers-node--python)
- [☁️ Cloud CLI (AWS/GCP/Azure)](#️-cloud-cli-aws--gcp--azure)
- [📊 Monitoring & Observability](#-monitoring--observability)
- [📨 Message Queue (Kafka/RabbitMQ)](#-message-queue-kafka--rabbitmq)
- [🔀 Nginx & Reverse Proxy](#-nginx--reverse-proxy)
- [🛡️ Firewall](#️-firewall-iptables--ufw--firewalld)
- [⚡ Performance Profiling](#-performance-profiling--debug-sâu)
- [🚀 ArgoCD & GitOps](#-argocd--gitops)
- [🔧 CI/CD (GitHub Actions / GitLab CI)](#-cicd-github-actions--gitlab-ci)
- [🔑 Secrets (Vault / kubeseal / SOPS)](#-secrets-vault--kubeseal--sops)
- [⚙️ systemd nâng cao](#️-systemd-nâng-cao)
- [📦 Container runtime khác (podman/crictl/nerdctl)](#-container-runtime-khác-podman--crictl--nerdctl)
- [🕸️ Service Mesh (Istio / Linkerd)](#️-service-mesh-istio--linkerd)
- [🐶 Công cụ TUI cho Kubernetes (k9s/stern/kubectx)](#-công-cụ-tui-cho-kubernetes-k9s--stern--kubectx)
- [☸️ Kubernetes vận hành nâng cao](#️-kubernetes-vận-hành-nâng-cao)
- [🔥 Load Testing & Benchmark](#-load-testing--benchmark)
- [🖥️ tmux & screen (giữ session)](#️-tmux--screen-giữ-session)
- [🌩️ Network debug sâu (tcpdump/mtr/tshark)](#️-network-debug-sâu-tcpdump--mtr--tshark)
- [💾 Backup & Disaster Recovery (Velero/etcd)](#-backup--disaster-recovery-velero--etcd)
- [📜 cert-manager (TLS tự động trên K8s)](#-cert-manager-tls-tự-động-trên-k8s)
- [🎛️ Vận hành & Backup Cluster K8s (manifest/data/DR)](#️-vận-hành--backup-cluster-k8s-manifest--data--dr)

---

## 🐳 Docker

### Container - Quản lý container
```bash
docker ps                              # Liệt kê container đang chạy
docker ps -a                           # Liệt kê tất cả container (cả đã dừng)
docker run <image>                     # Chạy container từ image
docker run -d <image>                  # Chạy nền (detached)
docker run -it <image> bash            # Chạy tương tác + mở bash
docker run --name myapp <image>        # Đặt tên cho container
docker run -p 8080:80 <image>          # Map port host:container
docker run -v $(pwd):/app <image>      # Mount volume (host:container)
docker run --env-file .env <image>     # Nạp biến môi trường từ file
docker run -e KEY=value <image>        # Set 1 biến môi trường
docker run --rm <image>                # Tự xóa container sau khi dừng
docker start <container>               # Khởi động lại container đã dừng
docker stop <container>                # Dừng container
docker restart <container>             # Restart container
docker rm <container>                  # Xóa container
docker rm -f <container>               # Xóa cưỡng bức (đang chạy vẫn xóa)
docker rm -f $(docker ps -aq)          # Xóa TẤT CẢ container
docker pause / unpause <container>     # Tạm dừng / tiếp tục process
docker rename <old> <new>              # Đổi tên container
```

### Exec & Logs - Vào container, xem log
```bash
docker exec -it <container> bash       # Mở shell trong container
docker exec -it <container> sh         # Dùng sh nếu không có bash
docker exec -it <container> <cmd>      # Chạy 1 lệnh trong container
docker logs <container>                # Xem log
docker logs -f <container>             # Theo dõi log realtime (follow)
docker logs --tail 100 <container>     # 100 dòng log cuối
docker logs --since 10m <container>    # Log trong 10 phút gần nhất
docker attach <container>              # Gắn vào tiến trình chính của container
docker top <container>                 # Xem process đang chạy trong container
docker stats                           # Theo dõi CPU/RAM realtime
docker inspect <container>             # Xem chi tiết cấu hình (JSON)
docker port <container>                # Xem mapping port
docker cp <container>:/path ./local    # Copy file từ container ra host
docker cp ./local <container>:/path    # Copy file từ host vào container
docker diff <container>                # Xem thay đổi filesystem
```

### Images - Quản lý image
```bash
docker images                          # Liệt kê image
docker pull <image>:<tag>              # Tải image về
docker push <image>:<tag>              # Đẩy image lên registry
docker build -t myapp:1.0 .            # Build image từ Dockerfile
docker build -t myapp:1.0 -f Dockerfile.dev .   # Chỉ định Dockerfile
docker build --no-cache -t myapp .     # Build không dùng cache
docker tag <image> myrepo/myapp:1.0    # Gắn tag mới cho image
docker rmi <image>                     # Xóa image
docker rmi -f <image>                  # Xóa cưỡng bức
docker rmi $(docker images -q)         # Xóa tất cả image
docker history <image>                 # Xem các layer của image
docker save -o app.tar <image>         # Xuất image ra file tar
docker load -i app.tar                 # Nạp image từ file tar
docker image prune                     # Xóa image dangling (không tag)
docker image prune -a                  # Xóa tất cả image không dùng
```

### Volume & Network
```bash
docker volume ls                       # Liệt kê volume
docker volume create <name>            # Tạo volume
docker volume rm <name>                # Xóa volume
docker volume inspect <name>           # Chi tiết volume
docker volume prune                    # Xóa volume không dùng

docker network ls                      # Liệt kê network
docker network create <name>           # Tạo network
docker network rm <name>               # Xóa network
docker network connect <net> <cont>    # Kết nối container vào network
docker network disconnect <net> <cont> # Ngắt kết nối
docker network inspect <name>          # Chi tiết network
```

### Registry - Đăng nhập / đăng xuất
```bash
docker login                           # Đăng nhập Docker Hub
docker login <registry-url>            # Đăng nhập registry riêng
docker logout                          # Đăng xuất
```

### Dọn dẹp (Cleanup) - Rất hữu ích khi hết dung lượng
```bash
docker system df                       # Xem dung lượng đang dùng
docker system prune                    # Dọn container/network/image dangling
docker system prune -a                 # Dọn TẤT CẢ (kể cả image không dùng)
docker system prune -a --volumes       # Dọn cả volume (cẩn thận mất data!)
docker container prune                  # Xóa container đã dừng
```

### Format tùy chỉnh (hay dùng)
```bash
docker ps --format "{{.Names}}"                              # Chỉ hiện tên
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"  # Bảng gọn
```

---

## 🐙 Docker Compose

> Docker mới dùng `docker compose` (có space), bản cũ dùng `docker-compose` (có gạch).

```bash
docker compose up                      # Khởi động các service
docker compose up -d                   # Khởi động chạy nền (detached)
docker compose up -d --build           # Build lại image rồi chạy
docker compose up --env-file .env -d   # Nạp biến môi trường từ file
docker compose down                    # Dừng & xóa container, network
docker compose down -v                 # Dừng và xóa cả volume (mất data!)
docker compose down --rmi all          # Dừng và xóa cả image
docker compose stop                    # Chỉ dừng, không xóa
docker compose start                   # Khởi động lại các service đã dừng
docker compose restart                 # Restart tất cả service
docker compose restart <service>       # Restart 1 service cụ thể
docker compose ps                      # Liệt kê service trong compose
docker compose logs                    # Xem log tất cả service
docker compose logs -f                 # Theo dõi log realtime
docker compose logs -f <service>       # Log của 1 service
docker compose exec <service> bash     # Mở shell trong service
docker compose run <service> <cmd>     # Chạy 1 lệnh one-off
docker compose build                   # Build lại image
docker compose build --no-cache        # Build không cache
docker compose pull                    # Tải image mới nhất
docker compose config                  # Kiểm tra & in ra config đã merge
docker compose top                     # Xem process của các service
docker compose -f docker-compose.prod.yml up -d   # Chỉ định file compose
docker compose --profile dev up        # Chạy theo profile
docker compose scale <service>=3       # Scale service lên 3 instance
```

---

## ⎈ Helm

### Repository
```bash
helm repo add <name> <url>             # Thêm chart repo
helm repo update                       # Cập nhật repo
helm repo list                         # Liệt kê repo
helm repo remove <name>                # Xóa repo
helm search repo <keyword>             # Tìm chart trong repo đã add
helm search hub <keyword>              # Tìm chart trên Artifact Hub
```

### Cài đặt & quản lý release
```bash
helm install <release> <chart>         # Cài đặt chart
helm install <release> <chart> -n <namespace>          # Chỉ định namespace
helm install <release> <chart> --create-namespace -n ns  # Tạo namespace nếu chưa có
helm install <release> <chart> -f values.yaml          # Dùng file values riêng
helm install <release> <chart> --set key=value         # Ghi đè 1 giá trị
helm install <release> <chart> --dry-run --debug       # Chạy thử, không cài thật
helm upgrade <release> <chart>         # Nâng cấp release
helm upgrade --install <release> <chart>   # Cài nếu chưa có, nâng cấp nếu đã có (hay dùng)
helm upgrade <release> <chart> -f values.yaml --atomic # Rollback tự động nếu fail
helm uninstall <release>               # Gỡ release
helm uninstall <release> -n <namespace>
helm rollback <release> <revision>     # Quay lại version trước
```

### Xem thông tin
```bash
helm list                              # Liệt kê release
helm list -A                           # Liệt kê tất cả namespace
helm list -n <namespace>               # Trong 1 namespace
helm status <release>                  # Trạng thái release
helm history <release>                 # Lịch sử các bản deploy
helm get values <release>              # Xem values đang dùng
helm get manifest <release>            # Xem manifest đã render
helm get all <release>                 # Xem tất cả thông tin
```

### Chart development
```bash
helm create <name>                     # Tạo chart mẫu
helm lint <chart>                      # Kiểm tra lỗi cú pháp chart
helm template <chart>                  # Render template ra YAML (không cài)
helm template <chart> -f values.yaml   # Render với values
helm package <chart>                   # Đóng gói chart thành .tgz
helm show values <chart>               # Xem values mặc định của chart
helm show chart <chart>                # Xem metadata của chart
helm dependency update <chart>         # Cập nhật dependency chart
```

---

## ☸️ kubectl

### Context & Cluster
```bash
kubectl config get-contexts            # Liệt kê context
kubectl config current-context         # Context hiện tại
kubectl config use-context <name>      # Chuyển context
kubectl config set-context --current --namespace=<ns>   # Đổi namespace mặc định
kubectl cluster-info                   # Thông tin cluster
kubectl version                        # Phiên bản client/server
kubectl get nodes                      # Liệt kê node
kubectl get nodes -o wide              # Chi tiết hơn (IP, OS...)
kubectl top nodes                      # CPU/RAM của node (cần metrics-server)
```

### Xem resource (get / describe)
```bash
kubectl get pods                       # Liệt kê pod
kubectl get pods -A                    # Tất cả namespace
kubectl get pods -n <namespace>        # Trong 1 namespace
kubectl get pods -o wide               # Chi tiết (node, IP...)
kubectl get pods -w                    # Theo dõi realtime (watch)
kubectl get pods -l app=nginx          # Lọc theo label
kubectl get all                        # Tất cả resource cơ bản
kubectl get svc                        # Service
kubectl get deploy                     # Deployment
kubectl get ns                         # Namespace
kubectl get nodes,pods,svc             # Nhiều loại cùng lúc
kubectl get pod <pod> -o yaml          # Xuất ra YAML
kubectl get pod <pod> -o json          # Xuất ra JSON
kubectl describe pod <pod>             # Chi tiết + events (debug rất hay)
kubectl describe node <node>
kubectl get events --sort-by=.metadata.creationTimestamp   # Xem events
```

### Tạo / sửa / xóa resource
```bash
kubectl apply -f <file.yaml>           # Tạo/cập nhật từ file (khai báo)
kubectl apply -f <dir>/                # Áp dụng cả thư mục
kubectl create -f <file.yaml>          # Tạo mới (báo lỗi nếu đã có)
kubectl delete -f <file.yaml>          # Xóa theo file
kubectl delete pod <pod>               # Xóa pod
kubectl delete pod <pod> --grace-period=0 --force   # Xóa cưỡng bức
kubectl delete deploy <name>           # Xóa deployment
kubectl edit deploy <name>             # Sửa trực tiếp (mở editor)
kubectl scale deploy <name> --replicas=3   # Scale số bản
kubectl rollout restart deploy <name>  # Restart deployment (rolling)
kubectl rollout status deploy <name>   # Xem tiến trình rollout
kubectl rollout undo deploy <name>     # Rollback về version trước
kubectl rollout history deploy <name>  # Lịch sử rollout
kubectl set image deploy/<name> <container>=<image>:<tag>   # Đổi image
```

### Debug & Log
```bash
kubectl logs <pod>                     # Xem log pod
kubectl logs -f <pod>                  # Theo dõi log realtime
kubectl logs <pod> -c <container>      # Log của 1 container (pod nhiều container)
kubectl logs --tail=100 <pod>          # 100 dòng cuối
kubectl logs --previous <pod>          # Log của lần chạy trước (khi pod crash)
kubectl exec -it <pod> -- bash         # Mở shell trong pod
kubectl exec -it <pod> -- sh           # Dùng sh
kubectl exec <pod> -- <cmd>            # Chạy 1 lệnh
kubectl cp <pod>:/path ./local         # Copy file từ pod ra host
kubectl cp ./local <pod>:/path         # Copy vào pod
kubectl port-forward <pod> 8080:80     # Forward port pod về máy local
kubectl port-forward svc/<svc> 8080:80 # Forward port của service
kubectl attach -it <pod>               # Gắn vào pod
kubectl debug <pod> -it --image=busybox  # Ephemeral container để debug
```

### Namespace & Config
```bash
kubectl create namespace <name>        # Tạo namespace
kubectl delete namespace <name>        # Xóa namespace
kubectl get configmap                  # Liệt kê configmap
kubectl get secret                     # Liệt kê secret
kubectl create secret generic <name> --from-literal=key=value  # Tạo secret
kubectl get secret <name> -o jsonpath='{.data.key}' | base64 -d # Đọc secret
kubectl create configmap <name> --from-file=<file>   # Tạo configmap từ file
```

### Tiện ích khác
```bash
kubectl api-resources                  # Liệt kê tất cả loại resource
kubectl explain <resource>             # Xem tài liệu về resource
kubectl label pod <pod> env=prod       # Gắn label
kubectl annotate pod <pod> note=abc    # Gắn annotation
kubectl top pods                       # CPU/RAM của pod
kubectl apply -k <dir>                 # Kustomize
kubectl run tmp --rm -it --image=busybox -- sh   # Pod tạm để test nhanh
```

### Alias siêu ngắn (nên add vào ~/.zshrc)
```bash
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deploy'
alias kaf='kubectl apply -f'
alias kdf='kubectl delete -f'
alias kl='kubectl logs -f'
alias kex='kubectl exec -it'
alias kns='kubectl config set-context --current --namespace'
alias kctx='kubectl config use-context'
```

---

## 🔍 grep & Xử lý text (log filtering)

### grep - Tìm kiếm trong text/log
```bash
grep "error" file.log                  # Tìm dòng chứa "error"
grep -i "error" file.log               # Không phân biệt hoa/thường (ignore case)
grep -v "debug" file.log               # Đảo ngược: dòng KHÔNG chứa "debug"
grep -n "error" file.log               # Kèm số dòng
grep -c "error" file.log               # Đếm số dòng khớp
grep -r "TODO" .                       # Tìm đệ quy trong thư mục
grep -w "id" file.log                  # Khớp đúng cả từ (whole word)
grep -l "error" *.log                  # Chỉ in tên file có khớp
grep -o "user_[0-9]*" file.log         # Chỉ in phần khớp, không cả dòng

# -E: dùng regex mở rộng (extended), hay đi cùng -i
grep -iE "error|fail|exception" app.log        # Tìm nhiều từ (OR) không phân biệt hoa thường
grep -iE "timeout|refused|reset" app.log       # Lọc lỗi network
grep -E "[0-9]{3}" access.log                  # Regex: 3 chữ số liền nhau

# -A / -B / -C: xem context quanh dòng khớp (rất hay khi debug)
grep -A 6 "Exception" app.log          # 6 dòng SAU (After) dòng khớp
grep -B 3 "Exception" app.log          # 3 dòng TRƯỚC (Before)
grep -C 5 "Exception" app.log          # 5 dòng trước VÀ sau (Context)
grep -A 10 -i "traceback" app.log      # Xem 10 dòng stack trace sau lỗi

# Kết hợp với các lệnh khác qua pipe
docker logs app 2>&1 | grep -iE "error|fatal"          # Lọc log docker
kubectl logs pod 2>&1 | grep -A 6 -i "exception"       # Lọc log k8s + context
tail -f app.log | grep --line-buffered -iE "error"     # Lọc log realtime
ps aux | grep -i nginx                                 # Tìm process
grep -riE "password|secret|token" .                    # Rà tìm secret bị lộ
```

### Các công cụ text khác hay dùng khi troubleshoot
```bash
tail -f app.log                        # Theo dõi log realtime
tail -n 100 app.log                    # 100 dòng cuối
head -n 50 app.log                     # 50 dòng đầu
less app.log                           # Xem file phân trang (/ để tìm, G xuống cuối)
wc -l app.log                          # Đếm số dòng
sort file.txt | uniq -c | sort -nr     # Đếm & sắp xếp theo tần suất (top lỗi)
awk '{print $1}' access.log | sort | uniq -c | sort -nr   # Top IP truy cập
awk '/error/ {print $0}' app.log       # Lọc bằng awk
sed -n '100,200p' app.log              # In dòng 100 đến 200
cut -d',' -f1,3 data.csv               # Cắt cột theo dấu phân cách
cat app.log | grep error | wc -l       # Đếm số lỗi
```

---

## 🚑 Troubleshooting sự cố (Linux)

### Process & Tài nguyên (CPU / RAM)
```bash
top                                    # Xem process realtime (CPU, RAM)
htop                                   # Bản đẹp hơn top (nếu có cài)
ps aux                                 # Liệt kê tất cả process
ps aux --sort=-%cpu | head            # Top process ăn CPU
ps aux --sort=-%mem | head            # Top process ăn RAM
ps -ef | grep <name>                   # Tìm process theo tên
pgrep -f <name>                        # Lấy PID theo tên
kill <PID>                             # Dừng process (gửi SIGTERM)
kill -9 <PID>                          # Dừng cưỡng bức (SIGKILL)
pkill -f <name>                        # Kill theo tên
free -h                                # Xem RAM/swap (dạng dễ đọc)
uptime                                 # Tải hệ thống (load average)
vmstat 1                               # Thống kê CPU/memory/IO mỗi giây
nproc                                  # Số CPU core
```

### Ổ đĩa (Disk)
```bash
df -h                                  # Dung lượng ổ đĩa còn trống
du -sh *                               # Kích thước từng thư mục/file
du -sh * | sort -rh | head             # Top thư mục nặng nhất
du -ah . | sort -rh | head -20         # Top file nặng nhất
lsof | grep deleted                    # File đã xóa nhưng process còn giữ (ngốn disk)
ncdu                                   # Duyệt dung lượng tương tác (nếu có cài)
```

### Network
```bash
ping <host>                            # Kiểm tra kết nối
curl -I <url>                          # Chỉ lấy HTTP header
curl -v <url>                          # Verbose (xem chi tiết handshake)
curl -w "%{http_code}" -o /dev/null -s <url>   # Chỉ lấy status code
ss -tulpn                              # Liệt kê port đang lắng nghe (thay netstat)
netstat -tulpn                         # (bản cũ) port đang mở
lsof -i :8080                          # Ai đang chiếm port 8080
lsof -iTCP -sTCP:LISTEN -P -n          # Tất cả port đang LISTEN
nslookup <domain>                      # Tra DNS
dig <domain>                           # Tra DNS chi tiết
traceroute <host>                      # Truy vết đường đi mạng
telnet <host> <port>                   # Test kết nối tới port
nc -zv <host> <port>                   # Test port bằng netcat (nhanh)
ip a                                   # Xem địa chỉ IP các interface
curl ifconfig.me                       # Xem IP public
tcpdump -i any port 80                 # Bắt gói tin (cần quyền root)
```

### Log hệ thống & Service (systemd)
```bash
journalctl -u <service>                # Log của 1 service
journalctl -u <service> -f             # Theo dõi realtime
journalctl -u <service> --since "10 min ago"   # Log gần đây
journalctl -xe                         # Log lỗi gần nhất (hay dùng khi service fail)
journalctl -p err -b                   # Chỉ log lỗi trong lần boot này
systemctl status <service>             # Trạng thái service
systemctl restart <service>            # Restart service
systemctl start / stop <service>       # Bật / tắt
systemctl enable / disable <service>   # Tự chạy khi boot / tắt
dmesg | tail                           # Log kernel (lỗi phần cứng, OOM...)
dmesg | grep -i "out of memory"        # Kiểm tra bị OOM kill
```

### Troubleshoot Docker / Kubernetes
```bash
docker stats                           # CPU/RAM realtime của container
docker inspect <container>             # Xem chi tiết cấu hình
docker logs --tail 200 <container>     # 200 dòng log cuối
docker events                          # Theo dõi sự kiện docker realtime
docker system df                       # Xem docker dùng bao nhiêu disk

kubectl get pods                       # Kiểm tra pod nào lỗi (CrashLoopBackOff...)
kubectl describe pod <pod>             # Xem Events để biết lý do lỗi (rất quan trọng)
kubectl logs --previous <pod>          # Log lần chạy trước khi pod crash
kubectl get events --sort-by=.metadata.creationTimestamp   # Events toàn cluster
kubectl top pods                       # Pod nào ăn nhiều CPU/RAM
```

---

## 🌿 Git - Troubleshooting & lệnh hay dùng

### Xem trạng thái & lịch sử
```bash
git status                             # Trạng thái working tree
git log --oneline --graph --decorate --all   # Xem lịch sử dạng cây (đẹp)
git log -p <file>                      # Lịch sử thay đổi 1 file kèm diff
git log --author="ten"                 # Commit của 1 người
git show <commit>                      # Chi tiết 1 commit
git diff                               # Thay đổi chưa staged
git diff --staged                      # Thay đổi đã staged
git diff main..feature                 # So sánh 2 branch
git blame <file>                       # Ai sửa dòng nào (rất hay khi truy lỗi)
git reflog                             # Lịch sử MỌI thao tác HEAD (cứu tinh khi lỡ tay)
```

### Undo / Reset - Sửa sai (hay cần khi có sự cố)
```bash
git restore <file>                     # Bỏ thay đổi chưa staged của file
git restore --staged <file>            # Bỏ staged (unstage), giữ thay đổi
git checkout -- <file>                 # (bản cũ) khôi phục file về HEAD
git reset --soft HEAD~1                # Bỏ commit cuối, GIỮ thay đổi (đã staged)
git reset HEAD~1                       # Bỏ commit cuối, giữ thay đổi (unstaged)
git reset --hard HEAD~1                # Bỏ commit cuối + XÓA thay đổi (cẩn thận!)
git reset --hard origin/main           # Ép branch về đúng như remote (mất local!)
git revert <commit>                    # Tạo commit mới đảo ngược 1 commit (an toàn)
git commit --amend                     # Sửa commit cuối (nội dung/message)
git commit --amend --no-edit           # Thêm file vào commit cuối, giữ message
git clean -fd                          # Xóa file/thư mục chưa track (cẩn thận!)
git clean -nd                          # Xem trước sẽ xóa gì (dry-run)
```

### Xử lý conflict khi merge/rebase
```bash
git merge <branch>                     # Merge branch vào branch hiện tại
git rebase main                        # Rebase branch hiện tại lên main
# Khi bị conflict:
git status                             # Xem file nào conflict
# ... sửa file, xóa dấu <<<<<<< ======= >>>>>>> ...
git add <file>                         # Đánh dấu đã giải quyết
git rebase --continue                  # Tiếp tục rebase
git merge --continue                   # Tiếp tục merge
git rebase --abort                     # Hủy rebase, quay về trạng thái cũ
git merge --abort                      # Hủy merge
git checkout --theirs <file>           # Lấy bản của branch được merge vào
git checkout --ours <file>             # Lấy bản của branch hiện tại
```

### Branch & Stash
```bash
git branch                             # Liệt kê branch
git branch -a                          # Cả branch remote
git checkout -b <branch>               # Tạo và chuyển sang branch mới
git branch -D <branch>                 # Xóa branch (cưỡng bức)
git branch -m <new-name>               # Đổi tên branch hiện tại
git stash                              # Cất tạm thay đổi (để chuyển branch)
git stash list                         # Liệt kê stash
git stash pop                          # Lấy lại + xóa stash mới nhất
git stash apply                        # Lấy lại nhưng giữ stash
git stash drop                         # Xóa stash
git cherry-pick <commit>               # Lấy 1 commit từ branch khác về
git fetch origin                       # Tải cập nhật từ remote (không merge)
git pull --rebase origin main          # Pull kiểu rebase (lịch sử sạch hơn)
git push --force-with-lease            # Force push an toàn (không đè của người khác)
git remote -v                          # Xem URL remote
```

### Tình huống hay gặp
```bash
git commit --allow-empty -m "trigger CI"   # Commit rỗng để trigger CI/CD
git rm --cached <file>                 # Bỏ file khỏi git nhưng giữ trên disk
git log --oneline | head               # Xem nhanh vài commit gần nhất
git reflog                             # Tìm lại commit "mất" sau reset --hard
git reset --hard <commit-tu-reflog>    # Khôi phục về commit đã lỡ xóa
```

---

## 🗄️ Database

### PostgreSQL (psql)
```bash
psql -U <user> -d <database>           # Kết nối vào database
psql -h <host> -p 5432 -U <user> -d <db>   # Kết nối remote
psql -U user -d db -c "SELECT 1;"      # Chạy 1 câu lệnh rồi thoát

# Trong psql:
\l                                     # Liệt kê database
\c <database>                          # Chuyển database
\dt                                    # Liệt kê bảng
\d <table>                             # Mô tả cấu trúc bảng
\du                                    # Liệt kê user/role
\dn                                    # Liệt kê schema
\x                                     # Bật/tắt hiển thị dọc (dễ đọc)
\timing                                # Bật đo thời gian query
\q                                     # Thoát

# Backup & Restore
pg_dump -U user -d db > backup.sql             # Backup database
pg_dump -U user -d db -t <table> > tbl.sql     # Backup 1 bảng
psql -U user -d db < backup.sql                # Restore
pg_dump -U user -Fc db > backup.dump           # Backup dạng nén
pg_restore -U user -d db backup.dump           # Restore từ dump nén

# Troubleshoot (kết nối chậm/treo)
# SELECT * FROM pg_stat_activity;              # Xem query đang chạy
# SELECT pg_terminate_backend(<pid>);         # Kill 1 query treo
```

### MySQL / MariaDB
```bash
mysql -u <user> -p                     # Kết nối (sẽ hỏi password)
mysql -h <host> -P 3306 -u user -p db  # Kết nối remote vào database

# Trong mysql:
SHOW DATABASES;                        # Liệt kê database
USE <database>;                        # Chọn database
SHOW TABLES;                           # Liệt kê bảng
DESCRIBE <table>;                      # Mô tả bảng
SHOW PROCESSLIST;                      # Xem query đang chạy (troubleshoot)
KILL <id>;                             # Kill query treo
SHOW STATUS;                           # Trạng thái server
EXIT;                                  # Thoát

# Backup & Restore
mysqldump -u user -p db > backup.sql           # Backup database
mysqldump -u user -p db table > table.sql      # Backup 1 bảng
mysqldump -u user -p --all-databases > all.sql # Backup tất cả
mysql -u user -p db < backup.sql               # Restore
```

### Redis (redis-cli)
```bash
redis-cli                              # Kết nối (mặc định localhost:6379)
redis-cli -h <host> -p 6379 -a <pass>  # Kết nối remote có password

# Trong redis-cli:
PING                                   # Kiểm tra sống (trả PONG)
KEYS *                                 # Liệt kê tất cả key (tránh dùng trên prod!)
SCAN 0                                 # Duyệt key an toàn hơn KEYS
GET <key>                              # Lấy giá trị
SET <key> <value>                      # Gán giá trị
DEL <key>                              # Xóa key
TTL <key>                              # Thời gian sống còn lại của key
EXPIRE <key> 60                        # Đặt hết hạn 60 giây
TYPE <key>                             # Kiểu dữ liệu của key
INFO                                   # Thông tin server (memory, clients...)
INFO memory                            # Thông tin RAM đang dùng
DBSIZE                                 # Số lượng key
FLUSHDB                                # Xóa toàn bộ DB hiện tại (cẩn thận!)
MONITOR                                # Xem mọi lệnh realtime (debug)
CLIENT LIST                            # Danh sách client đang kết nối
```

### MongoDB (mongosh)
```bash
mongosh                                # Kết nối local
mongosh "mongodb://user:pass@host:27017/db"    # Kết nối remote

# Trong mongosh:
show dbs                               # Liệt kê database
use <database>                         # Chọn database
show collections                       # Liệt kê collection
db.<coll>.find()                       # Truy vấn tất cả
db.<coll>.find({name: "abc"})          # Truy vấn có điều kiện
db.<coll>.countDocuments()             # Đếm document
db.<coll>.insertOne({...})             # Thêm 1 document
db.<coll>.deleteOne({...})             # Xóa
db.currentOp()                         # Xem thao tác đang chạy (troubleshoot)
db.stats()                             # Thống kê database

# Backup & Restore
mongodump --db <db> --out ./backup             # Backup
mongorestore --db <db> ./backup/<db>           # Restore
```

---

## 📁 File, Quyền & User (Linux)

### File & thư mục
```bash
ls -lah                                # Liệt kê chi tiết + ẩn + dung lượng dễ đọc
find . -name "*.log"                    # Tìm file theo tên
find . -type f -size +100M             # Tìm file > 100MB (ngốn disk)
find . -mtime -1                       # File sửa trong 1 ngày qua
find . -name "*.tmp" -delete           # Tìm và xóa
find /var/log -name "*.log" -mtime +30 -delete   # Dọn log cũ > 30 ngày
stat <file>                            # Thông tin chi tiết file
file <file>                            # Xác định loại file
readlink -f <file>                     # Đường dẫn thật của symlink
ln -s <target> <link>                  # Tạo symbolic link
watch -n 2 'ls -la'                    # Chạy lặp lệnh mỗi 2 giây (theo dõi)
```

### Quyền (permission) & Sở hữu (ownership)
```bash
chmod +x script.sh                     # Cấp quyền thực thi
chmod 755 file                         # rwxr-xr-x (chủ full, còn lại đọc+chạy)
chmod 644 file                         # rw-r--r-- (chủ đọc/ghi, còn lại đọc)
chmod -R 755 <dir>                     # Đệ quy cả thư mục
chown user:group file                  # Đổi chủ sở hữu
chown -R user:group <dir>              # Đệ quy
chgrp group file                       # Đổi nhóm
umask                                  # Xem quyền mặc định khi tạo file
sudo <command>                         # Chạy với quyền root
sudo -i                                # Mở shell root
sudo su - <user>                       # Chuyển sang user khác
sudo !!                                # Chạy lại lệnh trước với sudo
```

### User & Nhóm
```bash
whoami                                 # User hiện tại
id                                     # UID, GID, nhóm
groups                                 # Các nhóm của user
who / w                                # Ai đang đăng nhập
last                                   # Lịch sử đăng nhập
useradd <user> / userdel <user>        # Thêm / xóa user
passwd <user>                          # Đổi mật khẩu
usermod -aG <group> <user>             # Thêm user vào nhóm
```

---

## 🔐 SSH & Truyền file (transfer)

### SSH
```bash
ssh user@host                          # Kết nối SSH
ssh -p 2222 user@host                  # Chỉ định port
ssh -i key.pem user@host               # Dùng private key
ssh -v user@host                       # Verbose (debug lỗi kết nối)
ssh user@host "df -h"                  # Chạy 1 lệnh từ xa rồi thoát
ssh -L 8080:localhost:80 user@host     # Local port forward (tunnel)
ssh -D 1080 user@host                  # SOCKS proxy
ssh-keygen -t ed25519                  # Tạo cặp SSH key
ssh-copy-id user@host                  # Copy public key lên server (login không mật khẩu)
cat ~/.ssh/config                      # Cấu hình SSH (Host, HostName, User...)
```

### Truyền file
```bash
scp file.txt user@host:/path           # Copy file lên server
scp user@host:/path/file.txt ./        # Tải file về
scp -r <dir> user@host:/path           # Copy cả thư mục
rsync -avz <src> user@host:/dest       # Đồng bộ (nhanh, chỉ gửi phần khác)
rsync -avz --delete <src> <dest>       # Đồng bộ + xóa file thừa ở đích
rsync -avz --progress <src> <dest>     # Hiện tiến trình
rsync -avz -e "ssh -p 2222" <src> <dest>   # Rsync qua SSH port tùy chỉnh
```

### Nén & giải nén
```bash
tar -czvf archive.tar.gz <dir>         # Nén thư mục (gzip)
tar -xzvf archive.tar.gz               # Giải nén
tar -xzvf archive.tar.gz -C /dest      # Giải nén vào thư mục cụ thể
tar -tzvf archive.tar.gz               # Xem nội dung không giải nén
zip -r archive.zip <dir>               # Nén zip
unzip archive.zip                      # Giải nén zip
unzip -l archive.zip                   # Xem nội dung zip
gzip file / gunzip file.gz             # Nén / giải nén 1 file
```

---

## 🌐 HTTP, API & SSL/Certificate

### curl - Test API (rất hay khi debug service)
```bash
curl <url>                             # GET request
curl -I <url>                          # Chỉ lấy header
curl -v <url>                          # Verbose (xem handshake, header đầy đủ)
curl -L <url>                          # Follow redirect
curl -X POST <url> -d '{"a":1}' -H "Content-Type: application/json"   # POST JSON
curl -X POST <url> -H "Authorization: Bearer <token>"    # Kèm token
curl -o file.zip <url>                 # Tải và lưu file
curl -s <url> | jq                     # Lấy JSON rồi format đẹp
curl -w "\nTime: %{time_total}s Code: %{http_code}\n" -o /dev/null -s <url>  # Đo thời gian + status
curl -k <url>                          # Bỏ qua kiểm tra SSL (self-signed)
curl --resolve host:443:1.2.3.4 https://host/   # Test IP cụ thể (bỏ qua DNS)
wget <url>                             # Tải file
wget -c <url>                          # Tải tiếp (resume)
http GET <url>                         # HTTPie (dễ đọc hơn curl, nếu có cài)
```

### SSL / Certificate (check hết hạn, cấu hình)
```bash
# Xem cert của 1 domain (ngày hết hạn, issuer...)
echo | openssl s_client -connect <host>:443 -servername <host> 2>/dev/null | openssl x509 -noout -dates
openssl s_client -connect <host>:443 -servername <host>   # Xem full handshake
openssl x509 -in cert.pem -text -noout # Đọc nội dung cert file
openssl x509 -in cert.pem -noout -dates    # Chỉ xem ngày hiệu lực/hết hạn
openssl verify -CAfile ca.crt cert.pem # Kiểm tra chuỗi cert
curl -vI https://<host> 2>&1 | grep -i "expire"   # Xem hạn cert nhanh
nmap --script ssl-cert -p 443 <host>   # Quét thông tin SSL
```

### DNS
```bash
dig <domain>                           # Tra DNS đầy đủ
dig +short <domain>                    # Chỉ lấy IP
dig <domain> MX / TXT / NS             # Tra bản ghi cụ thể
dig @8.8.8.8 <domain>                  # Hỏi DNS server cụ thể
host <domain>                          # Tra nhanh
nslookup <domain>
whois <domain>                         # Thông tin đăng ký domain
```

---

## 🧰 Công cụ xử lý JSON/YAML & môi trường

### jq - Xử lý JSON (cực hữu ích cho DevOps)
```bash
cat data.json | jq                     # Format đẹp (pretty print)
jq '.name' data.json                   # Lấy field name
jq '.items[]' data.json                # Duyệt mảng
jq '.items[].id' data.json             # Lấy id của từng phần tử
jq '.items | length' data.json         # Đếm số phần tử
jq -r '.name' data.json                # Raw (bỏ dấu ngoặc kép)
jq '.items[] | select(.status=="ok")' data.json   # Lọc theo điều kiện
kubectl get pods -o json | jq '.items[].metadata.name'   # Kết hợp với kubectl
curl -s <api> | jq '.data'             # Kết hợp với curl
```

### yq - Xử lý YAML (giống jq nhưng cho YAML)
```bash
yq '.version' config.yaml              # Lấy field
yq -i '.image.tag = "1.2.3"' values.yaml   # Sửa file tại chỗ (hay khi CI/CD)
yq eval '.services | keys' docker-compose.yml   # Lấy danh sách service
yq -o=json config.yaml                 # Convert YAML sang JSON
```

### Biến môi trường (Environment variables)
```bash
env                                    # Liệt kê tất cả biến môi trường
printenv <VAR>                         # In 1 biến
echo $PATH                             # Xem biến PATH
export KEY=value                       # Set biến cho session hiện tại
export PATH=$PATH:/new/path            # Thêm vào PATH
unset KEY                              # Xóa biến
set -a; source .env; set +a            # Nạp toàn bộ file .env vào môi trường
KEY=value command                      # Set biến chỉ cho 1 lệnh
```

### Cron - Lập lịch chạy tự động
```bash
crontab -l                             # Xem lịch cron hiện tại
crontab -e                             # Sửa lịch cron
crontab -r                             # Xóa toàn bộ cron
# Cú pháp: phút giờ ngày tháng thứ  lệnh
# 0 2 * * *  /path/backup.sh           # Chạy 2h sáng mỗi ngày
# */5 * * * * /path/check.sh           # Mỗi 5 phút
systemctl list-timers                  # Xem systemd timer (thay thế cron)
```

---

## 🏗️ IaC & CI/CD (Terraform / Ansible)

### Terraform
```bash
terraform init                         # Khởi tạo (tải provider)
terraform plan                         # Xem trước thay đổi (không apply)
terraform apply                        # Áp dụng thay đổi
terraform apply -auto-approve          # Áp dụng không hỏi xác nhận
terraform destroy                      # Xóa toàn bộ resource
terraform validate                     # Kiểm tra cú pháp
terraform fmt                          # Format code
terraform state list                   # Liệt kê resource trong state
terraform show                         # Xem state hiện tại
terraform output                       # Xem output values
terraform plan -out=tf.plan            # Lưu plan ra file
```

### Ansible
```bash
ansible all -m ping                    # Kiểm tra kết nối tới tất cả host
ansible-playbook site.yml              # Chạy playbook
ansible-playbook site.yml --check      # Dry-run (không thay đổi thật)
ansible-playbook site.yml -i inventory # Chỉ định inventory
ansible-playbook site.yml --limit web  # Chỉ chạy trên nhóm web
ansible-playbook site.yml -vvv         # Verbose (debug)
ansible-vault encrypt secrets.yml      # Mã hóa file secret
ansible all -m shell -a "df -h"        # Chạy lệnh shell trên tất cả host
```

---

## 📦 Package Managers (Node / Python)

### npm / yarn / pnpm
```bash
npm install                            # Cài dependency theo package.json
npm install <pkg>                      # Cài 1 package
npm install -g <pkg>                   # Cài global
npm ci                                 # Cài sạch theo lock (dùng trong CI)
npm run <script>                       # Chạy script trong package.json
npm run build                          # Build project
npm outdated                           # Xem package cũ
npm audit / npm audit fix              # Kiểm tra & sửa lỗ hổng bảo mật
npm cache clean --force                # Xóa cache khi lỗi lạ
rm -rf node_modules package-lock.json && npm install   # Cài lại sạch (fix lỗi)

yarn / yarn install                    # Cài dependency
yarn add <pkg>                         # Thêm package
yarn <script>                          # Chạy script

pnpm install / pnpm add <pkg>          # Tương tự nhưng nhanh & tiết kiệm disk
```

### Python (pip / venv)
```bash
python -m venv venv                    # Tạo môi trường ảo
source venv/bin/activate               # Kích hoạt (Linux/Mac)
pip install <pkg>                      # Cài package
pip install -r requirements.txt        # Cài theo file
pip freeze > requirements.txt          # Lưu danh sách package
pip list                               # Liệt kê package đã cài
pip install --upgrade <pkg>            # Nâng cấp
deactivate                             # Thoát venv
```

---

## ☁️ Cloud CLI (AWS / GCP / Azure)

### AWS CLI
```bash
aws configure                          # Cấu hình credential + region
aws sts get-caller-identity            # Kiểm tra đang login bằng account nào
aws s3 ls                              # Liệt kê bucket
aws s3 ls s3://<bucket>/               # Liệt kê nội dung bucket
aws s3 cp file s3://<bucket>/          # Upload file
aws s3 cp s3://<bucket>/file ./        # Download file
aws s3 sync ./dir s3://<bucket>/       # Đồng bộ thư mục lên S3
aws ec2 describe-instances             # Liệt kê EC2 instance
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"   # Chỉ instance đang chạy
aws ec2 start-instances --instance-ids <id>    # Bật instance
aws ec2 stop-instances --instance-ids <id>     # Tắt instance
aws logs tail <log-group> --follow     # Xem CloudWatch log realtime
aws ecr get-login-password | docker login --username AWS --password-stdin <ecr-url>   # Login ECR
aws eks update-kubeconfig --name <cluster>     # Lấy kubeconfig cho EKS
aws ecs list-services --cluster <name>         # Liệt kê ECS service
aws --profile <profile> <command>      # Dùng profile khác
```

### Google Cloud (gcloud)
```bash
gcloud auth login                      # Đăng nhập
gcloud config set project <project>    # Chọn project
gcloud config list                     # Xem cấu hình hiện tại
gcloud compute instances list          # Liệt kê VM
gcloud compute ssh <instance>          # SSH vào VM
gcloud container clusters get-credentials <cluster> --region <r>   # Kubeconfig cho GKE
gcloud container clusters list         # Liệt kê GKE cluster
gcloud projects list                   # Liệt kê project
gcloud logging read "severity>=ERROR" --limit 20   # Đọc log lỗi
gcloud auth configure-docker           # Cấu hình Docker để push lên GCR/Artifact Registry
```

### Azure CLI (az)
```bash
az login                               # Đăng nhập
az account show                        # Xem subscription hiện tại
az account set --subscription <id>     # Chọn subscription
az vm list -o table                    # Liệt kê VM
az vm start --name <vm> -g <rg>        # Bật VM
az aks get-credentials --name <cluster> -g <rg>   # Kubeconfig cho AKS
az acr login --name <registry>         # Login Azure Container Registry
az group list -o table                 # Liệt kê resource group
```

---

## 📊 Monitoring & Observability

### Prometheus / PromQL (query metric)
```bash
# Truy vấn qua HTTP API
curl -s 'http://localhost:9090/api/v1/query?query=up' | jq         # Service nào đang up
curl -s 'http://localhost:9090/api/v1/targets' | jq '.data.activeTargets[].health'   # Sức khỏe target

# PromQL hay dùng (gõ trong Prometheus UI hoặc Grafana):
# up                                     -> service sống/chết (1/0)
# rate(http_requests_total[5m])          -> tốc độ request/giây
# sum(rate(http_requests_total[5m])) by (status)   -> request theo status code
# histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))  -> p95 latency
# node_memory_MemAvailable_bytes         -> RAM còn trống
# 100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)  -> % CPU dùng
# rate(container_cpu_usage_seconds_total[5m])   -> CPU container
```

### Xem metric endpoint & health check
```bash
curl -s localhost:8080/metrics         # Metric dạng Prometheus của app
curl -s localhost:8080/health          # Health check endpoint
curl -s localhost:8080/actuator/health # Spring Boot health
watch -n 2 'curl -s localhost:8080/health'   # Theo dõi health mỗi 2s
```

### Grafana / Loki / cAdvisor (tham khảo)
```bash
# Grafana:   thường ở http://localhost:3000 (admin/admin)
# Loki:      LogQL để query log, ví dụ: {app="myapp"} |= "error"
logcli query '{app="myapp"} |= "error"'   # Query Loki bằng CLI (nếu có logcli)
# cAdvisor:  http://localhost:8080 -> xem metric container realtime
```

---

## 📨 Message Queue (Kafka / RabbitMQ)

### Kafka
```bash
# Topic
kafka-topics.sh --bootstrap-server localhost:9092 --list                      # Liệt kê topic
kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic <t>       # Chi tiết topic
kafka-topics.sh --bootstrap-server localhost:9092 --create --topic <t> --partitions 3 --replication-factor 1   # Tạo topic

# Producer / Consumer (test message)
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic <t>        # Gửi message
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic <t> --from-beginning   # Đọc từ đầu

# Consumer group (troubleshoot lag)
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list              # Liệt kê group
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group <g>   # Xem LAG (rất quan trọng)
```

### RabbitMQ
```bash
rabbitmqctl status                     # Trạng thái node
rabbitmqctl list_queues                # Liệt kê queue + số message
rabbitmqctl list_queues name messages consumers   # Queue nào tồn message (troubleshoot)
rabbitmqctl list_exchanges             # Liệt kê exchange
rabbitmqctl list_connections           # Kết nối đang mở
rabbitmqctl list_consumers             # Danh sách consumer
rabbitmqctl purge_queue <queue>        # Xóa sạch message trong queue
rabbitmq-plugins enable rabbitmq_management   # Bật UI quản lý (cổng 15672)
```

---

## 🔀 Nginx & Reverse Proxy

```bash
nginx -t                               # Kiểm tra cú pháp config (LUÔN chạy trước khi reload)
nginx -T                               # In toàn bộ config đã merge
nginx -s reload                        # Reload config (không downtime)
nginx -s stop / quit                   # Dừng
systemctl restart nginx                # Restart qua systemd
nginx -V                               # Xem version + module đã build

# File config hay gặp
# /etc/nginx/nginx.conf                 -> config chính
# /etc/nginx/sites-enabled/             -> các site đang bật
# /etc/nginx/conf.d/                    -> config bổ sung

# Xem log (troubleshoot 502/504/404)
tail -f /var/log/nginx/access.log      # Log truy cập
tail -f /var/log/nginx/error.log       # Log lỗi (quan trọng nhất khi debug)
grep " 502 " /var/log/nginx/access.log # Lọc request bị 502
awk '{print $9}' access.log | sort | uniq -c | sort -rn   # Đếm theo status code
awk '{print $7}' access.log | sort | uniq -c | sort -rn | head   # Top URL bị gọi

# Ý nghĩa lỗi hay gặp:
# 502 Bad Gateway     -> backend chết / sai upstream / sai port
# 504 Gateway Timeout -> backend phản hồi chậm (tăng proxy_read_timeout)
# 413 Too Large       -> tăng client_max_body_size
# 499                 -> client tự ngắt kết nối
```

---

## 🛡️ Firewall (iptables / ufw / firewalld)

### ufw (Ubuntu - đơn giản)
```bash
ufw status                             # Trạng thái + rule
ufw status verbose                     # Chi tiết hơn
ufw enable / disable                   # Bật / tắt firewall
ufw allow 80/tcp                       # Mở port 80
ufw allow from 1.2.3.4                 # Cho phép 1 IP
ufw allow from 1.2.3.4 to any port 22  # Cho IP truy cập port 22
ufw deny 3306                          # Chặn port
ufw delete allow 80                    # Xóa rule
```

### iptables (chi tiết, thấp cấp hơn)
```bash
iptables -L -n -v                      # Liệt kê rule (kèm số gói tin)
iptables -L -n --line-numbers          # Kèm số thứ tự rule
iptables -A INPUT -p tcp --dport 80 -j ACCEPT   # Mở port 80
iptables -A INPUT -s 1.2.3.4 -j DROP   # Chặn 1 IP
iptables -D INPUT <line-number>        # Xóa rule theo số dòng
iptables -F                            # Xóa toàn bộ rule (cẩn thận mất SSH!)
iptables-save > rules.v4               # Lưu rule
iptables-restore < rules.v4            # Khôi phục rule
```

### firewalld (RHEL/CentOS)
```bash
firewall-cmd --state                   # Trạng thái
firewall-cmd --list-all                # Liệt kê rule
firewall-cmd --add-port=80/tcp --permanent   # Mở port (lưu vĩnh viễn)
firewall-cmd --reload                  # Áp dụng thay đổi
```

---

## ⚡ Performance Profiling & Debug sâu

### Theo dõi tài nguyên chi tiết
```bash
iostat -x 1                            # Thống kê I/O đĩa mỗi giây (tìm bottleneck disk)
mpstat -P ALL 1                        # CPU từng core mỗi giây
sar -u 1 5                             # Lịch sử CPU (5 mẫu, mỗi giây)
sar -r 1 5                             # Lịch sử RAM
pidstat 1                              # Thống kê tài nguyên theo process
iotop                                  # Process nào đọc/ghi disk nhiều (cần root)
nethogs                                # Process nào dùng nhiều băng thông mạng
dstat                                  # Tổng hợp CPU/disk/net/memory 1 màn hình
```

### Debug process ở mức system call
```bash
strace -p <PID>                        # Theo dõi syscall của process đang chạy
strace -f -e trace=network <cmd>       # Chỉ trace syscall mạng
strace -c <cmd>                        # Thống kê syscall (tìm cái gọi nhiều)
ltrace <cmd>                           # Theo dõi lời gọi thư viện
lsof -p <PID>                          # File/socket mà process đang mở
cat /proc/<PID>/status                 # Trạng thái chi tiết process
cat /proc/<PID>/limits                 # Giới hạn (ulimit) của process
ulimit -a                              # Xem giới hạn hiện tại (file descriptor...)
perf top                               # Hàm nào ngốn CPU nhất (realtime)
perf record -p <PID> / perf report     # Ghi & phân tích profile CPU
```

### Kiểm tra kết nối & OOM
```bash
ss -s                                  # Tổng hợp số kết nối theo trạng thái
ss -tan state established | wc -l      # Đếm kết nối ESTABLISHED
ss -tan state time-wait | wc -l        # Đếm TIME_WAIT (nhiều = vấn đề)
dmesg -T | grep -i "killed process"    # Kiểm tra process bị OOM killer giết
grep -i oom /var/log/syslog            # Log OOM (Ubuntu)
cat /proc/loadavg                      # Load average
```

---

## 🚀 ArgoCD & GitOps

### argocd CLI
```bash
argocd login <server>                  # Đăng nhập ArgoCD server
argocd login <server> --sso            # Đăng nhập qua SSO
argocd account get-user-info           # Xem user hiện tại

# Ứng dụng (Application)
argocd app list                        # Liệt kê tất cả app
argocd app get <app>                   # Chi tiết app (trạng thái sync/health)
argocd app create <app> \
  --repo <git-url> --path <path> \
  --dest-server https://kubernetes.default.svc --dest-namespace <ns>   # Tạo app
argocd app sync <app>                  # Đồng bộ app (deploy theo git)
argocd app sync <app> --prune          # Sync + xóa resource thừa
argocd app diff <app>                  # Xem khác biệt giữa git và cluster
argocd app history <app>               # Lịch sử deploy
argocd app rollback <app> <revision>   # Rollback về revision cũ
argocd app set <app> --sync-policy automated   # Bật auto-sync
argocd app delete <app>                # Xóa app
argocd app wait <app> --health         # Chờ đến khi app healthy (dùng trong CI)
argocd app logs <app>                  # Xem log của app

# Trạng thái hay gặp (troubleshoot):
# Synced        -> git khớp với cluster
# OutOfSync     -> git khác cluster (cần sync)
# Healthy       -> resource chạy tốt
# Degraded      -> resource lỗi (xem app get / describe pod)
# Progressing   -> đang rollout
# Missing       -> resource chưa được tạo
```

### GitOps qua kubectl (khi cài ArgoCD dạng CRD)
```bash
kubectl get applications -n argocd     # Liệt kê ArgoCD app
kubectl get app <app> -n argocd -o yaml   # Xem manifest app
kubectl describe app <app> -n argocd   # Xem điều kiện & lỗi sync
kubectl -n argocd get pods             # Kiểm tra ArgoCD component
argocd admin initial-password -n argocd    # Lấy mật khẩu admin ban đầu

# FluxCD (GitOps thay thế ArgoCD)
flux get kustomizations                # Liệt kê kustomization
flux reconcile kustomization <name>    # Ép đồng bộ ngay
flux get sources git                   # Xem git source
flux logs                              # Xem log controller
```

---

## 🔧 CI/CD (GitHub Actions / GitLab CI)

### GitHub Actions (gh CLI)
```bash
gh auth login                          # Đăng nhập
gh run list                            # Liệt kê các lần chạy workflow
gh run view <run-id>                   # Chi tiết 1 run
gh run view <run-id> --log             # Xem log
gh run view <run-id> --log-failed      # Chỉ log các job fail (debug nhanh)
gh run watch <run-id>                  # Theo dõi run realtime
gh run rerun <run-id>                  # Chạy lại
gh run rerun <run-id> --failed         # Chỉ chạy lại job fail
gh run cancel <run-id>                 # Hủy run
gh workflow list                       # Liệt kê workflow
gh workflow run <workflow>             # Trigger workflow thủ công
gh pr checks                           # Xem trạng thái CI của PR hiện tại
# act                                  # Chạy GitHub Actions ở local (công cụ nomad/act)
```

### GitLab CI
```bash
# Kiểm tra cú pháp .gitlab-ci.yml tại: GitLab > CI/CD > Editor > Validate
gitlab-runner verify                   # Kiểm tra runner
gitlab-runner list                     # Liệt kê runner đã đăng ký
gitlab-runner exec docker <job>        # Chạy 1 job ở local để test
glab ci list                           # Liệt kê pipeline (glab CLI)
glab ci view                           # Xem pipeline hiện tại
glab ci trace                          # Xem log job realtime
glab ci retry <id>                     # Chạy lại pipeline
```

---

## 🔑 Secrets (Vault / kubeseal / SOPS)

### HashiCorp Vault
```bash
export VAULT_ADDR='https://vault:8200' # Trỏ tới server
vault login                            # Đăng nhập
vault status                           # Trạng thái (sealed/unsealed)
vault kv get secret/myapp              # Đọc secret
vault kv get -field=password secret/myapp   # Lấy đúng 1 field
vault kv put secret/myapp password=abc123   # Ghi secret
vault kv list secret/                  # Liệt kê secret
vault kv delete secret/myapp           # Xóa
vault operator unseal                  # Unseal vault (khi bị sealed)
vault token lookup                     # Xem thông tin token hiện tại
```

### Kubernetes Secrets & mã hóa (GitOps-friendly)
```bash
# Secret thường
kubectl create secret generic <name> --from-literal=key=value
kubectl get secret <name> -o jsonpath='{.data.key}' | base64 -d   # Đọc giá trị

# Sealed Secrets (an toàn để commit vào git)
kubeseal --format yaml < secret.yaml > sealed-secret.yaml   # Mã hóa secret
kubectl apply -f sealed-secret.yaml    # Controller sẽ tự giải mã trong cluster

# SOPS (mã hóa file YAML/ENV để lưu git)
sops -e secrets.yaml > secrets.enc.yaml   # Mã hóa
sops -d secrets.enc.yaml               # Giải mã
sops secrets.enc.yaml                   # Mở editor sửa file đã mã hóa
```

---

## ⚙️ systemd nâng cao

```bash
systemctl list-units --type=service              # Liệt kê service đang chạy
systemctl list-units --state=failed              # Service nào đang fail (troubleshoot)
systemctl status <service>                       # Trạng thái + vài dòng log
systemctl is-active <service>                    # Đang chạy? (active/inactive)
systemctl is-enabled <service>                   # Có tự chạy khi boot?
systemctl daemon-reload                          # Nạp lại khi sửa file unit
systemctl cat <service>                          # Xem nội dung file unit
systemctl show <service>                         # Xem tất cả property
systemctl list-dependencies <service>            # Xem phụ thuộc
journalctl -u <service> --since today            # Log service hôm nay
journalctl -u <service> -p err                   # Chỉ log lỗi
journalctl --disk-usage                          # Log đang chiếm bao nhiêu disk
journalctl --vacuum-time=7d                      # Dọn log cũ hơn 7 ngày
systemd-analyze blame                             # Thời gian boot của từng service
systemctl list-timers --all                      # Xem timer (thay cron)
```

---

## 📦 Container runtime khác (podman / crictl / nerdctl)

### Podman (thay Docker, không cần daemon, chạy rootless)
```bash
podman ps / podman ps -a                # Liệt kê container (cú pháp giống docker)
podman run -d --name app <image>        # Chạy container
podman images / podman pull <image>     # Image
podman logs -f <container>              # Xem log
podman exec -it <container> bash        # Vào container
podman build -t app:1.0 .               # Build image
podman pod create --name mypod          # Tạo pod (nhóm container, giống k8s pod)
podman generate kube <container>        # Xuất ra manifest kubernetes
podman play kube pod.yaml               # Chạy từ manifest k8s
alias docker=podman                     # Dùng podman như docker
```

### crictl (debug container trực tiếp trên node k8s - không qua kubectl)
```bash
crictl ps                              # Liệt kê container đang chạy
crictl ps -a                           # Cả đã dừng
crictl images                          # Liệt kê image
crictl pods                            # Liệt kê pod (sandbox)
crictl logs <container-id>             # Xem log container
crictl exec -it <container-id> sh      # Vào container
crictl inspect <container-id>          # Chi tiết container
crictl stats                           # CPU/RAM container
# Rất hữu ích khi node lỗi mà kubectl/API server không truy cập được
```

### nerdctl (CLI giống Docker cho containerd)
```bash
nerdctl ps                             # Liệt kê container
nerdctl run -d <image>                 # Chạy
nerdctl build -t app .                 # Build
nerdctl compose up -d                  # Hỗ trợ cả compose
nerdctl -n k8s.io ps                   # Xem container của k8s (namespace k8s.io)
```

---

## 🕸️ Service Mesh (Istio / Linkerd)

### Istio (istioctl)
```bash
istioctl version                       # Phiên bản
istioctl install --set profile=demo    # Cài đặt
istioctl analyze                       # Phân tích lỗi cấu hình (rất hay)
istioctl analyze -n <namespace>        # Trong 1 namespace
kubectl label namespace <ns> istio-injection=enabled   # Bật sidecar injection
istioctl proxy-status                  # Trạng thái đồng bộ của các proxy (Envoy)
istioctl proxy-config routes <pod>     # Xem cấu hình route của sidecar
istioctl proxy-config cluster <pod>    # Xem cluster (upstream) của sidecar
istioctl dashboard kiali               # Mở dashboard Kiali (xem traffic)
istioctl dashboard grafana             # Mở Grafana
kubectl get virtualservice,destinationrule,gateway -A   # Xem resource Istio
```

### Linkerd
```bash
linkerd check                          # Kiểm tra sức khỏe cài đặt (chạy trước tiên)
linkerd install | kubectl apply -f -   # Cài đặt
linkerd viz install | kubectl apply -f -   # Cài dashboard
linkerd viz dashboard                  # Mở dashboard
linkerd viz stat deploy -n <ns>        # Thống kê success rate, RPS, latency
linkerd viz tap deploy/<name>          # Xem request realtime (live tap)
linkerd viz top deploy/<name>          # Route nào nhiều traffic nhất
```

---

## 🐶 Công cụ TUI cho Kubernetes (k9s / stern / kubectx)

```bash
k9s                                    # TUI quản lý cluster (điều hướng bằng phím)
k9s -n <namespace>                     # Mở thẳng namespace
# Trong k9s: :pods, :svc, :deploy để chuyển view; / để lọc; d=describe; l=logs; s=shell

stern <pod-query>                      # Xem log NHIỀU pod cùng lúc (theo tên/label)
stern <app> -n <ns>                    # Trong namespace
stern -l app=nginx                     # Theo label
stern <app> --since 10m                # Log 10 phút gần đây (nhiều pod)
# stern tiện hơn "kubectl logs" khi 1 service có nhiều replica

kubectx                                # Liệt kê & chuyển context nhanh
kubectx <context>                      # Chuyển context
kubens                                 # Liệt kê & chuyển namespace nhanh
kubens <namespace>                     # Chuyển namespace mặc định

kubectl krew install <plugin>          # Cài plugin kubectl (krew = package manager)
kubectl krew list                      # Liệt kê plugin đã cài
```

---

## ☸️ Kubernetes vận hành nâng cao

### Bảo trì node (drain / cordon / taint)
```bash
kubectl cordon <node>                  # Đánh dấu node không nhận pod mới
kubectl uncordon <node>                # Cho phép nhận pod lại
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data   # Đẩy pod ra khỏi node (trước khi bảo trì)
kubectl taint node <node> key=value:NoSchedule   # Cấm schedule (trừ pod có toleration)
kubectl taint node <node> key:NoSchedule-        # Gỡ taint (thêm dấu - ở cuối)
kubectl get node <node> -o jsonpath='{.spec.taints}'   # Xem taint của node
```

### Autoscaling & Resource
```bash
kubectl autoscale deploy <name> --min=2 --max=10 --cpu-percent=80   # Tạo HPA
kubectl get hpa                        # Xem trạng thái HPA (current/target)
kubectl describe hpa <name>            # Chi tiết (tại sao scale/không scale)
kubectl get resourcequota -n <ns>      # Giới hạn tài nguyên namespace
kubectl describe limitrange -n <ns>    # Giới hạn mặc định cho pod
kubectl get pods -o custom-columns='NAME:.metadata.name,CPU:.spec.containers[*].resources.requests.cpu,MEM:.spec.containers[*].resources.requests.memory'   # Xem request tài nguyên
```

### Troubleshoot pod không chạy được
```bash
kubectl get pod <pod> -o wide          # Xem pod ở node nào, IP gì
kubectl describe pod <pod>             # Xem Events (lý do Pending/CrashLoop)
kubectl get events --field-selector involvedObject.name=<pod>   # Events của riêng pod
# Nguyên nhân hay gặp:
# Pending           -> thiếu tài nguyên / không có node phù hợp / PVC chưa bound
# ImagePullBackOff  -> sai tên image / thiếu imagePullSecret
# CrashLoopBackOff  -> app crash liên tục -> xem logs --previous
# OOMKilled         -> vượt memory limit -> tăng limit hoặc tối ưu app
# Evicted           -> node hết tài nguyên (disk/memory pressure)

kubectl get pvc                        # Kiểm tra Persistent Volume Claim
kubectl get pv                         # Persistent Volume
kubectl rollout status deploy <name>   # Xem rollout có kẹt không
kubectl get pods --field-selector status.phase=Running   # Lọc theo trạng thái
```

### Bảo mật & phân quyền (RBAC)
```bash
kubectl auth can-i create pods         # Kiểm tra quyền của mình
kubectl auth can-i '*' '*' --as <user> # Kiểm tra quyền của user khác
kubectl get roles,rolebindings -n <ns> # Xem role trong namespace
kubectl get clusterroles               # Role toàn cluster
kubectl get serviceaccount -n <ns>     # Service account
```

---

## 🔥 Load Testing & Benchmark

```bash
# ab (Apache Bench) - đơn giản, nhanh
ab -n 1000 -c 50 http://localhost:8080/   # 1000 request, 50 đồng thời

# hey - hiện đại hơn ab
hey -n 1000 -c 50 http://localhost:8080/
hey -z 30s -c 20 http://localhost:8080/   # Chạy trong 30 giây

# wrk - hiệu năng cao
wrk -t4 -c100 -d30s http://localhost:8080/   # 4 thread, 100 kết nối, 30 giây
wrk -t4 -c100 -d30s --latency http://localhost:8080/   # Kèm phân phối latency

# k6 - script bằng JavaScript, mạnh nhất
k6 run script.js                       # Chạy test theo script
k6 run --vus 50 --duration 30s script.js   # 50 virtual user trong 30s

# siege - test bền
siege -c 50 -t 1M http://localhost:8080/   # 50 client trong 1 phút

# Test nhanh bằng curl (đo thời gian)
curl -w "@curl-format.txt" -o /dev/null -s http://localhost:8080/
# hoặc vòng lặp đơn giản:
for i in $(seq 1 100); do curl -s -o /dev/null -w "%{http_code} %{time_total}s\n" http://localhost:8080/; done
```

---

## 🖥️ tmux & screen (giữ session)

> Chạy lệnh dài (deploy, backup, migration) qua SSH mà không sợ mất khi rớt mạng.

### tmux (khuyên dùng)
```bash
tmux                                   # Tạo session mới
tmux new -s <name>                     # Tạo session có tên
tmux ls                                # Liệt kê session
tmux attach -t <name>                  # Gắn lại vào session
tmux kill-session -t <name>            # Xóa session
# Phím tắt (prefix mặc định = Ctrl+b, bấm rồi thả, rồi bấm phím sau):
#   Ctrl+b d   -> detach (thoát nhưng session vẫn chạy nền)
#   Ctrl+b c   -> tạo cửa sổ mới
#   Ctrl+b n/p -> chuyển cửa sổ next/prev
#   Ctrl+b %   -> chia dọc  |  Ctrl+b "  -> chia ngang
#   Ctrl+b <mũi tên>  -> chuyển giữa các pane
#   Ctrl+b [   -> chế độ cuộn (q để thoát)
```

### screen (có sẵn trên nhiều server)
```bash
screen                                 # Tạo session
screen -S <name>                       # Tạo session có tên
screen -ls                             # Liệt kê session
screen -r <name>                       # Gắn lại
# Ctrl+a d  -> detach   |   Ctrl+a c -> cửa sổ mới   |   Ctrl+a n -> cửa sổ tiếp

# Hoặc dùng nohup cho lệnh chạy nền không cần terminal:
nohup ./long-task.sh > out.log 2>&1 &  # Chạy nền, ghi log ra file
disown                                 # Gỡ khỏi shell hiện tại
```

---

## 🌩️ Network debug sâu (tcpdump / mtr / tshark)

### tcpdump - bắt gói tin (cần root)
```bash
tcpdump -i any                         # Bắt trên mọi interface
tcpdump -i eth0 port 80                 # Chỉ port 80
tcpdump -i any host 1.2.3.4             # Chỉ traffic tới/từ 1 IP
tcpdump -i any 'port 80 and host 1.2.3.4'   # Kết hợp điều kiện
tcpdump -i any -w capture.pcap          # Ghi ra file (mở bằng Wireshark)
tcpdump -r capture.pcap                 # Đọc lại file
tcpdump -i any -A port 80               # Hiện nội dung dạng ASCII (xem HTTP)
tcpdump -i any -nn port 443             # Không resolve DNS/port (nhanh)
tcpdump -i any 'tcp[tcpflags] & tcp-syn != 0'   # Chỉ gói SYN (debug bắt tay)
tcpdump -c 100 -i any port 53           # Bắt 100 gói DNS rồi dừng
```

### mtr - traceroute + ping realtime (tìm điểm mất gói)
```bash
mtr <host>                             # Xem đường đi + % mất gói realtime
mtr -r -c 100 <host>                    # Báo cáo 100 lần ping (dừng sau khi xong)
mtr --tcp --port 443 <host>            # Dùng TCP thay ICMP (khi ICMP bị chặn)
```

### Khác
```bash
tshark -i any -f "port 80"             # Wireshark bản CLI
iperf3 -s                              # Chạy server đo băng thông
iperf3 -c <server-ip>                  # Client đo băng thông tới server
arp -a                                 # Bảng ARP (IP <-> MAC)
ip route                               # Bảng định tuyến
ip route get 8.8.8.8                   # Xem gói đi ra interface nào
conntrack -L                           # Bảng theo dõi kết nối NAT (cần root)
ethtool eth0                           # Thông tin & tốc độ card mạng
```

---

## 💾 Backup & Disaster Recovery (Velero / etcd)

### Velero - backup/restore toàn bộ cluster Kubernetes
```bash
velero backup create <name>                        # Backup toàn cluster
velero backup create <name> --include-namespaces <ns>   # Chỉ 1 namespace
velero backup create <name> --selector app=myapp   # Theo label
velero backup get                                  # Liệt kê backup
velero backup describe <name>                      # Chi tiết backup
velero backup logs <name>                          # Log backup
velero restore create --from-backup <name>         # Khôi phục từ backup
velero restore get                                 # Liệt kê restore
velero schedule create daily --schedule="0 2 * * *"   # Lịch backup tự động 2h sáng
velero backup-location get                          # Nơi lưu backup (S3/GCS...)
```

### etcd - backup dữ liệu control plane (quan trọng nhất của K8s)
```bash
# Snapshot (chạy trên control plane node)
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

ETCDCTL_API=3 etcdctl snapshot status snapshot.db  # Kiểm tra snapshot
ETCDCTL_API=3 etcdctl endpoint health              # Kiểm tra sức khỏe etcd
ETCDCTL_API=3 etcdctl member list                  # Liệt kê member cluster
# Restore: etcdctl snapshot restore snapshot.db --data-dir /var/lib/etcd-restore
```

### Backup database & file (cron-friendly)
```bash
pg_dump -U user db | gzip > backup_$(date +%F).sql.gz   # Backup Postgres kèm ngày
mysqldump -u user -p db | gzip > backup_$(date +%F).sql.gz   # Backup MySQL
tar -czf backup_$(date +%F).tar.gz /data           # Backup thư mục kèm ngày
aws s3 cp backup.sql.gz s3://<bucket>/backups/      # Đẩy backup lên S3
find /backups -mtime +30 -delete                    # Xóa backup cũ > 30 ngày
```

---

## 📜 cert-manager (TLS tự động trên K8s)

```bash
kubectl get certificate -A             # Liệt kê certificate
kubectl get certificate <name> -n <ns> # Trạng thái cert (READY = True là ổn)
kubectl describe certificate <name>    # Chi tiết + lý do lỗi cấp phát
kubectl get certificaterequest -A      # Yêu cầu cấp cert (troubleshoot)
kubectl get order,challenge -A         # ACME order/challenge (Let's Encrypt)
kubectl describe challenge <name>      # Xem tại sao challenge fail (DNS/HTTP-01)
kubectl get clusterissuer             # Nơi cấp cert (Let's Encrypt...)
kubectl describe clusterissuer <name>  # Trạng thái issuer
cmctl status certificate <name>        # Kiểm tra chi tiết bằng cmctl CLI
cmctl renew <name>                     # Ép gia hạn cert ngay

# Trạng thái hay gặp:
# READY=True         -> cert đã cấp thành công
# READY=False        -> xem describe certificate -> certificaterequest -> challenge
# Challenge pending  -> thường do DNS/HTTP-01 chưa verify được (kiểm tra ingress/DNS)
```

---

## 🎛️ Vận hành & Backup Cluster K8s (manifest / data / DR)

> Khi vận hành cluster production, có **3 thứ phải backup**: (1) **etcd** — trạng thái cluster,
> (2) **Manifest/YAML** — cấu hình resource (nên để trong Git = GitOps), (3) **Dữ liệu Persistent Volume**.

### 1. Backup Manifest / cấu hình resource (YAML)

```bash
# Xuất toàn bộ resource của 1 namespace ra file YAML
kubectl get all -n <ns> -o yaml > backup-<ns>.yaml

# Nhưng "get all" KHÔNG bao gồm hết. Backup đầy đủ hơn:
for res in deploy sts ds svc cm secret ingress pvc hpa; do
  kubectl get $res -n <ns> -o yaml > backup-<ns>-$res.yaml
done

# Backup TẤT CẢ resource của mọi namespace (dùng khi migrate/DR)
for ns in $(kubectl get ns -o jsonpath='{.items[*].metadata.name}'); do
  kubectl get all,cm,secret,ingress,pvc -n $ns -o yaml > backup-$ns.yaml
done

# Backup resource cấp cluster (không thuộc namespace)
kubectl get pv,clusterrole,clusterrolebinding,storageclass,crd -o yaml > backup-cluster.yaml

# Xuất sạch để commit vào Git (bỏ field runtime: status, uid, resourceVersion...)
kubectl get deploy <name> -o yaml \
  | kubectl neat > clean.yaml          # cần plugin "kubectl neat" (qua krew)
```

**Khuyến nghị (best practice):** Đừng backup YAML thủ công — hãy để **toàn bộ manifest trong Git**
và deploy qua **ArgoCD/Flux (GitOps)**. Khi đó Git chính là bản backup manifest, dựng lại cluster
chỉ cần trỏ ArgoCD vào repo là xong.

```bash
# Công cụ backup manifest tự động (kèm cả PV) — khuyên dùng cho production:
velero backup create full-$(date +%F) --include-cluster-resources=true   # Backup cả manifest + volume
velero backup create ns-backup --include-namespaces prod --snapshot-volumes   # Kèm snapshot PV
```

### 2. Backup Dữ liệu (Persistent Volume)

```bash
# Cách A: Velero + snapshot volume (tích hợp cloud provider - khuyên dùng)
velero backup create data-$(date +%F) --snapshot-volumes --include-namespaces prod
velero restore create --from-backup data-2026-07-03    # Khôi phục cả PV

# Cách B: Backup thủ công dữ liệu trong pod (DB) ra ngoài
kubectl exec -n prod <postgres-pod> -- pg_dump -U user db | gzip > db-$(date +%F).sql.gz
kubectl exec -n prod <mysql-pod> -- mysqldump -u root -p$PW db | gzip > db-$(date +%F).sql.gz

# Cách C: Copy dữ liệu từ PV ra ngoài
kubectl cp prod/<pod>:/data ./pv-backup            # Copy thư mục data ra local
# Rồi đẩy lên object storage:
aws s3 cp db-$(date +%F).sql.gz s3://<bucket>/k8s-backups/

# Snapshot PVC bằng VolumeSnapshot (CSI driver hỗ trợ)
kubectl get volumesnapshot -n prod                 # Liệt kê snapshot
# (tạo VolumeSnapshot bằng manifest có spec.source.persistentVolumeClaimName)
```

### 3. Backup etcd (trạng thái cluster - QUAN TRỌNG NHẤT)

```bash
# Chạy trên control-plane node. Nên đặt vào cron chạy định kỳ.
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%F-%H%M).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-xxx.db --write-out=table   # Kiểm tra
aws s3 cp /backup/etcd-*.db s3://<bucket>/etcd/      # Đẩy lên storage ngoài cluster!
find /backup -name 'etcd-*.db' -mtime +14 -delete   # Giữ 14 ngày
```

### 4. Vận hành thường ngày (Day-2 Operations)

```bash
# --- Nâng cấp node an toàn (rolling) ---
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data   # 1. Đẩy pod ra
# ... nâng cấp / vá lỗi node ...
kubectl uncordon <node>                            # 2. Cho node nhận pod lại

# --- Kiểm tra sức khỏe cluster ---
kubectl get nodes                                  # Node Ready hết chưa
kubectl get pods -A | grep -vE 'Running|Completed' # Pod nào KHÔNG khỏe
kubectl top nodes && kubectl top pods -A           # Tài nguyên
kubectl get componentstatuses                      # Trạng thái control plane
kubectl get events -A --sort-by=.lastTimestamp | tail -30   # Sự kiện gần nhất

# --- etcd bảo trì ---
ETCDCTL_API=3 etcdctl endpoint health              # etcd còn khỏe không
ETCDCTL_API=3 etcdctl endpoint status --write-out=table   # Kích thước DB, leader
ETCDCTL_API=3 etcdctl defrag                       # Nén DB khi phình to (giảm dung lượng)

# --- Chứng chỉ (cert control plane hết hạn là cluster chết) ---
kubeadm certs check-expiration                     # Kiểm tra hạn cert
kubeadm certs renew all                            # Gia hạn tất cả

# --- Dọn dẹp ---
kubectl delete pod --field-selector status.phase=Failed -A    # Xóa pod Failed
kubectl delete pod --field-selector status.phase=Succeeded -A # Xóa pod đã xong
```

### 5. Quy trình khôi phục thảm họa (DR Runbook)

```text
Khi mất cluster / mất control plane:
  1. Dựng lại control-plane node (kubeadm init hoặc theo IaC)
  2. Khôi phục etcd từ snapshot:
     etcdctl snapshot restore /backup/etcd-xxx.db --data-dir /var/lib/etcd-new
     (trỏ static pod etcd vào data-dir mới, restart kubelet)
  3. Kiểm tra: kubectl get nodes && kubectl get pods -A
  4. Nếu KHÔNG có etcd backup nhưng có manifest trong Git:
     -> dựng cluster mới, để ArgoCD/Flux sync lại toàn bộ từ Git
  5. Khôi phục dữ liệu PV: velero restore, hoặc restore DB dump
  6. Verify: chạy smoke test, kiểm tra ingress/cert/DNS

Nguyên tắc vàng:
  - Backup phải để NGOÀI cluster (S3/GCS), không để trong chính cluster
  - Test restore định kỳ — backup không test = không có backup
  - 3-2-1: 3 bản sao, 2 loại lưu trữ, 1 bản offsite
```







