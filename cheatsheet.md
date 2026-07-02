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
