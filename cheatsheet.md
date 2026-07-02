# Cheatsheet: Docker, Docker Compose, Helm, kubectl

Tổng hợp các câu lệnh hay dùng. Ghi chú bằng tiếng Việt để dễ tra cứu.

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
