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

<details>
<summary><b>Bấm xem: giải nghĩa các cờ mục Container</b></summary>

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-a` | **a**ll | Hiện **TẤT CẢ** container, kể cả đã dừng. Không có `-a` thì `docker ps` **chỉ** hiện cái đang chạy — đây là lý do hay tưởng "container biến mất" |
| `-d` | **d**etached | Chạy **nền**. Trả prompt lại ngay, container chạy tiếp phía sau. Không có `-d` thì terminal bị "dính" vào container, Ctrl+C là container chết |
| `-i` | **i**nteractive | Giữ **STDIN mở** — bạn gõ được vào container |
| `-t` | **t**ty | Cấp một **terminal giả** (pseudo-TTY) để có prompt, màu, xoá dòng. `-it` luôn đi cặp: `-i` để gõ vào, `-t` để nhìn cho ra hình terminal |
| `--name` | name | Đặt **tên cố định**. Không đặt thì Docker tự sinh tên ngẫu nhiên kiểu `naughty_tesla`, mỗi lần tạo lại một tên khác nhau ⇒ script không gọi được |
| `-p` | **p**ublish | **Mở port ra host.** Cú pháp `host:container`. `-p 8080:80` = gõ `localhost:8080` ở máy thật thì vào port 80 trong container |
| `-v` | **v**olume | **Gắn thư mục** vào container, cú pháp `host:container`. Data nằm ở host ⇒ xoá container **không mất** data |
| `--env-file` | environment file | Nạp **cả file** biến môi trường (mỗi dòng `KEY=value`) |
| `-e` | **e**nvironment | Set **1 biến** môi trường |
| `--rm` | **r**e**m**ove | **Tự xoá xác container** khi nó dừng. Không có `--rm` thì container dừng vẫn nằm đó chiếm disk, phải `docker rm` tay |
| `-f` (trong `rm -f`) | **f**orce | Xoá **cưỡng bức**, container đang chạy cũng xoá. Không có `-f` thì Docker từ chối xoá container đang chạy |
| `-q` (trong `ps -aq`) | **q**uiet | **Chỉ in ID**, không in bảng. Sinh ra để **đưa vào lệnh khác** |

**Vì sao `docker rm -f $(docker ps -aq)` xoá được TẤT CẢ?** — đây là *command substitution* của shell:

```bash
docker rm -f $(docker ps -aq)
#         │   └──────────────── shell CHẠY TRƯỚC lệnh trong $( ),
#         │                     `ps -aq` trả về danh sách ID: "a1b2 c3d4 e5f6"
#         │                     rồi DÁN chuỗi đó vào chỗ này
#         └──────────────────── nên thành: docker rm -f a1b2 c3d4 e5f6
```

🛑 **Cảnh báo**: `$(docker ps -aq)` lấy **mọi** container trên máy, không phân biệt dự án. Trên máy dùng chung là xoá nhầm của người khác.

Bóc từng mảnh một lệnh đầy đủ:

```bash
docker run -d --name myapp -p 8080:80 -v $(pwd):/app --rm nginx
#          │  │            │          │              │    │
#          │  │            │          │              │    └─ image dùng để tạo container
#          │  │            │          │              └────── dừng là tự xoá xác
#          │  │            │          └───────────────────── mount thư mục hiện tại vào /app
#          │  │            │                                 ($(pwd) = shell thay bằng đường dẫn đang đứng)
#          │  │            └──────────────────────────────── port 8080 host -> 80 container
#          │  └───────────────────────────────────────────── đặt tên cố định "myapp"
#          └──────────────────────────────────────────────── chạy nền, trả prompt ngay
```

**`start` / `restart` / `pause` khác nhau chỗ nào?** (rất hay nhầm)

| Lệnh | Container phải đang | Làm gì | Tiến trình bên trong |
|---|---|---|---|
| `docker start` | **đã dừng** | Bật lại | Chạy **lại từ đầu** |
| `docker restart` | đang chạy / đã dừng | stop rồi start | Chạy **lại từ đầu** |
| `docker pause` | **đang chạy** | Đóng băng | **Giữ nguyên**, đứng im tại chỗ (SIGSTOP), RAM còn nguyên |

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa các cờ mục Exec & Logs</b></summary>

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-it` | interactive + tty | Cặp đôi để có **shell gõ được**. Thiếu `-i` thì gõ không vào; thiếu `-t` thì không có prompt, nhìn như treo |
| `-f` (trong `logs -f`) | **f**ollow | **Bám** theo log, dòng mới hiện ngay realtime. Ctrl+C để thoát. Không có `-f` thì in xong log cũ là thoát luôn |
| `--tail N` | tail = đuôi | Chỉ lấy **N dòng cuối**. Không có thì in **từ đầu đời** container — log vài GB là ngập terminal |
| `--since` | since = từ lúc | Chỉ log **từ mốc thời gian**. Nhận `10m`, `1h`, `2026-08-06T10:00:00` |

**`exec` vs `attach` — khác nhau căn bản** (đây là chỗ hay gõ nhầm rồi làm sập service):

| | `docker exec -it <c> bash` | `docker attach <c>` |
|---|---|---|
| Tạo ra cái gì | Một tiến trình **MỚI** (bash) bên cạnh app | **KHÔNG** tạo gì, nối vào đúng tiến trình chính đang chạy |
| Ctrl+C sẽ | Chỉ thoát bash. **App vẫn chạy** | **Gửi tín hiệu dừng cho app ⇒ container CHẾT** |
| Dùng khi | Vào ngó nghiêng, debug (99% trường hợp) | Muốn xem output trực tiếp của app |

⇒ **Gần như luôn dùng `exec`.** `attach` chỉ dùng khi biết rõ mình làm gì.

**`bash` vs `sh`** — `bash` là shell "đầy đủ". Image gọn (Alpine, distroless, nhiều image `-slim`) **không cài bash** để tiết kiệm dung lượng. Gặp `exec: "bash": executable file not found` ⇒ **không phải container hỏng**, chỉ là image không có bash ⇒ đổi sang `sh` (hầu như luôn có).

Bóc từng mảnh:

```bash
docker logs -f --tail 100 --since 10m myapp
#           │  │          │          └─ tên/ID container
#           │  │          └──────────── chỉ log 10 phút gần đây
#           │  └─────────────────────── bắt đầu từ 100 dòng cuối (không phải từ đầu đời)
#           └────────────────────────── rồi BÁM tiếp, dòng mới hiện ngay
```

**`docker cp` — chiều nào là vào, chiều nào là ra?** Quy tắc: **`<nguồn> <đích>`**, cái nào có dấu `:` là phía container.

```bash
docker cp myapp:/var/log/app.log ./local.log   # container ---> host  (LẤY RA)
#         └──── có dấu ':' = nguồn là container

docker cp ./config.yaml myapp:/etc/config.yaml # host ---> container  (ĐẨY VÀO)
#                       └──── có dấu ':' = đích là container
```

**Các lệnh chỉ đọc, an toàn chạy trên production:**

| Lệnh | Trả về gì | Dùng để |
|---|---|---|
| `docker top <c>` | Bảng process **bên trong** container | Xem app có thực sự chạy không, hay đã chết mà container vẫn sống |
| `docker stats` | CPU/RAM realtime, tự refresh | Bắt container ăn RAM. Ctrl+C thoát |
| `docker inspect <c>` | **JSON đầy đủ**: mount, env, IP, restart policy | Tra cấu hình thật container đang chạy (khác với file compose đã sửa sau đó) |
| `docker port <c>` | Chỉ bảng mapping port | Nhanh hơn đọc cả JSON của inspect |
| `docker diff <c>` | Filesystem đã đổi gì so với image | Phát hiện app ghi file lung tung vào container (mất khi restart!) |

`docker diff` in ra 3 ký tự đầu dòng: `A` = **A**dded (thêm mới), `C` = **C**hanged (sửa), `D` = **D**eleted (xoá).

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa các cờ mục Images</b></summary>

**Tiền đề — image vs container** (phải rõ cái này thì phần dưới mới có nghĩa):

| | Image | Container |
|---|---|---|
| Là gì | **Khuôn bánh** — chỉ đọc, bất biến | **Cái bánh** đúc ra từ khuôn — chạy được, sửa được |
| Lệnh xoá | `docker rmi` (**r**e**m**ove **i**mage) | `docker rm` (**r**e**m**ove) |
| Số lượng | 1 image | đúc ra được **N** container |

⇒ Đây là lý do có **hai** lệnh xoá gần giống nhau: `rm` (container) và `rm**i**` (image). Xoá image mà container còn dùng thì Docker từ chối.

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-t` | **t**ag | **Đặt tên:tag** cho image vừa build. Không có `-t` thì image không tên (`<none>:<none>`), chỉ gọi được bằng ID |
| `-f` (trong `build -f`) | **f**ile | Chỉ định **Dockerfile nào**. Mặc định tìm file tên đúng `Dockerfile` trong thư mục build |
| `--no-cache` | không cache | **Bỏ qua layer đã cache**, build lại từ đầu |
| `-o` (trong `save -o`) | **o**utput | Ghi ra **file** |
| `-i` (trong `load -i`) | **i**nput | Đọc **từ file** |
| `-a` (trong `prune -a`) | **a**ll | Xoá **cả** image có tag nhưng không container nào dùng |
| `-q` (trong `images -q`) | **q**uiet | Chỉ in ID, để đưa vào lệnh khác |

⚠️ **Dấu `.` cuối lệnh build là gì?** Đây là chỗ gây bối rối nhất — nó **KHÔNG** phải "thư mục hiện tại" theo nghĩa thông thường, mà là **build context**: toàn bộ thư mục này được **gửi cho Docker daemon**. Nên `COPY` trong Dockerfile chỉ copy được file **nằm trong context**. Để `.` ở thư mục có `node_modules` 2GB ⇒ build chậm khủng khiếp ⇒ dùng `.dockerignore` để loại trừ.

```bash
docker build -t myapp:1.0 -f Dockerfile.dev .
#            │            │                └─ BUILD CONTEXT: gửi cả thư mục này cho daemon
#            │            └────────────────── dùng Dockerfile.dev thay vì Dockerfile
#            └─────────────────────────────── đặt tên image = myapp, tag = 1.0
```

**Vì sao `--no-cache` tồn tại?** Docker cache **theo từng dòng** trong Dockerfile: dòng nào lệnh không đổi thì tái dùng kết quả cũ. Nhưng `RUN apt-get update` hay `RUN git clone` **giống hệt về mặt chữ** mà **nội dung ngoài đời đã đổi** ⇒ Docker vẫn coi là "chưa đổi" và dùng bản cũ ⇒ build ra image cũ mà không báo gì. `--no-cache` xử lý đúng ca này.

**`tag` — hiểu cho đúng, không phải "đổi tên":**

```bash
docker tag myapp:1.0 registry.company.vn/team/myapp:1.0
#          │         └─ tên MỚI (thêm vào, KHÔNG mất tên cũ)
#          └─────────── tên đang có
```

Sau lệnh này, **một** image có **hai** tên trỏ vào (giống hai shortcut cùng trỏ một file). Bắt buộc phải làm bước này trước khi `push` lên registry riêng, vì tên image **chính là địa chỉ** đẩy đi.

**`save`/`load` — cứu tinh cho môi trường air-gapped (VDI không ra internet):**

```bash
# Máy CÓ mạng:
docker save -o app.tar myapp:1.0     # image  -> file tar (bê qua USB/scp)
# Máy KHÔNG có mạng:
docker load -i app.tar               # file tar -> image (không cần registry)
```

⚠️ Đừng nhầm với cặp `export`/`import` — cặp đó làm việc trên **container** và **mất hết layer + metadata** (ENV, CMD, ENTRYPOINT). Chuyển image thì luôn dùng `save`/`load`.

**"dangling image" là gì?** Là image **không còn tag nào trỏ vào**, hiện dưới dạng `<none>:<none>`. Sinh ra khi bạn build lại `myapp:1.0` — tag `1.0` nhảy sang image mới, image cũ bị **bỏ rơi** nhưng **vẫn chiếm disk**. Đây là thủ phạm số 1 của "máy build hết dung lượng".

| Lệnh | Xoá cái gì | Mức độ nguy hiểm |
|---|---|---|
| `docker image prune` | Chỉ image `<none>` mồ côi | An toàn |
| `docker image prune -a` | **Cả** image có tag mà không container nào đang dùng | Phải `docker pull` lại — chậm, và **hỏng nếu đang offline** |

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa mục Volume & Network</b></summary>

**Bài toán gốc của volume**: filesystem trong container là **tạm**. `docker rm` là **bay sạch** data. Database chạy trong container không có volume = **mất toàn bộ dữ liệu** mỗi lần deploy lại. Volume là **thư mục sống ngoài vòng đời container**.

**Hai kiểu gắn data — hay bị gọi lẫn lộn là "volume":**

| | Named volume | Bind mount |
|---|---|---|
| Cú pháp | `-v mydata:/var/lib/postgresql/data` | `-v $(pwd)/data:/app/data` |
| Nhận dạng | vế trái là **cái tên** | vế trái là **đường dẫn** (có `/` hoặc `./`) |
| Data nằm ở | Docker quản lý (`/var/lib/docker/volumes/`) | **Đúng thư mục bạn chỉ** trên host |
| Hiện trong `docker volume ls` | **Có** | **Không** |
| Hợp cho | Production, data của DB | Dev — sửa code ở host, container thấy ngay |

⇒ Vì bind mount **không hiện** trong `docker volume ls`, đừng kết luận "không có volume nào" là "không có data nào được gắn". Muốn thấy hết, phải dùng `docker inspect <container>` và đọc mục `Mounts`.

| Lệnh | Làm gì | Lưu ý |
|---|---|---|
| `docker volume ls` | Liệt kê named volume | **Không** thấy bind mount |
| `docker volume inspect <n>` | JSON: `Mountpoint` = đường dẫn thật trên host | Cách tìm data thật nằm ở đâu |
| `docker volume prune` | Xoá volume **không container nào dùng** | 🛑 **Mất data vĩnh viễn, không hỏi lại lần hai** |

**Network — vì sao cần tự tạo?** Container trong **cùng một** network do bạn tạo thì **gọi nhau bằng TÊN** (`http://api:3000`) — Docker có sẵn DNS nội bộ. Ở network `bridge` mặc định thì **không có** tính năng đó, phải dùng IP — mà IP thì đổi mỗi lần restart. Đây chính là lý do `docker compose` **tự tạo** một network riêng cho mỗi project.

```bash
docker network create backend
docker run -d --name db     --network backend postgres
docker run -d --name api    --network backend myapi
#                            └─ giờ trong container api, gõ "db:5432" là tới được Postgres
```

`docker network inspect <name>` cho biết **container nào đang nối vào** network đó — dùng để trả lời câu "vì sao api gọi db không được": nếu hai container không cùng network thì **không có đường nào tới nhau**, dù cùng chạy trên một máy.

</details>

### Registry - Đăng nhập / đăng xuất
```bash
docker login                           # Đăng nhập Docker Hub
docker login <registry-url>            # Đăng nhập registry riêng
docker logout                          # Đăng xuất
```

<details>
<summary><b>Bấm xem: giải nghĩa mục Registry</b></summary>

**Registry là gì?** Là **kho chứa image** ở xa — nơi `docker push` đẩy lên và `docker pull` tải về. Docker Hub là registry công cộng mặc định; công ty thường tự dựng registry riêng (Harbor, Nexus, ECR...).

| Lệnh | Làm gì | Ghi chú |
|---|---|---|
| `docker login` | Đăng nhập **Docker Hub** (mặc định) | Không ghi URL = mặc định Hub |
| `docker login <registry-url>` | Đăng nhập **registry riêng** | Ví dụ `docker login harbor.company.vn` |
| `docker logout` | Xoá thông tin đăng nhập | Nên làm trên máy dùng chung |

⚠️ **Credential lưu ở đâu?** Trong `~/.docker/config.json`. Mặc định trên Linux nó chỉ là **base64 — KHÔNG phải mã hoá**, ai đọc được file là đọc được mật khẩu:

```bash
cat ~/.docker/config.json | jq '.auths'   # xem đang đăng nhập những registry nào
```

**Trong CI/CD, đừng gõ password tương tác** — dùng `--password-stdin` để password **không lọt vào log và không nằm trong lịch sử shell**:

```bash
echo "$REGISTRY_PASSWORD" | docker login harbor.company.vn -u "$USER" --password-stdin
#                                                          │   └─ đọc password từ đầu vào chuẩn
#                                                          └───── user (-u = --username)
```

🛑 Cách sai hay gặp: `docker login -u user -p 'MatKhau'` — password hiện nguyên văn trong `ps aux` (mọi user trên máy đều thấy) và trong log CI.

</details>

### Dọn dẹp (Cleanup) - Rất hữu ích khi hết dung lượng
```bash
docker system df                       # Xem dung lượng đang dùng
docker system prune                    # Dọn container/network/image dangling
docker system prune -a                 # Dọn TẤT CẢ (kể cả image không dùng)
docker system prune -a --volumes       # Dọn cả volume (cẩn thận mất data!)
docker container prune                  # Xóa container đã dừng
```

<details>
<summary><b>Bấm xem: giải nghĩa mục Cleanup — ĐỌC KỸ TRƯỚC KHI CHẠY</b></summary>

**Luôn chạy `docker system df` TRƯỚC** để biết cái gì đang ăn disk, rồi mới dọn đúng cái đó:

```bash
docker system df
#      │      └─ disk free (mượn tên lệnh `df` của Linux)
#      └──────── nhóm lệnh quản lý toàn hệ thống Docker
```

Output thật trông như thế này — cột **RECLAIMABLE** là phần **dọn được**:

```
TYPE            TOTAL   ACTIVE   SIZE      RECLAIMABLE
Images          38      6        12.4GB    9.1GB (73%)
Containers      12      4        1.2GB     840MB (70%)
Local Volumes   9       3        22.8GB    18.2GB (79%)   <- thủ phạm thường ở đây
Build Cache     201     0        6.7GB     6.7GB
```

**Thang mức độ nguy hiểm — đọc từ trên xuống:**

| Lệnh | Xoá | Nguy hiểm |
|---|---|---|
| `docker container prune` | Container **đã dừng** | 🟢 An toàn |
| `docker image prune` | Image `<none>` mồ côi | 🟢 An toàn |
| `docker system prune` | Container dừng + network thừa + image mồ côi + build cache | 🟡 An toàn với data, nhưng build sau chậm hơn |
| `docker system prune -a` | **Cộng thêm** mọi image không container nào chạy | 🟠 Phải pull lại. **Offline/VDI air-gapped là kẹt** |
| `docker system prune -a --volumes` | **Cộng thêm VOLUME** | 🔴 **MẤT DATABASE. Không hoàn tác được** |

🛑 `--volumes` xoá volume không được container **đang chạy** dùng. Container DB đang `stop` để bảo trì ⇒ volume của nó bị tính là "không dùng" ⇒ **bay sạch data**.

**Cách an toàn — luôn xem trước bằng `--filter`, và giữ lại đồ mới:**

```bash
docker image prune -a --filter "until=168h"
#                  │  └─────────────────── chỉ xoá image cũ hơn 168 giờ (=7 ngày)
#                  └────────────────────── cả image có tag không dùng

docker builder prune --filter "until=24h"   # chỉ dọn build cache — an toàn nhất, thường thu lại nhiều GB
```

</details>

### Format tùy chỉnh (hay dùng)
```bash
docker ps --format "{{.Names}}"                              # Chỉ hiện tên
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"  # Bảng gọn
```

<details>
<summary><b>Bấm xem: giải nghĩa cú pháp `--format`</b></summary>

`--format` cho phép **tự chọn cột** thay vì nhận bảng mặc định (rộng, tràn dòng, khó đọc). Cú pháp `{{.Ten}}` là **Go template** — Docker viết bằng Go nên mượn luôn cú pháp này.

```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
#                   │      │          │            └─ cột port mapping
#                   │      │          └────────────── cột trạng thái (Up 3 hours / Exited (1))
#                   │      └───────────────────────── cột tên container
#                   └──────────────────────────────── từ khoá "table" = in KÈM dòng tiêu đề
#                                                     (bỏ đi thì chỉ có data, hợp cho script)
#                        \t = ký tự Tab, Docker dùng để canh cột thẳng hàng
```

**Các trường hay dùng** (viết hoa chữ đầu, phân biệt hoa thường):

| Trường | Nội dung |
|---|---|
| `{{.Names}}` | Tên container |
| `{{.ID}}` | ID ngắn |
| `{{.Image}}` | Image đang chạy |
| `{{.Status}}` | `Up 3 hours` / `Exited (137) 2 min ago` |
| `{{.Ports}}` | Mapping port |
| `{{.Size}}` | Dung lượng chiếm |

💡 Muốn biết còn trường nào: `docker ps --format '{{json .}}' | jq` — in **toàn bộ** trường có sẵn dưới dạng JSON.

**Vì sao `{{.Names}}` số nhiều mà `{{.ID}}` số ít?** Vì một container **có thể có nhiều tên** (network alias), nên Docker đặt tên trường ở dạng số nhiều. Gõ `{{.Name}}` là **ra rỗng, không báo lỗi** — bẫy hay gặp nhất khi dùng `--format`.

**Lưu vào `~/.zshrc` cho khỏi gõ lại** (bạn đã có sẵn trong `docker.txt`):

```bash
alias dpsf='docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"'
```

⚠️ Exit code hay gặp trong cột Status: **`Exited (137)`** = bị **SIGKILL**, gần như luôn là **OOM — hết RAM**, kiểm chứng bằng `docker inspect <c> | jq '.[].State.OOMKilled'`. **`Exited (1)`** = app tự crash, xem `docker logs`. **`Exited (0)`** = chạy xong bình thường.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa các cờ Docker Compose</b></summary>

**Tiền đề — Compose giải bài toán gì?** Chạy tay 5 container (app + db + redis + nginx + worker) nghĩa là 5 lệnh `docker run` dài loằng ngoằng, phải nhớ đúng thứ tự bật, đúng network, đúng volume. Sai một cờ là hỏng. Compose gom **toàn bộ** mô tả đó vào **1 file YAML**, còn lệnh chỉ còn `up`/`down`.

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-d` | **d**etached | Chạy **nền**. Không có `-d` thì log mọi service đổ ra terminal và **Ctrl+C là tắt hết** |
| `--build` | build | **Build lại image** trước khi chạy. Không có nó, Compose dùng image cũ ⇒ **sửa code mà không thấy đổi gì** — lỗi kinh điển |
| `--no-cache` | không cache | Build bỏ qua cache, từ đầu |
| `-v` (trong `down -v`) | **v**olumes | 🔴 **XOÁ CẢ VOLUME** — mất sạch data DB |
| `--rmi all` | **r**e**m**ove **i**mages | Xoá luôn image sau khi down |
| `-f` (trong `-f file.yml`) | **f**ile | Chỉ định **file compose nào**. Mặc định tìm `docker-compose.yml`/`compose.yaml` ở thư mục hiện tại |
| `-f` (trong `logs -f`) | **f**ollow | Bám log realtime. **Cùng chữ `-f` nhưng nghĩa khác hẳn** tuỳ vị trí đứng |
| `--env-file` | environment file | Nạp file biến môi trường |
| `--profile` | profile | Chỉ bật nhóm service gắn nhãn profile đó (ví dụ service chỉ dùng khi dev) |

⚠️ **`-f` có HAI nghĩa khác nhau** trong Compose, phân biệt bằng **vị trí**:

```bash
docker compose -f docker-compose.prod.yml logs -f api
#              │                               │
#              │                               └─ ĐỨNG SAU lệnh con = follow (bám log)
#              └─────────────────────────────── ĐỨNG TRƯỚC lệnh con = file (chọn file compose)
```

**Thang mức độ nguy hiểm của `down`:**

| Lệnh | Xoá container | Xoá network | Xoá **volume (DATA)** | Xoá image |
|---|---|---|---|---|
| `docker compose stop` | ❌ (chỉ dừng) | ❌ | ❌ | ❌ |
| `docker compose down` | ✅ | ✅ | ❌ | ❌ |
| `docker compose down -v` | ✅ | ✅ | 🔴 **CÓ** | ❌ |
| `docker compose down --rmi all` | ✅ | ✅ | ❌ | ✅ |

🛑 `down -v` là lệnh **mất data không hoàn tác**. Muốn chỉ tắt tạm để bật lại ⇒ dùng **`stop`**, không dùng `down`.

**`restart` vs `up -d --build` — vì sao restart không thấy code mới?**

| Lệnh | Đọc lại file compose? | Build lại image? | Kết quả |
|---|---|---|---|
| `docker compose restart` | ❌ **KHÔNG** | ❌ | Chỉ tắt-bật tiến trình, **giữ nguyên container cũ** |
| `docker compose up -d` | ✅ | ❌ | Tạo lại container nếu config đổi |
| `docker compose up -d --build` | ✅ | ✅ | **Cách duy nhất chắc chắn** áp dụng code mới |

⇒ Sửa `docker-compose.yml` rồi gõ `restart` là **không có tác dụng gì** — đây là lý do hay tưởng "sửa xong mà không ăn".

**`run` vs `exec` — cùng là chạy lệnh nhưng khác hẳn:**

| | `docker compose exec api sh` | `docker compose run api sh` |
|---|---|---|
| Container | Vào container **đang chạy** | Tạo container **MỚI** |
| Cần service đang chạy? | **Có**, không thì báo lỗi | Không |
| Dùng khi | Debug service đang sống | Chạy migration, seed data, một lệnh one-off |

⚠️ `run` để lại container thừa mỗi lần chạy ⇒ nên thêm `--rm`: `docker compose run --rm api npm run migrate`

**Lệnh chẩn đoán quan trọng nhất — `config`:**

```bash
docker compose config
#              └─ đọc file YAML + thay hết biến ${...} + gộp các file -f
#                 rồi IN RA cấu hình CUỐI CÙNG mà Compose thực sự dùng
```

Đây là cách trả lời "biến môi trường của tôi có được nạp không?" — nếu `config` in ra `${DB_PASSWORD}` **còn nguyên dấu ngoặc** thì biến **chưa được nạp**, không phải lỗi app.

**Compose v1 vs v2 — vì sao có hai cách gõ?**

| | `docker-compose` (có gạch) | `docker compose` (có dấu cách) |
|---|---|---|
| Là gì | **Chương trình riêng** viết bằng Python (v1) | **Plugin** tích hợp trong Docker CLI (v2, Go) |
| Trạng thái | Đã ngừng hỗ trợ (EOL) | Bản hiện hành |
| Tên container sinh ra | `project_service_1` (gạch dưới) | `project-service-1` (gạch ngang) |

⚠️ Khác biệt gạch dưới/gạch ngang làm **script cũ grep theo tên container bị hỏng** khi nâng lên v2.

⚠️ `docker compose scale` là **cú pháp cũ**; bản mới dùng: `docker compose up -d --scale api=3`

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa mục Helm Repository</b></summary>

**Tiền đề — Helm là gì, giải bài toán gì?** Cài một app lên K8s cần chục file YAML (Deployment, Service, Ingress, ConfigMap, Secret, PVC...). Cài lên 3 môi trường dev/staging/prod = chép 3 bản, sửa tay từng chỗ khác nhau ⇒ lệch nhau lúc nào không biết. **Helm** đóng gói đống YAML đó thành một **chart** có **tham số** (`values.yaml`), nên một chart dùng cho cả 3 môi trường, chỉ đổi values.

**Ba từ phải phân biệt — hay bị dùng lẫn:**

| Từ | Là gì | Ví von |
|---|---|---|
| **Chart** | Gói template YAML + values mặc định | **Công thức nấu ăn** |
| **Repo** | Kho chứa nhiều chart (một URL) | **Cuốn sách công thức** |
| **Release** | Một lần **cài chart đó vào cluster**, có tên riêng | **Món đã nấu ra**, nấu 2 lần = 2 món |

⇒ Cùng 1 chart `postgresql` cài 2 lần thành 2 release `pg-dev` và `pg-prod`, **độc lập nhau**.

| Lệnh | Làm gì | Ghi chú |
|---|---|---|
| `helm repo add <name> <url>` | Thêm kho chart, `<name>` là **bí danh tự đặt** để gọi sau | `helm repo add bitnami https://charts.bitnami.com/bitnami` |
| `helm repo update` | **Tải lại danh mục** chart mới nhất từ các repo | ⚠️ Helm cache danh mục ở local. **Không `update` thì `search`/`install` vẫn ra bản CŨ** dù repo đã có bản mới |
| `helm repo list` | Liệt kê repo đã thêm | Kiểm tra tên/URL |
| `helm repo remove <name>` | Bỏ repo | |
| `helm search repo <kw>` | Tìm trong **repo đã add** (offline, đọc cache) | Chạy được trong môi trường air-gapped |
| `helm search hub <kw>` | Tìm trên **Artifact Hub** (toàn Internet) | **Cần mạng ra ngoài** — trên VDI bị chặn sẽ timeout |

**Vì sao có tới hai lệnh search?** Vì hai **phạm vi** khác nhau:

```bash
helm search repo postgres          # chỉ trong các kho TÔI ĐÃ ADD -> ra vài dòng
helm search hub  postgres          # toàn bộ Artifact Hub công cộng -> ra hàng trăm dòng
```

⇒ Trong môi trường nội bộ/air-gapped, `search hub` **không dùng được**; thay thế bằng `search repo` sau khi đã add repo nội bộ (Harbor, Nexus).

💡 Xem được các **phiên bản** của một chart (rất cần khi phải ghim version):

```bash
helm search repo bitnami/postgresql --versions
#                                   └─ liệt kê MỌI version, không chỉ bản mới nhất
```

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa các cờ install/upgrade — ĐỌC KỸ `--atomic` và `-n`</b></summary>

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-n` | **n**amespace | Cài vào **namespace nào**. ⚠️ Không ghi = vào namespace `default` (hoặc namespace đang set trong context) — hay bị cài nhầm chỗ |
| `--create-namespace` | tạo namespace | Tạo namespace nếu **chưa tồn tại**. Không có cờ này mà namespace chưa có ⇒ **lỗi ngay**, không tự tạo |
| `-f` | **f**ile (values) | Dùng **file values riêng** đè lên mặc định của chart. Ghi **nhiều `-f`** được, file sau đè file trước |
| `--set` | set | Đè **một giá trị** ngay trên dòng lệnh, không cần file |
| `--dry-run` | chạy khô | **Không cài thật** — chỉ render ra và gửi lên API server kiểm tra |
| `--debug` | debug | In **chi tiết**: manifest đã render + values cuối cùng |
| `--atomic` | nguyên tử | ⭐ **Fail thì TỰ ĐỘNG rollback về trạng thái cũ** |
| `--wait` | chờ | Chờ pod thật sự Ready mới coi là thành công (`--atomic` đã bao gồm `--wait`) |
| `--timeout` | hết giờ | Thời gian chờ tối đa, mặc định `5m0s` |
| `--version` | version | Ghim **phiên bản chart** — bắt buộc dùng ở production |

⭐ **`--atomic` — vì sao BẮT BUỘC dùng ở production?**

Không có `--atomic`: `helm upgrade` fail giữa chừng ⇒ release nằm ở trạng thái **`failed`**, **một nửa** resource đã đổi, một nửa chưa ⇒ hệ thống **lai tạp**, và lần upgrade sau bị chặn bởi lỗi `another operation is in progress`.

Có `--atomic`: fail ⇒ Helm **tự rollback** về đúng revision trước ⇒ hệ thống về nguyên trạng, không có trạng thái nửa vời.

```bash
helm upgrade --install myapp ./chart -n prod --create-namespace \
     -f values-prod.yaml --atomic --timeout 10m --version 1.4.2
#    │        │                   │             │        │        └─ ghim version chart
#    │        │                   │             │        └────────── chờ tối đa 10 phút
#    │        │                   │             └─────────────────── FAIL thì tự rollback
#    │        │                   └───────────────────────────────── values riêng cho prod
#    │        └───────────────────────────────────────────────────── chưa có thì CÀI, có rồi thì NÂNG CẤP
#    └────────────────────────────────────────────────────────────── nâng cấp
```

⭐ **`helm upgrade --install` — vì sao gần như luôn dùng dạng này?** Vì nó **idempotent** — chạy bao nhiêu lần cũng ra một kết quả:

| Tình huống | `helm install` | `helm upgrade` | `helm upgrade --install` |
|---|---|---|---|
| Release **chưa có** | ✅ cài | ❌ **lỗi** `release: not found` | ✅ cài |
| Release **đã có** | ❌ **lỗi** `cannot re-use a name` | ✅ nâng cấp | ✅ nâng cấp |

⇒ Trong CI/CD, chỉ `upgrade --install` là chạy được cả lần đầu lẫn các lần sau mà không cần rẽ nhánh if/else.

**`-f` vs `--set` — dùng cái nào?**

| | `-f values.yaml` | `--set key=value` |
|---|---|---|
| Hợp cho | Cấu hình dài, cố định, **commit vào Git** | Giá trị đổi mỗi lần deploy (image tag) |
| Kiểu dữ liệu | Giữ đúng kiểu YAML | ⚠️ Helm **tự đoán kiểu** — dễ sai |
| Lưu vết | Có, trong Git | Không, nằm trong lịch sử shell |

⚠️ Bẫy kiểu dữ liệu của `--set`: `--set tag=1.10` → Helm hiểu là **số `1.1`** (mất số 0), image tag hỏng. Cách sửa: `--set-string tag=1.10`, hoặc bọc `--set tag="1.10"`.

⚠️ Bẫy dấu chấm: `--set nodeSelector.kubernetes\.io/hostname=node1` — dấu `.` trong **tên key** phải escape bằng `\.`, nếu không Helm hiểu thành cây lồng nhau.

**`--dry-run` vs `helm template` — khác nhau ở đâu?** (đây là chỗ hay tưởng như nhau)

| | `helm template` | `helm install --dry-run` |
|---|---|---|
| Có gọi API server K8s? | **KHÔNG** — thuần local | **CÓ** — gửi lên để validate |
| Bắt được lỗi sai schema/CRD chưa cài? | ❌ Không | ✅ Có |
| Chạy được khi **chưa** có cluster? | ✅ Được | ❌ Không |

⇒ Kiểm tra cú pháp offline: `helm template`. Kiểm tra "cài lên cluster này có nuốt được không": `--dry-run`.

**Rollback:**

```bash
helm history myapp -n prod        # xem có những revision nào (1,2,3...) + trạng thái
helm rollback myapp 3 -n prod     # quay về revision 3
helm rollback myapp -n prod       # KHÔNG ghi số = quay về revision LIỀN TRƯỚC
```

⚠️ `helm rollback` **KHÔNG khôi phục dữ liệu** trong PVC — nó chỉ đưa **manifest** về bản cũ. Database đã bị migration sửa schema thì rollback Helm không cứu được.

⚠️ `helm uninstall` **mặc định xoá luôn lịch sử revision** ⇒ không rollback lại được. Muốn giữ: `helm uninstall myapp --keep-history`.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa các lệnh Helm xem thông tin</b></summary>

| Lệnh | Trả về gì | Dùng khi |
|---|---|---|
| `helm list` | Release trong **namespace hiện tại** | Xem nhanh |
| `helm list -A` | **A**ll namespaces — **mọi** release toàn cluster | ⭐ Dùng cái này để khỏi sót |
| `helm list -a` | **a**ll statuses — kể cả release `failed`, `pending` | Điều tra khi release biến mất khỏi `list` thường |
| `helm status <r>` | Trạng thái + NOTES.txt (hướng dẫn truy cập app) | Sau khi cài xong |
| `helm history <r>` | Bảng revision: số, thời gian, trạng thái, mô tả | Trước khi rollback |
| `helm get values <r>` | Values **bạn đã đè** vào | Trả lời "prod đang chạy config gì?" |
| `helm get values <r> -a` | Values **đầy đủ** = mặc định của chart + phần đè | Xem giá trị thật của mọi tham số |
| `helm get manifest <r>` | **YAML cuối cùng** đã gửi lên K8s | ⭐ Chân lý cuối cùng khi debug |
| `helm get all <r>` | Tất cả những thứ trên gộp lại | Điều tra sâu |

⚠️ **`-a` có hai nghĩa khác nhau tuỳ lệnh** — đây là điểm dễ nhầm:

| Lệnh | `-a` nghĩa là | |
|---|---|---|
| `helm list -a` | **a**ll statuses (cả failed/pending) | ≠ `-A` (all namespaces) — **chữ hoa chữ thường khác nghĩa hẳn** |
| `helm get values -a` | **a**ll values (cả giá trị mặc định) | |

**Quy trình debug chuẩn khi "cài xong mà app sai cấu hình":**

```bash
helm get values myapp -n prod          # 1. tôi đã đè những gì?
helm get values myapp -n prod -a       # 2. giá trị THẬT sau khi trộn với default là gì?
helm get manifest myapp -n prod        # 3. YAML thật đã lên cluster ra sao?
helm get manifest myapp -n prod | kubectl diff -f -   # 4. cluster có bị sửa tay lệch đi không?
#                                        │             └─ so YAML của Helm với thực tế cluster
#                                        └─ đọc từ stdin ("-" = đầu vào chuẩn)
```

Bước 4 trả lời câu hỏi kinh điển: *"Helm nói đã cài đúng, sao cluster lại khác?"* — thường do ai đó `kubectl edit` sửa tay, Helm không biết.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa các lệnh phát triển chart</b></summary>

| Lệnh | Làm gì | Chạy offline được? |
|---|---|---|
| `helm create <name>` | Sinh **bộ khung** chart mẫu (có sẵn Deployment/Service/Ingress/HPA) | ✅ |
| `helm lint <chart>` | Soi lỗi **cú pháp + quy ước** của chart | ✅ |
| `helm template <chart>` | Render template ra YAML thuần, **in ra màn hình** | ✅ Không cần cluster |
| `helm package <chart>` | Đóng gói thành file `.tgz` để đẩy lên repo | ✅ |
| `helm show values <chart>` | In `values.yaml` **mặc định** của chart | ✅ (nếu chart đã tải) |
| `helm show chart <chart>` | Metadata: version, appVersion, mô tả, dependency | ✅ |
| `helm dependency update <chart>` | Tải các **sub-chart** khai báo trong `Chart.yaml` về `charts/` | ❌ Cần mạng tới repo |

⭐ **`helm show values` — lệnh nên chạy ĐẦU TIÊN** khi dùng chart của người khác. Nó cho biết chart có **những tham số nào** để đè; không xem thì phải đoán tên key, và `--set` sai tên key thì Helm **im lặng bỏ qua, không báo lỗi** — đây là nguyên nhân số 1 của "đã set rồi mà không ăn".

```bash
helm show values bitnami/postgresql > default-values.yaml
#                                   └─ lưu ra file để đọc và chép phần cần đè
```

**Cấu trúc một chart sau `helm create`:**

```
mychart/
├── Chart.yaml        # metadata: tên, version chart, appVersion, dependency
├── values.yaml       # giá trị MẶC ĐỊNH (người dùng đè bằng -f / --set)
├── charts/           # sub-chart phụ thuộc (do `dependency update` tải về)
└── templates/
    ├── deployment.yaml   # YAML có chèn {{ .Values.xxx }}
    ├── _helpers.tpl      # hàm dùng chung (tên bắt đầu bằng _ thì KHÔNG render ra resource)
    └── NOTES.txt         # lời nhắn in ra sau khi cài xong
```

**Hai chữ "version" trong `Chart.yaml` — rất hay nhầm:**

| Trường | Là version của | Ai tăng |
|---|---|---|
| `version` | **Bản thân chart** (bộ template) | Tăng khi sửa template |
| `appVersion` | **Ứng dụng** bên trong (image tag) | Tăng khi app ra bản mới |

⇒ Sửa template mà quên tăng `version` ⇒ repo chart không nhận bản mới, người dùng vẫn tải bản cũ.

**Bộ ba kiểm tra trước khi commit chart:**

```bash
helm lint ./mychart                                  # 1. cú pháp & quy ước
helm template myapp ./mychart -f values-prod.yaml    # 2. render ra xem YAML thật đúng chưa
helm template myapp ./mychart | kubectl apply --dry-run=server -f -
#                                              │                  └─ đọc từ stdin
#                                              └─ 3. hỏi API SERVER: manifest này có hợp lệ không
#                                                 (=server: cluster kiểm; =client: chỉ kiểm cú pháp local)
```

💡 `helm template` mặc định **bỏ qua** template có điều kiện `if`. Muốn render cả những phần đó để soi: thêm `--debug`, hoặc bật đúng flag values tương ứng.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa mục Context & Cluster — hiểu kubeconfig trước</b></summary>

**Tiền đề — "context" là gì?** `kubectl` không tự biết phải nói chuyện với cluster nào. Nó đọc file **`~/.kube/config`** (kubeconfig), trong đó có 3 danh sách và một con trỏ:

| Thành phần | Là gì | Ví von |
|---|---|---|
| `clusters` | **Địa chỉ** API server + CA cert | Địa chỉ toà nhà |
| `users` | **Danh tính** (cert, token) để chứng minh mình là ai | Thẻ nhân viên |
| `contexts` | Một **cặp ghép** (cluster + user + namespace mặc định) | "Vào toà nhà A bằng thẻ B, tầng C" |
| `current-context` | Con trỏ đang chỉ vào **1 context** | Bạn **đang** ở đâu |

⇒ 🛑 Đây là lý do sự cố kinh điển: gõ `kubectl delete` mà **quên kiểm tra context** ⇒ xoá nhầm trên **production**. `kubectl` không hỏi lại, không cảnh báo.

| Lệnh | Làm gì |
|---|---|
| `kubectl config get-contexts` | Liệt kê mọi context; dấu **`*`** ở cột đầu = context **đang dùng** |
| `kubectl config current-context` | Chỉ in **tên** context hiện tại (gọn, hợp cho script/prompt) |
| `kubectl config use-context <n>` | **Chuyển** sang context khác |
| `kubectl config set-context --current --namespace=<ns>` | Đổi **namespace mặc định** của context đang dùng |
| `kubectl cluster-info` | Địa chỉ control plane + CoreDNS — kiểm tra **có kết nối được không** |
| `kubectl version` | Version **client** (kubectl trên máy) và **server** (API server) |
| `kubectl get nodes` | Danh sách node + trạng thái Ready |
| `kubectl top nodes` | CPU/RAM **thực đang dùng** của node |

**Bóc `set-context` — lệnh dài, dễ gõ sai:**

```bash
kubectl config set-context --current --namespace=ai-hub
#                          │         └─ đặt namespace mặc định thành ai-hub
#                          └─────────── áp dụng cho context ĐANG DÙNG
#                                       (không có --current thì phải ghi rõ TÊN context)
```

⇒ Sau lệnh này, `kubectl get pods` **không cần `-n ai-hub`** nữa. Đây là cách tránh gõ `-n` lặp đi lặp lại cả ngày.

⭐ **Thói quen an toàn — luôn kiểm tra trước khi làm việc nguy hiểm:**

```bash
kubectl config current-context     # đang ở cluster nào? ĐỌC KỸ trước mọi lệnh delete/apply
```

💡 Bạn đã có `alias kctx` và `alias kns` trong cheatsheet này; công cụ `kubectx`/`kubens` (mục TUI phía dưới) làm việc tương tự nhưng có menu chọn.

**`kubectl version` — vì sao phải quan tâm độ lệch?** K8s chỉ bảo đảm kubectl lệch **±1 minor version** so với API server. Lệch xa hơn (client 1.31 nói với server 1.25) ⇒ có lệnh **báo lỗi khó hiểu** hoặc **thiếu field** khi apply.

```bash
kubectl version -o yaml     # -o = output, in dạng YAML cho dễ đọc cả 2 phía
```

⚠️ `kubectl top nodes` báo `error: Metrics API not available` ⇒ **không phải cluster hỏng**, mà là **chưa cài `metrics-server`**. Đây là add-on riêng, không có sẵn. Thay thế tạm thời để xem tài nguyên: `kubectl describe node <node>` (mục *Allocated resources* — nhưng đó là **request đã đặt trước**, không phải mức dùng thật).

⇒ Phân biệt rõ: **`top`** = đang dùng thật bao nhiêu · **`describe node`** = đã hứa cấp phát bao nhiêu. Node "hết chỗ schedule" mà CPU nhàn rỗi là chuyện bình thường — vì tính theo **request**, không theo mức dùng thật.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa các cờ get/describe</b></summary>

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-A` | **A**ll namespaces | **Mọi** namespace. ⚠️ Chữ **HOA**. `-a` thường **không có** nghĩa này |
| `-n` | **n**amespace | Chỉ 1 namespace cụ thể |
| `-o` | **o**utput | Đổi định dạng in ra: `wide`, `yaml`, `json`, `name`, `jsonpath=...`, `custom-columns=...` |
| `-o wide` | wide = rộng | Thêm cột **NODE, IP, NOMINATED NODE** — biết pod nằm ở node nào |
| `-w` | **w**atch | **Bám** — có thay đổi là in thêm dòng mới. Ctrl+C thoát |
| `-l` | **l**abel selector | Lọc theo nhãn, ví dụ `-l app=nginx` |
| `--sort-by` | sắp xếp | Sắp theo một field JSONPath |
| `--show-labels` | hiện nhãn | Thêm cột labels |

**`get` vs `describe` — khác nhau căn bản, dùng sai là mất thời gian:**

| | `kubectl get pod <p>` | `kubectl describe pod <p>` |
|---|---|---|
| Nguồn dữ liệu | Chỉ đọc **object** trong etcd | Object **+ EVENTS liên quan** |
| Định dạng | Bảng gọn / YAML | Văn bản dài, có mục **Events** ở cuối |
| Trả lời được | "Nó đang ở trạng thái gì?" | ⭐ "**VÌ SAO** nó ở trạng thái đó?" |

⇒ Pod `Pending` mãi không chạy: `get` chỉ nói `Pending`; **`describe` mới nói lý do** ở mục Events, ví dụ `0/3 nodes are available: 3 Insufficient memory`.

⭐ **Mục Events trong `describe` là chỗ quan trọng nhất khi debug** — luôn kéo xuống cuối đọc trước tiên.

**Bóc lệnh xem events toàn cluster:**

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
#                  └─ sắp theo thời gian TẠO, tăng dần -> việc mới nhất nằm CUỐI
#                     (mặc định events KHÔNG sắp xếp -> đọc rất rối)
```

⚠️ **Bẫy thời gian**: `--sort-by=.metadata.creationTimestamp` sắp theo lúc event **được tạo lần đầu**; một sự kiện lặp lại nhiều lần thì mốc đó vẫn là mốc cũ. Muốn theo lần xảy ra **gần nhất**, dùng `.lastTimestamp`:

```bash
kubectl get events -A --sort-by=.lastTimestamp | tail -30
#                                                └─ chỉ lấy 30 dòng cuối = mới nhất
```

⚠️ **Events chỉ sống mặc định 1 giờ** rồi bị xoá. Sự cố xảy ra đêm qua thì sáng nay `get events` **không còn gì** — không phải "không có lỗi". Lúc đó phải xem log tập trung (Loki/ELK) hoặc `kubectl logs --previous`.

**`-o` — các dạng output và khi nào dùng:**

```bash
kubectl get pods -o wide                  # thêm cột NODE + IP
kubectl get pod x -o yaml                 # toàn bộ object, đọc được spec + status
kubectl get pods -o name                  # chỉ "pod/tenpod" -> đưa vào lệnh khác
kubectl get pods -o jsonpath='{.items[*].spec.nodeName}'   # bóc đúng 1 field
#                            │  │       │    └─ lấy field nodeName
#                            │  │       └────── [*] = mọi phần tử trong mảng
#                            │  └────────────── items = mảng kết quả của lệnh get
#                            └───────────────── {} bao quanh biểu thức JSONPath
```

**Viết tắt tên resource** (gõ được cả hai, ý nghĩa như nhau):

| Đầy đủ | Tắt | | Đầy đủ | Tắt |
|---|---|---|---|---|
| `pods` | `po` | | `services` | `svc` |
| `deployments` | `deploy` | | `namespaces` | `ns` |
| `statefulsets` | `sts` | | `configmaps` | `cm` |
| `daemonsets` | `ds` | | `persistentvolumeclaims` | `pvc` |

💡 Tra đầy đủ: `kubectl api-resources` (cột SHORTNAMES).

⚠️ **`kubectl get all` KHÔNG phải "tất cả"** — tên gọi gây hiểu nhầm nghiêm trọng. Nó chỉ lấy vài loại thông dụng (pod, svc, deploy, rs, sts, ds, job, cronjob) và **BỎ SÓT**: ConfigMap, Secret, Ingress, PVC, ServiceAccount, Role, CRD... ⇒ Dùng `get all` để backup là **mất cấu hình**. Muốn thật sự đủ, phải liệt kê rõ loại:

```bash
kubectl get all,cm,secret,ingress,pvc,sa -n <ns>
```

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa mục tạo/sửa/xoá — apply vs create, và `--force` nguy hiểm</b></summary>

⭐ **`apply` vs `create` — khác nhau về TƯ DUY, không chỉ cú pháp:**

| | `kubectl create -f` | `kubectl apply -f` |
|---|---|---|
| Kiểu | **Mệnh lệnh** (imperative): "tạo cái này" | **Khai báo** (declarative): "trạng thái phải như thế này" |
| Resource **đã tồn tại** | ❌ Lỗi `AlreadyExists` | ✅ Cập nhật phần khác biệt |
| Chạy lại lần 2 | ❌ Lỗi | ✅ Không sao (idempotent) |
| Dùng trong CI/CD | Không hợp | ⭐ **Đúng cách** |

⇒ `apply` là lệnh dùng hằng ngày. `create` chỉ hợp khi tạo nhanh thứ chưa có (`create namespace`, `create secret`).

**Vì sao `apply` biết phải sửa gì?** Nó ghi bản YAML bạn gửi vào annotation `kubectl.kubernetes.io/last-applied-configuration`, rồi so **3 bên**: bản cũ đã apply · bản mới bạn gửi · trạng thái thật trên cluster. Nhờ vậy nó biết field nào bạn **cố tình bỏ đi** để xoá, và field nào do component khác (HPA) đặt thì **không đụng vào**.

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-f` | **f**ile | Từ file, thư mục, hoặc URL. `-f -` = đọc từ **đầu vào chuẩn** (stdin) |
| `-k` | **k**ustomize | Áp dụng thư mục có `kustomization.yaml` |
| `-R` | **R**ecursive | Đệ quy cả thư mục con (không có thì **chỉ lấy tầng ngoài cùng**) |
| `--dry-run=client` | chạy khô phía client | Chỉ kiểm cú pháp trên máy |
| `--dry-run=server` | chạy khô phía server | ⭐ Gửi lên API server kiểm thật nhưng **không lưu** |
| `--grace-period` | thời gian ân hạn | Số giây cho pod tự tắt êm |
| `--force` | cưỡng bức | Xoá khỏi **etcd ngay**, không chờ kubelet xác nhận |
| `--replicas` | số bản | Số pod mong muốn |
| `--record` | ghi lại | ⚠️ **Đã bỏ** (deprecated) từ v1.18+ |

🛑 **`--grace-period=0 --force` — hiểu đúng kẻo hỏng dữ liệu:**

Bình thường xoá pod: K8s gửi `SIGTERM` → app có **30 giây** (mặc định) để đóng kết nối, ghi nốt dữ liệu, thoát êm → rồi mới `SIGKILL`.

Với `--grace-period=0 --force`: K8s **xoá bản ghi khỏi etcd ngay lập tức** và **không chờ kubelet báo đã dừng**. Hậu quả:

- App **không kịp** đóng transaction ⇒ dữ liệu hỏng dở dang.
- Với **StatefulSet** (DB, Kafka, etcd): K8s tưởng pod đã chết nên tạo pod mới **cùng danh tính** ⇒ 🔴 **hai tiến trình cùng ghi vào một volume ⇒ hỏng dữ liệu (split-brain)**.

⇒ Chỉ dùng khi node đã **thật sự chết** và pod kẹt `Terminating` mãi. **Không bao giờ** dùng như thói quen cho nhanh.

**`rollout` — nhóm lệnh quản lý quá trình cập nhật:**

```bash
kubectl rollout restart deploy/myapp -n prod
#               │       │            └─ namespace
#               │       └────────────── deploy/<tên> = dạng "loại/tên"
#               └────────────────────── restart KIỂU CUỐN CHIẾU: tạo pod mới,
#                                       chờ Ready rồi mới xoá pod cũ -> KHÔNG downtime
```

⚠️ **Vì sao `rollout restart` khác `delete pod`?**

| | `rollout restart deploy` | `delete pod` |
|---|---|---|
| Cách làm | Tạo pod mới **trước**, Ready rồi mới bỏ pod cũ | Giết pod **trước**, ReplicaSet mới tạo lại sau |
| Downtime | ❌ Không (nếu có nhiều replica) | ⚠️ Có khoảng trống |
| Có tạo revision mới? | ✅ Có ⇒ **rollback được** | ❌ Không |

**Bộ lệnh rollout:**

```bash
kubectl rollout status  deploy/myapp     # đang cuốn chiếu tới đâu (treo = có vấn đề)
kubectl rollout history deploy/myapp     # danh sách revision
kubectl rollout undo    deploy/myapp     # về revision LIỀN TRƯỚC
kubectl rollout undo    deploy/myapp --to-revision=3   # về đúng revision 3
kubectl rollout pause   deploy/myapp     # tạm dừng giữa chừng (canary thủ công)
kubectl rollout resume  deploy/myapp     # chạy tiếp
```

⚠️ `rollout status` **treo mãi không xong** = rollout **kẹt**, thường do pod mới không Ready (ImagePullBackOff, readinessProbe fail). Đừng chờ — mở terminal khác chạy `kubectl describe pod` để xem Events.

**`edit` — tiện nhưng nguy hiểm ở production:**

`kubectl edit deploy x` mở editor, lưu là áp dụng ngay. 🛑 Vấn đề: thay đổi **chỉ nằm trên cluster, không có trong Git** ⇒ lần deploy sau (ArgoCD/Helm) **ghi đè mất**, và không ai biết vì sao. Ở môi trường GitOps, `edit` chỉ dùng để **thử nghiệm khi chữa cháy**, sau đó phải sửa lại vào Git.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa các cờ debug/log — đặc biệt `--previous` và `port-forward`</b></summary>

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-f` | **f**ollow | Bám log realtime |
| `-c` | **c**ontainer | Chọn **container nào** trong pod (pod có nhiều container thì **bắt buộc**) |
| `--tail=N` | tail | N dòng cuối |
| `--since` | từ lúc | `--since=10m`, `--since=1h` |
| `--previous` / `-p` | previous = lần trước | ⭐ Log của **lần chạy TRƯỚC** (container đã chết) |
| `--all-containers` | mọi container | Log của tất cả container trong pod |
| `--timestamps` | dấu thời gian | Thêm mốc giờ vào mỗi dòng |
| `-it` | interactive + tty | Cặp cờ để có shell gõ được |
| `--` (hai gạch) | dấu ngăn | **Ngăn cách** cờ của kubectl với lệnh chạy trong pod |

⭐ **`--previous` — cờ quan trọng nhất khi pod `CrashLoopBackOff`:**

Pod crash ⇒ K8s **tạo container mới**. `kubectl logs <pod>` cho log của container **MỚI** — vừa khởi động, **chưa kịp lỗi**, nên trông như bình thường. Nguyên nhân crash nằm ở log của container **ĐÃ CHẾT**:

```bash
kubectl logs myapp-7d9f-x2k --previous
#                           └─ đọc log của lần chạy TRƯỚC ĐÓ = nơi có thông báo lỗi thật
```

⇒ Đây là lý do "xem log mà chẳng thấy lỗi gì" khi debug CrashLoopBackOff.

⚠️ `--previous` báo `previous terminated container not found` ⇒ container **chưa từng** restart lần nào (pod lỗi vì lý do khác, ví dụ chưa pull được image) ⇒ chuyển sang đọc `kubectl describe pod`.

⚠️ **Vì sao phải có `--` trong `kubectl exec`?**

```bash
kubectl exec -it mypod -- ls -la /app
#                      │  └─ từ đây trở đi là LỆNH CHẠY TRONG POD
#                      └──── dấu ngăn: báo kubectl "hết cờ của tao rồi"
```

Không có `--`, kubectl sẽ tưởng `-la` là cờ **của chính nó** ⇒ lỗi `unknown flag: -l`. Với lệnh không có cờ (`bash`, `sh`) thì bỏ `--` vẫn chạy — nên nhiều người không biết quy tắc này cho tới khi gặp lỗi.

**`port-forward` — công cụ debug hữu ích nhất mà không cần Ingress:**

```bash
kubectl port-forward svc/postgres 15432:5432 -n ai-hub
#                    │            │     │
#                    │            │     └─ port trên SERVICE (trong cluster)
#                    │            └─────── port trên MÁY BẠN (chọn số chưa ai chiếm)
#                    └──────────────────── forward tới service (cân bằng tải qua các pod)
#                                          ghi <tên-pod> thì tới ĐÚNG 1 pod đó
```

Sau lệnh này, ở máy bạn gõ `psql -h localhost -p 15432` là vào thẳng Postgres trong cluster, **không cần** mở Ingress hay NodePort.

| Đặc điểm | Ghi chú |
|---|---|
| Đường hầm đi qua | **API server** — nên chỉ cần quyền kubectl, không cần mở firewall |
| Sống bao lâu | Chỉ khi lệnh còn chạy. Ctrl+C là đứt |
| Pod chết | Đường hầm **đứt luôn**, phải chạy lại lệnh |
| Chỉ nghe ở | `127.0.0.1` — máy khác **không** vào được. Muốn cho máy khác: thêm `--address 0.0.0.0` (cân nhắc bảo mật) |

⇒ Rất hợp với môi trường **VDI bị hạn chế mạng**: không cần mở port ra ngoài, mọi thứ đi qua kênh kubectl sẵn có.

**`kubectl debug` — khi container không có shell (distroless):**

```bash
kubectl debug -it myapp-pod --image=busybox --target=app
#             │             │               └─ CHIA SẺ namespace tiến trình với container "app"
#             │             │                  -> nhìn thấy process & /proc của nó
#             │             └───────────────── image công cụ để chèn vào
#             └─────────────────────────────── tương tác
```

Nó thêm một **ephemeral container** (container tạm) vào pod **đang chạy** — không restart pod, không sửa manifest. Đây là cách duy nhất debug image distroless (không có `sh`, `ls`, `cat`).

⚠️ Ephemeral container **không xoá được** riêng lẻ; nó mất khi pod bị xoá/tạo lại.

**Sao chép file — nhớ chiều bằng dấu `:`** (giống `docker cp`):

```bash
kubectl cp ai-hub/mypod:/var/log/app.log ./app.log   # LẤY RA (nguồn có dấu :)
kubectl cp ./config.yaml ai-hub/mypod:/etc/config    # ĐẨY VÀO (đích có dấu :)
#          └─ dạng đầy đủ: <namespace>/<pod>:<đường-dẫn>
```

⚠️ `kubectl cp` **cần `tar` có sẵn trong container**. Image gọn không có `tar` ⇒ lỗi `tar: not found`. Cách thay thế:

```bash
kubectl exec mypod -- cat /var/log/app.log > ./app.log
#                     └─ đọc nội dung rồi shell ở MÁY BẠN hứng vào file
```

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa mục Namespace, ConfigMap & Secret</b></summary>

**Namespace là gì?** Là **vách ngăn logic** trong cùng một cluster — chia theo team/môi trường/ứng dụng. Tên resource chỉ cần **duy nhất trong một namespace**, nên `svc/api` ở `dev` và `svc/api` ở `prod` là **hai thứ khác nhau**, sống song song.

⚠️ 🔴 **`kubectl delete namespace <ns>` xoá SẠCH mọi thứ bên trong** — pod, service, secret, **PVC (mất data)** — và **không hỏi lại, không hoàn tác được**. Đây là một trong những lệnh nguy hiểm nhất của kubectl.

⚠️ Namespace kẹt ở trạng thái `Terminating` **mãi không xoá xong** = có resource còn **finalizer** (một cái móc yêu cầu dọn dẹp trước khi xoá) mà controller phụ trách đã chết. Chẩn đoán:

```bash
kubectl get ns <ns> -o json | jq '.spec.finalizers, .status.conditions'
#                             └─ xem còn vướng finalizer nào / điều kiện nào chưa xong
```

**ConfigMap vs Secret — khác nhau ít hơn bạn tưởng:**

| | ConfigMap | Secret |
|---|---|---|
| Chứa | Cấu hình **không nhạy cảm** | Mật khẩu, token, cert |
| Lưu trong etcd | Văn bản thường | **base64** — ⚠️ **KHÔNG phải mã hoá** |
| Ai đọc được | Người có quyền get | Người có quyền get (**cũng đọc được ngay**) |
| Giới hạn kích thước | 1MB | 1MB |

🛑 **base64 KHÔNG phải mã hoá** — nó chỉ là cách **mã hoá ký tự** để chứa dữ liệu nhị phân trong text, ai cũng giải ngược được bằng một lệnh. Secret của K8s **mặc định không an toàn** trước người có quyền đọc và trước người xem được ổ đĩa etcd.

⇒ **Cái gì thay thế/bổ sung?** (1) Bật **encryption at rest** cho etcd; (2) RBAC chặt để ít người `get secret`; (3) dùng **Vault** / **Sealed Secrets** / **SOPS** (xem mục 🔑 Secrets phía dưới) — đặc biệt bắt buộc nếu muốn commit secret vào Git.

**Bóc lệnh đọc secret — vì sao phải có `base64 -d`:**

```bash
kubectl get secret db-cred -o jsonpath='{.data.password}' | base64 -d
#                          │            │                   │      └─ decode: giải ngược về chữ gốc
#                          │            │                   └──────── công cụ base64 của Linux
#                          │            └────────────────────────── bóc đúng field password
#                          │                                        (giá trị đang ở dạng base64)
#                          └─────────────────────────────────────── in ra theo đường dẫn JSON
```

💡 Đọc **tất cả** field của secret một lần, không cần biết tên key:

```bash
kubectl get secret db-cred -o json | jq -r '.data | map_values(@base64d)'
#                                          │       └─ giải base64 cho MỌI giá trị
#                                          └───────── lấy phần data
```

**Tạo secret/configmap — ba nguồn dữ liệu:**

```bash
kubectl create secret generic db-cred --from-literal=user=admin --from-literal=pass='S3cr3t!'
#                     │               └─ gõ TRỰC TIẾP giá trị (⚠️ lọt vào lịch sử shell)
#                     └────────────────── generic = loại thường
#                                         (còn: docker-registry để pull image, tls cho cert)

kubectl create secret generic db-cred --from-file=./password.txt
#                                     └─ lấy từ FILE, TÊN FILE thành key, NỘI DUNG thành value
#                                        (an toàn hơn --from-literal: không lọt vào history)

kubectl create configmap app-cfg --from-env-file=./app.env
#                                └─ file mỗi dòng KEY=value -> thành nhiều key riêng biệt
```

⚠️ Phân biệt hai cờ dễ nhầm: **`--from-file=app.env`** tạo **MỘT** key tên `app.env` chứa **cả nội dung file**; **`--from-env-file=app.env`** tách thành **NHIỀU** key. Chọn nhầm ⇒ app đọc biến môi trường không thấy gì.

⚠️ **Sửa ConfigMap/Secret KHÔNG tự động khởi động lại pod.** Pod đã mount rồi thì vẫn giữ giá trị cũ (biến môi trường thì giữ **vĩnh viễn** cho tới khi pod tạo lại). Phải tự làm:

```bash
kubectl rollout restart deploy/myapp     # cuốn chiếu, không downtime
```

💡 Mẹo "tạo lại secret khi đã tồn tại" (vì `create` báo lỗi `AlreadyExists`):

```bash
kubectl create secret generic db-cred --from-literal=pass=new --dry-run=client -o yaml \
  | kubectl apply -f -
#   │                └─ đọc từ đầu vào chuẩn (stdin)
#   └─ create chỉ để SINH RA YAML (dry-run: không tạo thật), rồi apply mới là thứ ghi lên cluster
```

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa mục Tiện ích — `explain` và `run` rất đáng dùng</b></summary>

| Lệnh | Làm gì | Vì sao đáng nhớ |
|---|---|---|
| `kubectl api-resources` | Bảng **mọi loại resource** cluster hiểu được: tên, tên tắt, có thuộc namespace không, API version | Trả lời "CRD vừa cài tên gọi là gì để `get`?" |
| `kubectl explain <res>` | **Tài liệu ngay trong terminal** về từng field | ⭐ Không cần mở trình duyệt — cực hợp môi trường VDI chặn mạng |
| `kubectl label` | Gắn/gỡ nhãn | Nhãn là thứ Service/Selector dựa vào để tìm pod |
| `kubectl annotate` | Gắn chú thích | Chứa metadata cho công cụ khác đọc (ingress, cert-manager) |
| `kubectl top pods` | CPU/RAM **thực dùng** của pod | Cần metrics-server |
| `kubectl apply -k` | Áp dụng **Kustomize** | Bản thay thế Helm, không dùng template mà **vá** (patch) YAML |
| `kubectl run` | Tạo **một pod** ngay | Test nhanh |

⭐ **`explain` — công cụ tra cứu offline mạnh nhất của kubectl:**

```bash
kubectl explain pod.spec.containers.resources
#               └─ đi theo đường dẫn field, phân tách bằng dấu chấm

kubectl explain deploy --recursive | less
#                      │             └─ phân trang để cuộn (q để thoát)
#                      └─ in TOÀN BỘ cây field lồng nhau, không chỉ 1 tầng
```

⇒ Trả lời được "field này viết là `terminationGracePeriodSeconds` hay `gracePeriod`?" mà **không cần Internet**.

**`label` — gắn, sửa, và gỡ:**

```bash
kubectl label pod mypod env=prod           # gắn nhãn mới
kubectl label pod mypod env=dev --overwrite # SỬA nhãn đã có (thiếu --overwrite là BÁO LỖI)
kubectl label pod mypod env-                # GỠ nhãn (dấu TRỪ ở cuối tên = xoá)
#                            └─ quy ước "hậu tố dấu trừ" này dùng chung
#                               cho label, annotation và taint
```

⚠️ Nhãn không chỉ để cho đẹp: **Service tìm pod bằng nhãn**. Sửa nhãn `app=` của pod đang chạy ⇒ Service **mất pod đó khỏi endpoint** ⇒ **rơi traffic ngay lập tức**. Kiểm chứng ai đang được Service trỏ tới:

```bash
kubectl get endpoints <svc-name>    # rỗng = Service không tìm thấy pod nào khớp selector
```

⭐ **`kubectl run` — pod dùng một lần để test mạng nội bộ:**

```bash
kubectl run tmp --rm -it --image=busybox --restart=Never -- sh
#              │     │   │               │                 └─ lệnh chạy trong pod
#              │     │   │               └─ Never = tạo POD trần, KHÔNG tạo Deployment
#              │     │   │                  (thiếu cờ này ở bản cũ sẽ đẻ ra Deployment nằm lại)
#              │     │   └─ image công cụ (có ping, nslookup, wget)
#              │     └───── interactive + tty: có shell gõ được
#              └─────────── thoát là XOÁ pod, không để rác
```

Đây là cách chuẩn để kiểm tra **từ bên trong cluster**: DNS của service có phân giải được không, port có thông không — thứ mà đứng ngoài cluster không kiểm được.

💡 Image hợp cho việc debug mạng: `busybox` (nhẹ, có wget/nslookup) · `nicolaka/netshoot` (đầy đủ: dig, tcpdump, curl, mtr — nếu registry nội bộ có).

**`apply -k` (Kustomize) — khác Helm chỗ nào?**

| | Helm | Kustomize |
|---|---|---|
| Cách làm | **Template** có biến `{{ }}` | **Vá đè** lên YAML thuần |
| File gốc | Không chạy trực tiếp được (còn dấu ngoặc) | **Là YAML hợp lệ**, apply thẳng được |
| Cần cài thêm | Có (`helm`) | ❌ **Không** — đã tích hợp sẵn trong kubectl |

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa cách alias hoạt động + cách cài đặt</b></summary>

**`alias` là gì?** Là **tên gọi tắt** do shell thay thế **trước khi chạy**. Gõ `kgp`, shell đọc file cấu hình thấy `kgp='kubectl get pods'` rồi chạy đúng chuỗi đó. Nó **không phải** một chương trình mới.

```bash
alias kgp='kubectl get pods'
#     │   │└─ nội dung thật sẽ được thay vào
#     │   └── dấu nháy ĐƠN: giữ nguyên văn, shell KHÔNG diễn giải $ và {{ }} bên trong
#     └────── từ khoá khai báo
```

⚠️ **Vì sao phải là nháy đơn `'...'` chứ không phải nháy kép `"..."`?** Nháy kép làm shell **thay biến ngay lúc khai báo**. Với alias chứa `{{.Names}}` hay `$(...)` (như `dpsf` trong `docker.txt` của bạn), dùng nháy kép là hỏng.

**Đối số nối vào ĐUÔI** — đây là lý do alias vẫn linh hoạt:

```bash
kgp -n ai-hub          # shell mở ra thành: kubectl get pods -n ai-hub
kaf deploy.yaml        # -> kubectl apply -f deploy.yaml
```

🛑 **Giới hạn: alias chỉ chèn được vào ĐUÔI.** Cần đối số nằm ở **giữa** thì alias bó tay ⇒ phải dùng **function**:

```bash
# alias KHÔNG làm được việc này (cần <pod> đứng giữa, trước "-- bash"):
kshell() { kubectl exec -it "$1" -- bash; }
#      │                        │
#      │                        └─ "$1" = đối số thứ nhất bạn gõ vào
#      └─────────────────────────── khai báo hàm; gọi: kshell mypod
#  Bọc "$1" trong nháy kép để tên có dấu cách vẫn đúng
```

**Cài đặt — làm một lần:**

```bash
# 1. Thêm vào cuối file cấu hình shell
vi ~/.zshrc              # zsh (macOS mặc định — shell bạn đang dùng)
# vi ~/.bashrc           # bash (đa số server Linux)

# 2. Nạp lại để có hiệu lực NGAY, khỏi phải mở terminal mới
source ~/.zshrc
#  └─ đọc và chạy file trong shell HIỆN TẠI
#     (khác với `./file.sh` — cái đó chạy shell con, đặt biến xong là mất)

# 3. Kiểm tra
alias kgp                # in ra định nghĩa -> có nghĩa là đã nạp thành công
which kgp                # xác nhận nó là alias chứ không phải chương trình trùng tên
```

💡 Bạn đã có sẵn thói quen này: `alias ma='grep "^alias" ~/.zshrc'` trong `docker.txt` để liệt kê mọi alias đang có.

⭐ **Bật autocomplete cho `k`** — nếu chỉ đặt `alias k=kubectl` thì gõ `k get po<Tab>` **không gợi ý gì**, vì hệ thống hoàn thành lệnh gắn với chữ `kubectl`, không gắn với `k`:

```bash
source <(kubectl completion zsh)   # nạp bộ gợi ý của kubectl
complete -F __start_kubectl k      # gắn bộ gợi ý ĐÓ cho luôn chữ tắt `k`
#        │  │                 └─ tên tắt
#        │  └─ tên hàm gợi ý mà lệnh completion ở trên vừa tạo ra
#        └──── -F = dùng Function này để gợi ý
```

⚠️ Với **bash** thì đổi `zsh` → `bash` trong lệnh trên.

🛑 **Cẩn thận đặt alias trùng tên lệnh có sẵn.** Ví dụ đặt `alias k=kubectl` là an toàn, nhưng đặt `alias kill=...` hay `alias rm='rm -f'` thì **che mất lệnh gốc** và gây tai nạn về sau. Kiểm tra trước khi đặt: `which <tên>` — có kết quả nghĩa là **đã có gì đó mang tên đó**.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa TOÀN BỘ cờ grep (kể cả cờ đã biết)</b></summary>

**`grep` là gì?** Tên viết tắt của **g**lobally search a **r**egular **e**xpression and **p**rint — "tìm khắp nơi theo một mẫu rồi in ra". Mặc định nó in **cả dòng** nào **chứa** chuỗi bạn tìm.

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-i` | **i**gnore case | Bỏ qua hoa/thường. `ERROR`, `Error`, `error` đều khớp |
| `-v` | in**v**ert | **Đảo ngược** — in dòng **KHÔNG** khớp. Dùng để **loại bỏ nhiễu** |
| `-n` | line **n**umber | Kèm **số dòng** — để `vi +<số> file` nhảy thẳng tới đó |
| `-c` | **c**ount | Chỉ in **số lượng** dòng khớp, không in nội dung |
| `-r` | **r**ecursive | Tìm **đệ quy** trong thư mục và mọi thư mục con |
| `-w` | **w**ord | Khớp **nguyên từ**. `-w id` **không** khớp `uuid`, `idle` |
| `-l` | **l**ist files | Chỉ in **tên file** có khớp, không in nội dung |
| `-L` | ngược của `-l` | Chỉ in tên file **KHÔNG** khớp |
| `-o` | **o**nly matching | Chỉ in **phần khớp**, không in cả dòng |
| `-E` | **E**xtended regex | Bật regex mở rộng — để dùng được `|` (hoặc), `+`, `{n}`, `()` |
| `-F` | **F**ixed string | Coi mẫu là **chữ thường**, không phải regex — nhanh và an toàn khi chuỗi có `.` `*` |
| `-A N` | **A**fter | In thêm **N dòng SAU** dòng khớp |
| `-B N` | **B**efore | In thêm **N dòng TRƯỚC** |
| `-C N` | **C**ontext | In **N dòng cả trước lẫn sau** |
| `-q` | **q**uiet | Không in gì, chỉ trả **mã thoát** (0 = tìm thấy) — dùng trong `if` |
| `--line-buffered` | đệm theo dòng | ⭐ In **ngay** từng dòng thay vì gom đủ 4KB mới in |

⭐ **`-A`/`-B`/`-C` — vì sao là cờ quan trọng nhất khi debug?** Một dòng `Exception` **tự nó không nói gì**; nguyên nhân nằm ở **stack trace phía sau** nó:

```bash
grep -A 20 -i "exception" app.log
#     │  │  └─ không phân biệt hoa thường
#     │  └──── 20 dòng SAU mỗi dòng khớp = trọn stack trace
#     └─────── After
```

⭐ **`--line-buffered` — vì sao BẮT BUỘC khi theo dõi log realtime?**

Khi output đi vào **pipe** (không phải màn hình), grep chuyển sang **đệm khối**: gom đủ ~4KB mới in một lượt. Hậu quả: `tail -f app.log | grep error` **im lặng hàng phút**, rồi đổ ra một cục ⇒ tưởng "không có lỗi" trong khi lỗi đã xảy ra từ lâu.

```bash
tail -f app.log | grep --line-buffered -iE "error|fatal"
#                      └─ ép in NGAY mỗi khi có một dòng khớp
```

⇒ Nhớ quy tắc: **grep đứng sau dấu `|` mà cần thấy ngay ⇒ luôn thêm `--line-buffered`.**

**`-E` — bảng ký hiệu regex hay dùng:**

| Ký hiệu | Nghĩa | Ví dụ |
|---|---|---|
| `\|` | hoặc | `error\|fail` |
| `.` | một ký tự bất kỳ | |
| `*` | 0 lần trở lên | |
| `+` | 1 lần trở lên | `[0-9]+` = một dãy số |
| `{3}` | đúng 3 lần | `[0-9]{3}` = 3 chữ số |
| `^` | đầu dòng | `^ERROR` |
| `$` | cuối dòng | `timeout$` |
| `[0-9]` | một chữ số | |
| `()` | nhóm | `(GET\|POST) /api` |

⚠️ **Không có `-E` thì `|` là chữ `|` thường**, không phải "hoặc". Đây là lý do `grep "error|fail"` **không ra gì** — không phải vì log không có lỗi.

```bash
grep -iE "timeout|refused|reset|connection" app.log
#     ││  └─ nhờ có -E, dấu | mới mang nghĩa "hoặc"
#     │└─── Extended regex
#     └──── ignore case
```

**`2>&1` — mảnh ghép hay bị thiếu khi lọc log:**

```bash
docker logs app 2>&1 | grep -iE "error"
#               │└┴─ gộp luồng 2 (stderr) vào luồng 1 (stdout)
#               └─── vì `|` CHỈ chuyển luồng 1; không có 2>&1 thì
#                    thông báo lỗi (thường nằm ở luồng 2) KHÔNG qua được grep
```

⇒ Rất nhiều app in log lỗi ra **stderr**. Thiếu `2>&1` là grep **không bao giờ thấy** dòng lỗi, dù nó đang hiện đầy trên màn hình.

⚠️ **Bẫy `ps aux | grep nginx`**: kết quả **luôn có ít nhất một dòng** — chính tiến trình `grep` vừa chạy! Cách khử:

```bash
ps aux | grep -v grep | grep nginx    # cách 1: loại bỏ dòng chứa chữ "grep"
pgrep -af nginx                       # cách 2 (gọn hơn): -a hiện cả dòng lệnh, -f khớp toàn bộ
```

🛑 **`grep -riE "password|secret|token" .`** rất hữu ích để rà secret lộ, nhưng **kết quả in ra màn hình chứa đúng secret đó** ⇒ trên VDI phải chụp màn hình gửi đi thì **đừng chụp cả output này**. An toàn hơn — chỉ lấy **tên file**:

```bash
grep -rlniE "password|secret|token" . --exclude-dir={.git,node_modules}
#       │ │                            └─ bỏ qua thư mục nhiễu
#       │ └─ kèm số dòng
#       └─── -l: CHỈ in tên file, KHÔNG in nội dung -> không lộ giá trị secret
```

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa tail/head/less/awk/sed/sort/uniq/cut</b></summary>

| Lệnh & cờ | Viết tắt của | Làm gì |
|---|---|---|
| `tail -f` | **f**ollow | Bám đuôi file, dòng mới hiện ngay |
| `tail -F` | Follow (hoa) | ⭐ Như `-f` nhưng **theo tên file** — file bị xoay vòng (logrotate) vẫn bám tiếp |
| `tail -n 100` | **n**umber | 100 dòng cuối |
| `head -n 50` | | 50 dòng đầu |
| `wc -l` | **w**ord **c**ount, **l**ines | Đếm **số dòng** |
| `sort -n` | **n**umeric | Sắp theo **giá trị số** (không có `-n` thì `10` đứng trước `9` vì so theo chữ!) |
| `sort -r` | **r**everse | Đảo ngược (lớn → bé) |
| `sort -h` | **h**uman | Hiểu `1K`, `2M`, `3G` |
| `uniq -c` | **c**ount | Đếm số lần lặp — ⚠️ **chỉ gộp dòng GIỐNG NHAU LIỀN KỀ** |
| `sed -n '100,200p'` | **n**o auto-print + **p**rint | Chỉ in dòng 100→200 |
| `cut -d',' -f1,3` | **d**elimiter, **f**ields | Cắt cột 1 và 3, phân tách bằng dấu phẩy |

⭐ **`tail -f` vs `tail -F` — khác biệt quan trọng ở server:**

| | `-f` (thường) | `-F` (hoa) |
|---|---|---|
| Bám theo | **inode** (định danh file cụ thể) | **tên file** |
| Khi logrotate đổi tên file cũ và tạo file mới | 🛑 **Bám file CŨ mãi mãi, im lặng** — tưởng hết log | ✅ Tự chuyển sang file mới |

⇒ Theo dõi log qua đêm ở server có logrotate: **luôn dùng `-F`.**

⭐ **Bộ ba `sort | uniq -c | sort -nr` — mẫu đếm tần suất kinh điển:**

```bash
sort file.txt | uniq -c | sort -nr | head -20
#  │             │   │     │   ││     └─ chỉ lấy 20 dòng đầu = top 20
#  │             │   │     │   │└─ reverse: nhiều nhất lên đầu
#  │             │   │     │   └── numeric: so theo SỐ đếm
#  │             │   │     └────── sắp lại lần 2, lần này theo số lượng
#  │             │   └──────────── đếm số lần mỗi dòng lặp lại
#  │             └──────────────── gộp trùng
#  └────────────────────────────── sắp xếp để dòng GIỐNG NHAU nằm CẠNH NHAU
```

🛑 **Vì sao BẮT BUỘC `sort` trước `uniq`?** Vì `uniq` **chỉ so dòng liền kề**, không nhớ dòng đã gặp trước đó. Không sort trước, `A B A` cho ra **A xuất hiện 1 lần, hai lần riêng biệt** — số đếm **sai mà không báo lỗi**. Đây là lỗi âm thầm nguy hiểm nhất trong nhóm lệnh này.

**`awk` — hiểu một câu là đủ dùng 90%:** awk **tự tách mỗi dòng thành các cột** theo dấu cách, đặt tên `$1 $2 $3...`, `$0` là cả dòng.

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
#     │      └─ cột 1 của log Nginx = địa chỉ IP
#     └──────── {hành động} áp dụng cho MỌI dòng
#  => kết quả: TOP IP truy cập nhiều nhất (dùng để phát hiện tấn công/bot)
```

Với log Nginx dạng mặc định (combined), vị trí các cột:

| Cột | Nội dung | Lệnh lấy top |
|---|---|---|
| `$1` | IP client | `awk '{print $1}'` |
| `$7` | Đường dẫn URL | `awk '{print $7}'` |
| `$9` | **HTTP status code** | `awk '{print $9}'` |

```bash
awk '$9 >= 500 {print $0}' access.log     # chỉ in request bị lỗi 5xx
#    │          └─ in cả dòng
#    └──────────── ĐIỀU KIỆN đứng trước {} : chỉ xử lý dòng thoả điều kiện
```

**`sed -n '100,200p'` — bóc từng mảnh:**

```bash
sed -n '100,200p' app.log
#    │   │       └─ p = print (in ra)
#    │   └───────── khoảng dòng 100 đến 200
#    └───────────── -n = TẮT chế độ tự in mọi dòng
#                   (thiếu -n thì sed in HẾT file, cộng thêm 100-200 in LẦN HAI)
```

**`less` — phím tắt cần nhớ** (dùng khi file quá lớn để `cat`):

| Phím | Làm gì |
|---|---|
| `/chữ` | Tìm xuôi · `n` = kết quả tiếp, `N` = lùi lại |
| `?chữ` | Tìm ngược |
| `G` / `g` | Xuống cuối file / lên đầu |
| `F` | Chế độ bám như `tail -f` (Ctrl+C để dừng bám) |
| `q` | Thoát |

💡 `less` **không nạp cả file vào RAM** ⇒ mở được file log vài GB, trong khi `cat` file đó là treo terminal.

⚠️ **UUOC — "Useless Use Of Cat"**: `cat app.log | grep error | wc -l` chạy **3 tiến trình** cho việc mà một lệnh làm được:

```bash
grep -c error app.log     # gọn hơn, nhanh hơn, và -c đếm luôn
```

Không sai về kết quả, nhưng với file lớn thì lãng phí rõ rệt.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa mục Process & Tài nguyên — đọc load average cho đúng</b></summary>

| Lệnh & cờ | Viết tắt của | Làm gì |
|---|---|---|
| `ps aux` | **a**ll users + **u**ser format + **x** (cả tiến trình không gắn terminal) | Liệt kê **mọi** tiến trình. Đây là **cú pháp BSD**, không có dấu `-` |
| `ps -ef` | **e**very + **f**ull format | Tương đương `aux` nhưng là **cú pháp UNIX** (có dấu `-`) |
| `--sort=-%cpu` | sắp xếp | Dấu **`-`** phía trước = **giảm dần** (nhiều nhất lên đầu) |
| `pgrep -f` | **p**rocess grep, **f**ull | Tìm PID, khớp **toàn bộ dòng lệnh** chứ không chỉ tên chương trình |
| `kill <PID>` | | Gửi **SIGTERM (15)** — "xin hãy tự thoát êm" |
| `kill -9 <PID>` | | Gửi **SIGKILL (9)** — 🛑 chém ngay, không cho dọn dẹp |
| `pkill -f <tên>` | | Kill theo **tên/dòng lệnh** thay vì PID |
| `free -h` | **h**uman-readable | RAM/swap dạng dễ đọc (GB thay vì byte) |
| `vmstat 1` | virtual memory stat | Thống kê **mỗi 1 giây** |
| `nproc` | number of processors | Số CPU core — **con số cần để đọc load average** |

⭐ **`kill` vs `kill -9` — vì sao đừng quen tay dùng `-9`:**

| | `kill <PID>` (SIGTERM) | `kill -9 <PID>` (SIGKILL) |
|---|---|---|
| App có nhận được tín hiệu? | ✅ Có — chạy hàm dọn dẹp | ❌ **Kernel giết thẳng**, app không hay biết |
| App kịp làm gì | Đóng kết nối DB, ghi nốt buffer, xoá file tạm | **Không gì cả** |
| Hậu quả | Sạch sẽ | File tạm/lock còn lại · dữ liệu buffer **mất** · DB có thể phải phục hồi |

⇒ Luôn thử `kill` trước, chờ vài giây. `-9` chỉ dùng khi tiến trình **đã treo cứng** không phản hồi.

⚠️ **`kill -9` KHÔNG giết được tiến trình ở trạng thái `D`** (uninterruptible sleep — đang chờ I/O đĩa/NFS). Cột `STAT` trong `ps aux` hiện chữ `D` ⇒ không phải "lệnh kill hỏng", mà là tiến trình đang **kẹt trong kernel**. Cái gì xử lý được? Chỉ có **giải quyết nguồn I/O bị treo** (mount NFS chết, đĩa lỗi) hoặc **khởi động lại máy**.

**Bảng chữ `STAT` trong `ps aux` — dịch ra tiếng thường:**

| Chữ | Nghĩa |
|---|---|
| `R` | Đang chạy / sẵn sàng chạy |
| `S` | Ngủ, **đánh thức được** (bình thường — đa số tiến trình ở đây) |
| `D` | Ngủ **KHÔNG đánh thức được** — kẹt I/O. `kill -9` vô hiệu |
| `Z` | **Zombie** — đã chết nhưng cha chưa thu nhận |
| `T` | Bị dừng (SIGSTOP) |

⚠️ **Zombie không ăn RAM/CPU** — nó chỉ chiếm một dòng trong bảng tiến trình. Vài con zombie là **vô hại**; hàng nghìn con mới là dấu hiệu tiến trình cha viết sai. Không `kill` được zombie (nó đã chết rồi) — phải xử lý **tiến trình cha**.

⭐ **Đọc `uptime` / load average — con số bị hiểu sai nhiều nhất:**

```bash
uptime
# 14:32:01 up 42 days,  load average: 3.24, 2.98, 2.51
#                                     │     │     └─ trung bình 15 phút
#                                     │     └─────── trung bình 5 phút
#                                     └───────────── trung bình 1 phút
```

🛑 **Load average KHÔNG phải phần trăm CPU.** Nó là **số tiến trình đang chạy + đang chờ** (kể cả chờ I/O trên Linux). Phải so với **số core**:

```bash
nproc          # ví dụ ra: 4
```

| Load / số core | Nghĩa |
|---|---|
| `< 1.0` | Nhàn rỗi, còn dư sức |
| `= 1.0` | Dùng vừa hết công suất |
| `> 1.0` | **Có tiến trình phải xếp hàng chờ** |

⇒ Load `3.24` trên máy **4 core** = **bình thường** (3.24/4 = 81%). Cùng load đó trên máy **2 core** = **quá tải nặng**. Nhìn con số trần mà không chia cho `nproc` là kết luận sai.

⇒ So ba con số để biết **xu hướng**: `1 phút > 15 phút` = tải **đang tăng**; ngược lại = sự cố **đã qua đỉnh**.

**Bóc lệnh tìm thủ phạm ăn tài nguyên:**

```bash
ps aux --sort=-%mem | head -10
#  │    │      │       └─ 10 dòng đầu (kèm dòng tiêu đề)
#  │    │      └───────── DẤU TRỪ = giảm dần; bỏ dấu trừ là tăng dần (ít nhất lên đầu)
#  │    └──────────────── sắp theo cột %mem
#  └───────────────────── a=mọi user, u=định dạng có %CPU/%MEM, x=cả tiến trình nền
```

⚠️ **Cột `%MEM` và `RSS` trong `ps` đếm TRÙNG bộ nhớ dùng chung.** Cộng `%MEM` của mọi tiến trình rất hay ra **>100%** — không phải máy hỏng, mà vì thư viện dùng chung (libc...) bị tính cho từng tiến trình. Muốn số thật của một tiến trình: đọc `Pss` trong `/proc/<PID>/smaps_rollup`.

⚠️ **`free -h` — đừng hoảng khi thấy "hết RAM":**

```
              total   used   free   shared  buff/cache   available
Mem:           15Gi   9Gi    300Mi    1Gi       6Gi         5Gi
#                              │                 │            └─ ⭐ CON SỐ THẬT SỰ CÒN DÙNG ĐƯỢC
#                              │                 └─ Linux mượn RAM rỗi làm cache đĩa; SẼ TRẢ LẠI ngay khi app cần
#                              └─ "free" thấp là BÌNH THƯỜNG và TỐT — RAM rỗi là RAM lãng phí
```

⇒ Chỉ nhìn cột **`available`**. Cột `free` thấp **không** có nghĩa là thiếu RAM.

</details>

### Ổ đĩa (Disk)
```bash
df -h                                  # Dung lượng ổ đĩa còn trống
du -sh *                               # Kích thước từng thư mục/file
du -sh * | sort -rh | head             # Top thư mục nặng nhất
du -ah . | sort -rh | head -20         # Top file nặng nhất
lsof | grep deleted                    # File đã xóa nhưng process còn giữ (ngốn disk)
ncdu                                   # Duyệt dung lượng tương tác (nếu có cài)
```

<details>
<summary><b>Bấm xem: giải nghĩa mục Disk — và bẫy "df báo đầy mà du không thấy gì"</b></summary>

| Lệnh & cờ | Viết tắt của | Làm gì |
|---|---|---|
| `df -h` | **d**isk **f**ree, **h**uman | Dung lượng **theo từng phân vùng** (mount point) |
| `df -i` | **i**nodes | ⭐ Số **inode** còn lại — chỗ để **ĐẾM FILE**, khác với chỗ chứa dữ liệu |
| `du -sh` | **d**isk **u**sage, **s**ummarize, **h**uman | Tổng dung lượng, **không liệt kê từng file con** |
| `du -ah` | **a**ll | Liệt kê **cả file** lẻ, không chỉ thư mục |
| `sort -rh` | **r**everse + **h**uman | Sắp giảm dần, hiểu được `1.2G`, `500M` |
| `lsof` | **l**i**s**t **o**pen **f**iles | Liệt kê file **đang được tiến trình mở** |
| `ncdu` | NCurses disk usage | Duyệt cây thư mục **tương tác** (mũi tên + Enter) |

⭐ **`df` vs `du` — hai câu trả lời khác nhau, và đây là gốc của sự cố kinh điển:**

| | `df` | `du` |
|---|---|---|
| Hỏi ai | **Hệ thống file** (kernel) | **Duyệt từng file** rồi cộng lại |
| Đếm cả file đã xoá mà process còn giữ? | ✅ **CÓ** | ❌ **KHÔNG** (file không còn tên để duyệt) |

🛑 **Sự cố kinh điển: `df -h` báo 100% đầy, nhưng `du -sh /*` cộng lại chỉ vài GB.**

**Vì sao?** Trên Linux, xoá file (`rm`) chỉ gỡ **cái tên**. Nếu còn tiến trình **đang mở** file đó, dữ liệu **vẫn nằm trên đĩa** cho tới khi tiến trình đóng file hoặc bị restart. `du` duyệt theo tên nên **không thấy**, `df` hỏi kernel nên **vẫn tính**.

Ca hay gặp nhất: ai đó `rm` file log 50GB trong khi ứng dụng vẫn đang ghi vào nó → **disk không hề được giải phóng**.

**Cách chẩn đoán và xử lý:**

```bash
lsof | grep deleted
#  │     └─ lọc các dòng có chữ (deleted): file đã bị xoá tên nhưng CHƯA được giải phóng
#  └─────── liệt kê mọi file đang mở

lsof -nP +L1
#     ││  └─ +L1 = chỉ hiện file có SỐ LIÊN KẾT < 1, tức đã bị xoá tên  (cách gọn hơn)
#     │└─── -P: không đổi số port thành tên dịch vụ (nhanh hơn)
#     └──── -n: không tra ngược DNS (nhanh hơn)
```

**Cách giải phóng — hai lựa chọn:**

```bash
# Cách 1 (an toàn nhất): restart tiến trình đang giữ file
systemctl restart <service>

# Cách 2 (không cần restart): "làm rỗng" file qua đường /proc
: > /proc/<PID>/fd/<FD>
# │ │  └─ FD = số file descriptor lấy được từ cột FD của lệnh lsof
# │ └──── ghi đè bằng nội dung RỖNG -> đĩa được trả lại ngay
# └────── dấu hai chấm = lệnh "không làm gì", chỉ để tạo ra thao tác ghi
```

⭐ **`df -i` — thủ phạm thứ hai: hết inode, không phải hết dung lượng.**

**Inode là gì?** Mỗi file cần **một inode** để lưu metadata (chủ sở hữu, quyền, vị trí trên đĩa). Số inode được **định sẵn khi format** và **không tăng được**. Hết inode ⇒ **không tạo được file mới**, dù `df -h` còn trống 80%.

```bash
df -i
#  └─ cột IUse% = phần trăm inode ĐÃ DÙNG
```

Triệu chứng: `No space left on device` nhưng `df -h` báo còn nhiều chỗ. Nguyên nhân thường là **hàng triệu file nhỏ** (session PHP, cache, mail queue). Tìm thư mục nhiều file nhất:

```bash
for d in /var/*; do echo "$(find "$d" -type f 2>/dev/null | wc -l)  $d"; done | sort -rn | head
#         │           │      │          │       │              │
#         │           │      │          │       │              └─ đếm SỐ FILE (không phải dung lượng)
#         │           │      │          │       └─ giấu lỗi "permission denied"
#         │           │      │          └─ chỉ đếm file thường
#         │           │      └─ duyệt đệ quy thư mục đó
#         │           └─ in "số lượng  đường dẫn"
#         └─ lặp qua từng thư mục con của /var
```

**Quy trình tìm chỗ ăn đĩa — chạy theo thứ tự:**

```bash
df -h                                # 1. phân vùng NÀO đầy? (đừng đoán, phải xem)
df -i                                # 2. đầy dung lượng hay đầy INODE?
du -sh /* 2>/dev/null | sort -rh | head    # 3. thư mục gốc nào nặng nhất
#       │              └─ sắp giảm dần, hiểu đơn vị G/M
#       └─ bỏ qua lỗi không có quyền đọc
du -ah /var/log | sort -rh | head -20      # 4. đi sâu vào thư mục nghi ngờ, tìm FILE cụ thể
lsof | grep deleted                        # 5. nếu du không khớp df -> tìm file ma
```

⚠️ **`du` trên thư mục lớn rất chậm** (phải duyệt hết cây thư mục) và **làm tăng tải I/O**. Trên production giờ cao điểm, cân nhắc dùng `ncdu` (có tiến trình hiển thị, dừng được) hoặc chạy lúc vắng.

⚠️ `du -sh *` **bỏ sót file ẩn** (bắt đầu bằng dấu chấm) vì shell mở rộng `*` không lấy file ẩn. Dùng `du -sh .[!.]* *` hoặc đơn giản là `du -sh .` cho cả thư mục hiện tại.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa mục Network — ss, lsof, curl, dig</b></summary>

| Lệnh & cờ | Viết tắt của | Làm gì |
|---|---|---|
| `ss -tulpn` | **s**ocket **s**tatistics | Liệt kê socket. Bộ cờ này là **cụm hay dùng nhất** — bóc kỹ ở dưới |
| `netstat -tulpn` | network statistics | Bản **cũ** của `ss`, chậm hơn, nhiều bản Linux mới **không cài sẵn** |
| `lsof -i :8080` | list open files, **i**nternet | Tiến trình nào đang **chiếm port 8080** |
| `curl -I` | **I** = head | Chỉ lấy **header**, không tải nội dung |
| `curl -v` | **v**erbose | Hiện chi tiết: DNS, bắt tay TLS, header gửi & nhận |
| `curl -w` | **w**rite-out | In thêm thông tin sau khi xong (mã HTTP, thời gian) |
| `nc -zv` | netcat: **z**ero-I/O + **v**erbose | Chỉ **thử mở** kết nối rồi đóng ngay, báo mở/đóng |
| `dig +short` | | Chỉ in kết quả IP, bỏ phần trình bày dài |
| `traceroute` | | Truy vết từng chặng gói tin đi qua |

⭐ **Bóc kỹ `ss -tulpn`** — 5 chữ cái dính liền nhau, mỗi chữ một nghĩa:

```bash
ss -tulpn
#   │││││
#   ││││└─ n = numeric: in SỐ port (8080), không tra tên dịch vụ (http-alt). NHANH hơn nhiều
#   │││└── p = process: hiện TIẾN TRÌNH nào đang giữ port ⚠️ cần quyền root mới thấy đủ
#   ││└─── l = listening: chỉ socket đang LẮNG NGHE (bỏ qua kết nối đang hoạt động)
#   │└──── u = UDP
#   └───── t = TCP
```

⚠️ **Không có `sudo` thì cột PROCESS bị TRỐNG** với tiến trình của user khác — dễ tưởng "không ai giữ port". Luôn `sudo ss -tulpn` khi cần biết thủ phạm.

**Đọc kết quả — cột `Local Address:Port` cho biết dịch vụ nghe ở đâu:**

| Giá trị | Nghĩa | Máy khác kết nối được? |
|---|---|---|
| `0.0.0.0:8080` | Nghe trên **mọi** card mạng | ✅ Được |
| `127.0.0.1:8080` | **Chỉ** localhost | ❌ **Không** — dù firewall đã mở |
| `[::]:8080` | Mọi địa chỉ IPv6 | ✅ (qua IPv6) |

🛑 Đây là nguyên nhân cực kỳ phổ biến của "mở port firewall rồi mà vẫn không vào được": dịch vụ **chỉ nghe trên `127.0.0.1`**. Firewall không liên quan — phải sửa **cấu hình ứng dụng** (`bind_address`, `listen`, `--host 0.0.0.0`).

⭐ **`curl` đo hiệu năng — tách được "chậm ở đâu":**

```bash
curl -w "DNS:%{time_namelookup}s Connect:%{time_connect}s TLS:%{time_appconnect}s Total:%{time_total}s Code:%{http_code}\n" \
     -o /dev/null -s https://api.company.vn/health
#    │            │
#    │            └─ s = silent: tắt thanh tiến trình (nếu không nó lẫn vào output)
#    └─────────────── o = output: vứt NỘI DUNG vào thùng rác, chỉ giữ phần đo
```

Cách đọc kết quả — biết **chặng nào** là nút thắt:

| Chỉ số | Cao bất thường nghĩa là |
|---|---|
| `time_namelookup` | **DNS chậm** — đổi resolver, kiểm tra `/etc/resolv.conf` |
| `time_connect` | **Bắt tay TCP chậm** — mạng/độ trễ đường truyền |
| `time_appconnect` | **Bắt tay TLS chậm** — cert chain dài, OCSP |
| `time_total` cao mà các mục trên thấp | **Ứng dụng phía server xử lý chậm** |

**`nc -zv` — bóc mảnh:**

```bash
nc -zv db.internal 5432
#   ││  └─ đích: host + port
#   │└─── v = verbose: IN RA kết quả (không có -v thì lệnh IM LẶNG, chỉ có mã thoát)
#   └──── z = zero-I/O: chỉ kiểm tra mở được không rồi ĐÓNG NGAY, không gửi dữ liệu
```

⚠️ `nc` có nhiều **phiên bản khác nhau** (OpenBSD, GNU, busybox) với bộ cờ **không giống nhau** — `-z` không phải bản nào cũng có. Cách thay thế **luôn chạy được** nếu có bash (không cần cài gì):

```bash
timeout 5 bash -c "</dev/tcp/db.internal/5432" && echo MỞ || echo ĐÓNG
#       │       │   └─ /dev/tcp là TÍNH NĂNG DỰNG SẴN của bash, không phải file thật
#       │       └───── -c: chạy chuỗi lệnh này
#       └───────────── dừng sau 5 giây, tránh treo khi gói tin bị DROP im lặng
```

⚠️ **Phân biệt hai kiểu "không kết nối được"** — chẩn đoán khác nhau hoàn toàn:

| Hiện tượng | Nghĩa | Thủ phạm |
|---|---|---|
| **Trả lời ngay** `Connection refused` | Gói tin **tới nơi**, nhưng **không có ai nghe** port đó | Dịch vụ **chưa chạy** / nghe sai địa chỉ |
| **Treo rồi timeout** | Gói tin **bị nuốt**, không ai trả lời | **Firewall DROP** / sai route / sai security group |

⇒ `refused` = tin tốt (đường mạng thông). `timeout` = vấn đề nằm ở tầng mạng/firewall.

**`ping` không thông ≠ máy chết:** rất nhiều hệ thống **chặn ICMP** (giao thức ping) nhưng vẫn phục vụ TCP bình thường. Ping fail thì **phải kiểm chứng lại bằng TCP** (`nc -zv`, `curl`) trước khi kết luận.

**`dig` — bóc mảnh và mẹo so sánh:**

```bash
dig +short api.company.vn          # chỉ in IP, gọn cho script
dig api.company.vn +trace          # đi từ ROOT server xuống, xem tắc ở tầng nào
dig @8.8.8.8 api.company.vn        # hỏi THẲNG một DNS khác
#   └─ dấu @ = chỉ định server DNS để hỏi
```

⭐ **Mẹo chẩn đoán**: nếu `dig @8.8.8.8` **ra IP** mà `dig` (dùng DNS mặc định) **không ra** ⇒ vấn đề nằm ở **DNS nội bộ**, không phải ở domain. Đây là cách khoanh vùng nhanh nhất.

⚠️ `nslookup` là công cụ **cũ**, đôi khi hiện kết quả khác `dig` vì nó có logic tìm kiếm riêng. Khi hai lệnh cho kết quả khác nhau, **tin `dig`** — nó hiển thị đúng những gì DNS server trả về.

</details>

### Check IP/PORT từ TRONG Docker container (khi thiếu tool)
> Container thường bị cắt gọt (không có ping/nc/telnet). Đây là các cách check kết nối tới `IP:PORT`,
> ưu tiên cách không cần cài tool ngoài.
```bash
# 1. curl check TCP qua scheme telnet:// (curl thường có sẵn)
curl -v telnet://<IP>:<PORT>           # Kết nối được = "Connected to..."; fail = "Connection refused/timed out"

# 2. Check TCP KHÔNG cần tool ngoài (dùng /dev/tcp của bash - tiện nhất trong container)
timeout 5 bash -c "echo > /dev/tcp/<IP>/<PORT>" && echo OK || echo FAIL   # 5 = số giây chờ tối đa
timeout 3 bash -c "cat < /dev/tcp/<IP>/<PORT>"   # mở & đọc thử (tùy dịch vụ)

# 3. Check cả TLS/SSL (bắt tay chứng chỉ) — khi service dùng HTTPS/TLS
openssl s_client -connect <IP>:<PORT>              # Xem handshake + cert
openssl s_client -connect <IP>:<PORT> -servername <domain>   # kèm SNI (vhost TLS)
echo | openssl s_client -connect <IP>:<PORT> 2>/dev/null | openssl x509 -noout -dates   # ngày hết hạn cert

# Bổ sung nếu container CÓ sẵn tool:
nc -zv <IP> <PORT>                     # netcat (nếu có)
curl -v http://<IP>:<PORT>             # nếu là HTTP
getent hosts <hostname>                # resolve DNS không cần nslookup/dig
```

<details>
<summary><b>Bấm xem: giải nghĩa các cờ + vì sao `/dev/tcp` chạy được khi không cài gì</b></summary>

**Tiền đề — vì sao container thiếu tool?** Image production được **cắt gọt tối đa** để nhẹ và giảm bề mặt tấn công. `alpine` ~5MB, `distroless` thậm chí **không có shell**. Nên `ping`, `telnet`, `netstat`, `dig` thường **không có sẵn** — đó là **chủ ý thiết kế**, không phải image bị lỗi.

⚠️ Và trong môi trường **VDI air-gapped**, `apk add` / `apt install` **cũng không chạy được** vì không ra được Internet ⇒ bắt buộc phải biết cách dùng **thứ có sẵn**.

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-v` (curl) | **v**erbose | Hiện chi tiết từng bước kết nối |
| `timeout 5` | | Giết lệnh sau **5 giây** — tránh treo vô hạn |
| `-c` (bash) | **c**ommand | Chạy chuỗi lệnh truyền vào |
| `-z` (nc) | **z**ero-I/O | Chỉ thử mở port, không gửi data |
| `-servername` | SNI | Báo **tên miền** muốn truy cập khi một IP phục vụ nhiều domain TLS |
| `-noout` | no output | **Không** in cert dạng mã hoá thô, chỉ in phần đã bóc tách |
| `-dates` | | Chỉ in ngày hiệu lực & hết hạn |

⭐ **`/dev/tcp` — vì sao chạy được mà không cài gì?**

`/dev/tcp/<host>/<port>` **KHÔNG phải file thật** trên đĩa (`ls /dev/tcp` sẽ báo không tồn tại). Nó là **tính năng dựng sẵn bên trong bash**: khi bash thấy đường dẫn có dạng này, nó **tự mở một kết nối TCP** thay vì mở file. Vì nằm trong chính bash nên **không cần cài thêm bất cứ thứ gì**.

```bash
timeout 5 bash -c "echo > /dev/tcp/10.0.0.5/5432" && echo OK || echo FAIL
#       │  │    │   │     └─ bash biến đường dẫn này thành KẾT NỐI TCP
#       │  │    │   └─────── ghi một dòng rỗng vào đó = ép bash thật sự mở kết nối
#       │  │    └─────────── chuỗi lệnh cho bash chạy
#       │  └──────────────── BẮT BUỘC là bash (sh/dash KHÔNG có tính năng này)
#       └─────────────────── bỏ cuộc sau 5 giây
#                            && = lệnh trước THÀNH CÔNG thì chạy tiếp
#                            || = lệnh trước THẤT BẠI thì chạy cái này
```

🛑 **Giới hạn phải biết**: `sh` (dash) và `busybox sh` — vốn là shell mặc định của Alpine — **KHÔNG có `/dev/tcp`**. Gõ vào sẽ báo `No such file or directory`, dễ hiểu nhầm là "port đóng".

⇒ **Cái gì thay thế khi chỉ có `sh`?** Theo thứ tự ưu tiên:

```bash
curl -v telnet://10.0.0.5:5432        # 1. curl gần như luôn có sẵn
nc -zv 10.0.0.5 5432                  # 2. nếu image có netcat
wget -S -O- http://10.0.0.5:8080      # 3. Alpine luôn có wget (busybox)
#     │  └─ in nội dung ra màn hình thay vì lưu file
#     └──── in header phản hồi của server
bash -c "..."                          # 4. thử xem có bash không: `which bash`
```

**Cách đọc kết quả `curl -v telnet://`:**

| Output | Nghĩa | Thủ phạm |
|---|---|---|
| `Connected to 10.0.0.5` | ✅ **Thông** | — |
| `Connection refused` | Gói tin **tới nơi**, không ai nghe port đó | Service chưa chạy / nghe sai địa chỉ |
| Treo rồi `Connection timed out` | Gói tin **bị nuốt** | Firewall DROP · sai route · sai Security Group |
| `Could not resolve host` | **Lỗi DNS**, chưa tới bước kết nối | Xem mục Debug DNS bên dưới |

⇒ Bốn dòng này phân biệt được **bốn tầng sự cố khác nhau** — đừng gộp chung thành "không kết nối được".

**Kiểm tra TLS/HTTPS — bóc mảnh:**

```bash
echo | openssl s_client -connect api.company.vn:443 -servername api.company.vn 2>/dev/null \
     | openssl x509 -noout -dates -subject -issuer
# │    │       │                   │                  │           │      │
# │    │       │                   │                  │           │      └─ ai cấp cert
# │    │       │                   │                  │           └──────── cert cấp cho domain nào
# │    │       │                   │                  └─ vứt thông báo phụ đi cho gọn
# │    │       │                   └─ SNI: báo tên miền (một IP phục vụ nhiều domain thì BẮT BUỘC)
# │    │       └─ s_client = đóng vai TRÌNH DUYỆT, thực hiện bắt tay TLS
# │    └───────── bộ công cụ OpenSSL
# └────────────── gửi một dòng rỗng để openssl ĐÓNG kết nối, không treo chờ nhập
```

⚠️ **Thiếu `echo |` ở đầu, lệnh sẽ TREO** — vì `s_client` bắt tay xong sẽ mở phiên tương tác chờ bạn gõ. Đây là lý do lệnh "chạy mãi không xong".

⚠️ **Thiếu `-servername`** trên server dùng nhiều vhost TLS ⇒ nhận về **cert mặc định của server khác** ⇒ tưởng cert sai/hết hạn trong khi cert thật vẫn ổn.

**Kiểm tra nhanh cert sắp hết hạn** (hợp cho script cảnh báo):

```bash
echo | openssl s_client -connect api.company.vn:443 -servername api.company.vn 2>/dev/null \
  | openssl x509 -noout -checkend 604800 && echo "CÒN HẠN >7 ngày" || echo "SẮP HẾT HẠN!"
#                        └─ checkend N: trả về THÀNH CÔNG nếu cert còn hạn ít nhất N giây
#                           604800 giây = 7 ngày
```

</details>
> Ghi chú: `/dev/tcp` là tính năng của **bash** (không có trong `sh`/dash). Nếu container chỉ có `sh`,
> dùng `curl telnet://` hoặc `nc`. `timeout` giúp không bị treo khi port bị chặn (drop gói).

### Check UDP từ trong container
> UDP không có "handshake" như TCP nên khó xác nhận chắc chắn — không phản hồi có thể là "thông nhưng
> service im lặng" HOẶC "bị chặn". Cần suy luận theo ngữ cảnh.
```bash
# nc cho UDP (nếu có netcat)
nc -zvu <IP> <PORT>                    # -u = UDP; "open" hoặc "succeeded" là gửi được
nc -u <IP> <PORT>                      # mở phiên UDP, gõ data thử (Ctrl+C thoát)
echo "ping" | nc -u -w3 <IP> <PORT>    # gửi 1 gói, chờ 3s phản hồi

# /dev/udp của bash (không cần tool ngoài) — gửi được gói là "đường thông"
timeout 3 bash -c "echo -n 'test' > /dev/udp/<IP>/<PORT>" && echo SENT || echo FAIL

# UDP đặc thù dịch vụ (cách chắc chắn nhất: dùng đúng client của dịch vụ)
dig @<IP> -p <PORT> example.com        # test DNS server (UDP/53)
nc -zvu <IP> 123                       # NTP thường ở UDP/123
nmap -sU -p <PORT> <IP>                # quét UDP (chính xác hơn, cần cài nmap + quyền)
```

<details>
<summary><b>Bấm xem: vì sao UDP khó kiểm tra hơn TCP — và cách chắc chắn</b></summary>

⭐ **Tiền đề — vì sao UDP không kiểm được như TCP?**

| | TCP | UDP |
|---|---|---|
| Có bắt tay không? | ✅ **Có** (SYN → SYN-ACK → ACK) | ❌ **Không** — gửi xong là xong |
| Bên nhận có xác nhận? | ✅ Có | ❌ **Không** |
| Nên kiểm tra được? | ✅ Kết nối được = thông | ⚠️ **Gửi thành công KHÔNG chứng minh điều gì** |

🛑 Với UDP, `echo > /dev/udp/...` **luôn báo thành công** kể cả khi **không có ai ở đầu kia** — vì hệ điều hành chỉ ghi nhận "đã đẩy gói đi", không chờ ai trả lời. Nên **"gửi được" ≠ "thông"**.

⇒ **Im lặng có ba nghĩa khác nhau**, và không có cách nào phân biệt chỉ bằng `nc`:

1. Đường **thông**, service nhận được nhưng **giao thức không quy định trả lời** (ví dụ gửi rác cho DNS).
2. Đường **thông**, nhưng **service không chạy** ⇒ đáng lẽ có `ICMP port unreachable`, mà ICMP thường **bị firewall chặn** ⇒ vẫn im lặng.
3. Đường **bị chặn** hoàn toàn.

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-u` (nc) | **u**dp | Chuyển sang chế độ UDP (mặc định của `nc` là TCP) |
| `-z` (nc) | **z**ero-I/O | Chỉ thử, không gửi dữ liệu thật |
| `-w3` (nc) | **w**ait | Chờ tối đa 3 giây rồi bỏ |
| `-sU` (nmap) | **s**can **U**DP | Quét kiểu UDP |
| `-p` (dig) | **p**ort | Hỏi DNS ở port khác 53 |

⭐ **Cách đáng tin cậy duy nhất: dùng đúng CLIENT của giao thức đó.** Vì client biết **chờ đúng loại phản hồi**, thay vì đoán mò:

```bash
dig @10.0.0.53 -p 53 example.com +short +time=3
#   │          │                  │      └─ chờ tối đa 3 giây mỗi lần thử
#   │          │                  └──────── chỉ in kết quả cho gọn
#   │          └─ port (mặc định 53, ghi rõ khi dùng port khác)
#   └──────────── hỏi THẲNG server DNS này
#  => CÓ kết quả IP = UDP/53 chắc chắn THÔNG cả hai chiều
#     (vì đã đi được lượt đi VÀ nhận được lượt về)
```

| Dịch vụ UDP | Client kiểm tra đúng cách |
|---|---|
| DNS (53) | `dig @<IP> example.com` |
| NTP (123) | `ntpdate -q <IP>` (`-q` = query, chỉ hỏi **không chỉnh giờ máy**) |
| SNMP (161) | `snmpget -v2c -c public <IP> sysUpTime.0` |
| Syslog (514) | `logger -n <IP> -P 514 "test"` rồi **kiểm tra ở phía server nhận** |

⭐ **Nguyên tắc vàng cho UDP: kiểm chứng ở PHÍA NHẬN.** Gửi một gói rồi vào server đích xem log/tcpdump có thấy không — đây là cách duy nhất trả lời dứt khoát:

```bash
# Trên máy ĐÍCH (cần quyền root):
tcpdump -i any -n udp port 514
#           │   │  └─ chỉ bắt UDP port 514
#           │   └──── không tra DNS (nhanh, không nhiễu)
#           └──────── mọi card mạng
# Rồi từ container gửi thử -> thấy gói hiện ra = ĐƯỜNG THÔNG (chắc chắn 100%)
```

**`nmap -sU` — chính xác hơn nhưng có điều kiện:**

```bash
sudo nmap -sU -p 53,123 --reason 10.0.0.53
#    │     │   │         └─ GIẢI THÍCH vì sao kết luận như vậy (rất đáng thêm)
#    │     │   └─ danh sách port cần quét
#    │     └───── quét UDP
#    └─────────── ⚠️ BẮT BUỘC quyền root (cần tạo gói tin thô)
```

Cách đọc kết quả — 4 trạng thái, hai trong số đó **không phải kết luận chắc chắn**:

| Trạng thái | Nghĩa |
|---|---|
| `open` | Có phản hồi đúng ⇒ **chắc chắn thông** |
| `closed` | Nhận `ICMP port unreachable` ⇒ tới nơi nhưng **không ai nghe** |
| `open\|filtered` | ⚠️ **Không xác định được** — im lặng, có thể mở mà không trả lời, hoặc bị chặn |
| `filtered` | Có dấu hiệu bị firewall chặn |

⚠️ Quét UDP **rất chậm** (nmap phải chờ timeout từng port) và có thể bị hệ thống giám sát bảo mật của công ty ghi nhận là **hành vi quét mạng**. Trong môi trường doanh nghiệp, nên xin phép hoặc chỉ quét đúng vài port cần thiết.

</details>
> Với UDP, cách đáng tin nhất là **dùng client thật của giao thức** (dig cho DNS, ntpdate cho NTP...)
> vì nó chờ đúng loại phản hồi, thay vì đoán qua nc.

### Debug DNS trong container
> Trong container, "không kết nối được bằng tên nhưng bằng IP thì được" = lỗi DNS. Kiểm tra theo thứ tự:
```bash
# 1. Container đang dùng DNS server nào?
cat /etc/resolv.conf                   # nameserver, search domain, ndots
                                       # (Docker mặc định 127.0.0.11; K8s là ClusterIP của CoreDNS)

# 2. Resolve tên KHÔNG cần tool ngoài (getent luôn có trên image glibc)
getent hosts <hostname>                # ra IP = DNS OK; rỗng = resolve fail
getent hosts google.com                # test resolve ra ngoài

# 3. Nếu có dig/nslookup (bind-tools / dnsutils)
dig <hostname>                         # chi tiết, xem ANSWER + server trả lời
dig +short <hostname>                  # chỉ lấy IP
nslookup <hostname>                    # thay thế dig
dig @8.8.8.8 <hostname>                # hỏi thẳng DNS khác -> so sánh (loại trừ DNS nội bộ lỗi)

# 4. Phân biệt lỗi DNS vs lỗi mạng
getent hosts <hostname> || echo "DNS FAIL"        # tên -> IP?
curl -v http://<IP-truc-tiep>/         # nếu IP thông mà tên fail -> chắc chắn lỗi DNS

# 5. /etc/hosts (đôi khi bị override tên)
cat /etc/hosts

# Kubernetes: kiểm tra CoreDNS nếu resolve service fail
# kubectl get pods -n kube-system -l k8s-app=kube-dns
# kubectl exec <pod> -- nslookup <service>.<namespace>.svc.cluster.local
```

<details>
<summary><b>Bấm xem: giải nghĩa resolv.conf, ndots, và thứ tự chẩn đoán DNS</b></summary>

⭐ **Tiền đề — dấu hiệu nhận biết lỗi DNS trong một câu:** *kết nối bằng **IP thì được**, bằng **tên thì không*** ⇒ gần như chắc chắn là DNS, không phải mạng, không phải firewall.

**`/etc/resolv.conf` — file quyết định mọi việc phân giải tên:**

```
nameserver 10.96.0.10                 <- hỏi DNS server nào (K8s: ClusterIP của CoreDNS)
search ai-hub.svc.cluster.local svc.cluster.local cluster.local
#      └─ danh sách hậu tố sẽ THỬ GHÉP THÊM vào tên chưa đầy đủ
options ndots:5                       <- ⭐ con số gây nhiều hiểu nhầm nhất, giải thích bên dưới
```

| Thành phần | Nghĩa |
|---|---|
| `nameserver` | Địa chỉ DNS server. Docker mặc định `127.0.0.11`; K8s là ClusterIP của CoreDNS |
| `search` | Các hậu tố tự động ghép thêm ⇒ nhờ nó mà gõ `postgres` là ra `postgres.ai-hub.svc.cluster.local` |
| `ndots:N` | Tên có **ÍT HƠN N dấu chấm** thì **thử ghép `search` TRƯỚC**, mới thử tên nguyên bản sau |

⭐ **`ndots:5` — vì sao gọi ra ngoài Internet trong K8s lại chậm?**

`api.company.vn` có **2 dấu chấm** < 5 ⇒ Kubernetes coi đây là tên **chưa đầy đủ** nên thử ghép hậu tố **trước**:

```
1. api.company.vn.ai-hub.svc.cluster.local   -> NXDOMAIN (không có)
2. api.company.vn.svc.cluster.local          -> NXDOMAIN
3. api.company.vn.cluster.local              -> NXDOMAIN
4. api.company.vn                            -> ✅ mới ra kết quả ĐÚNG
```

⇒ **4 lượt truy vấn** cho một tên miền, mỗi lượt còn nhân đôi vì hỏi cả IPv4 (A) và IPv6 (AAAA) ⇒ **8 truy vấn**. Đây là nguyên nhân thật của "gọi API bên ngoài từ pod chậm bất thường".

**Cách xử lý** — thêm **dấu chấm ở cuối** để báo "đây là tên tuyệt đối, đừng ghép gì nữa":

```bash
curl https://api.company.vn./health
#                          └─ dấu chấm cuối = FQDN, bỏ qua toàn bộ danh sách search -> 1 truy vấn duy nhất
```

Hoặc chỉnh `dnsConfig.options.ndots: 2` trong manifest pod nếu app gọi ra ngoài nhiều.

**Thứ tự chẩn đoán — làm đúng theo bước, đừng nhảy cóc:**

```bash
# BƯỚC 1: container đang hỏi DNS server nào?
cat /etc/resolv.conf

# BƯỚC 2: phân giải được không (KHÔNG cần cài tool gì thêm)
getent hosts postgres
#  │     └─ cơ sở dữ liệu "hosts" = tra theo ĐÚNG cách ứng dụng thật tra
#  └─────── get entries: công cụ của glibc, có sẵn trên mọi image Debian/Ubuntu
#  => ra IP = DNS OK · rỗng (không in gì) = phân giải FAIL
```

⭐ **Vì sao ưu tiên `getent hosts` hơn `dig`/`nslookup`?**

| | `getent hosts` | `dig` / `nslookup` |
|---|---|---|
| Đi qua | **Toàn bộ chuỗi phân giải của hệ điều hành**: `/etc/hosts` → `/etc/nsswitch.conf` → DNS | **Hỏi thẳng DNS server**, bỏ qua `/etc/hosts` |
| Giống cách app tra? | ✅ **Đúng y hệt** | ❌ Không |
| Có sẵn trong container? | ✅ Hầu như luôn có | ❌ Thường phải cài |

🛑 **Đây là mảnh ghép hay bị thiếu**: `dig` ra IP đúng mà app vẫn không kết nối được ⇒ vì `/etc/hosts` có một dòng **đè lên** tên đó, mà `dig` **không đọc `/etc/hosts`**. `getent` thì thấy. Vì vậy **luôn kiểm tra `getent` trước, và luôn xem `/etc/hosts`**.

```bash
# BƯỚC 3: nếu có dig/nslookup thì soi kỹ hơn
dig postgres.ai-hub.svc.cluster.local +short
dig @8.8.8.8 google.com +short      # hỏi DNS NGOÀI để loại trừ "DNS nội bộ hỏng"
#   └─ @ = chỉ định server DNS

# BƯỚC 4: phân biệt lỗi DNS với lỗi mạng — bước quan trọng nhất
getent hosts postgres || echo "=> LỖI DNS"
curl -v telnet://10.0.0.5:5432       # thử bằng IP TRỰC TIẾP
#  => IP thông + tên fail = CHẮC CHẮN lỗi DNS (không phải mạng, không phải firewall)

# BƯỚC 5: /etc/hosts có đè tên không?
cat /etc/hosts
```

**Kubernetes — quy tắc đặt tên service (cần thuộc):**

```
<service>.<namespace>.svc.cluster.local
#  │         │         │        └─ tên miền gốc của cluster (đổi được khi cài)
#  │         │         └────────── loại: service
#  │         └──────────────────── namespace chứa service
#  └────────────────────────────── tên service
```

| Gọi từ đâu | Viết thế nào |
|---|---|
| **Cùng** namespace | `postgres` (ngắn nhất, nhờ `search`) |
| **Khác** namespace | `postgres.ai-hub` ⚠️ **bắt buộc** ghi kèm namespace |
| Muốn chắc chắn / trong config | `postgres.ai-hub.svc.cluster.local` |

⇒ Lỗi rất hay gặp: app ở namespace `default` gọi `postgres` mà service nằm ở `ai-hub` ⇒ **không tìm thấy**, nhưng thông báo lỗi chỉ nói "unknown host" nên dễ tưởng DNS hỏng.

**Nếu nghi CoreDNS có vấn đề:**

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns      # CoreDNS có chạy & Ready không
#                                └─ lọc theo nhãn (tên nhãn vẫn là kube-dns vì lý do lịch sử)
kubectl logs -n kube-system -l k8s-app=kube-dns --tail=50
kubectl get svc -n kube-system kube-dns                  # ClusterIP có khớp với resolv.conf không
```

⚠️ Cài tool DNS khi thiếu: `apk add bind-tools` (Alpine) · `apt install -y dnsutils` (Debian/Ubuntu). Nhưng trên **VDI air-gapped thì cả hai đều không chạy được** ⇒ `getent hosts` vẫn là phương án chính, và nó thường đã đủ để kết luận.

</details>
> Cài nhanh tool DNS khi container thiếu: `apk add bind-tools` (Alpine) · `apt install -y dnsutils`
> (Debian/Ubuntu). Nhưng `getent hosts` thường đã đủ để xác nhận DNS thông hay không mà không cần cài gì.

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

<details>
<summary><b>Bấm xem: giải nghĩa journalctl & systemctl</b></summary>

**Tiền đề — `journalctl` là gì và khác `/var/log` chỗ nào?** systemd thu log của **mọi** service vào một kho nhị phân tập trung (journal). Không đọc bằng `cat` được — phải qua `journalctl`. Ưu điểm: lọc theo service, theo mức độ, theo thời gian **mà không cần biết file log nằm ở đâu**.

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-u` | **u**nit | Lọc theo **service** (unit), ví dụ `-u nginx` |
| `-f` | **f**ollow | Bám realtime (giống `tail -f`) |
| `-x` | e**x**plain | Thêm **gợi ý giải thích** cho thông báo lỗi đã biết |
| `-e` | **e**nd | Nhảy thẳng **xuống cuối** (log mới nhất) |
| `-p` | **p**riority | Lọc theo **mức độ**: `err`, `warning`, `crit`... |
| `-b` | **b**oot | Chỉ lần **khởi động hiện tại**. `-b -1` = lần boot **trước** |
| `-n N` | **n**umber | N dòng cuối |
| `--since` / `--until` | | Khoảng thời gian: `"10 min ago"`, `"today"`, `"2026-08-06 09:00"` |
| `--disk-usage` | | Journal đang **chiếm bao nhiêu đĩa** |
| `--vacuum-time` | hút bụi | **Xoá** log cũ hơn mốc chỉ định |

⭐ **`journalctl -xe` — vì sao là phản xạ đầu tiên khi service fail?**

```bash
journalctl -xe
#           ││
#           │└─ e = nhảy xuống CUỐI = sự kiện MỚI NHẤT (lỗi vừa xảy ra nằm ở đây)
#           └── x = kèm dòng giải thích của systemd, gợi ý nguyên nhân
```

⇒ Ghép lại: "cho tôi xem **ngay chỗ mới nhất**, kèm **lời giải thích**". Đúng thứ cần khi vừa `systemctl start` mà báo failed.

**Bậc thang mức độ log (`-p`)** — từ nặng đến nhẹ, ghi số hoặc chữ đều được:

| Số | Tên | Nghĩa |
|---|---|---|
| 0 | `emerg` | Hệ thống không dùng được |
| 2 | `crit` | Nguy cấp |
| 3 | `err` | **Lỗi** — mức hay lọc nhất |
| 4 | `warning` | Cảnh báo |
| 6 | `info` | Thông tin thường |

```bash
journalctl -p err -b
#           │      └─ chỉ lần boot NÀY (bỏ đi là lấy cả log các lần boot trước)
#           └──────── mức err TRỞ LÊN (gồm cả crit, alert, emerg — không chỉ riêng err)
```

**Lọc theo thời gian — cú pháp linh hoạt:**

```bash
journalctl -u nginx --since "10 min ago"
journalctl -u nginx --since "2026-08-06 09:00" --until "2026-08-06 10:00"
journalctl -u nginx --since today -p err
journalctl -u nginx -f -n 50      # bám realtime, bắt đầu từ 50 dòng gần nhất
```

⚠️ **Log biến mất sau khi reboot?** Mặc định trên nhiều bản phân phối, journal lưu ở `/run/log/journal` (**RAM**) ⇒ **mất sạch khi khởi động lại**. Kiểm tra và bật lưu vĩnh viễn:

```bash
journalctl --disk-usage         # nếu báo dung lượng rất nhỏ -> nhiều khả năng đang lưu ở RAM
# Bật lưu xuống đĩa:
mkdir -p /var/log/journal && systemctl restart systemd-journald
#  │   └─ tạo cả thư mục cha nếu chưa có; KHÔNG báo lỗi nếu đã tồn tại
#  └───── chỉ cần thư mục này tồn tại là journald tự chuyển sang ghi đĩa
```

**Dọn log khi journal ăn hết đĩa** (thủ phạm hay gặp của `/var` đầy):

```bash
journalctl --disk-usage                 # 1. đang chiếm bao nhiêu
journalctl --vacuum-time=7d             # 2a. xoá log cũ hơn 7 ngày
journalctl --vacuum-size=500M           # 2b. hoặc giữ tối đa 500MB
```

Giới hạn **lâu dài** (khỏi phải dọn tay): sửa `SystemMaxUse=1G` trong `/etc/systemd/journald.conf` rồi `systemctl restart systemd-journald`.

**`systemctl` — nhóm lệnh điều khiển service:**

| Lệnh | Làm gì | Hiệu lực sau reboot |
|---|---|---|
| `systemctl start <s>` | Bật **ngay bây giờ** | ❌ Không |
| `systemctl enable <s>` | Đăng ký **tự chạy khi boot** | ✅ Có, nhưng **không bật ngay** |
| `systemctl enable --now <s>` | ⭐ **Cả hai** cùng lúc | ✅ |
| `systemctl reload <s>` | Nạp lại **cấu hình**, không dừng tiến trình ⇒ **không mất kết nối** | — |
| `systemctl restart <s>` | Dừng hẳn rồi bật lại ⇒ **có downtime** | — |
| `systemctl daemon-reload` | Nạp lại **file unit** đã sửa | — |

⚠️ **Hai chữ "reload" khác nhau hoàn toàn — nhầm là mất cả buổi:**

| | `systemctl reload nginx` | `systemctl daemon-reload` |
|---|---|---|
| Nạp lại cái gì | Cấu hình **của ứng dụng** (`/etc/nginx/nginx.conf`) | File **unit của systemd** (`/etc/systemd/system/*.service`) |
| Dùng sau khi sửa | File config của app | File `.service` |

🛑 **Sửa file `.service` mà quên `daemon-reload`** ⇒ systemd **vẫn chạy bản cũ trong bộ nhớ**, thay đổi của bạn **không có tác dụng** và **không có cảnh báo nào**. Bản systemd mới có nhắc `Warning: unit changed on disk`, bản cũ thì im lặng.

**`dmesg` — log của kernel (tầng thấp hơn systemd):**

```bash
dmesg -T | tail -50
#      └─ T = Timestamps: đổi dấu thời gian từ "giây kể từ lúc boot"
#         (dạng [12345.67] rất khó đọc) thành ngày giờ thật

dmesg -T | grep -i -E "oom|killed process|error|fail"
#                      └─ tìm dấu vết bị OOM killer giết, lỗi phần cứng, lỗi đĩa
```

⭐ **`dmesg` là nơi duy nhất ghi lại việc bị OOM killer giết.** Khi app "tự dưng chết mà log ứng dụng không có gì" — nó bị kernel giết vì hết RAM, và **chỉ có dmesg biết**:

```
Out of memory: Killed process 12345 (java) total-vm:8000000kB
```

⇒ Đây chính là bên Linux của hiện tượng `Exited (137)` / `OOMKilled` trong Docker/K8s.

</details>

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

<details>
<summary><b>Bấm xem: quy trình chẩn đoán Docker/K8s theo thứ tự</b></summary>

**Nguyên tắc chung — luôn đi theo thứ tự này**, đừng nhảy cóc:

```
1. Trạng thái là gì?      -> kubectl get pods / docker ps -a
2. VÌ SAO ở trạng thái đó? -> kubectl describe pod  (đọc mục EVENTS ở cuối)
3. App nói gì?            -> kubectl logs (thêm --previous nếu đã crash)
4. Vào tận nơi xem        -> kubectl exec -it ... -- sh
```

⇒ Bước 2 là bước hay bị bỏ qua nhất, và cũng là bước chứa câu trả lời thường xuyên nhất.

| Lệnh | Trả lời câu hỏi |
|---|---|
| `docker stats` | Container nào **đang ăn** CPU/RAM (realtime, Ctrl+C thoát) |
| `docker events` | Có gì **vừa xảy ra**: container bị kill, restart, OOM |
| `docker inspect <c>` | Cấu hình **thật đang chạy** (khác với file compose đã sửa sau đó) |
| `kubectl describe pod` | ⭐ **VÌ SAO** pod không chạy được |
| `kubectl logs --previous` | Log của container **đã chết** — nơi có lỗi thật |
| `kubectl get events` | Sự kiện toàn cluster |

**Bảng tra trạng thái pod → nguyên nhân → lệnh kiểm chứng:**

| Trạng thái | Nghĩa | Kiểm chứng bằng |
|---|---|---|
| `Pending` | **Chưa được xếp lên node nào** | `describe pod` → mục Events: thiếu CPU/RAM, không khớp nodeSelector/taint, PVC chưa bound |
| `ContainerCreating` | Đã chọn node, đang chuẩn bị | Kẹt lâu ⇒ thường do **mount volume fail** hoặc **pull image chậm** |
| `ImagePullBackOff` / `ErrImagePull` | **Không tải được image** | Sai tên/tag · thiếu `imagePullSecret` · registry chặn · **mạng VDI không ra được** |
| `CrashLoopBackOff` | App **khởi động rồi chết**, lặp lại | `logs --previous` ⭐ |
| `OOMKilled` | Vượt **memory limit** | `describe pod` → `Last State: Terminated, Reason: OOMKilled` |
| `Evicted` | Node **hết tài nguyên** nên đuổi pod đi | `describe node` → mục Conditions: DiskPressure/MemoryPressure |
| `Terminating` (kẹt) | Đang xoá mà không xong | Còn **finalizer**, hoặc node đã chết |
| `Running` nhưng chưa `Ready` | Container sống nhưng **readinessProbe fail** | `describe pod` → mục Events có dòng `Readiness probe failed` |

⚠️ **`Running` mà `READY 0/1` — trạng thái hay bị hiểu nhầm nhất.** Cột `STATUS` nói **container còn sống**; cột `READY` nói **có được nhận traffic không**. `0/1` = readinessProbe chưa qua ⇒ **Service KHÔNG gửi request tới pod này**. App vẫn chạy nhưng người dùng gặp 503.

```bash
kubectl get endpoints <svc>    # rỗng = không pod nào Ready = đó là lý do 503
```

**`BackOff` nghĩa là gì?** Là **cơ chế lùi dần** của K8s: crash lần 1 chờ 10s thử lại, lần 2 chờ 20s, 40s, 80s... **tối đa 5 phút**. Vì vậy pod crash lâu rồi thì **rất lâu mới thấy thử lại** — không phải K8s đã bỏ cuộc. Sửa xong nguyên nhân mà muốn nó thử ngay, đừng ngồi chờ:

```bash
kubectl delete pod <pod>              # pod do Deployment quản lý sẽ được tạo lại NGAY
kubectl rollout restart deploy/<name> # cách êm hơn nếu có nhiều replica
```

**Bóc lệnh lọc pod không khoẻ (rất đáng đưa vào alias):**

```bash
kubectl get pods -A | grep -vE 'Running|Completed'
#                │     │   ││   └─ hai trạng thái BÌNH THƯỜNG cần loại bỏ
#                │     │   │└──── E = regex mở rộng, để dùng dấu | (hoặc)
#                │     │   └───── v = đảo ngược: chỉ giữ dòng KHÔNG khớp
#                │     └───────── => còn lại đúng những pod CÓ VẤN ĐỀ
#                └─────────────── mọi namespace
```

⚠️ Lệnh trên **giữ lại cả dòng tiêu đề** (NAME READY STATUS...) vì nó cũng không chứa chữ Running — đó là bình thường, không phải lỗi.

⚠️ Nó **KHÔNG bắt được** pod `Running` mà `READY 0/1` (vì có chữ Running). Muốn bắt cả ca đó:

```bash
kubectl get pods -A | awk 'NR>1 && $3!="Running" || ($2 ~ /^0\// )'
#                          │       │                  └─ cột READY bắt đầu bằng "0/"
#                          │       └─ cột 3 (STATUS) khác Running
#                          └─ NR>1: bỏ qua dòng tiêu đề
```

**Docker — kiểm chứng container bị OOM:**

```bash
docker inspect <container> | jq '.[0].State | {OOMKilled, ExitCode, Error}'
#                             │              └─ bóc đúng 3 field cần
#                             └─ [0] vì inspect luôn trả về MỘT MẢNG, kể cả khi hỏi 1 container
```

`OOMKilled: true` ⇒ container vượt giới hạn RAM (`--memory` hoặc `mem_limit` trong compose), **không phải app tự crash**. Cách xử lý khác hẳn: tăng limit hoặc giảm mức dùng, chứ không phải sửa code lỗi.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa các cờ git log/diff/blame</b></summary>

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `--oneline` | một dòng | Mỗi commit **gọn một dòng** (hash ngắn + tiêu đề) |
| `--graph` | đồ thị | Vẽ **nhánh** bằng ký tự `* \| / \\` ở lề trái |
| `--decorate` | trang trí | Hiện **tên branch/tag** gắn vào commit |
| `--all` | tất cả | **Mọi** branch, không chỉ branch đang đứng |
| `-p` | **p**atch | Hiện **nội dung thay đổi** (diff) của từng commit |
| `--staged` | | Xem phần **đã `git add`** (đồng nghĩa `--cached`) |
| `--author` | | Lọc theo người viết |
| `-S<chuỗi>` | **S**tring pickaxe | ⭐ Tìm commit **thêm/xoá** một chuỗi cụ thể |

⭐ **`git log --oneline --graph --decorate --all` — vì sao là bộ tứ kinh điển?** Bốn cờ trả lời bốn câu: *có những commit nào* · *nhánh chia tách ra sao* · *branch đang ở đâu* · *có gì ở nhánh khác*. Đáng đặt thành alias (bạn đã có `glog` trong `docker.txt`).

⭐ **`git diff` — ba phạm vi khác nhau, đây là chỗ hay nhầm nhất:**

Git có **ba vùng**, và `diff` so sánh **các cặp khác nhau** tuỳ cờ:

```
Working tree  ──(git add)──>  Staging area  ──(git commit)──>  Repository
(file đang sửa)               (đã đánh dấu)                    (đã lưu)
```

| Lệnh | So sánh | Trả lời |
|---|---|---|
| `git diff` | Working tree ↔ **Staging** | "Tôi sửa gì mà **chưa** `add`?" |
| `git diff --staged` | Staging ↔ **Repository** | "Tôi sắp commit **cái gì**?" ⭐ Chạy trước mỗi lần commit |
| `git diff HEAD` | Working tree ↔ Repository | "Tổng cộng đã đổi gì so với commit cuối?" |

🛑 Đây là lý do "tôi sửa rồi mà `git diff` không hiện gì": bạn **đã `git add`** ⇒ thay đổi chuyển sang vùng staging ⇒ phải xem bằng `--staged`.

**So sánh hai branch — ⚠️ hai dấu chấm và ba dấu chấm KHÁC NHAU:**

```bash
git diff main..feature      # so TRẠNG THÁI CUỐI của hai nhánh
git diff main...feature     # so từ ĐIỂM RẼ NHÁNH -> chỉ thấy việc feature làm
#               └─ BA chấm = "kể từ lúc tách khỏi main"
```

⇒ Nếu `main` đã đi tiếp sau khi bạn tách nhánh, **hai chấm** sẽ hiện **cả thay đổi của người khác trên main** (dưới dạng đảo ngược) ⇒ nhìn rất rối. **Review PR thì dùng ba chấm** — đó chính là những gì GitHub hiển thị.

⭐ **`git log -S` — công cụ điều tra mạnh nhất mà ít người biết:**

```bash
git log -S "API_SECRET_KEY" --oneline
#       └─ tìm mọi commit làm SỐ LẦN XUẤT HIỆN của chuỗi này thay đổi
#          => tìm ra ai THÊM VÀO và ai XOÁ ĐI
```

Trả lời được: *"dòng cấu hình này ai thêm và vào lúc nào?"*, *"secret này lọt vào từ commit nào?"* — thứ mà `git blame` **không làm được** (blame chỉ cho biết ai sửa **lần cuối**, dòng đã bị xoá thì blame không thấy).

**`git blame` — kèm cờ để đỡ nhiễu:**

```bash
git blame -L 40,60 -w -C app/config.py
#         │        │  └─ C = phát hiện dòng được DI CHUYỂN/COPY từ file khác
#         │        └──── w = bỏ qua thay đổi chỉ về KHOẢNG TRẮNG (định dạng lại code)
#         └───────────── L = giới hạn từ dòng 40 đến 60
```

⚠️ Không có `-w`, một lần chạy formatter (prettier/black) sẽ khiến **mọi dòng đều mang tên người chạy formatter** ⇒ blame trở nên vô dụng.

⭐ **`git reflog` — cứu tinh khi lỡ tay, cần hiểu để dùng:**

`reflog` ghi lại **mọi lần HEAD di chuyển** trên máy bạn — kể cả commit đã bị `reset --hard` "mất". Commit không thực sự bị xoá ngay; nó chỉ **mất tên trỏ tới**, và vẫn nằm trong kho khoảng **90 ngày** trước khi bị dọn rác.

```bash
git reflog
# a1b2c3d HEAD@{0}: reset: moving to HEAD~1     <- thao tác vừa làm (lỡ tay)
# e4f5g6h HEAD@{1}: commit: thêm tính năng X    <- ⭐ commit "đã mất" VẪN CÒN ĐÂY
git reset --hard e4f5g6h                        # quay lại chính xác chỗ đó
```

🛑 **Giới hạn quan trọng**: `reflog` là **cục bộ trên máy bạn**, không đẩy lên remote, và **không cứu được** thay đổi **chưa từng commit** (`git clean -fd` xoá file chưa track là mất thật, không có gì cứu).

</details>

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

<details>
<summary><b>Bấm xem: reset --soft/--mixed/--hard, và vì sao nên dùng revert</b></summary>

⭐ **Ba chế độ `reset` — khác nhau ở chỗ "kéo theo bao nhiêu vùng":**

```
Working tree  ←→  Staging area  ←→  Repository (HEAD)
```

| Lệnh | HEAD lùi | Staging | Working tree (file của bạn) | Mất code? |
|---|---|---|---|---|
| `reset --soft HEAD~1` | ✅ | **giữ nguyên** | **giữ nguyên** | ❌ An toàn |
| `reset HEAD~1` (mặc định `--mixed`) | ✅ | **xoá** (thành chưa add) | **giữ nguyên** | ❌ An toàn |
| `reset --hard HEAD~1` | ✅ | xoá | 🔴 **GHI ĐÈ — mất sạch** | 🛑 **CÓ** |

⇒ Cách nhớ: **`--soft`** giữ cả hai vùng · **`--mixed`** giữ file, bỏ staging · **`--hard`** không giữ gì.

**`HEAD~1` nghĩa là gì?** `HEAD` = commit đang đứng; `~1` = **lùi lại 1 commit** theo dòng cha. `HEAD~3` = lùi 3 commit.

**Dùng khi nào:**

```bash
git reset --soft HEAD~1     # muốn GỘP commit cuối vào commit mới / sửa lại message
git reset HEAD~1            # muốn commit LẠI theo cách chia nhỏ khác
git reset --hard HEAD~1     # 🛑 muốn VỨT BỎ HOÀN TOÀN commit cuối và code của nó
```

🛑 **`git reset --hard origin/main` là lệnh xoá không hoàn tác:** nó ép branch của bạn **giống hệt remote**, **vứt sạch** mọi commit local chưa push và mọi thay đổi đang sửa dở. Kiểm tra trước khi chạy:

```bash
git status                       # còn gì đang sửa dở không
git log origin/main..HEAD --oneline    # ⭐ liệt kê commit CHỈ CÓ Ở LOCAL — sắp mất những cái này
#           └─ hai chấm: "có ở HEAD mà KHÔNG có ở origin/main"
git stash                        # cất tạm nếu muốn giữ lại
```

⭐ **`revert` vs `reset` — chọn cái nào?**

| | `git revert <commit>` | `git reset --hard` |
|---|---|---|
| Cách làm | Tạo commit **MỚI** đảo ngược thay đổi | **Xoá** commit khỏi lịch sử |
| Lịch sử | **Giữ nguyên**, có thêm dấu vết | **Bị viết lại** |
| Đã push lên remote | ✅ **An toàn, dùng cái này** | 🛑 Phải force-push ⇒ **hỏng repo của đồng đội** |
| Nhánh riêng chưa push | Được | Được |

🛑 **Quy tắc vàng: commit đã push lên nhánh chung ⇒ CHỈ dùng `revert`.** Vì `reset` + force-push sẽ xoá commit mà người khác đã pull về ⇒ lần pull sau của họ **conflict tùm lum** hoặc **mất code**.

**`restore` vs `checkout` — vì sao có hai lệnh làm cùng việc?**

`git checkout` cũ **gánh quá nhiều vai**: chuyển branch, khôi phục file, tạo branch... gây nhầm lẫn và nguy hiểm. Git 2.23 tách đôi cho rõ nghĩa:

| Việc | Lệnh mới (nên dùng) | Lệnh cũ |
|---|---|---|
| Bỏ sửa đổi của file | `git restore <file>` | `git checkout -- <file>` |
| Bỏ staged (giữ sửa đổi) | `git restore --staged <file>` | `git reset HEAD <file>` |
| Chuyển branch | `git switch <branch>` | `git checkout <branch>` |

🛑 **`git restore <file>` mất code vĩnh viễn** — thay đổi chưa commit thì **không có trong reflog**, không cứu được. Xem kỹ trước:

```bash
git diff <file>        # đọc chính xác những gì sắp bị vứt bỏ
```

**`--amend` — sửa commit cuối:**

```bash
git commit --amend --no-edit
#                  └─ giữ nguyên message cũ, chỉ nhét thêm file đã `add` vào commit cuối
```

🛑 `--amend` **tạo ra một commit HOÀN TOÀN MỚI** (hash khác), không phải sửa tại chỗ. Nếu commit cũ **đã push** ⇒ phải force-push ⇒ áp dụng đúng cảnh báo ở trên.

**`git clean` — luôn xem trước bằng `-n`:**

```bash
git clean -nd     # n = dry-run: CHỈ LIỆT KÊ những gì sẽ xoá, KHÔNG xoá thật
git clean -fd     # f = force (bắt buộc có), d = cả thư mục -> XOÁ THẬT
#         │└─ d: gồm cả thư mục chưa track
#         └── f: git bắt buộc phải có cờ này, coi như một bước xác nhận
```

🛑 `git clean -fd` xoá **file chưa từng được git theo dõi** ⇒ **reflog không cứu được** vì git chưa bao giờ biết đến chúng. ⚠️ Nó cũng xoá luôn **file `.env` chưa commit** — mất cấu hình local. Thêm `-x` còn xoá cả file trong `.gitignore` (`node_modules`, build output).

⇒ **Luôn chạy `git clean -nd` trước `git clean -fd`.** Không có ngoại lệ.

</details>

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

<details>
<summary><b>Bấm xem: hiểu conflict, và --ours/--theirs ĐẢO NGHĨA khi rebase</b></summary>

**Conflict xảy ra khi nào?** Khi **hai nhánh sửa CÙNG một vùng dòng** của cùng một file. Git ghép tự động được nếu hai bên sửa **chỗ khác nhau**; sửa **đè lên nhau** thì git **không đoán bừa** mà dừng lại hỏi bạn.

**Dấu hiệu trong file:**

```
<<<<<<< HEAD
mã của phía "OURS"
=======
mã của phía "THEIRS"
>>>>>>> feature-branch
```

⇒ Sửa xong phải **xoá cả ba dòng dấu** `<<<<<<<`, `=======`, `>>>>>>>`. Quên xoá ⇒ code chứa ký tự rác, thường **không chạy được** hoặc lọt lên production.

🛑 ⭐ **`--ours` và `--theirs` ĐẢO NGHĨA giữa merge và rebase** — đây là bẫy nguy hiểm nhất của git, lấy nhầm là **mất code mà không báo lỗi**:

| Đang làm | `--ours` = | `--theirs` = |
|---|---|---|
| **`git merge`** | Branch bạn **đang đứng** (nơi nhận) | Branch **được merge vào** |
| **`git rebase`** | 😵 Branch **gốc** (`main`) | 😵 **Branch của BẠN** (commit đang phát lại) |

**Vì sao đảo?** Vì `rebase` hoạt động bằng cách: **checkout sang `main` trước**, rồi **phát lại từng commit của bạn lên đó**. Nên trong con mắt của git, "chỗ đang đứng" (**ours**) là `main`, còn "cái đang được áp vào" (**theirs**) chính là **commit của bạn**.

⇒ Hệ quả thực tế: đang rebase, muốn giữ **code của mình** thì phải gõ **`--theirs`** — nghe hoàn toàn ngược với trực giác.

⭐ **Cách an toàn: đừng đoán, hãy XEM.** Git giữ đủ ba phiên bản trong lúc conflict:

```bash
git status                          # 1. file nào đang conflict (mục "Unmerged paths")
git diff                            # 2. xem cả hai phía cạnh nhau

git show :1:duong/dan/file.py       # 3. phiên bản GỐC CHUNG (tổ tiên, trước khi hai bên tách)
git show :2:duong/dan/file.py       #    phiên bản "ours"
git show :3:duong/dan/file.py       #    phiên bản "theirs"
#        └─ số 1/2/3 là ba ô nhớ tạm git dùng trong lúc merge
```

⇒ Xem `:1:` (bản gốc) là cách hiểu **mỗi bên đã đổi gì so với điểm chung** — rõ hơn nhiều so với chỉ nhìn hai bản cuối.

**Quy trình xử lý — merge và rebase khác nhau ở lệnh tiếp tục:**

```bash
# --- Đang MERGE ---
git status                    # xem file conflict
# ... sửa file, xoá dấu <<<<<<< ======= >>>>>>> ...
git add <file>                # ĐÁNH DẤU đã giải quyết (không phải "thêm file mới")
git merge --continue          # hoặc: git commit
git merge --abort             # 🔙 huỷ toàn bộ, quay về nguyên trạng

# --- Đang REBASE ---
git add <file>
git rebase --continue         # ⚠️ rebase xử lý TỪNG COMMIT -> có thể conflict LẶP LẠI nhiều lần
git rebase --skip             # bỏ qua commit đang gây conflict (⚠️ mất commit đó)
git rebase --abort            # 🔙 huỷ, quay về trước khi rebase
```

⚠️ **Rebase có thể bắt bạn giải conflict nhiều lần** cho cùng một chỗ, vì nó phát lại **lần lượt từng commit**. Đây là hành vi bình thường, không phải lỗi. Muốn git **nhớ** cách bạn đã giải để tự áp dụng lại:

```bash
git config --global rerere.enabled true
#                   └─ rerere = REuse REcorded REsolution ("dùng lại cách giải đã ghi")
```

⭐ **`--abort` luôn an toàn** — nó đưa mọi thứ về đúng trạng thái trước khi bắt đầu. Khi rối, đừng cố gỡ tiếp: **abort rồi làm lại từ đầu** bao giờ cũng nhanh hơn.

**Lấy nguyên một phía (khi chắc chắn):**

```bash
git checkout --ours  <file>    # lấy trọn phía "ours"  -> ⚠️ NHỚ KIỂM TRA đang merge hay rebase
git checkout --theirs <file>   # lấy trọn phía "theirs"
git add <file>                 # vẫn phải add để đánh dấu đã xong
```

⚠️ Cách này **vứt bỏ hoàn toàn** thay đổi của phía kia trong file đó — kể cả những đoạn **không hề conflict**. Chỉ dùng khi thực sự muốn ghi đè cả file (ví dụ file lock tự sinh: `package-lock.json`).

**`merge` vs `rebase` — chọn thế nào:**

| | `git merge main` | `git rebase main` |
|---|---|---|
| Lịch sử | Giữ nguyên, thêm **commit merge** | **Thẳng hàng**, sạch, như chưa từng rẽ nhánh |
| Hash commit | Không đổi | 🛑 **Đổi hết** (viết lại lịch sử) |
| Đã push nhánh chung | ✅ An toàn | 🛑 **Không được** — trừ khi chỉ mình bạn dùng nhánh đó |

⇒ Quy ước phổ biến: **rebase** trên nhánh cá nhân (cho lịch sử sạch trước khi mở PR) · **merge** khi đưa vào nhánh chung.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa branch, stash, cherry-pick, force-with-lease</b></summary>

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-a` (branch) | **a**ll | Cả branch remote (`remotes/origin/...`) |
| `-b` (checkout) | **b**ranch | Tạo branch **mới** rồi chuyển sang luôn |
| `-D` (branch) | **D**elete force | Xoá **cưỡng bức**, kể cả chưa merge |
| `-d` (branch) | **d**elete | Xoá **an toàn** — từ chối nếu chưa merge |
| `-m` (branch) | **m**ove | Đổi tên branch |
| `--rebase` (pull) | | Pull theo kiểu rebase thay vì merge |
| `--force-with-lease` | thuê có kiểm | ⭐ Force push **an toàn** |

⚠️ **`-d` và `-D` khác nhau ở một lớp bảo vệ:**

```bash
git branch -d feature-x    # từ chối nếu branch CHƯA merge -> "not fully merged"
git branch -D feature-x    # 🛑 xoá bất chấp -> commit chỉ có ở nhánh này thành MỒ CÔI
```

⇒ Lỗi `not fully merged` là **tính năng bảo vệ**, không phải sự cố. Gõ ngay `-D` cho xong là **vứt code**. Cứu được bằng `git reflog` trong ~90 ngày, nhưng đừng dựa vào đó.

⭐ **`git stash` — cất tạm để chuyển việc:**

```bash
git stash push -u -m "đang sửa dở phần login"
#          │   │  └─ đặt GHI CHÚ -> sau này `stash list` còn biết cái nào là cái nào
#          │   └──── u = untracked: ⭐ cất luôn FILE MỚI TẠO
#          └──────── push = cú pháp mới (bản cũ chỉ gõ `git stash`)
```

🛑 **`git stash` trần KHÔNG cất file mới tạo** (chưa `git add` lần nào). Hậu quả: chuyển branch xong thấy file lạ nằm lại, hoặc tưởng đã cất mà thực ra chưa ⇒ **luôn thêm `-u`**.

| Lệnh | Làm gì |
|---|---|
| `git stash list` | Danh sách: `stash@{0}` là **mới nhất** |
| `git stash show -p stash@{1}` | Xem **nội dung** stash đó (`-p` = patch) |
| `git stash pop` | Lấy lại **và xoá** khỏi danh sách |
| `git stash apply` | Lấy lại **nhưng GIỮ** — an toàn hơn nếu sợ conflict |
| `git stash drop stash@{0}` | Xoá một stash |

⚠️ **`pop` gặp conflict thì stash KHÔNG bị xoá** (may mắn thay) — nhưng nếu áp dụng thành công một phần rồi bạn hoảng lên `reset --hard` thì **mất cả hai**. ⇒ Khi không chắc, dùng **`apply`** trước, kiểm tra ổn rồi mới `drop`.

⭐ **`git cherry-pick` — lấy đúng một commit từ nhánh khác:**

```bash
git cherry-pick a1b2c3d           # chép commit đó sang nhánh hiện tại
git cherry-pick a1b2c3d..e4f5g6h  # chép cả một dải commit
git cherry-pick -n a1b2c3d        # n = no-commit: áp thay đổi nhưng CHƯA commit (để sửa thêm)
```

⚠️ Cherry-pick tạo commit **hash MỚI** (nội dung giống, danh tính khác). Nếu sau đó **merge luôn cả nhánh nguồn** vào, git thấy hai commit khác hash ⇒ có thể **conflict** hoặc **nhân đôi thay đổi**. ⇒ Dùng cherry-pick cho **hotfix cần đưa gấp sang nhánh release**, không dùng thay cho merge.

⭐ **`--force-with-lease` — vì sao KHÔNG BAO GIỜ dùng `--force` trần:**

| | `git push --force` | `git push --force-with-lease` |
|---|---|---|
| Kiểm tra remote trước khi đẩy | ❌ **Không** — đè bất chấp | ✅ Có — chỉ đẩy nếu remote **đúng như bạn thấy lần fetch cuối** |
| Đồng đội vừa push commit mới | 🛑 **Xoá mất commit của họ, im lặng** | ✅ **Từ chối**, báo lỗi cho bạn biết |

```bash
git push --force-with-lease origin feature-x
#              └─ "lease" = tôi ĐANG GIỮ CHỖ ở trạng thái X;
#                 nếu remote đã khác X thì HUỶ, đừng đẩy
```

⚠️ Bẫy: nếu bạn vừa `git fetch` (làm mới thông tin remote) mà **chưa xem lại**, `--force-with-lease` sẽ tưởng bạn **đã biết** commit mới đó ⇒ vẫn cho đè. ⇒ Sau khi fetch, **kiểm tra bằng mắt** trước khi force:

```bash
git log origin/feature-x --oneline -5    # remote đang có gì
```

**`git pull --rebase` — vì sao nên dùng:**

```bash
git pull --rebase origin main
#         └─ thay vì tạo commit merge, PHÁT LẠI commit của bạn lên trên bản mới nhất
```

| | `git pull` (mặc định: merge) | `git pull --rebase` |
|---|---|---|
| Lịch sử | Sinh commit `Merge branch 'main'...` rác | **Thẳng hàng, sạch** |
| Hash commit local | Giữ nguyên | Đổi (chưa push thì không sao) |

Đặt mặc định luôn cho khỏi phải nhớ gõ:

```bash
git config --global pull.rebase true
```

⚠️ **Lệnh nào cần mạng, lệnh nào không** — rất quan trọng khi làm việc trên VDI hạn chế:

| Cần mạng | Chạy offline được |
|---|---|
| `fetch`, `pull`, `push`, `clone` | `branch`, `checkout`, `commit`, `stash`, `log`, `diff`, `merge`, `rebase`, `cherry-pick` |

⇒ Git là **hệ phân tán**: toàn bộ lịch sử nằm trên máy bạn, nên gần như mọi thao tác đều làm được khi mất mạng.

</details>

### Tình huống hay gặp
```bash
git commit --allow-empty -m "trigger CI"   # Commit rỗng để trigger CI/CD
git rm --cached <file>                 # Bỏ file khỏi git nhưng giữ trên disk
git log --oneline | head               # Xem nhanh vài commit gần nhất
git reflog                             # Tìm lại commit "mất" sau reset --hard
git reset --hard <commit-tu-reflog>    # Khôi phục về commit đã lỡ xóa
```

<details>
<summary><b>Bấm xem: giải nghĩa các tình huống & lệnh cứu hộ</b></summary>

**1. Commit rỗng để kích hoạt CI/CD:**

```bash
git commit --allow-empty -m "trigger CI"
#                 └─ cho phép commit KHÔNG có thay đổi nào
#                    (bình thường git từ chối: "nothing to commit")
```

Dùng khi pipeline chỉ chạy theo sự kiện push mà bạn cần chạy lại — ví dụ secret vừa được sửa ở phía CI, code không đổi. ⚠️ Cách này để lại commit rác trong lịch sử; nếu nền tảng CI có nút "re-run" thì ưu tiên dùng nút đó.

**2. Gỡ file khỏi git nhưng vẫn giữ trên máy:**

```bash
git rm --cached .env
#         └─ chỉ gỡ khỏi vùng theo dõi của git, FILE TRÊN ĐĨA VẪN CÒN
#            (không có --cached thì XOÁ LUÔN file thật)

git rm -r --cached node_modules    # -r = đệ quy, cho thư mục
```

Dùng khi lỡ commit file không nên commit. **Bắt buộc làm đủ ba bước**, thiếu bước nào cũng hỏng:

```bash
echo ".env" >> .gitignore     # 1. chặn lần sau (thiếu bước này thì commit sau lại vào tiếp)
git rm --cached .env          # 2. gỡ khỏi theo dõi
git commit -m "chore: bỏ .env khỏi git"   # 3. ghi nhận
```

🛑 **Cảnh báo lớn: cách này KHÔNG xoá file khỏi LỊCH SỬ.** Mọi commit cũ **vẫn chứa nguyên nội dung** — ai clone repo về đều `git show <commit-cũ>` đọc được. ⇒ Nếu file đó chứa **secret thật**:

1. **Đổi ngay secret đó** (coi như đã lộ — đây là việc quan trọng nhất, làm trước tiên).
2. Muốn xoá khỏi lịch sử: dùng `git filter-repo` hoặc BFG Repo-Cleaner ⚠️ **viết lại toàn bộ lịch sử**, mọi người phải clone lại.

⇒ Thứ tự đúng: **đổi secret trước, dọn lịch sử sau**. Dọn lịch sử mà không đổi secret là vô nghĩa — bản sao đã nằm ở máy người khác và trên server CI.

**3. Tìm lại commit đã "mất" sau `reset --hard`:**

```bash
git reflog
# a1b2c3d HEAD@{0}: reset: moving to HEAD~1      <- thao tác lỡ tay
# e4f5g6h HEAD@{1}: commit: tính năng quan trọng <- ⭐ commit tưởng đã mất
git reset --hard e4f5g6h                          # quay về đúng đó
```

Hoặc an toàn hơn — **không đụng vào nhánh hiện tại**, mở nhánh mới để xem trước:

```bash
git branch cuu-ho e4f5g6h    # tạo nhánh trỏ vào commit đó, KHÔNG chuyển sang
git log cuu-ho --oneline     # kiểm tra đúng cái mình cần rồi hãy quyết định
```

⚠️ `reflog` **chỉ có trên máy bạn**. Reset nhầm trên máy A thì ngồi máy B **không tìm lại được**.

**4. Xem nhanh vài commit gần nhất:**

```bash
git log --oneline | head
git log --oneline -10        # gọn hơn: git tự giới hạn, không cần pipe qua head
```

**5. Một số tình huống khác đáng biết:**

```bash
# Lỡ commit vào nhầm branch (main) trong khi đáng lẽ phải ở nhánh riêng:
git branch feature-x         # 1. tạo nhánh mới TẠI ĐÂY (giữ lại commit)
git reset --hard HEAD~1      # 2. đẩy main lùi lại 1 commit
git checkout feature-x       # 3. sang nhánh mới -> commit vẫn còn nguyên ở đó

# Xem file ở phiên bản cũ mà KHÔNG cần chuyển branch:
git show HEAD~3:src/config.py          # in nội dung file tại commit đó ra màn hình
git show main:src/config.py > /tmp/cu.py   # lưu ra file để so sánh

# Ai đó sửa gì trên remote mà mình chưa có:
git fetch origin                        # tải về, KHÔNG động vào code đang làm
git log HEAD..origin/main --oneline     # liệt kê commit remote có mà mình CHƯA có
#           └─ hai chấm: "có ở origin/main mà KHÔNG có ở HEAD"

# Tìm commit gây ra lỗi (nhị phân, rất mạnh):
git bisect start
git bisect bad                # commit hiện tại: ĐANG LỖI
git bisect good v1.2.0        # phiên bản này: CHẠY TỐT
#  => git tự nhảy tới commit GIỮA, bạn test rồi gõ `git bisect good` / `git bisect bad`
#     lặp lại vài lần là khoanh đúng thủ phạm (100 commit chỉ cần ~7 lần thử)
git bisect reset              # xong thì quay lại chỗ cũ
```

⚠️ `git fetch` **an toàn tuyệt đối** — chỉ tải dữ liệu về, **không** đụng vào file bạn đang sửa và **không** đổi branch. Ngược lại, `git pull` = `fetch` + `merge` ⇒ **có thể gây conflict ngay lập tức**. ⇒ Khi không chắc tình hình, **`fetch` trước, xem `log`, rồi mới quyết định merge hay rebase**.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa cờ psql, lệnh gạch chéo, pg_dump/pg_restore</b></summary>

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-U` | **U**ser | Đăng nhập bằng user nào (⚠️ **chữ HOA**) |
| `-d` | **d**atabase | Database nào |
| `-h` | **h**ost | Máy chủ. Không ghi = kết nối qua **Unix socket** trên máy local |
| `-p` | **p**ort | Cổng, mặc định **5432** |
| `-c` | **c**ommand | Chạy **một câu SQL** rồi thoát ngay (hợp cho script) |
| `-f` | **f**ile | Chạy SQL từ file |
| `-t` | **t**uples only | Bỏ tiêu đề cột và dòng đếm — chỉ lấy **dữ liệu thuần** |
| `-A` | **A**ligned tắt | Bỏ canh lề bằng khoảng trắng |
| `-F` | **F**ield separator | Đổi ký tự phân tách cột |

⚠️ **`-U` là chữ HOA. `-u` viết thường sẽ báo lỗi** — khác với `mysql` dùng `-u` thường. Đây là chỗ hay gõ nhầm khi chuyển qua lại giữa hai loại DB.

**Nhập mật khẩu — đừng gõ vào lệnh:**

```bash
PGPASSWORD='matkhau' psql -h db -U app -d mydb -c "SELECT 1;"
# └─ biến môi trường đặt NGAY TRƯỚC lệnh: chỉ có hiệu lực cho ĐÚNG lệnh này
#    ⚠️ vẫn lọt vào lịch sử shell -> chỉ nên dùng trong script CI

# ⭐ Cách chuẩn: file ~/.pgpass (psql tự đọc, không cần gõ gì)
echo "db-host:5432:mydb:app:matkhau" >> ~/.pgpass
chmod 600 ~/.pgpass
#     └─ BẮT BUỘC 600 (chỉ chủ sở hữu đọc/ghi).
#        Quyền lỏng hơn thì psql TỪ CHỐI ĐỌC và IM LẶNG bỏ qua file
#        -> hiện tượng "đã tạo .pgpass mà vẫn hỏi mật khẩu"
```

**Lệnh gạch chéo `\` — chỉ có bên trong `psql`, không phải SQL:**

| Lệnh | Viết tắt của | Làm gì |
|---|---|---|
| `\l` | **l**ist | Liệt kê database |
| `\c <db>` | **c**onnect | Chuyển database |
| `\dt` | **d**escribe **t**ables | Liệt kê bảng |
| `\d <table>` | **d**escribe | Cấu trúc bảng: cột, kiểu, index, khoá ngoại |
| `\d+ <table>` | | Như trên **+ dung lượng, mô tả** |
| `\du` | **d**escribe **u**sers | Danh sách role/user |
| `\dn` | **d**escribe **n**amespaces | Danh sách schema |
| `\di` | **d**escribe **i**ndexes | Danh sách index |
| `\x` | e**x**panded | ⭐ Bật/tắt hiển thị **dọc** |
| `\timing` | | Bật đo thời gian mỗi query |
| `\q` | **q**uit | Thoát |

⭐ **`\x` — cứu tinh khi bảng có nhiều cột.** Bảng 20 cột in ngang thì **tràn dòng, không đọc nổi**. Bật `\x` đổi thành mỗi cột một dòng:

```
-[ RECORD 1 ]------------------
id       | 42
email    | user@company.vn
created  | 2026-08-07 09:15:00
```

**Truy vấn chẩn đoán — dùng khi DB chậm/treo:**

```sql
-- Query nào đang chạy và chạy bao lâu rồi
SELECT pid, now() - query_start AS thoi_gian, state, left(query, 80)
FROM pg_stat_activity
WHERE state != 'idle' AND pid != pg_backend_pid()
--                        └─ loại bỏ CHÍNH phiên đang gõ lệnh này
ORDER BY thoi_gian DESC;
```

⚠️ **Hai cách dừng query — khác nhau về mức độ:**

```sql
SELECT pg_cancel_backend(12345);     -- ⭐ THỬ CÁI NÀY TRƯỚC: huỷ query, GIỮ kết nối
SELECT pg_terminate_backend(12345);  -- 🛑 mạnh tay: cắt CẢ kết nối
```

⇒ `cancel` để app còn cơ hội xử lý lỗi tử tế. `terminate` khiến app **mất kết nối đột ngột** — có thể làm rơi cả pool kết nối.

⚠️ **Truy vấn phát hiện khoá (lock) — nguyên nhân số 1 của "DB treo":**

```sql
SELECT pid, wait_event_type, wait_event, left(query,60)
FROM pg_stat_activity WHERE wait_event_type = 'Lock';
--                          └─ đang CHỜ khoá do phiên khác giữ
```

⚠️ Trạng thái **`idle in transaction`** rất nguy hiểm: app mở transaction rồi **quên commit/rollback** ⇒ giữ khoá **vô thời hạn** ⇒ mọi query khác xếp hàng chờ. Đây thường là **lỗi code phía ứng dụng**, không phải lỗi database.

**Backup & Restore — chọn đúng định dạng:**

```bash
pg_dump -U app -d mydb -Fc -f backup.dump
#                       │   └─ ghi ra file (thay cho dùng dấu > )
#                       └──── F = Format, c = custom: NÉN SẴN + restore chọn lọc được
```

| Định dạng | Lệnh dump | Restore bằng | Ưu điểm |
|---|---|---|---|
| **Plain SQL** | `pg_dump -d db > b.sql` | `psql -d db < b.sql` | Đọc/sửa được bằng editor |
| **Custom (`-Fc`)** | `pg_dump -Fc -f b.dump` | `pg_restore -d db b.dump` | ⭐ Nén sẵn · **restore từng bảng** · chạy song song |

🛑 **Không dùng lẫn**: file `-Fc` là **nhị phân**, đưa vào `psql <` sẽ báo lỗi khó hiểu. Ngược lại file `.sql` đưa vào `pg_restore` cũng không chạy.

```bash
pg_restore -U app -d mydb -j 4 --clean --if-exists backup.dump
#                          │    │       └─ không báo lỗi khi object chưa tồn tại
#                          │    └───────── XOÁ object cũ trước khi tạo lại
#                          └────────────── j = jobs: restore SONG SONG 4 luồng (nhanh hơn nhiều)
```

⚠️ `pg_dump` **chỉ backup MỘT database**, **không** gồm role và tablespace (thứ nằm ở cấp cụm). Backup toàn bộ cụm phải dùng:

```bash
pg_dumpall -U postgres -f all.sql      # gồm cả role, quyền, mọi database
pg_dumpall -U postgres --roles-only -f roles.sql   # chỉ role (hay dùng kèm với pg_dump)
```

⚠️ **Phiên bản `pg_dump` phải ≥ phiên bản server.** Dùng `pg_dump` 13 để dump server 16 ⇒ lỗi `server version mismatch`. Trong container thì chạy `pg_dump` **bằng chính image của server** là chắc nhất.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa cờ mysql/mysqldump và các lệnh chẩn đoán</b></summary>

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-u` | **u**ser | User (⚠️ **chữ thường** — ngược với PostgreSQL dùng `-U` hoa) |
| `-p` | **p**assword | Hỏi mật khẩu. ⚠️ Xem cảnh báo về khoảng trắng bên dưới |
| `-h` | **h**ost | Máy chủ |
| `-P` | **P**ort | Cổng (⚠️ **chữ HOA** — `-p` thường là password!) |
| `-e` | **e**xecute | Chạy một câu SQL rồi thoát |
| `-N` | **N**o column names | Bỏ dòng tiêu đề |
| `-B` | **B**atch | Xuất dạng phân tách bằng Tab (hợp cho script) |

🛑 **Bẫy `-p` — dấu KHOẢNG TRẮNG đổi hoàn toàn ý nghĩa:**

```bash
mysql -u root -p mydb          # -p KHÔNG có giá trị -> HỎI mật khẩu, "mydb" là TÊN DATABASE
mysql -u root -pMatKhau mydb   # -p DÍNH LIỀN giá trị -> "MatKhau" là MẬT KHẨU
mysql -u root -p MatKhau       # 🛑 SAI: "MatKhau" bị hiểu là TÊN DATABASE -> lỗi unknown database
```

⇒ Quy tắc: **`-p` phải dính liền mật khẩu, không có dấu cách.** Nhưng cách này khiến mật khẩu lọt vào `ps aux` (mọi user trên máy đều đọc được) và lịch sử shell.

⭐ **Cách an toàn — file `~/.my.cnf`:**

```bash
cat > ~/.my.cnf <<'EOF'
[client]
user=app
password=matkhau
host=db-host
EOF
chmod 600 ~/.my.cnf
#     └─ bắt buộc: file chứa mật khẩu, không cho user khác đọc
# => từ giờ chỉ cần gõ `mysql mydb`, không cần cờ nào
```

| Lệnh SQL | Làm gì |
|---|---|
| `SHOW DATABASES;` | Liệt kê database |
| `USE <db>;` | Chọn database |
| `SHOW TABLES;` | Liệt kê bảng |
| `DESCRIBE <t>;` | Cấu trúc bảng (viết tắt: `DESC`) |
| `SHOW CREATE TABLE <t>;` | ⭐ Câu lệnh tạo bảng **đầy đủ** — gồm index, engine, charset |
| `SHOW PROCESSLIST;` | Query đang chạy |
| `SHOW FULL PROCESSLIST;` | ⭐ Như trên nhưng **không cắt cụt** câu query |
| `SHOW ENGINE INNODB STATUS;` | Chi tiết khoá, deadlock |
| `KILL <id>;` | Dừng một query |

⚠️ **Dấu `;` cuối câu là bắt buộc.** Quên `;` thì mysql hiện dấu nhắc `->` chờ bạn gõ tiếp — trông như **treo**. Gõ `;` rồi Enter, hoặc `\c` để huỷ câu đang gõ.

⭐ **`SHOW FULL PROCESSLIST` — vì sao phải có `FULL`?** Bản không `FULL` **cắt query còn 100 ký tự** ⇒ query dài bị chặt đúng chỗ cần xem. Khi truy tìm query chậm, luôn dùng `FULL`.

Đọc kết quả — cột **`Time`** (giây) và **`State`**:

| `State` | Nghĩa |
|---|---|
| `Sending data` | Đang đọc/xử lý dữ liệu (tên gây hiểu nhầm — **không phải** đang truyền mạng) |
| `Waiting for table metadata lock` | ⚠️ Bị chặn bởi **DDL** (ALTER TABLE) đang chạy |
| `Copying to tmp table` | ⚠️ Query phải tạo bảng tạm — thường **thiếu index** |
| `Locked` | Đang chờ khoá |

**Backup & Restore:**

```bash
mysqldump -u app -p --single-transaction --quick --routines --triggers mydb > backup.sql
#                    │                    │       │          └─ kèm trigger
#                    │                    │       └───────────── kèm stored procedure/function
#                    │                    └───────────────────── đọc từng dòng, KHÔNG nạp cả bảng vào RAM
#                    └──────────────────────────────────────── ⭐ backup NHẤT QUÁN mà KHÔNG KHOÁ BẢNG
```

⭐ **`--single-transaction` — cờ quan trọng nhất khi backup production.**

Không có nó, `mysqldump` **khoá toàn bộ bảng** trong suốt quá trình dump ⇒ ứng dụng **không ghi được** ⇒ **downtime** kéo dài hàng phút tới hàng giờ với DB lớn.

Có nó, dump chạy trong **một transaction** ⇒ thấy ảnh chụp nhất quán tại một thời điểm, **app vẫn ghi bình thường**.

🛑 **Giới hạn phải biết**: `--single-transaction` **chỉ hiệu lực với InnoDB**. Bảng dùng **MyISAM** vẫn bị khoá như thường. Kiểm tra engine trước:

```sql
SELECT table_name, engine FROM information_schema.tables WHERE table_schema='mydb';
```

⚠️ **`mysqldump` không tự backup user/quyền** (nằm ở database `mysql`). Muốn có, thêm `--all-databases` hoặc dump riêng.

**Restore — và mẹo tăng tốc:**

```bash
mysql -u app -p mydb < backup.sql

# File lớn -> xem tiến trình bằng pv (pipe viewer):
pv backup.sql | mysql -u app -p mydb
# └─ hiện thanh tiến trình + tốc độ + thời gian còn lại (cần cài pv)
```

⚠️ Restore **không tự xoá dữ liệu cũ** trừ khi file dump có sẵn lệnh `DROP TABLE` (mysqldump mặc định **có** thêm `DROP TABLE IF EXISTS`). Restore vào database **đang có dữ liệu khác** ⇒ dễ lẫn lộn ⇒ nên tạo database trống rồi restore vào đó.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa redis-cli — và vì sao KEYS * làm sập production</b></summary>

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-h` | **h**ost | Máy chủ |
| `-p` | **p**ort | Cổng, mặc định **6379** |
| `-a` | **a**uth | Mật khẩu ⚠️ lọt vào `ps aux` — xem cách an toàn bên dưới |
| `-n` | **n**umber | Chọn database số mấy (Redis có 16 DB đánh số 0-15) |
| `--scan` | | Duyệt key **an toàn** từ dòng lệnh |
| `--bigkeys` | | ⭐ Tìm key **chiếm nhiều RAM nhất** |
| `--stat` | | Thống kê realtime, tự cập nhật |
| `-i` | **i**nterval | Khoảng lặp lại (giây) |

⚠️ **Mật khẩu an toàn** — dùng biến môi trường thay vì `-a`:

```bash
REDISCLI_AUTH='matkhau' redis-cli -h redis-host ping
# └─ redis-cli tự đọc biến này; KHÔNG hiện trong `ps aux` như cờ -a
```

🛑🛑 **`KEYS *` — lệnh làm sập Redis production, phải hiểu vì sao:**

Redis là **đơn luồng**: nó xử lý **một lệnh tại một thời điểm**. `KEYS *` phải **duyệt toàn bộ** không gian khoá và **không nhường chỗ cho lệnh khác** trong suốt quá trình đó.

⇒ Với 10 triệu key, `KEYS *` chạy vài giây, và trong **toàn bộ vài giây đó Redis ĐỨNG IM** — mọi ứng dụng đang dùng Redis đều **treo**, timeout, health check fail, có thể kéo sập cả chuỗi dịch vụ.

⭐ **Cái gì thay thế? — `SCAN`**, duyệt theo **từng mẻ nhỏ**, có nhường chỗ giữa các mẻ:

```bash
redis-cli --scan --pattern 'session:*' --count 100
#          │      │                     └─ gợi ý mỗi mẻ ~100 key (không phải giới hạn tổng)
#          │      └─ lọc theo mẫu
#          └──────── tự động lặp SCAN cho tới hết, an toàn với production
```

| | `KEYS *` | `SCAN` |
|---|---|---|
| Chặn server | 🛑 **Có** — đứng im tới khi xong | ✅ Không |
| Kết quả | Chính xác tại một thời điểm | Có thể **trùng lặp**; key thêm/xoá giữa chừng có thể sót |
| Dùng ở production | ❌ **Tuyệt đối không** | ✅ Được |

⚠️ Trong `SCAN`, `--count` chỉ là **gợi ý cho mỗi vòng lặp**, không phải "lấy đúng N key rồi dừng" — hay bị hiểu nhầm.

**Lệnh nguy hiểm khác — cùng lý do đơn luồng:**

| Lệnh | Vì sao nguy hiểm | Dùng gì thay |
|---|---|---|
| `FLUSHDB` / `FLUSHALL` | 🔴 Xoá sạch, **không hỏi lại** | `FLUSHALL ASYNC` (xoá nền, không chặn) |
| `MONITOR` | In **mọi lệnh** của mọi client ⇒ ngốn CPU và băng thông | Chỉ bật **vài giây**, hoặc dùng `SLOWLOG` |
| `SAVE` | Ghi RAM xuống đĩa **đồng bộ** ⇒ chặn hoàn toàn | `BGSAVE` (chạy nền) |
| `DEBUG SLEEP` | Làm treo có chủ đích | — |

⭐ **Bộ lệnh chẩn đoán an toàn — chạy được trên production:**

```bash
redis-cli INFO memory | grep -E 'used_memory_human|maxmemory_human|evicted'
#                              └─ RAM đang dùng · giới hạn · số key bị ĐUỔI

redis-cli --bigkeys      # tìm key to nhất (nguyên nhân của "Redis phình RAM")
redis-cli --stat -i 2    # thống kê 2 giây/lần: số key, RAM, kết nối, ops/giây
redis-cli SLOWLOG GET 10 # ⭐ 10 lệnh CHẬM nhất gần đây — thay cho MONITOR
redis-cli CLIENT LIST | wc -l     # đếm số kết nối đang mở
```

⚠️ **Chỉ số quan trọng nhất khi Redis đầy RAM** — `evicted_keys`:

| Chỉ số | Ý nghĩa |
|---|---|
| `evicted_keys` tăng | Redis **đang phải xoá key** để lấy chỗ ⇒ cache miss tăng ⇒ tải dồn xuống database |
| `used_memory` ≈ `maxmemory` | Sắp/đã chạm trần |
| `mem_fragmentation_ratio` > 1.5 | RAM bị phân mảnh (không phải lỗi, nhưng đáng theo dõi) |

⚠️ **`TTL` trả về số âm — hai giá trị mang nghĩa khác nhau hoàn toàn:**

| Giá trị | Nghĩa |
|---|---|
| `-1` | Key **tồn tại** nhưng **KHÔNG có hạn** ⇒ sống mãi ⇒ ⚠️ nguồn gốc rò rỉ bộ nhớ |
| `-2` | Key **KHÔNG tồn tại** (đã hết hạn hoặc chưa từng có) |

⇒ Nhầm `-1` với "không có key" là chẩn đoán sai hoàn toàn. `-1` là **có key, và nó sẽ nằm đó vĩnh viễn**.

**Kiểm tra nhanh sức khoẻ:**

```bash
redis-cli ping                    # trả PONG = sống
redis-cli -h redis --no-auth-warning -a "$PASS" ping
#                   └─ tắt cảnh báo "dùng -a không an toàn" (khi buộc phải dùng trong script)
```

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa mongosh, chuỗi kết nối, và lệnh chẩn đoán</b></summary>

**Tiền đề — `mongosh` vs `mongo`:** `mongo` là shell **cũ, đã bị loại bỏ** từ MongoDB 6.0. Bản thay thế là **`mongosh`** (MongoDB Shell), hỗ trợ JavaScript hiện đại và tô màu cú pháp. Gõ `mongo` mà báo `command not found` trên server mới ⇒ **không phải chưa cài**, mà là đã đổi tên.

**Chuỗi kết nối (connection string) — bóc từng mảnh:**

```bash
mongosh "mongodb://app:matkhau@host1:27017,host2:27017/mydb?replicaSet=rs0&authSource=admin"
#         │        │   │       │                        │    │             └─ ⭐ user được TẠO ở database nào
#         │        │   │       │                        │    └─ tên replica set (bắt buộc khi có cụm)
#         │        │   │       │                        └─ database muốn dùng
#         │        │   │       └─ danh sách node, cách nhau bằng dấu phẩy
#         │        │   └─ mật khẩu (⚠️ ký tự đặc biệt phải mã hoá URL: @ -> %40)
#         │        └─ tên user
#         └─ giao thức (mongodb+srv:// nếu dùng bản ghi DNS SRV, ví dụ Atlas)
```

🛑 **`authSource` — nguyên nhân số 1 của lỗi "Authentication failed" dù mật khẩu đúng.** MongoDB lưu user **trong một database cụ thể**, thường là `admin`. Không ghi `authSource=admin` thì driver đi tìm user trong database `mydb` ⇒ **không thấy** ⇒ báo sai mật khẩu, gây hiểu lầm hoàn toàn.

⚠️ Mật khẩu chứa `@`, `:`, `/` phải **mã hoá URL** (`@` → `%40`), nếu không chuỗi kết nối bị cắt sai chỗ.

**Lệnh trong shell — cú pháp là JavaScript, không phải SQL:**

| Lệnh | Làm gì |
|---|---|
| `show dbs` | Liệt kê database |
| `use <db>` | Chọn database (⚠️ **tạo ngay cả khi chưa tồn tại** — không báo lỗi) |
| `show collections` | Liệt kê collection (tương đương "bảng") |
| `db.<c>.find().limit(5)` | ⭐ Truy vấn, **giới hạn 5 bản ghi** |
| `db.<c>.find().pretty()` | In JSON xuống dòng cho dễ đọc |
| `db.<c>.countDocuments()` | Đếm **chính xác** |
| `db.<c>.estimatedDocumentCount()` | Đếm **ước lượng** — nhanh hơn nhiều với collection lớn |
| `db.<c>.getIndexes()` | ⭐ Xem index hiện có |
| `db.currentOp()` | Thao tác đang chạy |
| `db.stats()` | Dung lượng database |

🛑 **`db.<c>.find()` trần trên collection lớn** sẽ kéo về hàng triệu bản ghi ⇒ treo shell, ngốn RAM server. **Luôn thêm `.limit()`**:

```javascript
db.users.find({ status: "active" }).limit(10).pretty()
//              └─ điều kiện lọc dạng đối tượng JSON
```

⭐ **`explain` — công cụ quan trọng nhất khi query chậm:**

```javascript
db.users.find({ email: "a@b.vn" }).explain("executionStats")
//                                          └─ chạy thật rồi báo cáo số liệu
```

Đọc kết quả — hai từ khoá quyết định:

| Giá trị `stage` | Nghĩa |
|---|---|
| `COLLSCAN` | 🛑 **Quét toàn bộ collection** — **KHÔNG dùng index** ⇒ nguyên nhân chậm |
| `IXSCAN` | ✅ Có dùng index |

⇒ Thấy `COLLSCAN` trên collection lớn là **phải tạo index**:

```javascript
db.users.createIndex({ email: 1 }, { background: true })
//                            │      └─ tạo NGẦM, không khoá collection (quan trọng ở production)
//                            └─ 1 = tăng dần, -1 = giảm dần
```

⚠️ So thêm `totalDocsExamined` với `nReturned`: xem **1 triệu** bản ghi để trả về **10** ⇒ index sai hoặc thiếu.

**Chẩn đoán query treo:**

```javascript
db.currentOp({ "secs_running": { $gt: 5 } })   // thao tác chạy quá 5 giây
db.killOp(12345)                               // dừng theo opid lấy được ở trên
```

**Backup & Restore:**

```bash
mongodump --uri="mongodb://user:pass@host:27017/mydb?authSource=admin" --out=./backup --gzip
#          │                                                            │            └─ nén lại
#          │                                                            └─ thư mục đích
#          └─ ⭐ dùng --uri thay vì --host/--db rời rạc: gọn và ít sai

mongorestore --uri="mongodb://..." --drop --gzip ./backup/mydb
#                                   └─ XOÁ collection cũ trước khi nạp
#                                      (không có --drop thì dữ liệu bị GỘP LẪN vào nhau)
```

⚠️ **`mongodump` không phải ảnh chụp nhất quán** trên replica set trừ khi thêm `--oplog`. Không có cờ đó, dữ liệu ghi **trong lúc dump** có thể tạo ra bản backup **không nhất quán giữa các collection**:

```bash
mongodump --uri="..." --oplog --out=./backup
#                      └─ ghi kèm nhật ký thao tác -> restore ra được một MỐC THỜI GIAN nhất quán
mongorestore --oplogReplay ./backup
```

⚠️ `mongodump` đọc **qua tầng ứng dụng** nên **chậm với dữ liệu lớn** (hàng trăm GB). Với quy mô đó nên dùng **snapshot mức ổ đĩa** (LVM/EBS/Longhorn) hoặc MongoDB Ops Manager.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa find, stat, ln, watch — đặc biệt find -delete</b></summary>

| Lệnh & cờ | Viết tắt của | Làm gì |
|---|---|---|
| `ls -lah` | **l**ong + **a**ll + **h**uman | Chi tiết + file ẩn + dung lượng dễ đọc |
| `find -name` | | Tìm theo tên, **phân biệt hoa/thường** |
| `find -iname` | **i**gnore case | Không phân biệt hoa/thường |
| `find -type f` | | Chỉ **f**ile thường (`d` = thư mục, `l` = symlink) |
| `find -size +100M` | | Lớn hơn 100MB (`-` = nhỏ hơn) |
| `find -mtime -1` | **m**odify time | Sửa **trong vòng** 1 ngày (`+30` = **cũ hơn** 30 ngày) |
| `find -delete` | | 🛑 Xoá luôn kết quả tìm được |
| `stat` | | Metadata đầy đủ: kích thước, quyền, inode, 3 mốc thời gian |
| `readlink -f` | | Đường dẫn **thật** sau khi đi hết chuỗi symlink |
| `ln -s` | **s**ymbolic | Tạo liên kết mềm |
| `watch -n 2` | | Chạy lặp lệnh mỗi 2 giây |

🛑 **`find ... -delete` — quy tắc bắt buộc: LUÔN chạy không có `-delete` trước.**

```bash
# BƯỚC 1 — XEM sẽ xoá những gì (bắt buộc, không có ngoại lệ)
find /var/log -name "*.log" -mtime +30 -type f

# BƯỚC 2 — chỉ khi danh sách trên đã ĐÚNG mới thêm -delete
find /var/log -name "*.log" -mtime +30 -type f -delete
#     │         │             │          │       └─ xoá, KHÔNG hỏi lại, KHÔNG vào thùng rác
#     │         │             │          └───────── chỉ file thường (⭐ tránh xoá nhầm thư mục)
#     │         │             └──────────────────── sửa lần cuối CŨ HƠN 30 ngày
#     │         └────────────────────────────────── khớp tên
#     └──────────────────────────────────────────── bắt đầu tìm từ đây (⚠️ gõ nhầm là thảm hoạ)
```

⚠️ **Thứ tự cờ trong `find` có ý nghĩa!** `-delete` đặt **trước** điều kiện lọc sẽ xoá **mọi thứ** rồi mới lọc:

```bash
find /data -delete -name "*.tmp"    # 🛑 SAI NGHIÊM TRỌNG: xoá SẠCH /data
find /data -name "*.tmp" -delete    # ✅ ĐÚNG: lọc trước, xoá sau
```

⚠️ **Ba mốc thời gian khác nhau, hay bị nhầm:**

| Cờ | Viết tắt | Đổi khi nào |
|---|---|---|
| `-mtime` | **m**odify | **Nội dung** file thay đổi |
| `-atime` | **a**ccess | File được **đọc** (⚠️ nhiều hệ thống tắt để tăng tốc ⇒ **không đáng tin**) |
| `-ctime` | **c**hange | **Metadata** đổi (quyền, chủ sở hữu) — **KHÔNG phải** "create time" |

🛑 `-ctime` **không phải thời gian tạo file** — đây là hiểu nhầm rất phổ biến. Linux truyền thống **không lưu** thời gian tạo file (ext4 mới có `crtime` nhưng `find` không tra được trực tiếp).

**`-exec` — khi cần làm gì đó phức tạp hơn xoá:**

```bash
find /var/log -name "*.log" -mtime +7 -exec gzip {} \;
#                                      │         │  └─ dấu chấm phẩy kết thúc (phải escape bằng \)
#                                      │         └──── {} = chỗ thay bằng TÊN FILE tìm được
#                                      └────────────── thực thi lệnh trên MỖI file

find . -name "*.log" -exec gzip {} +
#                                  └─ dấu + thay cho \; : gom NHIỀU file vào MỘT lần gọi lệnh
#                                     -> nhanh hơn nhiều khi có hàng nghìn file
```

⚠️ **Tên file có khoảng trắng làm hỏng pipeline.** `find ... | xargs rm` sẽ tách `my file.txt` thành hai file. Cách an toàn:

```bash
find . -name "*.tmp" -print0 | xargs -0 rm
#                     │              └─ đọc theo ký tự NULL
#                     └──────────────── ngăn cách bằng ký tự NULL thay vì xuống dòng
#                                       (NULL là ký tự KHÔNG THỂ có trong tên file -> luôn đúng)
```

**`stat` — đọc thông tin thật của file:**

```bash
stat -c '%s %U %a %n' app.log
#     │   │  │  │  └─ n = name
#     │   │  │  └──── a = quyền dạng SỐ (644)
#     │   │  └─────── U = tên chủ sở hữu
#     │   └────────── s = size (byte)
#     └────────────── c = custom format: chỉ in đúng thứ mình cần
```

**Symlink — và bẫy đường dẫn tương đối:**

```bash
ln -s /opt/app/v2 /opt/app/current
#     │           └─ TÊN LIÊN KẾT (cái mới tạo ra)
#     └───────────── ĐÍCH (⭐ nên dùng đường dẫn TUYỆT ĐỐI)

readlink -f /opt/app/current      # đi hết chuỗi symlink -> ra đường dẫn thật
ls -l /opt/app/current            # xem symlink trỏ đi đâu (cột cuối có mũi tên ->)
```

⚠️ Symlink tạo bằng đường dẫn **tương đối** sẽ **hỏng khi di chuyển** liên kết sang thư mục khác. Dùng đường dẫn tuyệt đối là an toàn nhất.

⚠️ **`watch` và dấu nháy:**

```bash
watch -n 2 'kubectl get pods -n ai-hub'
#      │    └─ ⭐ BẮT BUỘC bọc nháy khi lệnh có cờ/pipe,
#      │       nếu không `watch` nuốt mất phần sau và chạy sai
#      └────── n = số giây giữa mỗi lần chạy

watch -d -n 2 'kubectl get pods'
#      └─ d = differences: TÔ SÁNG phần vừa thay đổi so với lần trước ⭐ rất tiện khi theo dõi rollout
```

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa hệ thống quyền Linux từ số 0</b></summary>

**Tiền đề — quyền Linux đọc thế nào?** Mỗi file có **9 bit quyền**, chia **3 nhóm × 3 quyền**:

```
-rwxr-xr--
│└┬┘└┬┘└┬┘
│ │  │  └─ OTHERS: mọi người còn lại  -> r--  = chỉ đọc
│ │  └──── GROUP:  nhóm sở hữu        -> r-x  = đọc + chạy
│ └─────── USER:   chủ sở hữu         -> rwx  = đọc + ghi + chạy
└───────── loại: `-` file thường · `d` thư mục · `l` symlink
```

**Đổi ra số — mỗi quyền một giá trị, cộng lại:**

| Quyền | Chữ | Số |
|---|---|---|
| Đọc | `r` | **4** |
| Ghi | `w` | **2** |
| Chạy | `x` | **1** |

```
chmod 755 = rwx r-x r-x
#      │││
#      ││└─ others: 5 = 4+1 = r-x (đọc + chạy)
#      │└── group:  5 = 4+1 = r-x
#      └─── user:   7 = 4+2+1 = rwx (toàn quyền)
```

| Số | Nghĩa | Dùng cho |
|---|---|---|
| `755` | Chủ toàn quyền, còn lại đọc+chạy | **Thư mục**, script thực thi |
| `644` | Chủ đọc/ghi, còn lại chỉ đọc | **File thường**, file cấu hình |
| `600` | ⭐ **Chỉ chủ** đọc/ghi | **SSH key**, `.pgpass`, `.my.cnf`, file chứa secret |
| `700` | Chỉ chủ toàn quyền | Thư mục `~/.ssh` |
| `777` | 🛑 **Mọi người toàn quyền** | ❌ **Gần như không bao giờ đúng** |

🛑 **`chmod 777` không phải "cách sửa lỗi permission"** — nó là cách **tắt bỏ bảo mật**. Mọi user trên máy (kể cả tiến trình bị chiếm quyền) đều **ghi đè được** file đó. Lỗi permission thật sự cần sửa **chủ sở hữu** (`chown`), không phải mở toang quyền.

⭐ **`x` trên THƯ MỤC mang nghĩa khác hoàn toàn** — mảnh ghép hay thiếu:

| | Trên **file** | Trên **thư mục** |
|---|---|---|
| `r` | Đọc nội dung | **Liệt kê** tên file bên trong (`ls`) |
| `w` | Sửa nội dung | **Tạo/xoá** file bên trong |
| `x` | **Chạy** như chương trình | ⭐ **ĐI VÀO** (`cd`), truy cập file bên trong |

⇒ Hệ quả thực tế: thư mục có `r` nhưng **không có `x`** ⇒ `ls` thấy tên file nhưng **mở file nào cũng "Permission denied"**. Ngược lại có `x` mà không `r` ⇒ **không `ls` được**, nhưng biết chính xác tên file thì vẫn mở được.

⇒ Đây là lý do thư mục hầu như luôn là `755` chứ không phải `644`: thiếu `x` là **không ai vào được**.

**Cú pháp chữ — an toàn hơn khi chỉ muốn thêm/bớt một quyền:**

```bash
chmod +x script.sh        # thêm quyền chạy cho MỌI nhóm (chịu ảnh hưởng của umask)
chmod u+x script.sh       # chỉ thêm cho user (chủ sở hữu)
chmod go-w file           # BỚT quyền ghi của group và others
chmod u=rw,go=r file      # ĐẶT chính xác: = ghi đè, khác với + (thêm) và - (bớt)
#       │    └─ g=group, o=others
#       └────── u=user
```

⭐ **`chmod -R` trên thư mục — bẫy nguy hiểm:**

```bash
chmod -R 755 /var/www     # 🛑 gán 755 cho CẢ FILE lẫn thư mục
#                            -> mọi file .conf, .env đều thành CHẠY ĐƯỢC (không cần thiết)
```

Cách đúng — tách riêng file và thư mục:

```bash
find /var/www -type d -exec chmod 755 {} +    # thư mục: 755 (cần x để vào)
find /var/www -type f -exec chmod 644 {} +    # file:    644 (không cần x)
# hoặc gọn hơn, dùng chữ X HOA:
chmod -R u=rwX,go=rX /var/www
#              └─ X HOA = thêm x CHỈ CHO THƯ MỤC (và file vốn đã có x)
#                 khác hẳn x thường (thêm cho tất cả)
```

**`chown` — đổi chủ sở hữu:**

```bash
chown -R app:app /var/www
#     │   │   └─ nhóm
#     │   └───── user
#     └───────── đệ quy

chown :nhommoi file       # chỉ đổi NHÓM (bỏ trống phần user trước dấu :)
chown --reference=file-mau file-dich   # copy chủ sở hữu từ file khác
```

⚠️ **Trong container, quyền tính theo SỐ (UID/GID), không theo tên.** File thuộc user `app` (UID 1001) trong container, khi mount ra host sẽ thuộc về **UID 1001 của host** — có thể là một user hoàn toàn khác, hoặc không tồn tại (hiện là số trần). Đây là gốc của lỗi "Permission denied" khi dùng bind mount.

```bash
id -u && id -g            # xem UID/GID của mình để khớp với container
docker run --user "$(id -u):$(id -g)" ...    # chạy container bằng đúng danh tính của bạn
```

**`umask` — quyền mặc định khi tạo file mới:**

```bash
umask         # thường ra 0022
# Cách tính: file mới = 666 - umask = 644 · thư mục mới = 777 - umask = 755
#            (file KHÔNG bao giờ tự có quyền x, nên xuất phát từ 666 chứ không phải 777)
```

**`sudo` — vài dạng hay dùng:**

| Lệnh | Làm gì |
|---|---|
| `sudo -i` | Mở shell root, **nạp cả môi trường** của root |
| `sudo su - <user>` | Chuyển sang user khác (dấu `-` = nạp môi trường đăng nhập đầy đủ) |
| `sudo !!` | ⭐ Chạy lại **lệnh vừa gõ** với quyền root (khi quên sudo) |
| `sudo -l` | ⭐ Liệt kê **mình được phép chạy gì** bằng sudo |

⚠️ `sudo su -` và `sudo su` khác nhau: **có dấu `-`** thì nạp `.bashrc`/`.profile` của user đích (PATH, biến môi trường đúng); **không có** thì giữ môi trường cũ ⇒ hay gây lỗi "command not found" khó hiểu.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa lệnh quản lý user & nhóm</b></summary>

| Lệnh | Trả về / làm gì |
|---|---|
| `whoami` | Tên user **hiệu lực** hiện tại |
| `id` | UID, GID và **mọi nhóm** đang thuộc về |
| `id <user>` | Thông tin của user khác |
| `groups` | Chỉ danh sách nhóm |
| `who` | Ai đang đăng nhập |
| `w` | Như `who` **+ họ đang chạy lệnh gì** + load average |
| `last` | **Lịch sử** đăng nhập (đọc từ `/var/log/wtmp`) |
| `lastb` | Lịch sử đăng nhập **THẤT BẠI** ⭐ (dấu hiệu bị dò mật khẩu) |

⭐ **`id` là lệnh đáng dùng nhất** — nó cho biết cả nhóm phụ, thứ quyết định bạn truy cập được gì:

```bash
id
# uid=1000(kiennv) gid=1000(kiennv) groups=1000(kiennv),27(sudo),999(docker)
#     │              │                              │        └─ ⭐ thuộc nhóm docker = có quyền ngang root!
#     │              │                              └─ được dùng sudo
#     │              └─ nhóm CHÍNH (file mới tạo sẽ thuộc nhóm này)
#     └─ số định danh user (cái Linux thực sự dùng; tên chỉ để cho người đọc)
```

🛑 **Nhóm `docker` tương đương quyền root** — mảnh ghép an toàn hay bị bỏ qua. Ai vào được nhóm `docker` đều có thể mount toàn bộ ổ đĩa host vào container và chiếm quyền root:

```bash
docker run -v /:/host -it alpine chroot /host sh   # <- toàn quyền hệ thống, không cần sudo
```

⇒ Vì vậy, thêm user vào nhóm `docker` phải được coi là **cấp quyền quản trị**, không phải "cho tiện dùng docker".

**Tạo và quản lý user:**

```bash
useradd -m -s /bin/bash -G docker,sudo deployer
#        │  │            └─ G = nhóm PHỤ, nhiều nhóm cách nhau bằng dấu phẩy
#        │  └─ shell đăng nhập (không có thì mặc định /bin/sh hoặc nologin)
#        └──── m = tạo luôn thư mục /home/deployer (⚠️ THIẾU cờ này là user không có home)

adduser deployer      # ⭐ trên Debian/Ubuntu: bản thân thiện, HỎI TỪNG BƯỚC, tự tạo home
```

⚠️ **`useradd` vs `adduser`**: `useradd` là lệnh gốc, **không tự tạo home**, cần đủ cờ. `adduser` là script bao bọc (chỉ có trên Debian/Ubuntu), hỏi tương tác và đặt mặc định hợp lý. Trên RHEL/CentOS **chỉ có `useradd`**.

🛑 **`usermod -G` KHÔNG có `-a` là XOÁ hết nhóm cũ:**

```bash
usermod -aG docker deployer     # ✅ ĐÚNG: a = append, THÊM vào các nhóm đang có
usermod -G docker deployer      # 🛑 SAI: user bị GỠ khỏi MỌI nhóm khác, chỉ còn docker
#                                  -> mất luôn quyền sudo, không cảnh báo gì
```

⇒ Đây là lỗi kinh điển khiến người vừa được cấp quyền docker thì **mất quyền sudo**. Kiểm chứng ngay sau khi chạy:

```bash
id deployer     # đối chiếu danh sách nhóm có đủ như trước không
```

⚠️ **Thay đổi nhóm chỉ có hiệu lực ở phiên đăng nhập MỚI.** Vừa `usermod -aG docker` xong mà gõ `docker ps` vẫn báo permission denied ⇒ **không phải lệnh sai**, mà phiên hiện tại vẫn giữ danh sách nhóm cũ. Cách xử lý:

```bash
newgrp docker     # nạp nhóm mới cho shell hiện tại (không cần đăng xuất)
# hoặc: đăng xuất rồi đăng nhập lại (chắc chắn nhất)
```

**Mật khẩu & khoá tài khoản:**

```bash
passwd deployer            # đặt/đổi mật khẩu
passwd -l deployer         # l = lock: KHOÁ đăng nhập bằng mật khẩu (SSH key VẪN vào được!)
passwd -S deployer         # S = status: xem trạng thái (P=có mật khẩu, L=khoá, NP=trống)
chage -l deployer          # xem hạn mật khẩu, ngày hết hạn tài khoản
usermod -s /sbin/nologin deployer   # ⭐ CHẶN đăng nhập hoàn toàn (dùng cho user chạy service)
```

🛑 **`passwd -l` KHÔNG chặn được SSH key** — chỉ khoá đường đăng nhập bằng mật khẩu. Muốn chặn triệt để user đã bị thu hồi quyền:

```bash
usermod -s /sbin/nologin deployer          # 1. bỏ shell đăng nhập
usermod -L -e 1 deployer                   # 2. khoá + đặt hạn tài khoản vào quá khứ
mv /home/deployer/.ssh/authorized_keys{,.disabled}   # 3. ⭐ vô hiệu hoá SSH key
#                                    └─ cú pháp mở rộng dấu ngoặc của bash:
#                                       đổi tên thành authorized_keys.disabled
```

**Xoá user:**

```bash
userdel deployer            # xoá user, GIỮ lại thư mục /home
userdel -r deployer         # r = remove: xoá CẢ thư mục home và hộp thư
```

⚠️ Trước khi xoá, **kiểm tra user còn tiến trình nào đang chạy không** — xoá user mà tiến trình còn sống sẽ để lại tiến trình "mồ côi" thuộc về một UID không còn tên:

```bash
ps -u deployer          # liệt kê tiến trình của user này
pkill -u deployer       # dừng hết trước khi xoá
```

⚠️ **File cũ của user bị xoá vẫn nằm rải rác** trên hệ thống, giờ hiện chủ sở hữu là **số UID trần** (ví dụ `1003`). Tìm chúng:

```bash
find / -nouser -o -nogroup 2>/dev/null
#       │         └─ không thuộc nhóm nào còn tồn tại
#       └─────────── không thuộc user nào còn tồn tại
```

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa cờ SSH, tunnel, và file ~/.ssh/config</b></summary>

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-p` | **p**ort | Cổng SSH (⚠️ **chữ thường**; `scp` lại dùng `-P` HOA) |
| `-i` | **i**dentity | Dùng **private key** cụ thể |
| `-v` | **v**erbose | Debug. `-vv`, `-vvv` càng chi tiết hơn |
| `-L` | **L**ocal forward | ⭐ Mở port ở **máy bạn**, chuyển tới đích qua server |
| `-R` | **R**emote forward | Ngược lại: mở port **trên server**, chuyển về máy bạn |
| `-D` | **D**ynamic | SOCKS proxy (như một VPN nhẹ) |
| `-N` | **N**o command | **Không** mở shell — chỉ dựng tunnel |
| `-f` | **f**ork | Đẩy xuống chạy nền |
| `-J` | **J**ump | Nhảy qua máy trung gian (bastion) |
| `-o` | **o**ption | Đặt tuỳ chọn ngay trên dòng lệnh |

⭐ **`-L` — Local port forward, bóc kỹ vì hay gõ nhầm thứ tự:**

```bash
ssh -L 15432:db-noibo:5432 user@bastion
#      │     │         │    └─ SSH VÀO máy này (máy trung gian)
#      │     │         └────── port ở ĐÍCH CUỐI
#      │     └──────────────── ĐÍCH CUỐI, tính TỪ GÓC NHÌN CỦA BASTION
#      └────────────────────── port mở trên MÁY BẠN
```

⇒ Sau lệnh này: ở máy bạn gõ `psql -h localhost -p 15432` là **vào được database nội bộ** mà máy bạn **không hề có đường mạng trực tiếp** tới nó.

🛑 **Điểm hay hiểu sai nhất**: `db-noibo` được phân giải **trên bastion**, không phải trên máy bạn. Máy bạn không cần biết tên đó là gì, không cần DNS phân giải được.

**Dựng tunnel không mở shell — dạng hay dùng nhất:**

```bash
ssh -fNL 15432:db-noibo:5432 user@bastion
#    │││
#    ││└─ L: local forward
#    │└── N: không chạy lệnh nào, chỉ giữ tunnel
#    └─── f: đẩy xuống nền, trả prompt lại ngay
```

Đóng tunnel sau khi xong (vì nó chạy nền, Ctrl+C không tắt được):

```bash
pkill -f "ssh -fNL 15432"        # tìm theo dòng lệnh rồi dừng
ss -tlnp | grep 15432            # kiểm tra port đã được giải phóng chưa
```

⭐ **`-J` (Jump) — thay cho tunnel lồng nhau rối rắm:**

```bash
ssh -J user@bastion user@server-noibo
#    └─ tự động nhảy qua bastion rồi tới đích, KHÔNG cần dựng tunnel thủ công
#       nhiều chặng: -J user@bastion1,user@bastion2
```

⭐ **`~/.ssh/config` — viết một lần, đỡ gõ mãi mãi:**

```
Host prod-db
    HostName 10.20.30.40
    User deployer
    Port 2222
    IdentityFile ~/.ssh/id_ed25519_prod
    ProxyJump bastion              # <- tự nhảy qua bastion, tương đương -J
    ServerAliveInterval 60         # <- ⭐ gửi gói giữ nhịp mỗi 60 giây
    ServerAliveCountMax 3          #    3 lần không hồi đáp thì mới ngắt

Host *
    AddKeysToAgent yes
    ServerAliveInterval 60
```

⇒ Từ đó chỉ cần gõ **`ssh prod-db`**. `scp`, `rsync`, `git` **đều đọc file này**, nên cấu hình một lần dùng cho mọi công cụ.

⭐ **`ServerAliveInterval` — chữa dứt điểm bệnh "SSH tự rớt khi để yên vài phút".** Nguyên nhân là firewall/NAT ở giữa **dọn bỏ kết nối im lặng**. Gói giữ nhịp làm kết nối luôn "có hoạt động" nên không bị dọn.

**Tạo và cài key:**

```bash
ssh-keygen -t ed25519 -C "kiennv@company.vn" -f ~/.ssh/id_ed25519_prod
#           │          │                      └─ tên file (⭐ đặt riêng cho từng hệ thống)
#           │          └───── C = comment: ghi chú để sau này biết key của ai/việc gì
#           └──────────────── t = type: ed25519 ⭐ hiện đại, ngắn, nhanh, an toàn
#                             (RSA cũ hơn: nếu buộc phải dùng thì tối thiểu -b 4096)

ssh-copy-id -i ~/.ssh/id_ed25519_prod.pub user@host
#            └─ ⚠️ đẩy file .pub (khoá CÔNG KHAI). TUYỆT ĐỐI không đưa file không có .pub đi đâu
```

🛑 **Quyền file SSH bắt buộc — sai là SSH TỪ CHỐI, và thông báo lỗi không nói rõ:**

| Đường dẫn | Quyền bắt buộc |
|---|---|
| `~/.ssh` | `700` |
| `~/.ssh/id_ed25519` (private key) | **`600`** |
| `~/.ssh/id_ed25519.pub` | `644` |
| `~/.ssh/authorized_keys` | `600` |
| `~` (thư mục home) | Không được cho **group/others quyền ghi** |

```bash
chmod 700 ~/.ssh && chmod 600 ~/.ssh/id_* ~/.ssh/authorized_keys
```

⇒ Lỗi `WARNING: UNPROTECTED PRIVATE KEY FILE!` chính là ca này. Và ít ai ngờ: **thư mục home bị `chmod 777`** cũng khiến SSH key **im lặng không hoạt động**.

**Debug khi không vào được — đọc `-v` đúng chỗ:**

```bash
ssh -vvv user@host 2>&1 | grep -iE "debug1: (Offering|Authentications|Next auth)"
#    │                            └─ ba dòng quan trọng nhất trong đống output
#    └─ ba chữ v = mức chi tiết cao nhất
```

| Dòng trong output | Nghĩa |
|---|---|
| `Offering public key: ...` | Client **đang thử** key này |
| `Authentications that can continue: publickey,password` | Server **chấp nhận** những cách nào |
| `Permission denied (publickey)` | Server **chỉ nhận key**, và key của bạn **không khớp** |

⇒ Nếu không thấy dòng `Offering` nào ⇒ client **chưa tìm ra key** ⇒ vấn đề ở phía bạn (sai đường dẫn `-i`, sai config), không phải ở server.

⚠️ Debug từ **phía server** (khi có quyền): `journalctl -u sshd -f` — server ghi rõ lý do từ chối (sai quyền file, user bị khoá, key không có trong `authorized_keys`).

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa scp/rsync — và dấu `/` cuối đường dẫn quyết định mọi thứ</b></summary>

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-r` (scp) | **r**ecursive | Copy cả thư mục |
| `-P` (scp) | **P**ort | ⚠️ **Chữ HOA** — `ssh` dùng `-p` thường! |
| `-a` (rsync) | **a**rchive | ⭐ Gói gọn: đệ quy + giữ quyền, chủ sở hữu, thời gian, symlink |
| `-v` (rsync) | **v**erbose | Liệt kê file đang truyền |
| `-z` (rsync) | **z**ip | Nén khi truyền (⭐ đáng giá khi mạng chậm) |
| `-h` (rsync) | **h**uman | Số liệu dễ đọc |
| `--progress` | | Thanh tiến trình từng file |
| `--delete` | | 🛑 **Xoá** file ở đích mà nguồn không có |
| `--dry-run` / `-n` | | ⭐ **Diễn tập**, không truyền thật |
| `-e` | **e**xecute | Chỉ định lệnh kết nối (để đổi port SSH) |
| `--exclude` | | Bỏ qua theo mẫu |

⚠️ **`-P` hoa của scp vs `-p` thường của ssh** — nguồn gõ nhầm kinh điển:

```bash
ssh  -p 2222 user@host           # ssh:  p THƯỜNG
scp  -P 2222 file user@host:/tmp # scp:  P HOA (p thường ở scp nghĩa là "giữ mốc thời gian")
rsync -e "ssh -p 2222" src dst   # rsync: không có cờ port riêng, phải truyền qua -e
```

🛑🛑 **Dấu `/` ở CUỐI đường dẫn nguồn của rsync — khác biệt lớn nhất, sai là đổ nhầm chỗ:**

```bash
rsync -av /data/  user@host:/backup/     # CÓ dấu / -> đổ NỘI DUNG BÊN TRONG data vào /backup
#              └─ kết quả: /backup/file1, /backup/file2

rsync -av /data   user@host:/backup/     # KHÔNG có / -> đổ CẢ THƯ MỤC data vào trong
#                                           kết quả: /backup/data/file1, /backup/data/file2
```

⇒ Cách nhớ: **dấu `/` cuối = "lấy ruột bên trong"** · **không có `/` = "lấy cả cái hộp"**.

🛑 **`--delete` — kết hợp với sai dấu `/` là thảm hoạ.** Nó xoá **mọi thứ ở đích không có trong nguồn**. Chỉ định nhầm nguồn ⇒ **xoá sạch thư mục đích**.

⇒ **Quy tắc bắt buộc: luôn chạy `--dry-run` trước khi dùng `--delete`:**

```bash
# BƯỚC 1 — diễn tập, đọc kỹ danh sách sẽ xoá
rsync -avn --delete /data/ user@host:/backup/
#         └─ n = dry-run: chỉ IN RA sẽ làm gì, KHÔNG đụng vào file
#            (dòng bắt đầu bằng "deleting " chính là thứ sắp mất)

# BƯỚC 2 — chỉ khi danh sách đã đúng
rsync -av --delete /data/ user@host:/backup/
```

⭐ **`rsync` hơn `scp` ở đâu?**

| | `scp` | `rsync` |
|---|---|---|
| Truyền lại lần 2 | **Copy lại toàn bộ** | ⭐ **Chỉ gửi phần khác biệt** |
| Bị đứt giữa chừng | Phải làm lại từ đầu | ⭐ Chạy lại là **tiếp tục** |
| Giữ quyền/thời gian | Hạn chế | ✅ Đầy đủ (với `-a`) |
| Loại trừ file | ❌ Không | ✅ `--exclude` |
| Xem trước | ❌ Không | ✅ `--dry-run` |

⇒ **Gần như luôn nên dùng `rsync`.** `scp` chỉ tiện cho một file nhỏ, và bản OpenSSH mới đã coi `scp` là **lỗi thời** (nó chuyển sang chạy nền bằng giao thức SFTP).

**Đồng bộ thực tế — bộ cờ hay dùng:**

```bash
rsync -avz --progress --exclude={'.git','node_modules','*.log'} \
      -e "ssh -p 2222 -i ~/.ssh/id_ed25519_prod" \
      ./app/ deployer@server:/opt/app/
#      │                    └─ đích
#      └─ nguồn (⭐ CÓ dấu / = đổ ruột vào /opt/app)
```

⚠️ **`-a` không bao gồm `-z`.** Mạng nội bộ nhanh thì `-z` **làm chậm hơn** (tốn CPU nén). Chỉ dùng `-z` khi đường truyền là nút thắt (VPN, Internet).

⚠️ **`-a` giữ nguyên chủ sở hữu** ⇒ cần **quyền root ở đích** mới đặt được, nếu không rsync báo lỗi hoặc âm thầm gán cho user hiện tại. Không cần giữ chủ sở hữu thì bỏ bớt:

```bash
rsync -rlptDvz src/ dst/      # như -a nhưng KHÔNG giữ owner/group
#      ││││└─ D = giữ file thiết bị
#      │││└── t = giữ mốc thời gian
#      ││└─── p = giữ quyền
#      │└──── l = giữ symlink
#      └───── r = đệ quy
```

⭐ **Kiểm tra sau khi truyền** — đừng tin "không báo lỗi là xong":

```bash
rsync -avn --checksum /data/ user@host:/backup/
#          └─ so sánh bằng NỘI DUNG (checksum) thay vì kích thước+thời gian
#             chậm hơn nhưng chắc chắn; không in ra gì = hai bên GIỐNG HỆT
```

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa cờ tar — cụm chữ cái khó nhớ nhất Linux</b></summary>

**Vì sao `tar` khó nhớ?** Vì tên nó là **t**ape **ar**chive — sinh ra từ thời lưu vào **băng từ**, nên cú pháp giữ lại lối cũ: các chữ cái **dính liền nhau, không có dấu gạch**.

| Chữ | Viết tắt của | Làm gì |
|---|---|---|
| `c` | **c**reate | **TẠO** file nén mới |
| `x` | e**x**tract | **GIẢI NÉN** |
| `t` | lis**t** | **XEM** nội dung, **không** giải nén |
| `z` | g**z**ip | Nén/giải bằng gzip (`.gz`) |
| `j` | b**j**zip2 | Bzip2 (`.bz2`) — nén nhỏ hơn, chậm hơn |
| `J` | **J** = xz | XZ (`.xz`) — nhỏ nhất, chậm nhất |
| `v` | **v**erbose | Liệt kê từng file khi chạy |
| `f` | **f**ile | ⭐ **Chỉ định tên file** — phải là **chữ CUỐI CÙNG** |
| `-C` | **C**hange dir | Chuyển thư mục **trước khi** làm việc |

⭐ **Ba cụm cần thuộc — chỉ khác chữ đầu:**

```bash
tar -czvf archive.tar.gz thumuc/    # TẠO   (c)
tar -xzvf archive.tar.gz            # GIẢI  (x)
tar -tzvf archive.tar.gz            # XEM   (t) - an toàn, không ghi gì ra đĩa
#    ││││
#    │││└─ f = file: tên file, PHẢI đứng cuối vì chữ ngay sau f là tên file
#    ││└── v = verbose (bỏ được nếu không muốn ngập màn hình)
#    │└─── z = gzip
#    └──── c/x/t = tạo / giải / xem
```

🛑 **`f` phải là chữ CUỐI.** Gõ `tar -cfzv archive.tar.gz thumuc/` ⇒ tar hiểu tên file là **`z`** ⇒ tạo ra một file tên `z` và báo lỗi khó hiểu.

⭐ **Luôn `t` (xem) trước khi `x` (giải nén)** — vì có loại "tar bomb": file nén không có thư mục gốc, giải ra là **hàng trăm file đổ thẳng vào thư mục hiện tại**:

```bash
tar -tzvf archive.tar.gz | head       # xem trước 10 dòng: mọi thứ có nằm trong MỘT thư mục không?
```

**`-C` — giải nén vào đúng chỗ mình muốn:**

```bash
tar -xzvf backup.tar.gz -C /opt/app/
#                        └─ CHUYỂN vào thư mục này rồi mới giải nén
#                           (thay cho: cd /opt/app && tar -xzvf ...)

tar -czvf app.tar.gz -C /opt app
#                     │      └─ nén thư mục "app"
#                     └─ đứng từ /opt mà nhìn
#  => bên trong file nén, đường dẫn là "app/..." chứ KHÔNG phải "/opt/app/..."
```

⚠️ **Vì sao cần `-C` khi nén?** Nếu gõ `tar -czvf app.tar.gz /opt/app`, tar cảnh báo *"Removing leading `/`"* và lưu đường dẫn `opt/app/...`. Giải nén ra sẽ tạo thêm tầng `opt/app` không mong muốn. Dùng `-C` cho đường dẫn bên trong gọn và đúng ý.

**So sánh các kiểu nén — chọn theo tình huống:**

| Kiểu | Cờ | Tốc độ | Tỷ lệ nén | Dùng khi |
|---|---|---|---|---|
| gzip | `z` | Nhanh | Trung bình | ⭐ Mặc định, tương thích rộng nhất |
| bzip2 | `j` | Chậm | Tốt | Ít dùng |
| xz | `J` | Rất chậm | ⭐ Tốt nhất | Lưu trữ lâu dài, đẩy qua mạng chậm |
| Không nén | (bỏ) | Nhanh nhất | Không | Dữ liệu vốn đã nén (ảnh, video, `.gz`) |

💡 Bản `tar` mới **tự nhận diện kiểu nén** khi giải nén — `tar -xvf file.tar.xz` chạy được mà không cần `J`. Nhưng khi **tạo** thì vẫn phải ghi rõ.

**Loại trừ và giữ đúng thứ cần:**

```bash
tar -czvf app.tar.gz --exclude='node_modules' --exclude='*.log' -C /opt app
#                     └─ đặt TRƯỚC đường dẫn nguồn; dùng nhiều lần được
```

⚠️ Bọc mẫu trong **nháy đơn** để shell không tự mở rộng `*` trước khi tar nhìn thấy.

**zip / unzip — khi phải trao đổi với Windows:**

```bash
zip -r archive.zip thumuc/ -x '*.log' '*/node_modules/*'
#    │                      └─ x = exclude (⚠️ zip dùng -x, tar dùng --exclude)
#    └─ r = recursive (⚠️ tar KHÔNG cần cờ này, zip thì CÓ)

unzip -l archive.zip           # l = list: XEM trước, không giải
unzip archive.zip -d /opt/app  # d = directory: giải vào thư mục chỉ định (tar dùng -C)
```

⚠️ **`zip` KHÔNG giữ quyền file Unix** đầy đủ và hay làm hỏng dấu tiếng Việt trong tên file khi qua lại giữa Windows/Linux. ⇒ Trong nội bộ Linux, **luôn ưu tiên `tar.gz`**.

**Vài mẹo hay dùng:**

```bash
# Nén và truyền thẳng qua SSH, KHÔNG tạo file trung gian (tiết kiệm disk)
tar -czf - /opt/app | ssh user@host "cat > /backup/app.tar.gz"
#         └─ dấu - = ghi ra ĐẦU RA CHUẨN thay vì ra file

# Xem dung lượng thật bên trong mà không giải nén
tar -tzvf archive.tar.gz | awk '{s+=$3} END {print s/1024/1024 " MB"}'
#                                │            └─ đổi byte sang MB
#                                └─ cột 3 là kích thước từng file
```

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa TOÀN BỘ cờ curl hay dùng</b></summary>

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-I` | head (chữ **I** hoa) | Chỉ lấy **header**, gửi request kiểu `HEAD` |
| `-i` | **i**nclude | ⭐ Lấy **header + nội dung** (khác `-I` hoa!) |
| `-v` | **v**erbose | Chi tiết: DNS, TCP, bắt tay TLS, header hai chiều |
| `-L` | **L**ocation | **Đi theo** chuyển hướng (301/302) |
| `-X` | request method | Đổi phương thức: `POST`, `PUT`, `DELETE` |
| `-d` | **d**ata | Nội dung gửi lên (tự thành `POST` nếu chưa có `-X`) |
| `-H` | **H**eader | Thêm header |
| `-o` | **o**utput | Lưu vào file |
| `-O` | **O** hoa | Lưu, **giữ nguyên tên file** trên URL |
| `-s` | **s**ilent | Tắt thanh tiến trình |
| `-S` | **S**how error | Vẫn hiện lỗi khi đang `-s` (cặp `-sS` rất hay dùng) |
| `-k` | (insecure) | 🛑 **Bỏ qua kiểm tra chứng chỉ TLS** |
| `-w` | **w**rite-out | In thêm số liệu sau khi xong |
| `-f` | **f**ail | Trả **mã thoát khác 0** khi HTTP lỗi (⭐ cần cho script) |
| `--resolve` | | Ép một domain trỏ về IP chỉ định |
| `-u` | **u**ser | Xác thực cơ bản `user:pass` |
| `--max-time` | | Tổng thời gian tối đa |

⚠️ **`-I` hoa vs `-i` thường — hai lệnh hoàn toàn khác nhau:**

| | `-I` (hoa) | `-i` (thường) |
|---|---|---|
| Gửi phương thức | **HEAD** | GET (hoặc method bạn chọn) |
| Nhận về | Chỉ header | Header **+ nội dung** |
| Rủi ro | ⚠️ Một số server **xử lý HEAD khác GET** ⇒ kết quả gây hiểu lầm | Phản ánh đúng thực tế |

⇒ Khi debug "API trả về gì", dùng **`-i`**. `-I` chỉ để xem nhanh header.

🛑 **`-k` — hiểu rõ hậu quả trước khi dùng.** Nó **tắt toàn bộ việc xác thực danh tính server** ⇒ không phân biệt được server thật với kẻ đứng giữa. Chỉ chấp nhận được khi **debug nội bộ với cert tự ký**.

⇒ **Cái gì thay thế?** Nạp đúng CA nội bộ thay vì tắt kiểm tra:

```bash
curl --cacert /etc/ssl/certs/company-ca.crt https://api.noibo.vn/health
#     └─ tin CA này -> vẫn xác thực đầy đủ, không mở lỗ hổng
```

⭐ **Đo hiệu năng — tách được chậm ở chặng nào:**

```bash
curl -o /dev/null -sS -w "DNS:%{time_namelookup}s TCP:%{time_connect}s TLS:%{time_appconnect}s TTFB:%{time_starttransfer}s Total:%{time_total}s Code:%{http_code}\n" \
     https://api.company.vn/health
#     │                       └─ TTFB = Time To First Byte: server bắt đầu trả lời sau bao lâu
#     └─ vứt nội dung đi, chỉ giữ số liệu
```

| Chỉ số cao bất thường | Nút thắt nằm ở |
|---|---|
| `time_namelookup` | **DNS** |
| `time_connect` | **Mạng/TCP** (độ trễ đường truyền) |
| `time_appconnect` | **Bắt tay TLS** |
| `time_starttransfer` cao, các mục trên thấp | ⭐ **Ứng dụng phía server xử lý chậm** |

**POST JSON — dạng chuẩn:**

```bash
curl -sS -X POST https://api.company.vn/v1/users \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer $TOKEN" \
     -d '{"name":"Kien","role":"devops"}'
#        └─ ⭐ nháy ĐƠN bọc JSON: giữ nguyên dấu " bên trong, shell không diễn giải $
```

⚠️ **Dữ liệu dài thì đọc từ file** thay vì nhét vào dòng lệnh (tránh lỗi escape và lộ trong `ps`):

```bash
curl -sS -X POST <url> -H "Content-Type: application/json" -d @payload.json
#                                                             └─ dấu @ = ĐỌC TỪ FILE
```

⚠️ **Token trong dòng lệnh lọt vào `ps aux` và lịch sử shell.** An toàn hơn:

```bash
curl -sS -H @<(echo "Authorization: Bearer $TOKEN") <url>
# hoặc dùng file cấu hình:
curl -sS --config curl-auth.txt <url>     # file chứa: header = "Authorization: Bearer xxx"
```

⭐ **`--resolve` — test server cụ thể mà không cần sửa `/etc/hosts`:**

```bash
curl -v --resolve api.company.vn:443:10.0.0.5 https://api.company.vn/health
#                 │                │
#                 │                └─ ép trỏ về IP này
#                 └─ domain:port
```

⇒ Rất hữu ích khi có **nhiều pod/node sau load balancer** và muốn kiểm tra **đúng một cái**, đồng thời vẫn gửi đúng tên miền trong SNI và header `Host` (nên cert vẫn hợp lệ).

⭐ **`-f` — bắt buộc trong script:** mặc định curl trả **mã thoát 0** ngay cả khi server trả **404 hoặc 500** (vì "kết nối thành công"). Script sẽ tưởng là ổn:

```bash
curl -fsS https://api.company.vn/health || echo "API LỖI"
#     │└┴─ s = im lặng, S = nhưng vẫn hiện lỗi
#     └─── f = HTTP >= 400 thì TRẢ LỖI thật sự
```

**Thử lại và giới hạn thời gian — cần cho môi trường mạng không ổn định:**

```bash
curl -sS --retry 3 --retry-delay 2 --max-time 30 --connect-timeout 5 <url>
#         │         │               │            └─ chỉ riêng bước kết nối: 5 giây
#         │         │               └─ TỔNG thời gian tối đa: 30 giây
#         │         └─ chờ 2 giây giữa các lần thử
#         └─ thử lại tối đa 3 lần
```

⚠️ **`wget` vs `curl`**: `wget` chuyên **tải file** (có `-c` để tải tiếp khi đứt, tải đệ quy cả site); `curl` chuyên **gọi API** (in ra màn hình, nhiều tuỳ chọn giao thức hơn). Trên Alpine thường **chỉ có `wget`** của busybox — bộ cờ rút gọn hơn nhiều.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa openssl — và vì sao thiếu `echo |` thì bị treo</b></summary>

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `s_client` | SSL client | Đóng vai **trình duyệt**, thực hiện bắt tay TLS |
| `-connect` | | `host:port` cần nối tới |
| `-servername` | SNI | ⭐ Báo **tên miền** muốn truy cập |
| `x509` | | Bộ công cụ đọc **chứng chỉ** |
| `-in` | input | Đọc từ file |
| `-noout` | | **Không** in cert dạng mã hoá thô |
| `-text` | | In **toàn bộ** nội dung cert dạng người đọc được |
| `-dates` | | Chỉ ngày hiệu lực & hết hạn |
| `-subject` | | Cert cấp **cho ai** |
| `-issuer` | | **Ai cấp** |
| `-checkend N` | | Trả **thành công** nếu cert còn hạn ít nhất N giây |
| `-showcerts` | | Hiện **cả chuỗi** cert trung gian |

🛑 **Vì sao BẮT BUỘC có `echo |` ở đầu?**

`openssl s_client` bắt tay TLS xong sẽ **mở phiên tương tác** chờ bạn gõ dữ liệu (giống telnet). Không có gì đẩy vào ⇒ nó **đứng chờ vô hạn** ⇒ trông y như lệnh bị treo.

```bash
echo | openssl s_client -connect api.company.vn:443 -servername api.company.vn 2>/dev/null \
     | openssl x509 -noout -dates -subject -issuer
# │                                                  │
# │                                                  └─ vứt thông báo phụ (không phải lỗi)
# └─ gửi một dòng rỗng -> openssl đóng kết nối và thoát ngay
```

⇒ **Cái gì thay thế `echo |`?** Cờ `-brief` (bản mới) hoặc `< /dev/null`:

```bash
openssl s_client -connect api.company.vn:443 -servername api.company.vn < /dev/null
#                                                                       └─ đọc từ "hư vô" = EOF ngay
```

🛑 **`-servername` (SNI) — thiếu là chẩn đoán sai.** Một địa chỉ IP thường phục vụ **nhiều tên miền TLS**. Không báo tên miền ⇒ server trả về **cert mặc định của domain khác** ⇒ bạn kết luận "cert sai/hết hạn" trong khi cert thật hoàn toàn ổn.

⇒ **Quy tắc: luôn ghi `-servername` giống hệt phần host trong `-connect`.**

**Kiểm tra hạn cert — dùng cho script cảnh báo:**

```bash
echo | openssl s_client -connect api.company.vn:443 -servername api.company.vn 2>/dev/null \
  | openssl x509 -noout -checkend 604800 \
  && echo "OK: còn hạn trên 7 ngày" || echo "CẢNH BÁO: hết hạn trong 7 ngày!"
#                    └─ 604800 giây = 7 × 24 × 3600
```

⭐ **`-checkend` trả về mã thoát**, nên ghép thẳng vào `&&`/`||` được — không cần tự so sánh ngày tháng.

**Đọc file cert trên đĩa:**

```bash
openssl x509 -in cert.pem -noout -text | head -20   # xem tổng quan
openssl x509 -in cert.pem -noout -dates             # chỉ ngày
openssl x509 -in cert.pem -noout -ext subjectAltName
#                                 └─ ⭐ SAN: danh sách MỌI domain cert này có hiệu lực
```

⭐ **SAN (Subject Alternative Name) — mảnh ghép hay thiếu.** Trình duyệt hiện đại **KHÔNG còn đọc trường `CN`** để xác định domain — chúng **chỉ đọc SAN**. Cert có `CN=api.company.vn` mà **SAN không chứa** tên đó ⇒ trình duyệt vẫn báo **không hợp lệ**, dù nhìn `-subject` thấy đúng tên.

⇒ Khi gặp lỗi "certificate name mismatch" mà `-subject` trông đúng, **hãy kiểm tra SAN**.

**Kiểm tra khớp giữa cert và private key — trước khi cài lên server:**

```bash
openssl x509 -noout -modulus -in cert.pem | openssl md5
openssl rsa  -noout -modulus -in key.pem  | openssl md5
#                    └─ modulus là phần "chung" của cặp khoá
#  => HAI kết quả md5 phải GIỐNG HỆT nhau. Khác nhau = cert và key KHÔNG PHẢI một cặp
```

⇒ Đây là nguyên nhân của lỗi `SSL_CTX_use_PrivateKey: key values mismatch` khi nginx từ chối khởi động.

**Kiểm tra chuỗi cert đầy đủ — lỗi rất hay gặp:**

```bash
echo | openssl s_client -connect api.company.vn:443 -servername api.company.vn -showcerts 2>/dev/null \
  | grep -E "^(subject|issuer|-----BEGIN)"
```

⚠️ **Lỗi "chuỗi cert thiếu mắt xích"**: server chỉ gửi cert của mình mà **quên cert trung gian**. Hiện tượng đặc trưng: **trình duyệt vào bình thường** (vì trình duyệt tự tải bổ sung được) nhưng **`curl` và ứng dụng thì báo lỗi** — vì chúng không tự tải.

```bash
openssl verify -CAfile ca-chain.crt cert.pem
# "cert.pem: OK" = chuỗi đầy đủ và hợp lệ
```

⇒ Cách sửa: nối cert của bạn với các cert trung gian vào **một file** (đúng thứ tự: cert lá trước, trung gian sau) rồi trỏ nginx vào file đó.

**Vài lệnh kiểm tra khác:**

```bash
# Cert trong Kubernetes Secret (dạng TLS)
kubectl get secret my-tls -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -noout -dates
#                                              └─ ⚠️ dấu chấm trong tên key phải escape: tls\.crt

# Quét nhanh nhiều host xem cái nào sắp hết hạn
for h in api.company.vn web.company.vn; do
  printf "%-25s " "$h"
  echo | openssl s_client -connect "$h:443" -servername "$h" 2>/dev/null \
    | openssl x509 -noout -enddate
done
#   └─ printf canh lề cho dễ đọc (%-25s = chuỗi căn trái, rộng 25 ký tự)
```

⚠️ `nmap --script ssl-cert -p 443 <host>` cũng xem được cert, nhưng nmap là **công cụ quét mạng** — trong mạng doanh nghiệp có thể bị hệ thống giám sát ghi nhận là hành vi dò quét. Với việc chỉ xem cert, `openssl` là lựa chọn kín đáo và chuẩn xác hơn.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa dig/host/nslookup và các loại bản ghi</b></summary>

| Cờ | Làm gì |
|---|---|
| `+short` | Chỉ in kết quả gọn (bỏ toàn bộ phần trình bày) |
| `+trace` | ⭐ Truy vết **từ root server** xuống — thấy tắc ở tầng nào |
| `+noall +answer` | Chỉ hiện phần ANSWER (gọn hơn mặc định, chi tiết hơn `+short`) |
| `@<server>` | Hỏi **thẳng** một DNS server chỉ định |
| `-x` | **Tra ngược**: từ IP ra tên miền |
| `+time=N` | Chờ tối đa N giây |
| `+tcp` | Hỏi qua **TCP** thay vì UDP |

**Các loại bản ghi cần biết:**

| Loại | Trả về | Dùng để |
|---|---|---|
| `A` | Địa chỉ **IPv4** | Phổ biến nhất |
| `AAAA` | Địa chỉ **IPv6** | |
| `CNAME` | **Bí danh** trỏ tới tên khác | ⚠️ Không đặt được ở tên miền gốc (apex) |
| `MX` | Máy chủ **thư** | Debug email |
| `TXT` | Văn bản tự do | SPF, DKIM, xác minh sở hữu domain |
| `NS` | **Name server** quản lý miền | Kiểm tra uỷ quyền DNS |
| `SRV` | Dịch vụ + port | K8s dùng cho headless service |
| `PTR` | Tên miền từ IP (tra ngược) | Kiểm tra danh tiếng mail server |

```bash
dig api.company.vn A +short          # chỉ IPv4
dig company.vn MX +short             # máy chủ thư
dig company.vn TXT +short            # xem bản ghi SPF/xác minh
dig -x 8.8.8.8 +short                # tra NGƯỢC: IP -> tên miền
```

⭐ **`+trace` — công cụ mạnh nhất khi "domain mới trỏ mà chưa vào được":**

```bash
dig api.company.vn +trace
#                   └─ đi tuần tự: root (.) -> .vn -> company.vn -> api.company.vn
#                      thấy DỪNG ở tầng nào = biết tầng đó cấu hình sai/chưa uỷ quyền
```

⇒ Trả lời được câu hỏi mà `+short` không trả lời được: *"không ra kết quả là do domain chưa cấu hình, hay do DNS tôi đang dùng chưa cập nhật?"*

⭐ **Mẹo khoanh vùng nhanh — so sánh hai nguồn:**

```bash
dig api.company.vn +short              # dùng DNS mặc định của máy
dig @8.8.8.8 api.company.vn +short     # hỏi DNS công cộng
dig @$(awk '/^nameserver/{print $2; exit}' /etc/resolv.conf) api.company.vn +short
#     └─ lấy nameserver đầu tiên trong resolv.conf để hỏi trực tiếp
```

| Kết quả | Kết luận |
|---|---|
| Cả hai **giống nhau** | DNS đã lan truyền đầy đủ ⇒ vấn đề **không nằm ở DNS** |
| `8.8.8.8` ra, DNS nội bộ **không** | ⇒ **DNS nội bộ** chưa cập nhật / cấu hình sai |
| `8.8.8.8` **không** ra, nội bộ ra | ⇒ Đây là **bản ghi nội bộ**, đúng như thiết kế |
| **Cả hai đều không** | ⇒ Bản ghi **chưa được tạo** ở nhà cung cấp DNS |

**Kiểm tra TTL — biết phải chờ bao lâu sau khi đổi bản ghi:**

```bash
dig api.company.vn +noall +answer
# api.company.vn.  300  IN  A  10.0.0.5
#                  └─ TTL = 300 giây: các DNS trung gian sẽ CACHE kết quả này 5 phút
```

⇒ Đổi bản ghi DNS xong mà chưa thấy hiệu lực ⇒ **chờ hết TTL cũ**, không phải cấu hình sai. ⭐ Mẹo: **hạ TTL xuống 60 giây TRƯỚC khi** chuyển đổi hệ thống, xong xuôi mới nâng lại.

⚠️ **`nslookup` — công cụ cũ, có thể gây hiểu nhầm:**

| | `dig` | `nslookup` |
|---|---|---|
| Hiển thị | ⭐ **Đúng nguyên văn** phản hồi DNS | Đã diễn giải lại, ẩn bớt chi tiết |
| Đọc `/etc/hosts` | ❌ Không | ❌ Không |
| Có sẵn | Gói `dnsutils`/`bind-tools` | Cùng gói |

🛑 **Cả `dig` và `nslookup` đều KHÔNG đọc `/etc/hosts`.** Nên chúng có thể ra kết quả **khác với thứ ứng dụng thật sự dùng**. Muốn biết ứng dụng nhìn thấy gì, dùng:

```bash
getent hosts api.company.vn
#  └─ đi qua ĐÚNG chuỗi phân giải của hệ điều hành: /etc/hosts -> nsswitch -> DNS
```

⇒ Đây là mảnh ghép then chốt: `dig` ra IP đúng mà app vẫn kết nối sai ⇒ gần như chắc chắn có dòng đè trong `/etc/hosts`.

**`whois` — thông tin đăng ký tên miền:**

```bash
whois company.vn | grep -iE "expir|status|name server"
#                            └─ ngày hết hạn ĐĂNG KÝ · trạng thái · name server
```

⚠️ **Phân biệt hai loại "hết hạn"** — hoàn toàn khác nhau:

| | Hết hạn **domain** (`whois`) | Hết hạn **cert TLS** (`openssl`) |
|---|---|---|
| Hậu quả | Mất tên miền, **toàn bộ dịch vụ chết** | Trình duyệt cảnh báo, HTTPS lỗi |
| Gia hạn ở | Nhà đăng ký tên miền | Nơi cấp cert (Let's Encrypt, CA nội bộ) |

⚠️ `whois` cần **kết nối ra Internet** ⇒ trên VDI air-gapped thường không chạy được. Thông tin này phải tra từ cổng quản trị tên miền của công ty.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa cú pháp jq — từ số 0</b></summary>

**Tiền đề — vì sao cần `jq`?** `kubectl`, `aws`, `docker inspect`, hầu hết API trả về **JSON**. JSON đọc bằng mắt thì được với vài dòng, nhưng vài trăm dòng lồng nhau thì **không tìm nổi** field cần. `jq` là "grep dành riêng cho JSON" — biết đi vào cấu trúc lồng nhau, không chỉ tìm chữ.

| Ký hiệu | Nghĩa |
|---|---|
| `.` | **Gốc** của JSON — dùng một mình để in nguyên văn, format đẹp |
| `.ten` | Lấy **field** tên `ten` |
| `.a.b.c` | Đi **lồng** vào nhiều tầng |
| `.items[]` | ⭐ **Duyệt qua** mảng — mỗi phần tử ra một dòng kết quả riêng |
| `.items[0]` | Chỉ lấy phần tử **thứ 0** (JSON đếm từ 0) |
| `.items[].id` | Duyệt mảng **rồi** lấy field `id` của từng phần tử |
| `-r` | **r**aw: bỏ dấu ngoặc kép quanh chuỗi kết quả |
| `select(...)` | **Lọc** — chỉ giữ phần tử thoả điều kiện |
| `\|` | **Nối** nhiều bước lại, giống pipe của shell nhưng bên trong jq |

⭐ **Phân biệt `.a.b` và `.a[]` — nhầm là ra lỗi khó hiểu:**

```bash
echo '{"user":{"name":"Kien"}}' | jq '.user.name'
# "Kien"     <- .a.b: ĐI VÀO một OBJECT (dấu {})

echo '{"items":[{"id":1},{"id":2}]}' | jq '.items[]'
# {"id":1}
# {"id":2}   <- .a[]: DUYỆT một ARRAY (dấu [])
```

🛑 Dùng `.items.id` (thiếu `[]`) trên một **mảng** ⇒ jq báo lỗi `Cannot index array with string "id"` — vì mảng không có "tên field", nó có "chỉ số".

**`-r` — vì sao gần như luôn cần khi đưa vào script:**

```bash
jq '.name' data.json        # -> "Kien"   (CÓ dấu ngoặc kép — vẫn là JSON string)
jq -r '.name' data.json     # -> Kien     (chuỗi THUẦN — dùng được luôn trong shell)
```

⇒ Thiếu `-r` mà gán vào biến shell: `NAME=$(jq '.name' data.json)` ⇒ `$NAME` sẽ **CHỨA CẢ dấu ngoặc kép**, đưa vào lệnh khác gây lỗi lạ (ví dụ `curl -H "Authorization: Bearer \"abc\""`).

⭐ **`select()` — lọc theo điều kiện, ghép với `|`:**

```bash
kubectl get pods -A -o json | jq -r '.items[] | select(.status.phase!="Running") | .metadata.name'
#                                     │           │                                └─ lấy tên
#                                     │           └─ CHỈ GIỮ pod KHÔNG chạy
#                                     └─ duyệt qua từng pod
```

Cách đọc `|` bên trong jq: **kết quả bước trước** trở thành **đầu vào của bước sau** — y hệt pipe của shell, chỉ khác là hoạt động **bên trong một JSON**.

**Đếm, gộp, sắp xếp:**

```bash
jq '.items | length' data.json                    # đếm phần tử trong mảng
jq -r '.items[] | .name' data.json | sort | uniq -c   # đếm theo giá trị (kết hợp với sort/uniq của shell)
jq '[.items[] | select(.active)] | length' data.json  # ⭐ đếm phần tử THOẢ điều kiện
#   └─ dấu [ ] bọc lại: gom kết quả thành MỘT mảng mới, rồi mới đếm
```

⚠️ Thiếu cặp `[ ]` bọc quanh, `length` sẽ báo lỗi hoặc tính sai — vì `select` sau `|` cho ra **nhiều giá trị rời rạc**, không phải một mảng, và `length` cần **một mảng** để đếm.

**Kết hợp với `kubectl`/`curl` — công thức hay dùng nhất:**

```bash
kubectl get pods -o json | jq -r '.items[].metadata.name'
#                                              └─ ⭐ .metadata.name, KHÔNG PHẢI .name
#                                                 (object của K8s luôn lồng field trong metadata/spec/status)

kubectl get pods -o json \
  | jq -r '.items[] | "\(.metadata.name)  \(.status.phase)"'
#                       └─ ⭐ chuỗi nội suy: \(...) chèn giá trị field vào giữa văn bản
#  => in ra: "myapp-7d9f  Running"
```

**Sửa JSON — `jq` không chỉ đọc, còn ghi được:**

```bash
jq '.replicas = 5' values.json > values-new.json
#   └─ ĐẶT lại giá trị field (jq KHÔNG sửa file tại chỗ — luôn phải ghi RA FILE MỚI)

cat data.json | jq '.tags += ["prod"]'    # thêm phần tử vào mảng có sẵn
```

🛑 **`jq` không có cờ sửa tại chỗ** như `sed -i`. Viết `jq '...' file.json > file.json` (cùng tên) sẽ **làm rỗng file trước khi jq kịp đọc** — vì shell mở `>` để ghi **trước khi** chạy lệnh. Cách đúng:

```bash
jq '.replicas = 5' values.json > tmp.json && mv tmp.json values.json
#                                └─ ⭐ dùng file tạm rồi đổi tên đè lên, TUYỆT ĐỐI không > cùng tên
```

**Debug khi jq báo lỗi khó hiểu:**

```bash
jq . data.json          # chỉ format lại, không lọc gì -> nếu LỖI ngay bước này = JSON gốc SAI CÚ PHÁP
jq 'type' data.json      # kiểm tra gốc là object hay array
```

</details>

### yq - Xử lý YAML (giống jq nhưng cho YAML)
```bash
yq '.version' config.yaml              # Lấy field
yq -i '.image.tag = "1.2.3"' values.yaml   # Sửa file tại chỗ (hay khi CI/CD)
yq eval '.services | keys' docker-compose.yml   # Lấy danh sách service
yq -o=json config.yaml                 # Convert YAML sang JSON
```

<details>
<summary><b>Bấm xem: giải nghĩa yq — và cảnh báo có 2 chương trình cùng tên "yq"</b></summary>

🛑 **Cảnh báo quan trọng nhất trước khi dùng `yq`: có HAI chương trình khác nhau cùng tên.**

| | `yq` bản Go (mikefarah) | `yq` bản Python (kislyuk) |
|---|---|---|
| Cú pháp | Giống `jq` | Bọc quanh `jq` thật, cú pháp khác |
| `yq -i` | Sửa **tại chỗ** | Không hỗ trợ giống nhau |
| Cách nhận biết | `yq --version` ra `yq (https://github.com/mikefarah/yq...)` | ra bản có chữ `python-yq` |

⇒ Copy lệnh `yq` từ Internet mà báo lỗi cú pháp lạ ⇒ **kiểm tra `yq --version` trước** — rất có thể máy đang cài bản kia. Cheatsheet này dùng **bản Go (mikefarah)** — bản phổ biến hơn trong giới DevOps.

| Cú pháp | Giống jq ở | Khác jq ở |
|---|---|---|
| `.field`, `.a.b.c` | ✅ Giống hệt | |
| `.items[]` | ✅ Giống hệt | |
| `-i` | | ⭐ **Sửa file TẠI CHỖ** — yq CÓ, jq **KHÔNG CÓ** |
| `eval` | | Cú pháp đầy đủ (`yq` là viết tắt của `yq eval`) |
| `-o=json` | | Đổi **định dạng đầu ra** |
| `-P` | | **P**retty: format JSON đẹp khi xuất ra |

⭐ **`-i` — điểm khác biệt lớn nhất so với `jq`:**

```bash
yq -i '.image.tag = "1.2.3"' values.yaml
#  │                          └─ ⭐ SỬA TRỰC TIẾP file này, không cần > file khác
#  └─ in-place: khác hẳn jq (jq luôn phải ghi ra file mới)
```

⇒ Đây là lý do `yq` rất được ưa dùng trong **CI/CD** để đổi image tag trước khi deploy:

```bash
yq -i '.spec.template.spec.containers[0].image = "myapp:'"$TAG"'"' deployment.yaml
#                          └─ [0] = container ĐẦU TIÊN trong danh sách containers
#                                  (pod nhiều container thì PHẢI chỉ đúng index)
```

⚠️ `yq -i` **sửa file gốc, không có bước xác nhận**. Trong pipeline CI, luôn `git diff` sau bước này để chắc chắn thay đổi đúng ý, trước khi commit tự động.

**Đọc dữ liệu:**

```bash
yq '.version' config.yaml                     # lấy 1 field
yq eval '.services | keys' docker-compose.yml # ⭐ liệt kê TÊN các key (không phải giá trị)
#              └─ "keys" lấy danh sách TÊN của một object — hữu ích để biết "có những service nào"
```

**Chuyển đổi định dạng — chỗ `yq` hơn hẳn `jq`:**

```bash
yq -o=json config.yaml              # YAML -> JSON
yq -p=json -o=yaml data.json        # JSON -> YAML (⭐ -p = định dạng ĐẦU VÀO)
yq -o=props config.yaml             # YAML -> dạng key=value (Java properties)
```

⚠️ `-p` (parse, đầu **vào**) và `-o` (output, đầu **ra**) — bóc nhầm hai cờ này là đọc/ghi sai định dạng.

**Kiểm tra cú pháp YAML trước khi apply — rất đáng làm với K8s manifest:**

```bash
yq eval '.' deployment.yaml > /dev/null && echo "YAML hợp lệ" || echo "LỖI CÚ PHÁP"
#            └─ đọc và in lại (vứt đi) -> chỉ để BUỘC yq phải PARSE toàn bộ file
```

⚠️ **`yq eval '.'` chỉ bắt lỗi CÚ PHÁP YAML** (thụt lề sai, thiếu dấu `:`), **không** bắt lỗi **ngữ nghĩa K8s** (thiếu field bắt buộc, field không tồn tại trong schema). Muốn kiểm cả ngữ nghĩa, dùng `kubectl apply --dry-run=server` (đã nói ở mục Helm/kubectl phía trên).

**Xử lý nhiều document trong một file** (YAML cho phép nhiều tài liệu cách nhau bằng `---`, K8s hay dùng kiểu này):

```bash
yq eval-all '.' -                    # gộp NHIỀU document, đọc từ stdin
yq eval 'select(.kind == "Deployment")' manifests.yaml
#               └─ lọc ra CHỈ document có kind=Deployment trong file nhiều tài liệu
```

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa export/unset và ba tầng "phạm vi" của biến shell</b></summary>

**Tiền đề — ba tầng biến, phải phân biệt:**

| | Biến shell thường | Biến **export**ed | Biến trong `.env` |
|---|---|---|---|
| Khai báo | `KEY=value` | `export KEY=value` | dòng `KEY=value` trong file |
| Tiến trình CON có thấy không? | ❌ **KHÔNG** | ✅ **CÓ** | Tuỳ cách nạp |
| Ví dụ | biến tạm trong script | `PATH`, `HOME` | `DB_PASSWORD` cho app đọc |

🛑 **Đây là bẫy hay gặp nhất**: gõ `KEY=value` (thiếu `export`) rồi chạy chương trình khác trong CÙNG shell ⇒ chương trình đó **không thấy** biến này, vì biến chỉ tồn tại trong **shell hiện tại**, không truyền xuống tiến trình con.

```bash
KEY=value          # chỉ shell HIỆN TẠI biết
export KEY=value   # ⭐ shell hiện tại VÀ mọi tiến trình con (app, script gọi từ đây) đều thấy
```

**Lệnh cơ bản:**

| Lệnh | Làm gì |
|---|---|
| `env` | In **mọi biến đã export** |
| `printenv <VAR>` | In **một** biến (gọn hơn `env \| grep`) |
| `echo $PATH` | Đọc trực tiếp bằng cú pháp shell — hoạt động với **cả biến chưa export** |
| `export KEY=value` | Set và **cho tiến trình con thấy** |
| `unset KEY` | Xoá hẳn biến |
| `set -a; source .env; set +a` | Nạp cả file `.env` **và tự động export từng dòng** |

⭐ **`export PATH=$PATH:/new/path` — bóc để hiểu vì sao KHÔNG được đặt trước:**

```bash
export PATH=$PATH:/new/path
#             │    └─ đường dẫn MỚI thêm vào
#             └────── LẤY GIÁ TRỊ CŨ của PATH trước (nếu không có $PATH, PATH cũ bị GHI ĐÈ MẤT)
```

⚠️ Shell tìm chương trình theo **PATH, từ trái sang phải, dừng ở lệnh khớp ĐẦU TIÊN**. `export PATH=/new/path:$PATH` (đặt **trước**) khiến thư mục mới được **ưu tiên hơn** hệ thống; `export PATH=$PATH:/new/path` (đặt **sau**) thì **hệ thống được ưu tiên hơn**. Đây là công cụ để chọn "phiên bản nào của một chương trình chạy trước" khi có nhiều bản cài song song (ví dụ nhiều version Python).

⭐ **`set -a; source .env; set +a` — vì sao ba lệnh cần đi cùng nhau:**

```bash
set -a          # BẬT: mọi biến khai báo TỪ ĐÂY TRỞ ĐI tự động export
source .env     # nạp file: mỗi dòng KEY=value trở thành biến shell VÀ tự export nhờ set -a
set +a          # TẮT lại chế độ tự export (để các lệnh sau đó không bị ảnh hưởng)
```

🛑 Chỉ `source .env` (thiếu `set -a`) ⇒ biến chỉ tồn tại trong **shell hiện tại**, tiến trình con (app bạn sắp chạy) **không thấy được**.

⚠️ Cách khác hay gặp — `export $(cat .env | xargs)` — **hỏng ngay** khi file có **giá trị chứa khoảng trắng** hoặc dòng comment (`#`); shell tách nhầm chỗ. `set -a; source` là cách **an toàn nhất**, xử lý đúng theo cú pháp file.

**Set biến chỉ cho MỘT lệnh — không ảnh hưởng phiên làm việc:**

```bash
KEY=value command
#   └─ biến này CHỈ tồn tại trong đúng THỜI GIAN chạy "command",
#      KHÔNG lưu lại trong shell sau khi command chạy xong
DEBUG=true npm start
```

⇒ Cách này **an toàn hơn `export`** khi chỉ cần đổi hành vi tạm thời một lần — không sợ quên `unset` rồi ảnh hưởng lệnh sau.

**Kiểm tra ở đâu ra một biến** (đặt ở nhiều file dễ chồng chéo):

```bash
type -a KEY 2>/dev/null; echo "KEY=$KEY"    # xem giá trị hiện tại
env | grep KEY                              # có được export hay không (không thấy = chưa export)
```

⚠️ **Thứ tự nạp file cấu hình shell** — biến định nghĩa ở file nạp **sau** sẽ **đè** file nạp **trước**: `~/.zshenv` → `~/.zprofile` → `~/.zshrc` (zsh); `~/.bash_profile`/`~/.profile` → `~/.bashrc` (bash). Đặt cùng một biến ở hai chỗ khác nhau là nguồn gốc của "tôi đã set rồi mà sao vẫn giá trị cũ".

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa 5 trường cron và các lỗi hay gặp</b></summary>

| Lệnh | Làm gì |
|---|---|
| `crontab -l` | **L**ist — xem lịch hiện tại |
| `crontab -e` | **E**dit — sửa (mở editor, mặc định `vi`) |
| `crontab -r` | **R**emove — 🛑 xoá **TOÀN BỘ**, không hỏi lại |
| `crontab -u <user>` | Thao tác cron của **user khác** (cần quyền root) |

🛑 **`crontab -r` không hỏi xác nhận** — xoá sạch mọi cron job của user đó ngay lập tức. Trước khi sửa cron, luôn sao lưu:

```bash
crontab -l > cron-backup-$(date +%F).txt    # backup trước khi động vào
```

**Cấu trúc 5 trường — phải thuộc để đọc/viết đúng:**

```
* * * * *  lệnh
│ │ │ │ │
│ │ │ │ └─ Thứ trong tuần (0-7, cả 0 VÀ 7 đều là Chủ Nhật)
│ │ │ └─── Tháng (1-12)
│ │ └───── Ngày trong tháng (1-31)
│ └─────── Giờ (0-23)
└───────── Phút (0-59)
```

**Ký hiệu đặc biệt:**

| Ký hiệu | Nghĩa | Ví dụ |
|---|---|---|
| `*` | **Mọi** giá trị | `* * * * *` = mỗi phút |
| `*/N` | Mỗi N đơn vị | `*/5 * * * *` = mỗi 5 phút |
| `,` | Liệt kê nhiều giá trị | `0 9,17 * * *` = 9h **và** 17h |
| `-` | Khoảng | `0 9-17 * * *` = mỗi giờ từ 9h đến 17h |

**Ví dụ đọc — luyện đọc từ trái sang phải theo đúng thứ tự trường:**

```bash
0 2 * * *        /path/backup.sh        # phút=0 giờ=2  -> 2:00 sáng MỖI NGÀY
*/5 * * * *       /path/check.sh         # mỗi 5 PHÚT
0 9 * * 1-5       /path/report.sh        # 9:00 sáng, THỨ 2 ĐẾN THỨ 6 (bỏ qua cuối tuần)
0 0 1 * *         /path/monthly.sh       # 0:00 NGÀY 1 mỗi tháng
```

🛑 **Ba lỗi kinh điển khiến "cron không chạy" — kiểm tra theo đúng thứ tự này:**

**1. PATH của cron KHÁC PATH của bạn khi gõ tay.** Cron chạy với môi trường **rất tối giản**, không nạp `.bashrc`/`.zshrc` ⇒ lệnh chạy ngon khi gõ tay nhưng cron báo `command not found`.

```bash
# Trong crontab, LUÔN dùng đường dẫn TUYỆT ĐỐI cho cả lệnh và file:
0 2 * * * /usr/bin/python3 /home/deployer/backup.py
#         └─ tuyệt đối       └─ tuyệt đối, KHÔNG dùng ~ hay đường dẫn tương đối
```

💡 Tìm đường dẫn tuyệt đối: `which python3`.

**2. Không thấy log lỗi vì output bị "rơi vào hư không".** Mặc định, output của cron job được **gửi mail cho user** — nhưng đa số server **không cấu hình mail** ⇒ output biến mất hoàn toàn, không cách nào biết job đã chạy hay lỗi.

```bash
0 2 * * * /path/backup.sh >> /var/log/backup.log 2>&1
#                          │                      └─ gộp CẢ lỗi (stderr) vào cùng file
#                          └─ >> nối thêm, KHÔNG ghi đè file cũ
```

⚠️ Dùng `>` (một dấu) thay vì `>>` sẽ **XOÁ log lần chạy trước** mỗi khi cron chạy lại — chỉ còn log của lần gần nhất.

**3. Ai đó `sudo crontab -e` thay vì `crontab -e`** — sửa nhầm **cron của root**, không phải của mình:

```bash
crontab -l              # xem cron CỦA TÔI
sudo crontab -l -u root # xem cron CỦA ROOT (khác hẳn — hay bị nhầm khi debug)
```

**Kiểm tra cron có thực sự chạy hay không:**

```bash
grep CRON /var/log/syslog | tail -20         # Debian/Ubuntu
journalctl -u cron -f                         # systemd — bám realtime
journalctl -u cron --since "1 hour ago"       # log 1 giờ gần đây
```

⭐ **`systemctl list-timers`— công cụ thay thế hiện đại của cron:**

```bash
systemctl list-timers --all
#                     └─ hiện cả timer CHƯA từng chạy lần nào
```

| | cron | systemd timer |
|---|---|---|
| Log | Phải tự cấu hình (như trên) | ⭐ Tự động vào `journalctl` |
| Phụ thuộc | Không | ✅ Chờ được service khác xong mới chạy |
| Chạy bù nếu máy tắt đúng giờ | ❌ Không | ✅ Có (`Persistent=true`) |
| Có sẵn trên container/K8s | ❌ Cần cài thêm | K8s dùng `CronJob` riêng, không dùng systemd |

⇒ Trên hệ thống dùng systemd, **timer là lựa chọn hiện đại hơn** cron truyền thống, đặc biệt khi cần chạy bù việc bị lỡ (máy tắt đúng lúc lịch chạy).

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa lệnh Terraform — vì sao phải plan trước apply</b></summary>

**Tiền đề — Terraform "biết" trạng thái hạ tầng bằng cách nào?** Nó lưu một file **state** (`terraform.tfstate`) ghi lại **những gì nó đã tạo**. Mọi lệnh (`plan`, `apply`) đều **so sánh 3 bên**: code bạn viết · state đã lưu · **thực tế trên cloud** — rồi tính ra cần làm gì để khớp.

| Lệnh | Làm gì | Đụng vào hạ tầng thật? |
|---|---|---|
| `terraform init` | Tải **provider** (plugin gọi API AWS/GCP...) + cấu hình nơi lưu state | ❌ Không |
| `terraform validate` | Kiểm tra **cú pháp** file `.tf` | ❌ Không |
| `terraform fmt` | Format lại code theo chuẩn | ❌ Không |
| `terraform plan` | ⭐ **Tính toán** sẽ tạo/sửa/xoá gì, **chỉ in ra xem** | ❌ Không |
| `terraform apply` | **Thực thi** thật lên cloud | ✅ **CÓ** |
| `terraform destroy` | 🔴 **Xoá** mọi resource đang quản lý | ✅ **CÓ, và MẤT** |
| `terraform state list` | Liệt kê resource **trong state** | ❌ Không |
| `terraform show` | In state hiện tại dạng đọc được | ❌ Không |
| `terraform output` | In các giá trị `output` đã khai báo | ❌ Không |

⭐ **`plan` trước `apply` — không phải thói quen tốt, mà là BẮT BUỘC ở production.** `plan` cho biết chính xác **3 con số**: bao nhiêu resource sẽ `+ create`, `~ update`, `- destroy` — trước khi bất cứ thứ gì thật sự xảy ra.

```
Plan: 2 to add, 1 to change, 1 to destroy.
#          │         │            └─ 🛑 ĐỌC KỸ dòng này — "destroy" nghĩa là XOÁ THẬT
#          │         └────────────── sửa tại chỗ, không mất resource
#          └──────────────────────── tạo mới
```

🛑 **"1 to change" đôi khi ẩn giấu một "destroy + create"** — Terraform gọi là **replace**, hiện dấu `-/+` chứ không phải `~`. Một số thay đổi (ví dụ đổi tên EBS volume, đổi AZ) **không sửa tại chỗ được** ⇒ cloud provider buộc phải **xoá rồi tạo lại** ⇒ **mất dữ liệu** nếu là ổ đĩa/database. ⇒ Luôn đọc kỹ dấu ở đầu mỗi dòng trong `plan`, không chỉ đọc dòng tổng kết.

```bash
terraform plan -out=tf.plan
#               └─ LƯU kết quả tính toán ra file
terraform apply tf.plan
#               └─ ⭐ áp dụng ĐÚNG những gì đã plan, KHÔNG tính lại
#                  (nếu apply không truyền file, Terraform TÍNH LẠI plan ngay trước khi hỏi —
#                   giữa lúc plan và lúc bạn gõ "yes" có thể đã đổi -> apply thẳng file plan an toàn hơn)
```

⇒ Đây là mẫu chuẩn trong CI/CD: bước **plan** chạy riêng (cho người review đọc), bước **apply** dùng đúng file đã review — tránh tình trạng "cái tôi review khác cái được apply".

⚠️ **`-auto-approve` — cân nhắc kỹ, KHÔNG dùng tuỳ tiện ở production:**

```bash
terraform apply -auto-approve
#                └─ bỏ qua bước hỏi "yes" -> ⭐ CHỈ hợp cho pipeline CI đã được review kỹ,
#                   KHÔNG hợp để gõ tay ở terminal khi làm production
```

🛑 **`terraform destroy` — hiểu phạm vi trước khi chạy:**

```bash
terraform destroy                        # 🔴 xoá TOÀN BỘ resource trong state hiện tại
terraform destroy -target=aws_instance.web   # chỉ xoá MỘT resource cụ thể
#                  └─ target: giới hạn phạm vi (nhưng CẢNH BÁO: Terraform documentation
#                     khuyên KHÔNG lạm dụng -target vì có thể để state lệch khỏi thực tế)
```

⭐ **`terraform state list` + `show` — công cụ điều tra khi "apply nói có thay đổi mà tôi không sửa gì":**

```bash
terraform state list                              # mọi resource ĐANG quản lý
terraform state show aws_instance.web             # chi tiết MỘT resource, đúng như Terraform thấy
terraform plan -refresh-only                       # ⭐ chỉ ĐỐI CHIẾU với thực tế, KHÔNG đổi state
#              └─ trả lời: "có ai sửa tay trên console cloud mà Terraform chưa biết không?"
```

⚠️ **Nguyên nhân số 1 của "plan báo thay đổi lạ"**: ai đó **sửa trực tiếp trên AWS/GCP Console** (thêm tag, đổi security group) mà không qua Terraform ⇒ lần `plan` sau sẽ đòi **sửa lại cho khớp code** — đây chính là "giật lại quyền kiểm soát", không phải Terraform bị lỗi.

⚠️ **State chứa dữ liệu nhạy cảm ở dạng KHÔNG mã hoá** (password, private key có thể nằm trong đó) ⇒ **không bao giờ commit `.tfstate` vào Git**. Luôn dùng **remote backend** (S3 + DynamoDB lock, Terraform Cloud) để lưu state tập trung và **khoá** khi nhiều người cùng chạy — tránh hai người `apply` đồng thời làm hỏng state.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa cờ Ansible — và vì sao --check không đáng tin 100%</b></summary>

**Tiền đề — khác Terraform ở điểm nào?** Terraform quản lý **hạ tầng** (tạo máy chủ, mạng). Ansible chạy **bên trong** máy đã có sẵn để **cấu hình** nó (cài package, sửa file, khởi động service) — qua SSH, **không cần cài agent** trên máy đích.

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-m` | **m**odule | Chạy đúng **một module** thay vì cả playbook |
| `-i` | **i**nventory | Danh sách máy đích (file hoặc script động) |
| `--check` | | ⭐ **Diễn tập** — dự đoán thay đổi, không áp dụng thật |
| `--diff` | | Hiện **nội dung khác biệt** của file sẽ bị sửa |
| `--limit` | | Chỉ chạy trên **một nhóm/host** trong inventory |
| `-v` / `-vvv` | **v**erbose | Càng nhiều `v` càng chi tiết |
| `-a` | **a**rguments | Tham số truyền cho module |
| `--tags` | | Chỉ chạy **task được gắn tag** đó |
| `-K` | ask become pass | Hỏi mật khẩu `sudo` trên máy đích |

⭐ **`ansible all -m ping` — lệnh kiểm tra đầu tiên, luôn chạy trước mọi playbook:**

```bash
ansible all -m ping
#       │    └─ module "ping": KHÔNG PHẢI lệnh ping ICMP mạng thường!
#       │                      nó kiểm tra: SSH vào được + Python sẵn sàng chạy module
#       └────── nhóm "all" trong inventory
```

🛑 **`ansible ping` ≠ lệnh `ping` của hệ điều hành.** Trả lời `pong` nghĩa là: SSH OK, xác thực OK, **Python trên máy đích hoạt động** (Ansible cần Python để chạy hầu hết module). "ping" mạng thông thường không kiểm tra được ba điều này.

**`-m` — chạy nhanh một việc mà không cần viết playbook:**

```bash
ansible web -m shell -a "df -h"
#       │    │        └─ tham số: lệnh shell cần chạy
#       │    └────────── module "shell": chạy qua /bin/sh, HIỂU pipe/redirect (|, >)
#       └───────────── nhóm "web" trong inventory

ansible web -m command -a "df -h"
#            └─ module "command": ⭐ AN TOÀN HƠN shell — KHÔNG qua shell,
#               không hiểu pipe/redirect/biến môi trường -> tránh injection
```

⚠️ **`shell` vs `command` — chọn `command` khi có thể.** `shell` cho phép cú pháp shell đầy đủ (`|`, `&&`, `$VAR`) nhưng đó cũng là **rủi ro bảo mật** nếu tham số đến từ input không tin cậy. Chỉ dùng `shell` khi thực sự cần pipe/redirect.

⭐ **`--check` — vì sao KHÔNG đáng tin 100%, phải biết giới hạn:**

```bash
ansible-playbook site.yml --check --diff
#                          │       └─ CÙNG lúc: hiện rõ nội dung file sẽ đổi thế nào
#                          └─────── dry-run: dự đoán, KHÔNG áp dụng thật
```

🛑 **Giới hạn quan trọng**: `--check` hoạt động tốt với module quản lý **file/package** (biết dự đoán chính xác). Nhưng với module chạy **lệnh tuỳ ý** (`command`, `shell`) — Ansible **không có cách nào biết trước** lệnh đó sẽ làm gì ⇒ nó **BỎ QUA hoàn toàn** bước đó trong chế độ check (trừ khi task được đánh dấu rõ `check_mode: false` để buộc chạy, hoặc `changed_when` được set thủ công).

⇒ Playbook dùng nhiều `shell`/`command` thì `--check` **chỉ cho một bức tranh KHÔNG ĐẦY ĐỦ**. Đừng coi "check chạy sạch" là bằng chứng chắc chắn "apply thật cũng sẽ chạy sạch".

**`--limit` — thu hẹp phạm vi, tránh chạy nhầm cả fleet:**

```bash
ansible-playbook site.yml --limit web01.company.vn      # đúng MỘT máy
ansible-playbook site.yml --limit web              # đúng MỘT nhóm
ansible-playbook site.yml --limit 'web:!web03'      # nhóm web, TRỪ web03
#                                  └─ dấu ! = loại trừ
```

⭐ **Quy trình an toàn khi đổi cấu hình trên fleet lớn:**

```bash
ansible-playbook site.yml --check --diff --limit web01     # 1. thử trên 1 máy trước, xem diff
ansible-playbook site.yml --limit web01                    # 2. apply thật trên máy đó, kiểm tra kỹ
ansible-playbook site.yml --limit web                       # 3. mới áp dụng cho CẢ nhóm
```

⚠️ **`ansible-vault` — mã hoá secret để commit an toàn vào Git:**

```bash
ansible-vault encrypt secrets.yml           # mã hoá file
ansible-vault view secrets.yml              # xem nội dung mà KHÔNG giải mã ra đĩa
ansible-vault edit secrets.yml              # sửa (tự giải mã -> sửa -> tự mã hoá lại)
ansible-playbook site.yml --ask-vault-pass  # chạy playbook, hỏi mật khẩu vault
```

⚠️ Sau `encrypt`, file trên đĩa là **văn bản mã hoá** — `git diff` sẽ **không đọc được nội dung thay đổi thật**, chỉ thấy toàn bộ khối mã hoá đổi khác. Đây là đánh đổi chấp nhận được để giữ secret an toàn trong Git.

⚠️ `-vvv` in ra **cả nội dung biến**, kể cả biến đã đưa qua vault sau khi giải mã ⇒ **cẩn thận khi dán log debug** ra nơi công khai — mảnh log đó có thể chứa secret dạng rõ.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa npm install vs ci, và semver</b></summary>

**Tiền đề — `package.json` vs `package-lock.json`:**

| File | Chứa | Ai đọc |
|---|---|---|
| `package.json` | Version **mong muốn**, thường ghi dạng khoảng (`^1.2.0`) | Người viết code |
| `package-lock.json` | Version **CHÍNH XÁC** đã cài lần gần nhất, cho **mọi** package con | npm dùng để cài lại y hệt |

⭐ **`npm install` vs `npm ci` — khác biệt quan trọng nhất, quyết định CI có ổn định không:**

| | `npm install` | `npm ci` |
|---|---|---|
| Đọc theo | `package.json` (khoảng version) | ⭐ **CHỈ** `package-lock.json` (chính xác tuyệt đối) |
| Có thể **SỬA** `package-lock.json`? | ✅ Có (nếu version mới ra mà vẫn khớp `^`) | ❌ **KHÔNG BAO GIỜ** |
| `node_modules` cũ | Giữ, cài chồng lên | 🛑 **XOÁ SẠCH trước khi cài** |
| Tốc độ | Chậm hơn | ⭐ **Nhanh hơn** |
| Thiếu file lock | Tự tạo mới | ❌ **Báo lỗi**, dừng ngay |

⇒ **Quy tắc: máy dev dùng `install`, CI/CD LUÔN dùng `ci`.** Vì `install` có thể âm thầm cài **bản mới hơn** một chút so với lock file (nếu nằm trong khoảng `^`/`~`) ⇒ "chạy trên máy tôi thì được" nhưng **build trên CI ra khác** — chính `ci` được sinh ra để triệt tiêu sự khác biệt này.

**Đọc ký hiệu version (semver) trong `package.json`:**

| Ký hiệu | Nghĩa | Ví dụ `^1.2.3` chấp nhận |
|---|---|---|
| `^` | Không đổi số **đầu tiên khác 0** | `1.2.4`, `1.9.0` — **không** `2.0.0` |
| `~` | Chỉ đổi **bản vá** (số cuối) | `1.2.4` — **không** `1.3.0` |
| (không có gì) | **Chính xác** đúng version đó | Chỉ `1.2.3` |
| `*` hoặc `latest` | 🛑 Bất kỳ version nào | Rất nguy hiểm cho production |

⚠️ Ba số trong version `MAJOR.MINOR.PATCH` theo quy ước: **MAJOR** đổi = có thể **hỏng tương thích ngược** · **MINOR** = thêm tính năng, vẫn tương thích · **PATCH** = chỉ sửa lỗi. `^` tin tưởng rằng tác giả package tuân thủ đúng quy ước này — **nhưng không phải ai cũng tuân thủ nghiêm**, đây là rủi ro tiềm ẩn của `^`.

**Dọn dẹp khi gặp lỗi lạ:**

```bash
npm cache clean --force
#                └─ --force BẮT BUỘC phải có, vì npm CỐ Ý chặn lệnh này
#                   (dọn cache có thể gây ra đúng lỗi mà bạn đang cố sửa, nên npm cảnh báo trước)

rm -rf node_modules package-lock.json && npm install
#                  └─ 🛑 xoá CẢ file lock -> npm sẽ TỰ TÍNH LẠI version mới nhất
#                     có thể kéo theo version KHÁC với trước (không giống hệt máy khác)
```

⚠️ **Cách "xoá cả lock rồi install" chỉ nên dùng khi hết cách khác.** Nó xoá luôn "bản ghi chính xác" đã hoạt động — nếu sau đó version mới có breaking change, bug xuất hiện mà không rõ nguyên nhân do đâu. Ưu tiên thử trước:

```bash
rm -rf node_modules && npm install    # GIỮ lock file, chỉ cài lại theo đúng version cũ
```

**Bảo mật:**

```bash
npm audit                  # liệt kê lỗ hổng đã biết trong dependency
npm audit fix               # tự sửa những cái sửa được MÀ KHÔNG phá vỡ semver (an toàn)
npm audit fix --force       # ⚠️ có thể NÂNG CẤP major version -> có thể HỎNG CODE, cần test kỹ sau đó
```

**yarn / pnpm — điểm khác biệt cần biết:**

```bash
yarn install --frozen-lockfile   # ⭐ tương đương `npm ci` — KHÔNG được sửa lock file
pnpm install --frozen-lockfile   # tương tự
```

| | npm | yarn | pnpm |
|---|---|---|---|
| Cách lưu package | Copy riêng cho từng project | Copy riêng | ⭐ **Symlink** tới kho chung trên máy |
| Tốc độ cài lần 2 | Chậm | Nhanh hơn | ⭐ **Nhanh nhất** |
| Dung lượng đĩa | Cao (trùng lặp) | Cao | ⭐ **Thấp nhất** (không trùng lặp) |

⇒ `pnpm` tiết kiệm đĩa đáng kể khi máy có nhiều project Node — đáng cân nhắc trên môi trường VDI hạn chế dung lượng.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa venv — và vì sao PHẢI dùng, không cài thẳng vào máy</b></summary>

⭐ **Tiền đề — venv giải quyết bài toán gì?**

Máy chỉ có **một** bản Python hệ thống. Project A cần `django==3.2`, project B cần `django==4.2` ⇒ cài thẳng vào máy thì **một trong hai sẽ hỏng** vì chỉ giữ được một version tại một thời điểm. `venv` tạo ra **một bản Python + pip riêng biệt cho từng project**, không đụng vào nhau và không đụng vào Python hệ thống.

| Lệnh | Làm gì |
|---|---|
| `python -m venv venv` | Tạo môi trường ảo trong thư mục `venv/` |
| `source venv/bin/activate` | **Kích hoạt** — từ giờ `python`/`pip` trỏ vào bản trong `venv/` |
| `deactivate` | Thoát, trả `python`/`pip` về bản hệ thống |
| `pip install <pkg>` | Cài package **vào venv đang active** |
| `pip freeze` | In ra **version chính xác** đang cài |
| `pip list` | Danh sách package đã cài (dạng bảng, dễ đọc hơn `freeze`) |
| `pip install --upgrade <pkg>` | Nâng cấp lên bản mới nhất |

**Bóc lệnh tạo venv:**

```bash
python -m venv venv
#      │  │     └─ tên thư mục sẽ chứa venv (thường đặt "venv" hoặc ".venv")
#      │  └─────── module "venv" có sẵn trong Python (không cần cài thêm)
#      └────────── -m: chạy MODULE này như một chương trình
```

⭐ **Làm sao biết đang ở TRONG hay NGOÀI venv?** Dấu nhắc shell **tự đổi**, hiện tên venv trong ngoặc ở đầu dòng:

```
(venv) user@host:~/project$
 └──── ⭐ có cái này = đang Ở TRONG venv, pip install sẽ CHỈ ảnh hưởng project này
```

🛑 **Quên `activate` trước khi `pip install`** là nguyên nhân số 1 của "tôi cài rồi mà sao vẫn báo thiếu module" — package bị cài vào **Python hệ thống** hoặc **venv khác**, không phải venv của project đang mở.

```bash
which python      # kiểm tra ĐANG dùng python NÀO
#  Trong venv:    /home/user/project/venv/bin/python  <- đường dẫn nằm TRONG venv
#  Ngoài venv:    /usr/bin/python3                     <- đường dẫn hệ thống
```

**`pip freeze` — vì sao là bước bắt buộc trước khi bàn giao/deploy:**

```bash
pip freeze > requirements.txt
#            └─ ghi CHÍNH XÁC version đang cài (django==4.2.7, không phải django>=4.0)
```

⇒ Không có bước này, người khác `pip install -r requirements.txt` với version **thoáng** (`django>=4.0`) có thể cài phải **bản mới hơn** đã có breaking change ⇒ "chạy máy tôi được, máy bạn lỗi".

⚠️ **`pip freeze` liệt kê CẢ dependency của dependency** (transitive) — file `requirements.txt` sinh ra có thể dài, lẫn lộn cái bạn **chủ động cài** với cái bị **kéo theo**. Công cụ như `pip-tools` (`pip-compile`) giải quyết vấn đề này bằng cách tách riêng "cái tôi cần" và "cái được khoá version".

**Cài từ file:**

```bash
pip install -r requirements.txt
#               └─ đọc TỪNG DÒNG trong file, cài đúng version ghi trong đó

pip install -r requirements.txt --no-deps
#                                 └─ ⭐ CHỈ cài đúng những gì liệt kê,
#                                    KHÔNG tự kéo thêm dependency phụ
#                                    (dùng khi requirements.txt ĐÃ liệt kê đủ mọi thứ, tránh xung đột version)
```

⚠️ **`venv` là tính năng của Python 3** (module có sẵn). Python 2 hoặc project cũ dùng công cụ tên `virtualenv` (phải cài riêng) — cú pháp tương tự nhưng là **chương trình khác**.

⚠️ **Đường dẫn venv là TUYỆT ĐỐI, không di chuyển được.** Thư mục `venv/` ghi cứng đường dẫn máy lúc tạo ra nó ⇒ copy `venv/` sang máy khác hoặc đổi tên thư mục cha ⇒ **hỏng**, phải xoá và tạo lại (`rm -rf venv && python -m venv venv && pip install -r requirements.txt`). Đây là lý do `venv/` luôn nằm trong `.gitignore` — không bao giờ commit nó vào Git.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa AWS CLI — profile, sts, và cách tránh xoá nhầm production</b></summary>

**Tiền đề — AWS CLI lấy credential từ đâu?** Nó tìm theo **thứ tự ưu tiên**: cờ dòng lệnh → biến môi trường (`AWS_ACCESS_KEY_ID`...) → file `~/.aws/credentials` → IAM role (nếu chạy trên EC2/EKS). Không hiểu thứ tự này ⇒ hay bối rối "tôi đổi profile rồi mà sao vẫn login tài khoản cũ" (do biến môi trường đang **đè lên** file credentials).

| Lệnh/cờ | Làm gì |
|---|---|
| `aws configure` | Thiết lập access key + secret + region + output format |
| `aws configure list` | Xem cấu hình **đang thực sự áp dụng**, và **nguồn** của từng giá trị |
| `--profile` | Dùng bộ credential khác (nhiều tài khoản AWS trên một máy) |
| `sts get-caller-identity` | ⭐ "Tôi đang là ai" |
| `--filters` | Lọc kết quả ngay tại server (nhanh hơn lọc bằng `jq` phía client) |
| `--dry-run` | (một số lệnh) chỉ kiểm tra quyền, không thực thi |

⭐ **`aws sts get-caller-identity` — lệnh đầu tiên PHẢI chạy trước MỌI thao tác nguy hiểm:**

```bash
aws sts get-caller-identity
# {
#   "UserId": "AIDAI...",
#   "Account": "123456789012",   <- ⭐ SỐ TÀI KHOẢN — kiểm tra đây có phải PROD không
#   "Arn": "arn:aws:iam::123456789012:user/kiennv"
# }
```

🛑 **Đây là bước phòng sự cố quan trọng nhất khi làm việc với nhiều tài khoản AWS** (dev/staging/prod tách riêng). Không kiểm tra trước ⇒ chạy `terraform destroy` hay `aws ec2 terminate-instances` **trên nhầm tài khoản** — không có `Ctrl+Z` cho việc này.

⭐ **`--profile` — quản lý nhiều tài khoản trên một máy:**

```bash
cat ~/.aws/config
# [profile dev]
# region = ap-southeast-1
# [profile prod]
# region = ap-southeast-1

aws --profile prod sts get-caller-identity      # kiểm tra ĐÚNG profile trước khi làm gì
aws --profile prod s3 ls
```

💡 Đặt biến môi trường để khỏi gõ `--profile` mỗi lệnh trong một phiên làm việc: `export AWS_PROFILE=prod` — nhưng **cẩn thận quên đã set**, dễ chạy nhầm ở phiên terminal khác đang mở.

**S3 — ba lệnh dễ nhầm khi copy hàng loạt:**

```bash
aws s3 cp file.txt s3://mybucket/           # copy MỘT file
aws s3 cp ./localdir s3://mybucket/ --recursive   # ⭐ copy thư mục PHẢI có --recursive
aws s3 sync ./localdir s3://mybucket/       # ⭐ đồng bộ: chỉ gửi phần KHÁC BIỆT, không gửi lại hết
```

⚠️ **`cp --recursive` vs `sync`**: `cp` copy đè, không xoá file thừa ở đích. `sync` **giống `rsync`** — chỉ đẩy phần thay đổi, và với `--delete` sẽ **xoá ở đích** những gì không còn ở nguồn:

```bash
aws s3 sync ./dir s3://mybucket/ --delete
#                                 └─ 🛑 XOÁ trên S3 những file KHÔNG CÒN ở local
#                                    -> LUÔN thử --dryrun trước:
aws s3 sync ./dir s3://mybucket/ --delete --dryrun
```

**EC2 — lọc và các trạng thái:**

```bash
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[].Instances[].[InstanceId,Tags[?Key==`Name`].Value|[0]]' \
  --output table
#  │        │                                        │                └─ [0]: lấy PHẦN TỬ ĐẦU
#  │        │                                        │                   (JMESPath filter trả về mảng)
#  │        │                                        └─ lọc Tags theo Key="Name"
#  │        └─ ⭐ JMESPath: ngôn ngữ truy vấn RIÊNG của AWS CLI (không phải jq, cú pháp khác)
#  └────────── lọc NGAY TẠI SERVER AWS, nhanh hơn tải hết về rồi lọc bằng jq
```

⚠️ **`--filters` (lọc ở server) khác `--query` (lọc kết quả trả về, dùng JMESPath).** `--filters` chỉ áp dụng được cho **field mà AWS API hỗ trợ lọc** (khác nhau tuỳ dịch vụ); `--query` lọc được **bất kỳ field nào** trong JSON trả về nhưng **tải hết dữ liệu về trước** rồi mới cắt.

⚠️ **`start-instances`/`stop-instances` KHÔNG xoá gì** (dữ liệu ổ đĩa vẫn còn — trừ instance dùng instance-store). Nhưng `terminate-instances` **xoá vĩnh viễn** trừ khi ổ đĩa gắn `DeleteOnTermination=false`. Ba lệnh nghe gần giống nhau nhưng hậu quả khác hẳn:

| Lệnh | Ảnh hưởng |
|---|---|
| `stop-instances` | Tắt máy, **giữ** ổ đĩa, vẫn tính phí lưu trữ |
| `terminate-instances` | 🔴 **Xoá hẳn** máy (và ổ đĩa, trừ khi cấu hình giữ lại) |

**ECR — login trước khi push/pull image riêng của công ty:**

```bash
aws ecr get-login-password --region ap-southeast-1 \
  | docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-southeast-1.amazonaws.com
#                 │                └─ ⭐ đọc password từ ĐẦU VÀO CHUẨN, không hiện trong lịch sử lệnh
#                 └─ user luôn CỐ ĐỊNH là chữ "AWS" cho ECR (không phải tên user thật)
```

⚠️ Token đăng nhập ECR **hết hạn sau 12 giờ** — script chạy dài ngày phải **login lại định kỳ**, không phải login một lần là dùng mãi.

**EKS — lấy kubeconfig:**

```bash
aws eks update-kubeconfig --name mycluster --region ap-southeast-1 --profile prod
#                                                                    └─ ⭐ nhớ đúng profile
#                                                                       (kubeconfig sẽ NHỚ profile này
#                                                                        để lần sau kubectl tự gọi lại)
```

⚠️ Lệnh này **ghi thêm** vào `~/.kube/config` (không xoá context cũ) ⇒ dùng xong nhiều cluster, luôn `kubectl config current-context` để chắc đang đứng đúng chỗ (xem lại mục Context & Cluster của kubectl phía trên).

**CloudWatch Logs:**

```bash
aws logs tail /aws/lambda/myfunction --follow --since 10m
#                                     │        └─ chỉ log 10 phút gần đây
#                                     └───────── bám realtime (giống kubectl logs -f)
```

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa gcloud — project vs account, hai khái niệm hay nhầm</b></summary>

⭐ **Tiền đề — `gcloud` có HAI khái niệm "đang là ai" tách biệt, đây là nguồn gốc nhầm lẫn phổ biến nhất:**

| | **Account** (tài khoản) | **Project** (dự án) |
|---|---|---|
| Là gì | Email đăng nhập Google | Nơi chứa tài nguyên (VM, GKE, IP...) và **tính phí** |
| Lệnh xem | `gcloud auth list` | `gcloud config get-value project` |
| Lệnh đổi | `gcloud config set account <email>` | `gcloud config set project <id>` |

🛑 Đăng nhập đúng tài khoản (**account**) nhưng **project đang trỏ sai** ⇒ mọi lệnh `gcloud compute ...` chạy **nhầm chỗ** hoàn toàn — tạo VM ở dự án dev trong khi tưởng đang ở prod. Luôn kiểm tra CẢ HAI trước khi làm việc quan trọng:

```bash
gcloud config list
# [core]
# account = kiennv@company.vn
# project = company-prod-123456      <- ⭐ ĐỌC KỸ dòng này trước mọi lệnh nguy hiểm
```

| Lệnh | Làm gì |
|---|---|
| `gcloud auth login` | Đăng nhập bằng **trình duyệt** (cần mở được web — khó trên VDI air-gapped) |
| `gcloud auth login --no-launch-browser` | ⭐ In ra **URL + mã** để copy sang máy có mạng đăng nhập hộ |
| `gcloud auth activate-service-account` | Đăng nhập bằng **file key JSON** — dùng trong CI/CD, script tự động |
| `gcloud config configurations list` | ⭐ Liệt kê **bộ cấu hình đã lưu** (nhiều account+project cùng lúc) |
| `gcloud config configurations activate <name>` | Chuyển sang bộ cấu hình khác |

⭐ **`--no-launch-browser` — cứu tinh cho môi trường VDI/server không có trình duyệt:**

```bash
gcloud auth login --no-launch-browser
#                  └─ thay vì TỰ MỞ trình duyệt (sẽ lỗi trên server/VDI air-gapped),
#                     in ra một URL + LỆNH để bạn copy sang máy CÓ mạng, đăng nhập,
#                     rồi dán MÃ XÁC THỰC ngược lại vào đây
```

**Multiple configurations — làm việc với nhiều account/project không phải đổi qua đổi lại thủ công:**

```bash
gcloud config configurations create prod-config
gcloud config set account kiennv-prod@company.vn --configuration=prod-config
gcloud config set project company-prod-123456 --configuration=prod-config

gcloud config configurations activate prod-config    # chuyển TOÀN BỘ (account+project) một lệnh
gcloud config configurations list                     # xem đang active cái nào
```

⇒ Cách này an toàn hơn nhiều so với chỉ đổi `project` một mình — tránh ca "project đúng nhưng account vẫn của môi trường khác".

**Compute & GKE:**

```bash
gcloud compute instances list --filter="status=RUNNING"
#                              └─ cú pháp lọc RIÊNG của gcloud, không phải JMESPath (khác AWS)

gcloud compute ssh myinstance --zone=asia-southeast1-a
#                              └─ ⚠️ GCP yêu cầu chỉ định ZONE (không tự đoán như AWS region)
#                                 -> thiếu cờ này, gcloud có thể hỏi lại hoặc báo lỗi tuỳ cấu hình

gcloud container clusters get-credentials mycluster --region asia-southeast1
#                                                     └─ region CHO CỤM REGIONAL
#                                                        (cụm zonal thì dùng --zone thay vì --region)
```

⚠️ **`--zone` vs `--region`** — GKE có hai loại cụm: **zonal** (một vùng khả dụng, rẻ hơn) dùng `--zone`; **regional** (trải nhiều vùng, chịu lỗi tốt hơn) dùng `--region`. Gõ nhầm cờ ⇒ báo lỗi "cluster not found" dù cluster có thật.

**Logging:**

```bash
gcloud logging read "severity>=ERROR" --limit 20 --format=json
#                    └─ cú pháp lọc log RIÊNG của GCP (Cloud Logging query language)
#                       khác hẳn cú pháp PromQL hay LogQL đã gặp ở mục Monitoring
```

⚠️ Không có `--limit`, lệnh này có thể kéo về **rất nhiều** dòng nếu hệ thống log nhiều — luôn giới hạn khi thăm dò lần đầu.

**Docker/Artifact Registry:**

```bash
gcloud auth configure-docker
#      └─ cấu hình MỘT LẦN để `docker push/pull` tới GCR/Artifact Registry
#         tự động dùng credential của gcloud, KHÔNG cần `docker login` riêng
```

⚠️ Lệnh trên sửa file `~/.docker/config.json`, thêm credential helper — nếu sau đó gặp lỗi push lạ, kiểm tra file này có đúng cấu hình không bị công cụ khác ghi đè.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa az — subscription là khái niệm tương đương project của GCP</b></summary>

**Tiền đề — cấu trúc phân cấp của Azure, cần biết để đọc lệnh:**

```
Tenant (tổ chức)
  └─ Subscription (đơn vị TÍNH PHÍ — tương đương "project" của GCP, "account" của AWS)
       └─ Resource Group (nhóm chứa resource — Azure BẮT BUỘC mọi resource thuộc 1 group)
            └─ Resource (VM, Storage Account...)
```

⭐ **Khác biệt lớn nhất so với AWS/GCP: MỌI resource của Azure PHẢI nằm trong một Resource Group.** Đây không phải tuỳ chọn — khi tạo VM, Azure luôn hỏi (hoặc cần chỉ định) `--resource-group` (viết tắt `-g`). Xoá cả resource group là **xoá sạch mọi thứ bên trong nó cùng lúc** — cách nhanh nhất để dọn một môi trường test, nhưng cũng nguy hiểm nhất nếu gõ nhầm tên.

| Lệnh | Làm gì |
|---|---|
| `az login` | Đăng nhập (mở trình duyệt, hoặc `--use-device-code` cho máy không có trình duyệt) |
| `az account show` | ⭐ Subscription **đang active** |
| `az account list` | Mọi subscription mà account này có quyền |
| `az account set --subscription <id>` | Chuyển subscription |
| `-g` / `--resource-group` | Chỉ định Resource Group |
| `-o table` | Định dạng bảng dễ đọc (thay JSON mặc định) |

⭐ **`--use-device-code` — như `--no-launch-browser` của gcloud, dùng khi không có trình duyệt:**

```bash
az login --use-device-code
#         └─ in ra MÃ, bạn nhập mã đó vào https://microsoft.com/devicelogin trên MÁY KHÁC CÓ MẠNG
```

**Kiểm tra trước khi làm việc nguy hiểm — giống hệt tinh thần `aws sts get-caller-identity`:**

```bash
az account show --output table
#                └─ ⭐ LUÔN kiểm tra SubscriptionName trước khi az vm delete / az group delete
```

**Resource Group — hiểu rõ trước khi xoá:**

```bash
az group list -o table                    # liệt kê mọi resource group
az group show -n my-rg                    # chi tiết một group
az resource list -g my-rg -o table         # ⭐ xem TRƯỚC những gì nằm trong group này
```

🛑 **`az group delete -n my-rg` xoá TOÀN BỘ resource bên trong, không hỏi lại từng cái.** Đây là cách nhanh nhất dọn một môi trường thử nghiệm, nhưng cũng là lệnh nguy hiểm nhất trong Azure CLI — luôn `az resource list -g <rg>` xem trước.

```bash
az group delete -n my-rg --yes --no-wait
#                         │     └─ chạy NGẦM, trả lệnh lại ngay (không chờ xoá xong)
#                         └────── bỏ qua câu hỏi xác nhận — ⚠️ CHỈ dùng khi CHẮC CHẮN đúng group
```

**VM:**

```bash
az vm list -o table
az vm start --name myvm -g my-rg           # -g bắt buộc: Azure cần biết VM nằm ở group nào
az vm deallocate --name myvm -g my-rg      # ⭐ khác `stop` — xem bảng dưới
```

⚠️ **`az vm stop` vs `az vm deallocate` — khác biệt về TIỀN, không chỉ về trạng thái:**

| Lệnh | Máy tắt? | Vẫn tính phí **compute**? | Vẫn giữ IP public? |
|---|---|---|---|
| `az vm stop` | ✅ | 🛑 **CÓ** (tài nguyên vẫn được giữ chỗ) | ✅ Giữ |
| `az vm deallocate` | ✅ | ✅ **Không** (giải phóng phần cứng) | ⚠️ Có thể mất IP động |

⇒ Muốn thật sự **ngừng tính phí** khi tắt VM tạm thời (qua đêm, cuối tuần), phải dùng **`deallocate`**, không phải `stop`.

**AKS (Kubernetes trên Azure):**

```bash
az aks get-credentials --name mycluster -g my-rg
#                                         └─ ⚠️ vẫn cần -g, vì AKS cũng nằm trong resource group
```

**ACR (Container Registry):**

```bash
az acr login --name myregistry
#             └─ ⭐ chỉ cần TÊN registry, az tự suy ra domain đầy đủ (myregistry.azurecr.io)
#                az login trước đó đã có sẵn quyền -> KHÔNG cần thêm docker login riêng
```

⚠️ `az acr login` cần **Docker daemon đang chạy** trên máy (nó gọi `docker login` phía sau) — báo lỗi nếu Docker chưa khởi động, dễ nhầm tưởng lỗi xác thực Azure.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa PromQL từ số 0 — rate(), histogram_quantile</b></summary>

**Tiền đề — Prometheus lưu dữ liệu khác database thường thế nào?** Nó lưu **time series**: mỗi metric là một **dòng số theo thời gian**, có nhãn (label) để phân biệt (`method="GET"`, `pod="api-1"`). PromQL là ngôn ngữ **truy vấn** những dòng số đó — khác hẳn SQL, quen tay SQL sẽ thấy lạ lẫm ban đầu.

| API endpoint | Làm gì |
|---|---|
| `/api/v1/query` | Query tại **một thời điểm** (mặc định: bây giờ) |
| `/api/v1/query_range` | Query theo **khoảng thời gian** — vẽ được biểu đồ |
| `/api/v1/targets` | Danh sách **mục tiêu** Prometheus đang scrape (thu thập) |

```bash
curl -s 'http://localhost:9090/api/v1/query?query=up' | jq
#                                          └─ tham số "query" chính là câu PromQL, URL-encode nếu có ký tự đặc biệt
curl -s 'http://localhost:9090/api/v1/targets' | jq '.data.activeTargets[] | {job: .labels.job, health: .health}'
#                                                     └─ ⭐ .health = "up"/"down": target đang SỐNG hay MẤT KẾT NỐI
```

**`up` — metric đầu tiên cần biết:** Prometheus **tự sinh** metric này cho mọi target — `1` = scrape thành công (service đang trả metric), `0` = scrape thất bại (service sập **hoặc** không mở được cổng metric).

⭐ **`rate()` — hàm quan trọng nhất PromQL, và lý do phải dùng nó thay vì đọc số thô:**

Metric kiểu **counter** (`http_requests_total`) chỉ **tăng dần**, không bao giờ giảm (trừ khi service restart về 0). Đọc **giá trị thô** của counter (ví dụ "đã có 58 triệu request") **vô nghĩa** để biết tải hiện tại — phải biết **tốc độ tăng**.

```promql
rate(http_requests_total[5m])
#    └──────────────────────┘└─ [5m]: cửa sổ THỜI GIAN tính trung bình — 5 PHÚT gần nhất
#    └─ counter cần tính rate
# => kết quả: SỐ REQUEST MỖI GIÂY, trung bình trong 5 phút qua
```

🛑 **`[5m]` không phải "làm mượt biểu đồ cho đẹp"** — nó là **cửa sổ tính trung bình bắt buộc**. Cửa sổ **quá ngắn** (`[30s]`) ⇒ đồ thị **giật cục**, nhiễu theo từng lần scrape. Cửa sổ **quá dài** (`[1h]`) ⇒ **san phẳng mất** những đợt tăng đột biến ngắn (spike) — cái mà bạn thường **cần thấy nhất** khi debug sự cố.

⚠️ **Cửa sổ `[5m]` phải LỚN HƠN khoảng thời gian scrape** (`scrape_interval`, thường 15s-30s) — ít nhất gấp 4 lần, để luôn có đủ ít nhất 2 điểm dữ liệu để tính rate. Cửa sổ quá sát với scrape_interval sẽ ra kết quả **rỗng hoặc `NaN`** thất thường.

⭐ **`sum by (...)` — gộp nhiều dòng số lại theo nhãn:**

```promql
sum(rate(http_requests_total[5m])) by (status)
#                                    └─ GỘP tất cả pod/instance lại, CHỈ giữ phân biệt theo "status"
# => ra: {status="200"}: 120  {status="500"}: 3   -- tổng request/giây theo TỪNG mã lỗi
```

⚠️ Không có `sum by`, kết quả sẽ là **hàng chục dòng riêng** (mỗi pod một dòng) — không nhìn ra được bức tranh tổng.

⭐ **`histogram_quantile` — công thức tính p95/p99 latency, mảnh khó nhất PromQL:**

```promql
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
#                   │                                            └─ ⭐ PHẢI là metric có hậu tố "_bucket"
#                   │                                               (không phải _sum hay _count)
#                   └─ 0.95 = phân vị thứ 95 (p95): "95% request nhanh hơn con số này"
```

**Tiền đề bắt buộc phải hiểu**: một metric **histogram** trong Prometheus thực chất sinh ra **3 metric riêng**:

| Hậu tố | Ý nghĩa |
|---|---|
| `_bucket{le="0.5"}` | Đếm số request có độ trễ **≤** giá trị `le` (less-or-equal), theo từng ngưỡng |
| `_sum` | **Tổng** thời gian tất cả request cộng lại |
| `_count` | **Số lượng** request |

⇒ `histogram_quantile` chỉ dùng được với `_bucket` vì nó cần **phân bố** theo ngưỡng, không phải tổng hay đếm đơn thuần.

⭐ **p95 vs trung bình (average) — vì sao dashboard nên ưu tiên p95:**

Trung bình bị **một request rất chậm kéo lệch nhẹ**, nhưng vẫn có thể nhìn "ổn" nếu đa số request nhanh. **p95** trả lời đúng câu hỏi vận hành thật sự quan tâm: *"95% người dùng trải nghiệm nhanh hơn bao nhiêu?"* — phản ánh trải nghiệm của **đa số**, không bị outlier che lấp cũng không bị outlier bỏ qua.

**Các câu PromQL hay dùng khác — bóc nhanh:**

```promql
node_memory_MemAvailable_bytes                         # RAM CÒN DÙNG ĐƯỢC (không phải "free" trần)
100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
#                                                └─ mode="idle" = %thời gian CPU RẢNH RỖI
#      └─ 100% trừ đi %rảnh = %ĐANG DÙNG (đây là công thức chuẩn, không có metric "%cpu used" trực tiếp)
```

⚠️ Prometheus **không lưu vĩnh viễn** — mặc định giữ **15 ngày** rồi tự xoá dữ liệu cũ (cấu hình bằng `--storage.tsdb.retention.time`). Cần lưu lâu hơn (audit, so sánh theo quý) phải remote-write sang **Thanos/Cortex/Mimir** hoặc kho lưu trữ dài hạn khác.

</details>

### Xem metric endpoint & health check
```bash
curl -s localhost:8080/metrics         # Metric dạng Prometheus của app
curl -s localhost:8080/health          # Health check endpoint
curl -s localhost:8080/actuator/health # Spring Boot health
watch -n 2 'curl -s localhost:8080/health'   # Theo dõi health mỗi 2s
```

<details>
<summary><b>Bấm xem: /metrics vs /health khác nhau chỗ nào</b></summary>

⭐ **Hai endpoint cùng "kiểm tra service" nhưng trả lời hai câu hỏi khác hẳn:**

| | `/health` | `/metrics` |
|---|---|---|
| Trả lời | *"Service này SỐNG hay CHẾT?"* | *"Service đang vận hành RA SAO?"* |
| Định dạng | Thường gọn: `{"status":"ok"}` hoặc chỉ mã HTTP 200 | ⭐ Dạng **Prometheus text format**, hàng trăm dòng số |
| Ai gọi | K8s (**liveness/readiness probe**), load balancer | Prometheus (**scrape** định kỳ, ví dụ mỗi 15s) |
| Tần suất gọi | Vài giây/lần | Vài chục giây/lần |

```bash
curl -s localhost:8080/health
# {"status":"ok","database":"connected"}
#              └─ ⭐ health check TỐT nên tự kiểm tra CẢ dependency (DB, cache),
#                    không chỉ trả "tôi còn sống" mà bỏ qua DB đã chết

curl -s localhost:8080/metrics | head -20
# # HELP http_requests_total Total HTTP requests
# # TYPE http_requests_total counter
# http_requests_total{method="GET",status="200"} 15234
#                       └─ ⭐ đây chính là DÒNG mà Prometheus đọc vào để chạy PromQL ở mục trên
```

⭐ **Đọc format Prometheus text — 3 dòng lặp lại cho mỗi metric:**

```
# HELP <tên>  <mô tả>       <- dòng giải thích, không phải dữ liệu
# TYPE <tên>  <loại>        <- counter / gauge / histogram / summary
<tên>{nhãn="giá trị"} <số>  <- ⭐ ĐÂY mới là dữ liệu thật
```

| `TYPE` | Có thể **giảm** không? | Ví dụ |
|---|---|---|
| `counter` | ❌ Chỉ tăng (reset về 0 khi restart) | Tổng số request |
| `gauge` | ✅ Lên xuống tự do | RAM đang dùng, số kết nối hiện tại |
| `histogram` | (đặc biệt — sinh ra `_bucket`/`_sum`/`_count`, xem mục PromQL) | Độ trễ request |

⚠️ **Nhầm `counter` với `gauge` khi query là lỗi hay gặp**: dùng `rate()` (dành cho counter) trên một `gauge` sẽ cho ra **con số vô nghĩa**, vì gauge không có tính chất "tăng dần đều".

⭐ **`watch -n 2 'curl -s localhost:8080/health'` — theo dõi mà không cần Grafana/Prometheus:**

```bash
watch -n 2 'curl -s -o /dev/null -w "%{http_code} %{time_total}s\n" localhost:8080/health'
#                     │                └─ %{http_code}: mã HTTP · %{time_total}: thời gian phản hồi
#                     └─ vứt nội dung, chỉ giữ 2 số liệu quan tâm
# => mỗi 2 giây in ra 1 dòng "200 0.012s" -> theo dõi service có đang GIẬT/CHẬM DẦN không
```

⭐ **Ba loại probe của Kubernetes dùng chung endpoint kiểu health, nhưng Ý NGHĨA khác nhau — hiểu sai là gây downtime:**

| Probe | Fail thì K8s làm gì | Nên kiểm tra gì bên trong |
|---|---|---|
| `livenessProbe` | 🔴 **Restart container** | Chỉ tiến trình còn sống không (deadlock, treo) |
| `readinessProbe` | Gỡ khỏi Service, **KHÔNG restart** | Đã sẵn sàng nhận traffic chưa (kết nối DB xong chưa) |
| `startupProbe` | Cho thời gian khởi động dài trước khi 2 probe trên bắt đầu tính | App khởi động xong lần đầu chưa |

🛑 **Sai lầm hay gặp**: `livenessProbe` gọi `/health` mà `/health` lại **kiểm tra cả kết nối database**. Database chậm tạm thời (không phải app lỗi) ⇒ `/health` fail ⇒ K8s **restart cả container** ⇒ mất kết nối đang xử lý dở, làm tình hình **tệ hơn** thay vì tốt lên. ⇒ `liveness` nên **đơn giản** (chỉ kiểm tra tiến trình còn phản hồi), việc kiểm tra dependency nên để cho `readiness`.

</details>

### Grafana / Loki / cAdvisor (tham khảo)
```bash
# Grafana:   thường ở http://localhost:3000 (admin/admin)
# Loki:      LogQL để query log, ví dụ: {app="myapp"} |= "error"
logcli query '{app="myapp"} |= "error"'   # Query Loki bằng CLI (nếu có logcli)
# cAdvisor:  http://localhost:8080 -> xem metric container realtime
```

<details>
<summary><b>Bấm xem: giải nghĩa LogQL — và vì sao Loki khác ELK</b></summary>

⭐ **Tiền đề — Loki khác Elasticsearch (ELK) ở TRIẾT LÝ, không chỉ ở cú pháp:**

| | Elasticsearch (ELK) | Loki |
|---|---|---|
| Đánh index | **Toàn bộ nội dung** log (full-text search) | ⭐ **CHỈ index nhãn** (label) — `app`, `namespace`, `pod` |
| Nội dung log | Được **phân tích, đánh index** để tìm bất kỳ từ nào | Lưu **nguyên khối nén**, KHÔNG đánh index nội dung |
| Chi phí lưu trữ | Cao (index nặng) | ⭐ **Thấp hơn nhiều** |
| Tốc độ tìm theo nhãn | Nhanh | ⭐ Rất nhanh (đây là sở trường) |
| Tốc độ tìm full-text | Nhanh | Chậm hơn (phải quét nội dung từng khối) |

⇒ Loki đánh đổi: **tìm theo nhãn cực nhanh và rẻ**, đổi lấy **tìm chữ trong nội dung chậm hơn** ELK. Đây là lý do Loki phù hợp với Kubernetes — nơi *"log của pod X trong namespace Y"* (tìm theo nhãn) là truy vấn phổ biến nhất.

**LogQL — cú pháp mượn ý tưởng từ PromQL nhưng cho log:**

```logql
{app="myapp"} |= "error"
 │            │  └─ chuỗi cần TÌM (phải khớp nguyên văn, phân biệt hoa/thường)
 │            └──── |= : "PHẢI CHỨA" chuỗi này (đảo ngược bằng != hoặc !~)
 └───────────────── ⭐ SELECTOR: BẮT BUỘC phải có, lọc theo NHÃN trước
                     (đây chính là phần Loki index và tìm nhanh)
```

⚠️ **Selector `{}` không được để trống hoặc quá rộng.** `{app=~".+"}` (khớp mọi giá trị) buộc Loki phải quét **mọi log của mọi ứng dụng** ⇒ chậm, tốn tài nguyên. Luôn thu hẹp nhãn nhất có thể trước khi thêm điều kiện nội dung.

**Toán tử lọc nội dung:**

| Toán tử | Nghĩa |
|---|---|
| `\|=` | **Chứa** chuỗi này |
| `!=` | **KHÔNG chứa** chuỗi này |
| `\|~` | Khớp theo **regex** |
| `!~` | **KHÔNG** khớp regex |

```logql
{namespace="ai-hub", app="litellm"} |= "error" != "healthcheck"
#                                    │              └─ loại bỏ nhiễu (log healthcheck không phải lỗi thật)
#                                    └─ có chữ "error"
#  {namespace="ai-hub", app="litellm"}  <- ⭐ hai nhãn CÙNG LÚC, cách nhau bằng dấu phẩy = VÀ (AND)
```

⭐ **Đếm số dòng lỗi theo thời gian — LogQL cũng làm được thống kê như PromQL:**

```logql
sum(count_over_time({app="myapp"} |= "error" [5m]))
#      └─────────┘  └────────────────────────┘└──┘
#      đếm SỐ DÒNG   điều kiện lọc log NHƯ TRÊN  cửa sổ 5 phút
#  => ra một con số THEO THỜI GIAN, vẽ được lên Grafana y hệt một metric Prometheus
```

⇒ Đây là điểm mạnh cực lớn của Loki: **cùng một Grafana, cùng ngôn ngữ tư duy** như PromQL, để vừa xem metric vừa xem log — không cần học một hệ thống hoàn toàn khác.

**`logcli` — chạy LogQL từ terminal, không cần mở Grafana:**

```bash
logcli query '{app="myapp"} |= "error"' --since=1h --limit=100
#                                        │           └─ giới hạn số dòng trả về
#                                        └─────────── chỉ log trong 1 giờ gần đây
```

**Grafana — vài điều cần biết khi mới cài:**

```
http://localhost:3000    (admin/admin mặc định — ⚠️ BẮT BUỘC đổi ngay lần đăng nhập đầu)
```

⚠️ Grafana **không tự lưu trữ dữ liệu** — nó chỉ là **lớp hiển thị**, kết nối vào Prometheus/Loki/database khác qua "Data Source". Xoá Grafana không mất dữ liệu metric/log; xoá Prometheus/Loki thì **mất thật**.

**cAdvisor — metric container ở tầng thấp hơn `docker stats`:**

```
http://localhost:8080    -> giao diện web xem CPU/RAM/network/disk của TỪNG container, realtime
```

💡 cAdvisor thường **được tích hợp sẵn bên trong kubelet** trên mỗi node K8s (không cần cài riêng) — chính là nguồn dữ liệu phía sau `kubectl top pods`. Cài độc lập chỉ cần khi chạy Docker đơn lẻ, không qua K8s.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa Kafka — partition, offset, consumer group, và LAG</b></summary>

**Tiền đề — 3 khái niệm phải hiểu trước khi đọc lệnh:**

| Khái niệm | Là gì | Ví von |
|---|---|---|
| **Topic** | "Chủ đề" chứa message | Tên một cuốn sổ |
| **Partition** | Topic được **chia nhỏ** thành nhiều mảnh song song | Nhiều cuốn sổ CON của cùng chủ đề, đánh số 0,1,2... |
| **Offset** | Vị trí **thứ tự** của một message TRONG một partition | Số trang trong cuốn sổ đó |
| **Consumer Group** | Nhóm consumer **chia nhau** đọc các partition | Nhiều người cùng đọc, mỗi người phụ trách vài cuốn sổ con |

⭐ **Vì sao chia partition?** Một partition chỉ đọc/ghi được **tuần tự bởi một consumer tại một thời điểm** (trong cùng group) ⇒ chia nhiều partition để **nhiều consumer xử lý song song** ⇒ tăng thông lượng. Đánh đổi: **thứ tự chỉ được đảm bảo TRONG một partition**, không đảm bảo giữa các partition với nhau.

**Quản lý Topic:**

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --list
#                └─ ⭐ "bootstrap": chỉ cần biết 1-2 broker để hỏi,
#                   Kafka tự trả về địa chỉ TOÀN BỘ cluster từ đó

kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic orders
#                                                   │          └─ xem CHI TIẾT: bao nhiêu partition,
#                                                   │             ai là leader, ai là replica của từng partition
#                                                   └──────────── mô tả

kafka-topics.sh --bootstrap-server localhost:9092 --create --topic orders \
  --partitions 3 --replication-factor 2
#                  │                   └─ ⭐ MỖI partition có 2 BẢN SAO (1 leader + 1 replica)
#                  │                      -> mất 1 broker vẫn CÒN dữ liệu
#                  └───────────────────── chia thành 3 mảnh song song
```

⚠️ **`--partitions` chỉ TĂNG được, KHÔNG GIẢM được** sau khi tạo. Và tăng partition sau này sẽ **phá vỡ thứ tự** dựa trên key cũ (message cùng key trước đây vào cùng partition, sau khi tăng partition có thể **rơi vào partition khác**). ⇒ Cân nhắc kỹ số partition **trước khi** đưa vào production, tăng partition không phải thao tác "chỉnh sửa nhỏ".

**Producer / Consumer — công cụ test nhanh, không dùng cho production thật:**

```bash
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic orders
#  (gõ từng dòng, Enter là gửi 1 message; Ctrl+D thoát)

kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic orders --from-beginning
#                                                                          └─ ⭐ đọc TỪ ĐẦU
#                                                                             (thiếu cờ này = chỉ thấy message MỚI từ giờ trở đi,
#                                                                              message cũ đã có trước khi bạn chạy consumer thì KHÔNG thấy)
```

⭐ **Consumer Group & LAG — bảng chẩn đoán quan trọng nhất khi vận hành Kafka:**

```bash
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group order-service
```

```
GROUP          TOPIC   PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
order-service  orders  0          15234           15234           0      <- đã đọc HẾT, không tồn đọng
order-service  orders  1          8200            15500           7300   <- ⭐ TỒN 7300 message CHƯA XỬ LÝ
```

| Cột | Nghĩa |
|---|---|
| `CURRENT-OFFSET` | Consumer đã **đọc tới đâu** |
| `LOG-END-OFFSET` | Message **mới nhất** đã được ghi vào partition đó |
| **`LAG`** | ⭐ `LOG-END-OFFSET − CURRENT-OFFSET` = số message **đang chờ xử lý** |

🛑 **`LAG` là chỉ số quan trọng bậc nhất khi vận hành hệ thống hàng đợi.** `LAG` tăng liên tục theo thời gian ⇒ **consumer xử lý chậm hơn tốc độ producer gửi vào** ⇒ hàng đợi phình to mãi ⇒ cần: thêm consumer (nếu còn partition rảnh để chia), tối ưu code xử lý, hoặc consumer đang **bị treo/crash** mà không ai biết.

⚠️ **Số consumer HIỆU QUẢ tối đa = số partition.** Thêm consumer thứ 4 vào group trong khi topic chỉ có 3 partition ⇒ consumer thứ 4 đó **ngồi không, không nhận việc gì** — vì một partition chỉ giao cho đúng một consumer trong cùng group tại một thời điểm.

```bash
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
#                                                            └─ liệt kê MỌI consumer group đang tồn tại
```

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa rabbitmqctl — queue, exchange, và purge nguy hiểm</b></summary>

**Tiền đề — khác Kafka ở mô hình:** Kafka là **log bất biến** (message ở lại, nhiều consumer đọc lại được). RabbitMQ là **hàng đợi truyền thống**: message được **giao cho một consumer rồi biến mất khỏi queue** (trừ khi cấu hình đặc biệt). Chọn công cụ nào tuỳ bài toán: cần **replay lại lịch sử** ⇒ Kafka; cần **định tuyến phức tạp** (routing key, fanout) ⇒ RabbitMQ.

| Khái niệm | Là gì |
|---|---|
| **Exchange** | "Trạm phân loại" — nhận message rồi **quyết định đẩy vào queue nào** theo routing key |
| **Queue** | Nơi message **nằm chờ** được consumer lấy |
| **Consumer** | Tiến trình lấy message ra xử lý |
| **Connection** | Kết nối TCP tới RabbitMQ |

```bash
rabbitmqctl status
#           └─ trạng thái NODE: version, RAM, disk còn lại, số kết nối

rabbitmqctl list_queues name messages consumers
#                       └──────────────────────┘ chỉ định RÕ các CỘT muốn xem
#                       (không ghi gì -> mặc định chỉ hiện name + messages, thiếu consumers)
```

Đọc kết quả:

| Cột | Ý nghĩa |
|---|---|
| `messages` | Số message **đang tồn đọng** trong queue |
| `consumers` | ⭐ Số tiến trình **đang lắng nghe** queue đó |

🛑 **`consumers = 0` mà `messages` đang tăng — dấu hiệu sự cố nghiêm trọng nhất cần canh:**

```bash
rabbitmqctl list_queues name messages consumers | awk '$3==0 && $2>0'
#                                                       │       └─ có message tồn
#                                                       └─ KHÔNG consumer nào đang nghe
#  => queue này bị "mồ côi": message vào liên tục mà KHÔNG AI xử lý -> phình vô hạn
```

⇒ Nguyên nhân thường là: consumer đã crash, deploy sai làm mất kết nối, hoặc chưa từng có ai subscribe đúng queue này.

**Chẩn đoán nghẽn:**

```bash
rabbitmqctl list_connections            # kết nối TCP đang mở — nhiều bất thường có thể là leak
rabbitmqctl list_consumers               # ⭐ chi tiết TỪNG consumer: queue nào, ack_required, prefetch
rabbitmqctl list_exchanges               # danh sách exchange + loại (direct/topic/fanout/headers)
```

⚠️ **`ack_required` (trong `list_consumers`) — vì sao quan trọng:** RabbitMQ chỉ **xoá hẳn** message khỏi queue sau khi consumer gửi **acknowledgement** (xác nhận đã xử lý xong). Consumer **crash TRƯỚC KHI ack** ⇒ message **tự động quay lại queue** để giao cho consumer khác — đây là cơ chế **đảm bảo không mất message**, nhưng cũng là lý do một message có thể được xử lý **nhiều lần** nếu code không idempotent (chạy lại vẫn ra kết quả đúng).

🛑 **`purge_queue` — xoá sạch, không hoàn tác:**

```bash
rabbitmqctl purge_queue orders
#                       └─ 🔴 XOÁ TOÀN BỘ message ĐANG CHỜ trong queue này, MẤT VĨNH VIỄN
```

⇒ Chỉ dùng khi **chắc chắn** dữ liệu trong queue không còn giá trị (ví dụ dọn queue test, hoặc sau khi đã xác nhận nghiệp vụ chấp nhận bỏ qua các message tồn đọng). Không có bước hỏi lại — gõ Enter là mất ngay.

**Bật giao diện quản lý (web UI):**

```bash
rabbitmq-plugins enable rabbitmq_management
#                        └─ tên plugin CỐ ĐỊNH, không đổi được
# => truy cập http://<host>:15672  (user/pass mặc định thường là guest/guest —
#    ⚠️ tài khoản guest CHỈ được phép đăng nhập từ localhost theo mặc định bảo mật của RabbitMQ)
```

⚠️ Đăng nhập `guest/guest` từ xa (không phải localhost) sẽ bị **từ chối theo thiết kế bảo mật mặc định** — không phải plugin lỗi. Cần tạo user riêng có quyền phù hợp để truy cập UI từ máy khác:

```bash
rabbitmqctl add_user admin 'MatKhauManh'
rabbitmqctl set_user_tags admin administrator
#                              └─ gán quyền QUẢN TRỊ đầy đủ trên UI
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
#                            └─ vhost "/" (mặc định)  -- ba dấu ".*" = quyền configure/write/read TRÊN MỌI resource
```

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa nginx -t/-s/-T và bảng mã lỗi 502/504/413</b></summary>

⭐ **`nginx -t` — LỆNH BẮT BUỘC chạy trước MỌI lần reload, không có ngoại lệ:**

| Cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-t` | **t**est | ⭐ Kiểm tra **cú pháp** config, KHÔNG áp dụng gì |
| `-T` | **T** hoa | Như `-t` nhưng **in luôn toàn bộ config đã gộp** ra màn hình |
| `-s reload` | **s**ignal | Gửi tín hiệu **nạp lại config**, giữ nguyên tiến trình worker cũ tới khi worker mới sẵn sàng — **không downtime** |
| `-s stop` | | Dừng **ngay lập tức** (nặng tay) |
| `-s quit` | | Dừng **êm** — xử lý nốt request đang dở rồi mới thoát |
| `-V` | **V**ersion | Version + danh sách **module đã build kèm** |

🛑 **Không `nginx -t` trước khi reload = đánh cược cả service.** Nếu config có lỗi cú pháp, `nginx -s reload` sẽ **thất bại và giữ nguyên config CŨ đang chạy** — nghe có vẻ an toàn, nhưng nếu đang trong quy trình tự động (CI/CD) mà không kiểm tra kỹ, bạn có thể tưởng đã áp dụng bản mới trong khi **thực tế vẫn chạy bản cũ**, gây lệch giữa cái bạn nghĩ đang chạy và cái thật sự đang chạy.

```bash
nginx -t && nginx -s reload
#     │      └─ CHỈ reload nếu -t THÀNH CÔNG (nhờ &&)
#     └──────── luôn kiểm tra CÚ PHÁP trước
```

⚠️ **`-t` chỉ bắt lỗi CÚ PHÁP** (thiếu dấu `;`, sai tên directive) — **KHÔNG bắt được** lỗi logic (route sai upstream, mở port trùng process khác). Cú pháp đúng không đảm bảo hành vi đúng.

⭐ **`reload` vs `stop`/`quit` — vì sao reload không gây downtime:**

Nginx dùng mô hình **master + nhiều worker**. `reload` bảo master: **tạo worker MỚI với config mới**, để worker **cũ** xử lý nốt request đang dở, rồi mới tắt worker cũ. Trong toàn bộ quá trình, **luôn có ít nhất một worker đang phục vụ** ⇒ client không hề thấy gián đoạn.

```bash
systemctl reload nginx      # ⭐ tương đương nginx -s reload, ưu tiên dùng khi chạy qua systemd
systemctl restart nginx     # 🛑 dừng HẲN rồi bật lại -> CÓ khoảng trống ngắn, mất kết nối đang mở
```

**Vị trí file config — thứ tự đọc quan trọng khi debug "sửa mà không ăn":**

```
/etc/nginx/nginx.conf        # file GỐC, include các file dưới
/etc/nginx/conf.d/*.conf     # config bổ sung, TỰ ĐỘNG được include (không cần symlink)
/etc/nginx/sites-available/  # ⭐ CHỈ LÀ NƠI LƯU — nginx KHÔNG tự đọc thư mục này
/etc/nginx/sites-enabled/    # ⭐ nginx CHỈ đọc những gì NẰM Ở ĐÂY (thường là symlink trỏ vào sites-available)
```

🛑 Sửa file trong `sites-available/` mà **quên tạo symlink** sang `sites-enabled/` ⇒ nginx **không bao giờ đọc** file đó, dù cú pháp hoàn toàn đúng:

```bash
ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/myapp
#     └─ ĐÍCH (file thật)                └─ TÊN LIÊN KẾT (chỗ nginx thực sự nhìn vào)
```

**`-T` — công cụ điều tra khi nghi ngờ "config trên đĩa khác config đang chạy":**

```bash
nginx -T | grep -A 5 "server_name api.company.vn"
#     │                └─ ⭐ in TOÀN BỘ config đã GỘP các include lại,
#     │                     đúng CHÍNH XÁC những gì nginx nhìn thấy — không phải bạn tự suy luận từ nhiều file
```

**Bảng mã lỗi — đọc log rồi tra ngay, đừng đoán:**

| Mã | Tên | Nguyên nhân | Kiểm tra |
|---|---|---|---|
| `502` | Bad Gateway | **Backend không phản hồi** — chết, sai `proxy_pass`, sai port | `curl` thẳng vào backend, `docker ps`/`kubectl get pods` |
| `504` | Gateway Timeout | Backend **có phản hồi** nhưng **quá chậm**, vượt `proxy_read_timeout` | Tăng timeout **hoặc** tối ưu backend — hai hướng khác nhau |
| `413` | Request Entity Too Large | Body request **vượt** `client_max_body_size` (mặc định chỉ **1MB**) | `client_max_body_size 20M;` |
| `499` | (riêng của Nginx) | **Client tự đóng** kết nối trước khi server trả lời xong | Không phải lỗi server — thường do client timeout ngắn hơn |

⚠️ **502 và 504 nghe giống nhau nhưng chẩn đoán ngược hướng nhau:** 502 = **không kết nối được tới backend** (backend coi như không tồn tại với nginx); 504 = **kết nối được, nhưng chờ mãi không xong** (backend tồn tại, chỉ là chậm). Tăng `proxy_read_timeout` chữa được 504 nhưng **không chữa được** 502 — 502 phải sửa ở tầng kết nối/backend.

**Đếm lỗi theo mã trạng thái — công thức đã gặp ở mục awk phía trên:**

```bash
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head
#            └─ ⭐ cột 9 trong log format "combined" mặc định = mã HTTP status
#               (nếu log format tùy chỉnh khác, số cột có thể khác — kiểm tra log_format trong config)
```

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa ufw — và bẫy tự khoá mất SSH của chính mình</b></summary>

**Tiền đề — `ufw` là gì?** **U**ncomplicated **F**ire**w**all — lớp bọc **dễ dùng hơn** cho `iptables`/`nftables` bên dưới (Ubuntu/Debian mặc định dùng). Bạn gõ `ufw allow 80/tcp`, nó tự sinh ra các rule iptables phức tạp phía sau.

| Lệnh | Làm gì |
|---|---|
| `ufw status` | Trạng thái + danh sách rule đang bật |
| `ufw status verbose` | ⭐ Thêm: **default policy**, logging level |
| `ufw status numbered` | ⭐ Đánh **số thứ tự** — cần số này để `delete` chính xác |
| `ufw enable` / `disable` | Bật / tắt toàn bộ firewall |
| `ufw allow <port>/<proto>` | Mở cổng |
| `ufw allow from <ip>` | Cho phép **toàn bộ port** từ một IP |
| `ufw allow from <ip> to any port <port>` | Chỉ cho IP đó vào **đúng** port này |
| `ufw deny <port>` | Chặn cổng |
| `ufw delete allow <port>` | Xoá rule (ghi **lại đúng** rule đã tạo) |

🛑🛑 **Bẫy nguy hiểm nhất: `ufw enable` TỰ KHOÁ MẤT SSH của chính bạn.**

```bash
ufw enable
#    └─ 🛑 nếu CHƯA có rule "allow 22" (hoặc "allow ssh") từ trước,
#       ufw áp default policy "deny incoming" cho MỌI THỨ, kể cả port bạn đang SSH vào
#       -> KẾT NỐI SSH HIỆN TẠI BỊ CẮT, và bạn KHÔNG THỂ SSH LẠI để sửa
```

⇒ **Quy tắc bắt buộc, không có ngoại lệ**: LUÔN `ufw allow ssh` (hoặc đúng port SSH đang dùng) **TRƯỚC KHI** `ufw enable`:

```bash
ufw allow OpenSSH          # hoặc: ufw allow 22/tcp  (nếu SSH đổi port khác 22, ghi đúng số đó)
ufw allow ssh              # tương đương, tên "ssh" được ufw hiểu sẵn theo /etc/services
ufw enable                 # BÂY GIỜ mới bật, sau khi đã chắc SSH được phép
```

⚠️ Nếu **đã lỡ** enable mà mất SSH: chỉ còn cách vào qua **console vật lý/console cloud provider** (AWS Session Manager, GCP Serial Console...) để `ufw allow 22` từ bên trong.

⭐ **Đọc `ufw status verbose` — hiểu "default deny" để tránh bất ngờ:**

```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
#         └─ ⭐ MẶC ĐỊNH: mọi kết nối VÀO đều bị CHẶN, trừ khi có rule allow rõ ràng
#            (đây là mô hình "trắng danh sách" — an toàn hơn "đen danh sách")
To                         Action      From
22/tcp                     ALLOW       Anywhere
```

**Xoá rule — dùng số thứ tự để chính xác, tránh gõ nhầm cú pháp:**

```bash
ufw status numbered
# [1] 22/tcp    ALLOW IN    Anywhere
# [2] 80/tcp    ALLOW IN    Anywhere
ufw delete 2
#           └─ ⭐ xoá THEO SỐ — an toàn hơn gõ lại "ufw delete allow 80"
#              (nếu rule gốc phức tạp, gõ lại dễ SAI CÚ PHÁP và không khớp -> ufw báo không tìm thấy)
```

⚠️ **Số thứ tự thay đổi sau mỗi lần xoá** — xoá rule `[2]` xong thì rule cũ `[3]` trở thành `[2]`. Luôn `ufw status numbered` **lại** trước khi xoá tiếp rule khác, đừng xoá liên tiếp theo một danh sách số đã lấy từ trước.

**Giới hạn theo IP cụ thể — thu hẹp bề mặt tấn công:**

```bash
ufw allow from 10.0.0.0/24 to any port 5432
#              │                       └─ CHỈ port 5432 (Postgres)
#              └─ ⭐ CHỈ dải mạng nội bộ này được vào — không mở toang ra Internet
```

⇒ Đây là cách đúng đắn để mở database chỉ cho mạng nội bộ, thay vì `ufw allow 5432` (mở cho **mọi IP trên Internet**).

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa iptables — chain, target, và vì sao -D xoá nhầm rule âm thầm</b></summary>

**Tiền đề — mô hình chain/target, bắt buộc hiểu trước khi đọc rule:**

Gói tin đi qua **chain** (chuỗi luật) theo thứ tự, mỗi rule trong chain có một **target** (hành động):

| Chain | Áp dụng cho gói tin |
|---|---|
| `INPUT` | Đi **VÀO** máy này |
| `OUTPUT` | Đi **RA** từ máy này |
| `FORWARD` | Đi **QUA** máy này (khi máy đóng vai trò router) |

| Target | Làm gì |
|---|---|
| `ACCEPT` | Cho qua |
| `DROP` | 🛑 Chặn, **im lặng** không phản hồi gì (bên gửi chờ rồi timeout) |
| `REJECT` | Chặn, **có gửi phản hồi từ chối** (bên gửi biết ngay là bị từ chối) |

⚠️ **`DROP` vs `REJECT` — chọn sai ảnh hưởng tới TRẢI NGHIỆM debug của người khác:** `DROP` khiến máy nhìn như **không tồn tại** (kẻ dò quét mất nhiều thời gian hơn) — thường dùng cho bảo mật. `REJECT` phản hồi ngay `Connection refused` — thân thiện hơn cho **debug nội bộ** (đồng nghiệp biết ngay là bị chặn, không phải mất mạng).

| Lệnh & cờ | Viết tắt của | Làm gì |
|---|---|---|
| `-L` | **L**ist | Liệt kê rule (chỉ đọc) |
| `-n` | **n**umeric | In **số** IP/port, không tra DNS ngược (nhanh hơn nhiều) |
| `-v` | **v**erbose | Kèm **counter**: bao nhiêu gói/byte đã khớp rule này |
| `--line-numbers` | | ⭐ Kèm **số thứ tự** — cần để `-D` chính xác |
| `-A` | **A**ppend | Thêm rule vào **CUỐI** chain |
| `-I` | **I**nsert | Chèn rule vào **ĐẦU** chain (hoặc vị trí chỉ định) |
| `-D` | **D**elete | Xoá rule |
| `-F` | **F**lush | 🔴 Xoá **TOÀN BỘ** rule trong chain |
| `-p` | **p**rotocol | `tcp`/`udp`/`icmp` |
| `--dport` | destination port | Cổng **đích** |
| `-s` | **s**ource | Địa chỉ **nguồn** |
| `-j` | **j**ump | Nhảy tới **target** (ACCEPT/DROP/REJECT) |

⭐ **`iptables -L -n -v` — bóc từng mảnh:**

```bash
iptables -L -n -v --line-numbers
#         │  │  │  └─ đánh số THỨ TỰ mỗi rule -> cần để xoá đúng rule
#         │  │  └──── verbose: hiện SỐ GÓI TIN đã khớp mỗi rule (0 = rule này CHƯA từng khớp gói nào)
#         │  └─────── numeric: KHÔNG tra DNS ngược cho IP -> NHANH HƠN RẤT NHIỀU
#         └────────── liệt kê (chỉ đọc, an toàn)
```

⭐ **`-A` vs `-I` — khác nhau vị trí, và vì sao vị trí QUYẾT ĐỊNH kết quả:**

**iptables xử lý rule THEO THỨ TỰ TỪ TRÊN XUỐNG, dừng lại ở rule ĐẦU TIÊN khớp** (không tiếp tục xét các rule bên dưới). Đây là điểm khác biệt căn bản, và là nguồn gốc của rất nhiều rule "thêm rồi mà không có tác dụng":

```bash
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
#         └─ A = Append: thêm vào CUỐI
#            -> nếu phía TRÊN đã có rule "DROP tất cả" thì rule ACCEPT này KHÔNG BAO GIỜ được xét tới

iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT
#         │        └─ chèn vào ĐÚNG VỊ TRÍ 1 = ĐẦU chain -> LUÔN được xét TRƯỚC MỌI rule khác
#         └──────── I = Insert
```

⇒ **Quy tắc thực hành**: rule `ACCEPT` cho port quan trọng (SSH) nên **`-I` lên đầu**; rule tổng quát (`DROP` cho phần còn lại) đặt **cuối cùng**.

🛑🛑 **`iptables -D INPUT <line-number>` — vì sao là lệnh nguy hiểm cần hiểu kỹ:**

`-D` xoá theo **SỐ THỨ TỰ**, không phải theo nội dung rule. Nếu bạn xem `--line-numbers` **rồi có ai đó (script khác, cron) thêm/xoá rule TRONG LÚC** bạn đang đọc ⇒ số thứ tự **DỊCH CHUYỂN** ⇒ `-D 5` xoá **nhầm rule khác** — và lệnh **chạy thành công, không báo lỗi gì**, im lặng xoá sai.

⇒ **Cách an toàn hơn — xoá theo NỘI DUNG rule thay vì số:**

```bash
iptables -D INPUT -p tcp --dport 3306 -j DROP
#         └─ ghi ĐÚNG cú pháp rule cần xoá (giống hệt lúc tạo) -> iptables tự tìm và xoá đúng rule đó
#            AN TOÀN hơn nhiều so với xoá theo số, vì không phụ thuộc thứ tự hiện tại
```

⚠️ **Vì sao lỗi này "im lặng"?** `iptables -D <chain> <n>` **không kiểm tra** rule tại vị trí đó có đúng ý định của bạn hay không — nó **chỉ xoá đúng cái đang nằm ở vị trí n tại thời điểm chạy**. Không có bước xác nhận "bạn có chắc muốn xoá rule NÀY không".

🔴 **`iptables -F` — xoá sạch có thể khiến bạn MẤT SSH ngay lập tức:**

```bash
iptables -F
#         └─ 🛑 xoá TOÀN BỘ rule trong chain -> nếu default policy đang là DROP,
#            máy LẬP TỨC từ chối MỌI kết nối, kể cả SSH đang mở -> TỰ KHOÁ MÌNH RA NGOÀI
```

⇒ Trước khi `-F`, luôn kiểm tra `iptables -L INPUT | head -1` xem **default policy** là gì (ACCEPT hay DROP) — quyết định hậu quả có nghiêm trọng hay không.

**Backup & Restore — làm trước khi thay đổi lớn:**

```bash
iptables-save > rules-backup-$(date +%F).v4    # lưu TOÀN BỘ rule ra file
iptables-restore < rules-backup-2026-08-08.v4  # khôi phục nguyên trạng khi lỡ tay
```

⚠️ **Rule iptables KHÔNG tự động sống sót qua reboot** trừ khi có cấu hình lưu riêng (`iptables-persistent` trên Debian/Ubuntu, hoặc script trong `/etc/rc.local`). Chỉnh xong nhớ lưu lại bằng gói phù hợp cho bản phân phối, không chỉ tin vào rule đang chạy trong RAM.

</details>

### firewalld (RHEL/CentOS)
```bash
firewall-cmd --state                   # Trạng thái
firewall-cmd --list-all                # Liệt kê rule
firewall-cmd --add-port=80/tcp --permanent   # Mở port (lưu vĩnh viễn)
firewall-cmd --reload                  # Áp dụng thay đổi
```

<details>
<summary><b>Bấm xem: giải nghĩa firewalld — zone và runtime vs permanent</b></summary>

⭐ **Tiền đề — khái niệm "zone" là điểm khác biệt lớn nhất so với `ufw`/`iptables` thô:**

`firewalld` nhóm các interface mạng vào **zone** (vùng tin cậy khác nhau — `public`, `internal`, `trusted`...), mỗi zone có bộ rule **riêng**. Cùng một port có thể **mở ở zone này, đóng ở zone khác** — tuỳ interface mạng đó nằm ở zone nào.

```bash
firewall-cmd --get-active-zones
#            └─ interface mạng nào đang thuộc zone nào — BƯỚC ĐẦU TIÊN cần xem trước khi mở port
firewall-cmd --get-default-zone
#            └─ zone MẶC ĐỊNH áp dụng cho interface chưa gán rõ
```

| Lệnh | Làm gì |
|---|---|
| `--state` | Firewalld có đang chạy không |
| `--list-all` | Rule của **zone mặc định** |
| `--list-all --zone=<z>` | Rule của **một zone cụ thể** |
| `--add-port` | Mở port |
| `--add-service` | Mở theo **tên dịch vụ đã định nghĩa sẵn** (`http`, `https`, `ssh`...) |
| `--permanent` | ⭐ Ghi **vĩnh viễn**, nhưng KHÔNG có hiệu lực ngay |
| `--reload` | Áp dụng các thay đổi `--permanent` vào runtime |

🛑🛑 **Bẫy lớn nhất của firewalld: `--permanent` KHÔNG có hiệu lực ngay — thiếu `--reload` là tưởng đã mở mà chưa mở:**

```bash
firewall-cmd --add-port=8080/tcp --permanent
#                                 └─ ⭐ chỉ GHI VÀO CẤU HÌNH TRÊN ĐĨA,
#                                    RUNTIME (đang chạy trong RAM) CHƯA ĐỔI GÌ CẢ
#                                    -> port 8080 VẪN CHƯA MỞ ngay lúc này
firewall-cmd --reload
#            └─ ⭐ BẮT BUỘC — nạp lại cấu hình permanent VÀO runtime, port MỚI THỰC SỰ mở
```

⇒ Đây là nguyên nhân của than phiền kinh điển: *"tôi mở port rồi mà vẫn không connect được"* — chỉ vì thiếu bước `--reload`.

⭐ **Hai tầng cấu hình — hiểu rõ để không rối:**

| Tầng | Hiệu lực | Sống qua reboot? |
|---|---|---|
| **Runtime** (mặc định, không có `--permanent`) | ⭐ Ngay lập tức | ❌ **Mất khi restart firewalld/reboot** |
| **Permanent** (có `--permanent`) | Chỉ sau khi `--reload` | ✅ **Giữ được** |

⇒ **Cách làm việc đúng đắn — test trước, ghi vĩnh viễn sau khi chắc chắn:**

```bash
# BƯỚC 1: mở tạm ở RUNTIME để test ngay, KHÔNG ảnh hưởng cấu hình lâu dài
firewall-cmd --add-port=8080/tcp
#            └─ KHÔNG có --permanent -> có hiệu lực NGAY, nhưng MẤT nếu reload/reboot

# BƯỚC 2: test xong, thấy đúng ý -> mới ghi permanent
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload
```

⚠️ **`--reload` có làm rớt kết nối đang mở không?** Không — `firewalld` reload **giữ nguyên state của các kết nối đã được cho phép trước đó** (connection tracking), không giống việc restart hẳn service. Nhưng vẫn nên làm ở khung giờ ít rủi ro với hệ thống quan trọng.

**Mở theo tên dịch vụ — tiện hơn nhớ số port:**

```bash
firewall-cmd --list-services                    # dịch vụ đã cho phép ở zone hiện tại
firewall-cmd --add-service=https --permanent     # tương đương add-port=443/tcp nhưng DỄ ĐỌC hơn
firewall-cmd --get-services | tr ' ' '\n' | grep -i post
#                              └─ đổi khoảng trắng thành xuống dòng -> grep dễ tìm dịch vụ theo tên
```

**Kiểm tra một port cụ thể đã mở hay chưa — nhanh hơn đọc cả `--list-all`:**

```bash
firewall-cmd --query-port=8080/tcp
#            └─ trả lời "yes"/"no" trực tiếp, và MÃ THOÁT tương ứng -> dùng được trong script if
```

**Rich rule — khi cần giới hạn theo IP nguồn cụ thể (tương đương `ufw allow from`):**

```bash
firewall-cmd --add-rich-rule='rule family="ipv4" source address="10.0.0.0/24" port port="5432" protocol="tcp" accept' --permanent
firewall-cmd --reload
```

⚠️ Cú pháp rich rule **dài và dễ gõ sai** — luôn kiểm tra lại bằng `--list-rich-rules` sau khi thêm, và cân nhắc dùng `--permanent` ngay từ đầu với rule phức tạp này (đỡ phải gõ lại hai lần).

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa iostat/mpstat/sar/pidstat — tìm nút thắt hiệu năng</b></summary>

**Tiền đề — vì sao cần bộ công cụ này khi `top`/`htop` không đủ?** `top` cho biết **tổng quan** CPU/RAM, nhưng khi nghi ngờ **disk I/O** hay **một core cụ thể** là nút thắt, cần công cụ **đi sâu vào từng thành phần**. Bộ `sysstat` (chứa `iostat`, `mpstat`, `sar`, `pidstat`) là bộ công cụ chuẩn cho việc này — thường phải cài thêm (`apt/yum install sysstat`).

| Lệnh | Nhìn vào | Trả lời |
|---|---|---|
| `iostat -x 1` | **Đĩa** | Disk có phải nút thắt không? |
| `mpstat -P ALL 1` | **Từng CPU core** | Có core nào bị dồn tải lệch không? |
| `sar -u` / `sar -r` | CPU / RAM **lịch sử** | Sự cố xảy ra lúc nào, kéo dài bao lâu? |
| `pidstat 1` | **Từng tiến trình** | Tiến trình NÀO gây ra tải này? |

⭐ **`iostat -x 1` — bóc cột quan trọng nhất: `%util` và `await`:**

```bash
iostat -x 1
#         │  └─ lặp lại MỖI 1 GIÂY (dòng đầu tiên là TRUNG BÌNH TỪ LÚC BOOT — thường BỎ QUA, đọc dòng thứ 2 trở đi)
#         └──── x = extended: thêm các cột chi tiết (await, %util...)
```

```
Device  r/s   w/s   rMB/s  wMB/s  await  %util
sda     2.0   150   0.1    18.5   45.2   98.5
#                          └─ chờ TRUNG BÌNH (ms) cho MỖI thao tác I/O
#                                    └─ ⭐ % THỜI GIAN đĩa đang BẬN xử lý -> 98.5% = GẦN NHƯ BÃO HOÀ
```

| Cột | Ngưỡng đáng lo | Ý nghĩa |
|---|---|---|
| `%util` | > 90% liên tục | Đĩa **gần như luôn bận** — nút thắt thật sự |
| `await` | Vài chục ms trở lên (tuỳ loại đĩa) | Thời gian **chờ** mỗi thao tác — cao = ứng dụng cảm nhận I/O chậm |

⚠️ **`%util` cao KHÔNG TỰ ĐỘNG nghĩa là "cần đĩa nhanh hơn".** Với đĩa SSD/NVMe hỗ trợ **nhiều thao tác song song**, `%util = 100%` vẫn có thể còn dư sức — phải nhìn kèm `await`: `await` **thấp** dù `%util` cao ⇒ đĩa **bận nhưng vẫn đáp ứng nhanh**, chưa hẳn là vấn đề.

⭐ **`mpstat -P ALL 1` — phát hiện "một core gánh hết", điều `top` không thấy rõ:**

```bash
mpstat -P ALL 1
#         │    └─ ALL = TỪNG core RIÊNG, không gộp trung bình
#         └────── P = processor
```

```
CPU  %usr  %sys  %iowait  %idle
 0   95.0   3.0   0.5      1.5   <- core 0 GẦN NHƯ KIỆT SỨC
 1    5.0   1.0   0.2     93.8   <- core 1 hầu như RẢNH
```

⇒ Load trung bình toàn máy có thể **trông ổn** (vì tính trung bình các core), nhưng nếu ứng dụng **đơn luồng** (chỉ dùng được 1 core), một core bị dồn tải 95% trong khi các core khác rảnh là **nút thắt thật sự** mà `top` gộp chung khó thấy ngay.

⚠️ **`%iowait` cao** (không phải `%usr` hay `%sys`) nghĩa là CPU **đang RẢNH nhưng phải CHỜ dữ liệu từ đĩa** — đây là dấu hiệu **disk là nút thắt**, không phải CPU. Kết luận sai (tưởng CPU yếu, nâng cấp CPU) sẽ **không giải quyết được gì**.

⭐ **`sar` — xem LỊCH SỬ, khác `top`/`iostat` chỉ xem hiện tại:**

```bash
sar -u 1 5      # u = CPU, lấy 5 mẫu, mỗi mẫu cách nhau 1 giây (giống chạy tạm thời)
sar -u -f /var/log/sysstat/sa08     # ⭐ đọc LOG LỊCH SỬ đã có sẵn của ngày mùng 8
#           └─ sar mặc định TỰ GHI log định kỳ (qua cron của gói sysstat) -> tra lại được SỰ CỐ ĐÃ QUA
```

⇒ Đây là điểm khác biệt cốt lõi với `top`/`htop`/`iostat`: chúng chỉ cho biết **BÂY GIỜ**; `sar` (nếu đã bật ghi log định kỳ từ trước) cho biết **LÚC NÃY, ĐÊM QUA** — sự cố xảy ra rồi mới đi tra thì chỉ `sar` giúp được, các lệnh kia đã "muộn".

**`pidstat` — tìm ĐÚNG tiến trình gây tải, không chỉ biết "máy đang bận":**

```bash
pidstat 1                       # CPU của TỪNG tiến trình, mỗi giây
pidstat -d 1                    # d = disk: I/O của TỪNG tiến trình
pidstat -r 1                    # r = RAM: bộ nhớ của từng tiến trình
```

💡 Kết hợp: `iostat` nói "đĩa đang bận 98%", nhưng **không nói ai gây ra** ⇒ `pidstat -d` mới trả lời được "tiến trình PID nào đang ghi/đọc đĩa nhiều nhất".

⚠️ Toàn bộ nhóm lệnh `sysstat` **không có sẵn mặc định** trên nhiều distro tối giản (container, minimal install) — phải `apt install sysstat` hoặc `yum install sysstat`, và trên **VDI air-gapped** thì cần tải gói offline trước.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa strace/ltrace/perf — công cụ cuối cùng khi mọi thứ khác bó tay</b></summary>

⭐ **Tiền đề — khi nào cần "system call" thay vì log ứng dụng?** Khi ứng dụng **treo mà không log gì**, hoặc **chậm bất thường mà không rõ đang chờ cái gì** — nghĩa là vấn đề nằm **ở tầng dưới code ứng dụng**, trong cách nó tương tác với kernel (mở file, đọc mạng, cấp phát bộ nhớ). `strace` cho thấy **chính xác** ứng dụng đang gọi hàm hệ thống nào, chờ cái gì.

| Lệnh & cờ | Viết tắt của | Làm gì |
|---|---|---|
| `strace -p <PID>` | trace + **p**rocess | Gắn vào tiến trình **đang chạy**, xem mọi syscall realtime |
| `strace -f` | **f**ollow | Bám theo cả **tiến trình con** được fork ra |
| `strace -e trace=<nhóm>` | **e**xpression | Chỉ lọc **một nhóm** syscall (network, file, ...) |
| `strace -c` | **c**ount | ⭐ Thống kê: syscall nào được gọi **nhiều/lâu nhất** |
| `ltrace` | library trace | Như strace nhưng theo dõi **lời gọi thư viện** (mức cao hơn kernel) |
| `lsof -p <PID>` | list open files | File/socket **tiến trình đang giữ mở** |
| `perf top` | | Hàm nào ngốn CPU nhất, **realtime** |

🛑 **`strace` LÀM CHẬM tiến trình đáng kể** — mỗi syscall bị chặn lại để ghi log rồi mới cho chạy tiếp. Trên production, chỉ dùng **trong thời gian ngắn**, và cân nhắc kỹ trước khi gắn vào tiến trình đang phục vụ traffic thật.

⭐ **`strace -p <PID>` — gắn vào tiến trình ĐANG chạy để xem nó bị TREO ở đâu:**

```bash
strace -p 12345
#          └─ PID của tiến trình đang treo/chậm (lấy từ `ps aux` hoặc `pgrep`)
# Ctrl+C để thoát, KHÔNG làm chết tiến trình đang theo dõi
```

Đọc kết quả — dòng CUỐI CÙNG (chỗ nó đang treo) là quan trọng nhất:

```
read(5, ...                    <- ĐANG TREO ở đây: chờ ĐỌC dữ liệu từ file descriptor số 5
connect(6, {sa_family=AF_INET, sin_port=htons(5432)}, ...   <- ĐANG chờ KẾT NỐI TỚI port 5432 (Postgres!)
```

⇒ Dòng cuối chưa hoàn thành (không có kết quả trả về ở cuối dòng) chính là **chỗ tiến trình đang bị nghẽn** — thường trả lời ngay câu hỏi "app treo vì chờ database hay chờ gì khác".

⭐ **`-e trace=network` — lọc CHỈ syscall mạng, bớt nhiễu:**

```bash
strace -f -e trace=network -p 12345
#       │  └────────────────┘
#       │  chỉ các syscall LIÊN QUAN MẠNG: connect, sendto, recvfrom...
#       └─ bám cả TIẾN TRÌNH CON (nhiều app đa luồng/đa tiến trình fork con để xử lý request)
```

⚠️ **Thiếu `-f` với ứng dụng đa tiến trình** (Nginx, PostgreSQL, nhiều app Python dùng worker) ⇒ `strace` chỉ theo dõi **tiến trình cha**, trong khi việc thật sự xảy ra ở **tiến trình con** ⇒ trace ra **rỗng**, tưởng nhầm là "không có gì xảy ra".

⭐ **`strace -c <cmd>` — thống kê thay vì xem từng dòng, hợp để TÌM nút thắt tổng quan:**

```bash
strace -c curl -s https://api.company.vn/health > /dev/null
```

```
% time     seconds  usecs/call     calls    syscall
------ ----------- ----------- --------- ------------
 65.2    0.021453        2145        10  connect      <- ⭐ TỐN THỜI GIAN NHẤT: connect (khớp với nghi ngờ chậm mạng/DNS)
 20.1    0.006612          82        80  read
```

⇒ Cột `% time` cho biết ngay **loại syscall nào ăn thời gian nhiều nhất** — không cần đọc từng dòng log dài dằng dặc.

**`ltrace` — khi vấn đề nằm ở THƯ VIỆN, không phải kernel:**

```bash
ltrace -p 12345
#           └─ theo dõi lời gọi HÀM THƯ VIỆN (malloc, strcpy, các hàm của libc/openssl...)
#              KHÁC strace: strace theo dõi gọi KERNEL, ltrace theo dõi gọi THƯ VIỆN
```

⚠️ `ltrace` **chậm hơn `strace` nhiều lần** và **không hoạt động tốt với binary tối ưu tĩnh** (static linking) — vì nó cần "móc" vào lời gọi hàm động (dynamic linking). Với ứng dụng Go/Rust build static, `ltrace` thường **không thấy gì**.

**`lsof -p <PID>` — tiến trình đang mở những gì:**

```bash
lsof -p 12345
# COMMAND  PID  USER  FD    TYPE  DEVICE  SIZE/OFF   NODE  NAME
# myapp   12345 app   5u    IPv4  123456  0t0        TCP   10.0.0.5:5432 (ESTABLISHED)
#                └─ ⭐ FD (file descriptor) SỐ 5 chính là cột trong strace "read(5, ..." ở trên
#                     -> NỐI được strace và lsof lại với nhau: FD 5 LÀ kết nối này
```

⇒ Ghép `strace` (thấy FD nào đang treo) với `lsof` (biết FD đó **là cái gì**) là công thức chẩn đoán mạnh: *"tiến trình đang treo ở `read(5,...)`, và FD 5 chính là kết nối TCP tới Postgres 10.0.0.5:5432"* — kết luận chắc chắn, không phải đoán.

**`/proc/<PID>/limits` — kiểm tra giới hạn hệ thống, nguyên nhân hay bị bỏ sót:**

```bash
cat /proc/<PID>/limits | grep -i "open files"
#                                └─ ⭐ giới hạn SỐ FILE DESCRIPTOR tối đa của tiến trình này
ulimit -a          # giới hạn của SHELL HIỆN TẠI (tiến trình mới tạo từ đây kế thừa các giá trị này)
```

⚠️ App báo lỗi `too many open files` mà code không có gì sai ⇒ thường là **chạm giới hạn `ulimit -n`** (mặc định nhiều hệ thống chỉ **1024**), không phải app leak file descriptor thật. Kiểm tra bằng lệnh trên trước khi đi soát code.

**`perf top` — hàm nào ngốn CPU, ở mức sâu hơn `top` (tận native function):**

```bash
perf top
#        └─ cần quyền root + kernel hỗ trợ perf_events
#           hiện danh sách HÀM (không phải tiến trình) đang chiếm CPU nhiều nhất, cập nhật realtime
```

⚠️ `perf` **cần symbol debug** để hiện tên hàm dễ đọc — thiếu debug symbol, kết quả chỉ là **địa chỉ bộ nhớ** khó diễn giải. Đây là công cụ **sâu nhất** trong bộ này, thường chỉ cần tới khi `strace`/`pidstat` đã loại trừ hết các khả năng dễ hơn.

</details>

### Kiểm tra kết nối & OOM
```bash
ss -s                                  # Tổng hợp số kết nối theo trạng thái
ss -tan state established | wc -l      # Đếm kết nối ESTABLISHED
ss -tan state time-wait | wc -l        # Đếm TIME_WAIT (nhiều = vấn đề)
dmesg -T | grep -i "killed process"    # Kiểm tra process bị OOM killer giết
grep -i oom /var/log/syslog            # Log OOM (Ubuntu)
cat /proc/loadavg                      # Load average
```

<details>
<summary><b>Bấm xem: giải nghĩa ss -s, TIME_WAIT, và cách đọc dmesg OOM</b></summary>

| Lệnh & cờ | Viết tắt của | Làm gì |
|---|---|---|
| `ss -s` | **s**ummary | Tổng hợp **số lượng** kết nối theo từng trạng thái |
| `ss -tan` | tcp + **a**ll + numeric | Liệt kê **từng** kết nối TCP (không chỉ tổng hợp) |
| `state established` | | Lọc theo **trạng thái** kết nối |
| `dmesg -T` | | Log kernel, **T**imestamp dạng ngày giờ thật |
| `grep -i oom` | | Tìm dấu vết OOM killer |

**Bảng trạng thái TCP cần biết để đọc `ss`:**

| Trạng thái | Nghĩa |
|---|---|
| `ESTABLISHED` | Kết nối **đang hoạt động** bình thường |
| `TIME_WAIT` | ⭐ Kết nối **đã đóng**, hệ điều hành giữ lại ~60s để đảm bảo gói cuối không lạc |
| `CLOSE_WAIT` | ⚠️ Phía kia đã đóng, **nhưng ứng dụng của MÌNH chưa gọi `close()`** |
| `SYN_SENT` | Đang **chờ** bắt tay (đã gửi SYN, chưa nhận SYN-ACK) |

```bash
ss -s
# TCP:   1520 (estab 340, closed 1050, orphaned 0, timewait 1040)
#              │           │                                └─ ⭐ SỐ LƯỢNG TIME_WAIT
#              └─ đang hoạt động thật sự
```

⭐ **`TIME_WAIT` nhiều — bình thường hay bất thường?** Phải nhìn theo NGỮ CẢNH:

| Số lượng | Bình thường khi | Đáng lo khi |
|---|---|---|
| Vài nghìn, ổn định | Server có **traffic ngắn hạn cao** (web server bình thường) | — |
| Tăng KHÔNG NGỪNG, gần chạm giới hạn port (~28000-64000) | — | ⚠️ **Sắp cạn port khả dụng**, kết nối mới sẽ bị từ chối |

```bash
ss -tan state time-wait | wc -l           # đếm CHÍNH XÁC số lượng
cat /proc/sys/net/ipv4/ip_local_port_range   # dải PORT có thể dùng để mở kết nối MỚI
```

⇒ `TIME_WAIT` tăng liên tục thường là dấu hiệu ứng dụng đang **mở kết nối mới liên tục thay vì tái sử dụng** (thiếu connection pooling) — không phải lỗi hệ điều hành.

🛑 **`CLOSE_WAIT` nhiều — dấu hiệu RÕ RÀNG NHẤT của resource leak trong CHÍNH ứng dụng của bạn:**

```bash
ss -tan state close-wait | wc -l
```

⇒ Trạng thái này nghĩa là: **phía đối tác đã đóng kết nối từ lâu**, nhưng **code ứng dụng của bạn quên gọi `close()`/`disconnect()`** ⇒ tài nguyên (file descriptor, bộ nhớ) **bị giữ lại vĩnh viễn** cho tới khi tiến trình hết file descriptor hoặc bị restart. Đây là bug ở **code**, không sửa được bằng cấu hình hệ thống — phải tìm và fix đoạn code không đóng connection/response body đúng cách.

⭐ **`dmesg -T` — nơi DUY NHẤT ghi lại việc bị OOM killer giết** (đã nhắc sơ ở mục Log hệ thống, đào sâu thêm ở đây):

```bash
dmesg -T | grep -i "killed process"
```

```
[Mon Aug  8 14:32:01 2026] Out of memory: Killed process 12345 (java) total-vm:8500000kB, anon-rss:3900000kB
#                                                          │              │                └─ RAM THẬT đang dùng lúc bị giết
#                                                          │              └─ tổng bộ nhớ ẢO đã cấp phát (thường LỚN HƠN RAM thật rất nhiều — bình thường)
#                                                          └─ ⭐ TÊN + PID tiến trình bị giết -> đây chính là thủ phạm ăn RAM
```

⭐ **`total-vm` lớn KHÔNG đáng lo — `anon-rss` mới là con số thật.** Nhiều ứng dụng (đặc biệt JVM) xin cấp **bộ nhớ ảo** rất lớn ngay từ đầu nhưng **chưa dùng hết** — kernel chỉ cấp **trang vật lý thật** khi tiến trình thực sự chạm tới (cơ chế "lazy allocation"). `anon-rss` mới là **RAM VẬT LÝ thực sự đang chiếm** — con số quyết định máy có OOM hay không.

**Cách kernel CHỌN nạn nhân khi hết RAM — không phải ngẫu nhiên:**

```bash
cat /proc/<PID>/oom_score          # điểm "đáng bị giết" của tiến trình — CÀNG CAO càng dễ bị chọn
cat /proc/<PID>/oom_score_adj      # ⭐ tự CHỈNH điểm: -1000 = MIỄN NHIỄM tuyệt đối với OOM killer
```

⇒ Kernel tính điểm dựa trên **RAM đang chiếm** (nhiều RAM = điểm cao = dễ bị giết trước) và **có ưu tiên gì đặc biệt không**. Với tiến trình **tuyệt đối không được chết** (ví dụ tiến trình quản trị chính), có thể đặt `oom_score_adj=-1000` để loại khỏi danh sách ứng viên — nhưng dùng cẩn trọng, vì nếu tiến trình đó **chính là** thứ đang ăn hết RAM thì việc miễn nhiễm nó sẽ đẩy OOM killer đi **giết tiến trình KHÁC** thay thế, có thể còn tệ hơn.

⚠️ **`grep -i oom /var/log/syslog`** (Ubuntu/Debian) là cách thay thế khi `dmesg` đã bị **xoay vòng mất** (buffer kernel giới hạn dung lượng, log cũ bị đẩy ra) — `/var/log/syslog` thường giữ log lâu hơn buffer của `dmesg`.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa argocd — Synced/Healthy và vì sao hai trạng thái độc lập nhau</b></summary>

⭐ **Tiền đề — ArgoCD giải bài toán gì?** Đây là công cụ **GitOps**: thay vì `kubectl apply` thủ công, bạn **commit YAML vào Git**, ArgoCD **tự động** phát hiện thay đổi và đồng bộ vào cluster. Git trở thành **nguồn sự thật duy nhất** (single source of truth) — muốn biết cluster đang chạy gì, chỉ cần đọc Git, không cần SSH vào cluster.

| Lệnh | Làm gì |
|---|---|
| `argocd login <server>` | Đăng nhập vào ArgoCD server (không phải cluster K8s) |
| `argocd app list` | Danh sách mọi Application đang quản lý |
| `argocd app get <app>` | Chi tiết: trạng thái, resource con, lịch sử |
| `argocd app sync <app>` | ⭐ **Ép đồng bộ** cluster theo đúng Git ngay bây giờ |
| `argocd app diff <app>` | Khác biệt **Git vs Cluster** — trước khi sync |
| `argocd app rollback <app> <rev>` | Quay lại revision cũ |
| `argocd app set --sync-policy automated` | Bật **tự động sync** khi Git đổi (không cần gõ `sync` tay) |

⭐⭐ **Synced/OutOfSync và Healthy/Degraded — HAI TRỤC ĐỘC LẬP, đây là điều hay bị hiểu lầm nhất:**

Mới nhìn, "app khoẻ" tưởng chỉ có một trạng thái. Thực ra ArgoCD theo dõi **HAI câu hỏi khác nhau, không phụ thuộc nhau**:

| Trục | Câu hỏi | Giá trị |
|---|---|---|
| **Sync Status** | *"Cluster có GIỐNG Git không?"* | `Synced` / `OutOfSync` |
| **Health Status** | *"Resource đang chạy có TỐT không?"* | `Healthy` / `Degraded` / `Progressing` / `Missing` |

⇒ Bốn tổ hợp có thể xảy ra, và **mỗi tổ hợp cần một hành động khác nhau**:

| Sync | Health | Ý nghĩa | Cần làm gì |
|---|---|---|---|
| `Synced` | `Healthy` | ✅ Mọi thứ ổn | Không cần làm gì |
| `Synced` | `Degraded` | 🛑 Cluster **ĐÚNG** như Git yêu cầu, nhưng bản thân config đó **có vấn đề** (image sai, thiếu resource) | Sửa **code trong Git**, không phải sync lại |
| `OutOfSync` | `Healthy` | Ai đó sửa tay trên cluster (hoặc Git vừa đổi mà chưa sync), nhưng **hiện tại vẫn chạy tốt** | Sync để đồng bộ lại, hoặc điều tra ai sửa tay |
| `OutOfSync` | `Degraded` | Cả hai vấn đề cùng lúc | Ưu tiên xem **diff** trước khi sync mù quáng |

🛑 **Bẫy hay gặp nhất: thấy `OutOfSync` liền bấm `sync` ngay mà KHÔNG xem `diff` trước.** Nếu ai đó vừa sửa tay trên cluster để **chữa cháy khẩn cấp** (ví dụ tăng replicas gấp vì đang quá tải), `sync` sẽ **ghi đè** về đúng Git — có thể **xoá mất bản vá khẩn cấp đó**, đưa hệ thống về lại trạng thái gây ra sự cố ban đầu.

```bash
argocd app diff myapp
#               └─ ⭐ LUÔN chạy lệnh này TRƯỚC "sync" khi thấy OutOfSync bất ngờ
#                  để hiểu RÕ đang có khác biệt gì, không sync mù quáng
```

⭐ **`--prune` — vì sao mặc định sync KHÔNG xoá resource thừa:**

```bash
argocd app sync myapp                # chỉ THÊM/SỬA theo Git, KHÔNG xoá resource không còn trong Git
argocd app sync myapp --prune        # 🛑 CỘNG THÊM: XOÁ resource nào KHÔNG CÒN khai báo trong Git
```

⇒ Đây là thiết kế **an toàn theo mặc định**: xoá một Deployment khỏi file YAML rồi commit, ArgoCD sẽ **không tự xoá** Deployment đó khỏi cluster trừ khi có `--prune`. Bảo vệ khỏi việc lỡ tay xoá file YAML thì kéo theo mất luôn resource thật.

**`argocd app wait` — dùng trong CI/CD để chờ deploy XONG mới báo pipeline thành công:**

```bash
argocd app sync myapp
argocd app wait myapp --health --timeout 300
#               │      └───────┘  └─ chờ tối đa 300 giây rồi BỎ CUỘC (không treo vô hạn)
#               └─ chờ tới khi Health = Healthy (không chỉ chờ Sync xong)
```

⚠️ **Chờ `--health`, không chỉ chờ sync xong** — vì `sync` chỉ nghĩa là "đã GỬI lệnh apply lên K8s", **không đảm bảo** pod mới đã thực sự Ready. Pipeline CI nếu chỉ đợi `sync` xong đã coi là thành công, có thể báo "deploy OK" trong khi pod đang `CrashLoopBackOff`.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa CRD của ArgoCD và FluxCD — dùng khi không có argocd CLI</b></summary>

**Tiền đề — ArgoCD Application là CRD, nghĩa là gì?** ArgoCD **mở rộng** Kubernetes bằng một loại resource mới tên `Application` (CRD = Custom Resource Definition — "định nghĩa loại resource tuỳ chỉnh"). Vì nó **là** một resource K8s bình thường, mọi lệnh `kubectl` quen thuộc đều dùng được — hữu ích khi **không cài `argocd` CLI riêng** hoặc chỉ có quyền `kubectl`.

```bash
kubectl get applications -n argocd
#                          └─ ⭐ CRD của ArgoCD sống trong namespace "argocd" theo mặc định
#                             (khác các app THẬT của bạn — chúng nằm ở namespace RIÊNG như "ai-hub")

kubectl get app <app> -n argocd -o yaml
#            └─ "app" là TÊN TẮT (shortname) của "applications" — cả hai đều gõ được
```

⭐ **Đọc trạng thái NGAY TỪ YAML — không cần CLI `argocd` cũng biết Synced/Healthy:**

```bash
kubectl get app myapp -n argocd -o jsonpath='{.status.sync.status} {.status.health.status}'
#                                              └────────┬────────┘ └─────────┬─────────┘
#                                              Sync Status (Synced/OutOfSync)  Health Status (Healthy/Degraded)
```

⇒ Đây chính là **hai trục độc lập** đã giải thích ở mục `argocd CLI` phía trên — trong YAML thô, chúng nằm ở `.status.sync.status` và `.status.health.status` riêng biệt, xác nhận lại đúng là hai khái niệm khác nhau chứ không phải một.

```bash
kubectl describe app myapp -n argocd
#                └─ ⭐ mục "Conditions" ở cuối = LÝ DO chi tiết khi sync/health có vấn đề
#                   (giống hệt tinh thần "luôn đọc Events" đã nói ở mục Debug pod của kubectl)

kubectl -n argocd get pods
#           └─ kiểm tra CHÍNH ArgoCD có khoẻ không (application-controller, repo-server, api-server...)
#              trước khi nghi ngờ app CỦA BẠN có vấn đề — đôi khi ArgoCD chính nó mới là thứ đang lỗi
```

⚠️ **Sửa trực tiếp CRD `Application` bằng `kubectl edit`** kỹ thuật vẫn chạy được, nhưng đi ngược tinh thần GitOps — cấu hình của Application (trỏ tới repo nào, path nào) **nên nằm trong Git**, không sửa tay qua `kubectl`. Việc `kubectl edit` CRD Application chỉ nên dùng để **debug/xem**, không nên dùng để **thay đổi lâu dài**.

**Lấy mật khẩu admin ban đầu — bước đầu tiên sau khi cài mới ArgoCD:**

```bash
argocd admin initial-password -n argocd
#            └─ đọc TỪ Secret mà ArgoCD tự sinh khi cài lần đầu
# (cách thay thế không cần argocd CLI, đọc thẳng qua kubectl:)
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d
```

⚠️ **Secret này chỉ tồn tại tới lần đầu tiên đổi mật khẩu** — sau khi đổi, ArgoCD tự xoá Secret đó. Không tìm thấy secret này ⇒ nghĩa là mật khẩu **đã được đổi rồi**, không phải lỗi.

**FluxCD — công cụ GitOps thay thế, cách tiếp cận khác ArgoCD:**

⚠️ Khác biệt triết lý: ArgoCD có **UI + Application CRD tập trung**, coi trọng khả năng quan sát trực quan. FluxCD **thuần CLI/CRD**, chia nhỏ thành nhiều loại resource (`GitRepository`, `Kustomization`, `HelmRelease`) — kết hợp linh hoạt hơn nhưng **không có UI mặc định** đi kèm.

```bash
flux get kustomizations
#        └─ tương đương "argocd app list" nhưng cho Flux — Kustomization là đơn vị đồng bộ chính

flux reconcile kustomization myapp
#    └─ ⭐ reconcile = ÉP đồng bộ NGAY, tương đương "argocd app sync"
#       (không có reconcile thì Flux tự đồng bộ theo CHU KỲ định sẵn, mặc định thường VÀI PHÚT một lần)

flux get sources git             # xem Git repository đang theo dõi, commit nào đang được dùng
flux logs                        # log của controller — nơi tra khi Flux không đồng bộ như mong đợi
```

⇒ Cả ArgoCD và FluxCD đều theo cùng một triết lý gốc GitOps: **Git là nguồn sự thật**, công cụ chỉ là **cầu nối kéo trạng thái Git vào cluster** — hiểu triết lý này thì chuyển đổi giữa hai công cụ không khó, dù cú pháp lệnh khác nhau.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa gh CLI — và khác biệt run/workflow</b></summary>

**Tiền đề — hai khái niệm phải phân biệt trước khi đọc lệnh:**

| | **Workflow** | **Run** |
|---|---|---|
| Là gì | File `.yml` định nghĩa **quy trình** (định nghĩa TĨNH) | **Một lần thực thi** cụ thể của workflow đó |
| Số lượng | Cố định theo số file trong `.github/workflows/` | Mỗi lần push/trigger tạo ra **một run mới** |
| Ví von | Công thức nấu ăn | Một lần nấu theo công thức đó |

| Lệnh | Làm gì |
|---|---|
| `gh workflow list` | Liệt kê **file workflow** (công thức) |
| `gh run list` | Liệt kê **các lần chạy** (đã nấu bao nhiêu lần) |
| `gh run view <id>` | Chi tiết một lần chạy |
| `gh run view <id> --log` | ⭐ Log **đầy đủ** mọi job |
| `gh run view <id> --log-failed` | Chỉ log của **job/step bị fail** — bỏ qua job thành công |
| `gh run watch <id>` | Bám realtime, tự cập nhật tới khi xong |
| `gh run rerun <id>` | Chạy lại **toàn bộ** |
| `gh run rerun <id> --failed` | ⭐ Chỉ chạy lại **job đã fail**, giữ nguyên job đã pass |

⭐ **`--log-failed` — vì sao đáng dùng hơn `--log` khi debug:**

```bash
gh run view 123456 --log-failed
#                   └─ CI có 10 job, 9 job pass, 1 job fail
#                      -> chỉ in log của ĐÚNG job fail đó, không phải cuộn qua 9 job kia trước
```

⇒ Với pipeline nhiều job/step, `--log` thường ra hàng nghìn dòng — `--log-failed` đi thẳng vào chỗ cần xem.

⭐ **`gh run rerun --failed` — tiết kiệm thời gian và tiền (phút CI):**

```bash
gh run rerun 123456 --failed
#                     └─ ⭐ chỉ chạy lại job ĐàFAIL, các job đã PASS giữ nguyên kết quả cũ
#                        (không cần build lại từ đầu nếu chỉ 1 trong 10 job bị lỗi mạng tạm thời)
```

⚠️ **`--failed` chỉ hợp lý khi lỗi có tính "tạm thời"** (timeout mạng, flaky test). Nếu lỗi do **code sai thật sự**, sửa code trước rồi mới rerun — rerun không sửa được code.

**Trigger workflow thủ công — cần khai báo `workflow_dispatch` trong file YAML trước:**

```bash
gh workflow run deploy.yml -f environment=staging -f version=1.2.3
#                          └─ -f: truyền INPUT cho workflow (phải khớp tên input định nghĩa trong YAML)
```

⚠️ Lệnh này **chỉ chạy được** nếu workflow YAML có khai báo trigger `on: workflow_dispatch:` — thiếu khai báo đó, `gh workflow run` sẽ báo lỗi không tìm thấy cách trigger thủ công.

**Kiểm tra CI của PR hiện tại — chạy ngay trong thư mục repo, không cần biết PR số mấy:**

```bash
gh pr checks
#        └─ ⭐ tự nhận diện PR gắn với BRANCH hiện tại đang checkout, in bảng trạng thái từng check
```

**`act` — chạy GitHub Actions ngay trên máy local, không cần push lên GitHub:**

```bash
act push                    # giả lập sự kiện "push" để test workflow cục bộ
act -j build                # chỉ chạy MỘT job tên "build"
```

⇒ Rất hữu ích để **lặp nhanh** khi sửa file `.yml` — không phải commit + push + chờ CI thật mỗi lần thử. ⚠️ `act` chạy bằng Docker container mô phỏng runner GitHub — **không giống 100%** môi trường thật (thiếu một số secret/context), nên vẫn cần chạy thật trên GitHub trước khi merge.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa gitlab-runner và glab CLI</b></summary>

**Tiền đề — GitLab CI khác GitHub Actions ở kiến trúc runner:** GitLab tách rõ **server** (nơi lưu file `.gitlab-ci.yml`, điều phối pipeline) và **runner** (máy thực thi job) — runner có thể tự host trên hạ tầng nội bộ công ty, phù hợp môi trường **VDI air-gapped** không muốn gửi code ra ngoài.

| Lệnh | Làm gì |
|---|---|
| `gitlab-runner verify` | Kiểm tra **runner** có kết nối được với GitLab server không |
| `gitlab-runner list` | Danh sách runner **đã đăng ký** trên máy này |
| `gitlab-runner exec docker <job>` | ⭐ Chạy **một job** ngay trên máy local, không cần push |
| `glab ci list` | Liệt kê pipeline (tương đương `gh run list`) |
| `glab ci view` | Xem pipeline hiện tại |
| `glab ci trace` | ⭐ Log **realtime** của job đang chạy |
| `glab ci retry <id>` | Chạy lại pipeline |

⭐ **`gitlab-runner exec docker <job>` — thử nghiệm local trước khi push, tương tự `act` của GitHub:**

```bash
gitlab-runner exec docker build-job
#                          └─ TÊN JOB, phải khớp CHÍNH XÁC tên định nghĩa trong .gitlab-ci.yml
```

⚠️ **`exec` KHÔNG mô phỏng 100%** — nó **không có** các biến CI mà GitLab server cấp tự động khi pipeline chạy thật (`CI_COMMIT_SHA`, `CI_PIPELINE_ID`...), và **không chạy được** các job phụ thuộc `needs`/`stage` liên kết nhau như pipeline thật. Chỉ hợp để test **nhanh cú pháp và logic của một job đơn lẻ**.

⭐ **`glab ci trace` — bám log THEO ĐÚNG job đang chạy, không phải toàn pipeline:**

```bash
glab ci trace                    # bám job hiện tại của pipeline MỚI NHẤT trên branch đang đứng
glab ci trace --job=<job-id>     # chỉ định rõ job nào nếu pipeline có nhiều job song song
```

**Kiểm tra cú pháp `.gitlab-ci.yml` TRƯỚC khi commit — tránh phải chờ push rồi mới biết sai:**

Cách nhanh nhất là dùng **CI Lint** trên giao diện web: *GitLab → CI/CD → Editor → tab Validate* — dán nội dung YAML vào, GitLab server tự kiểm tra cú pháp **và** logic (`needs`, `rules`, biến tham chiếu) mà không cần chạy pipeline thật.

⚠️ Cũng có API để lint từ dòng lệnh (không cần mở trình duyệt) — hữu ích khi làm việc trên VDI hạn chế:

```bash
curl -sS --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  --data-urlencode "content@.gitlab-ci.yml" \
  "https://gitlab.company.vn/api/v4/ci/lint" | jq '.valid, .errors'
#                                                    └─ true/false          └─ danh sách lỗi cụ thể nếu có
```

**Debug pipeline fail — quy trình giống ArgoCD/K8s: luôn xem chi tiết trước khi đoán:**

```bash
glab ci view --web              # mở trực tiếp trên trình duyệt để xem log đầy đủ, có màu, dễ đọc
glab ci status                  # trạng thái GỌN của pipeline mới nhất trên branch hiện tại
```

⚠️ **Biến CI/CD nhạy cảm (secret) khai báo trên UI GitLab (Settings → CI/CD → Variables)** — không nằm trong file `.gitlab-ci.yml` — nên **không thấy trong Git history**, đây là chỗ đúng để lưu token/password thay vì viết cứng vào YAML.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa Vault — sealed/unsealed và KV v1 vs v2</b></summary>

⭐ **Tiền đề — "sealed" là gì, và vì sao Vault sinh ra khái niệm này?**

Vault mã hoá **mọi thứ** trên đĩa bằng một **master key**. Khi Vault khởi động (hoặc restart), nó ở trạng thái **`sealed`** (bị khoá) — dữ liệu **nằm đó nhưng KHÔNG đọc được**, kể cả bởi chính Vault, cho tới khi có đủ **"khoá gộp"** (unseal keys) đưa vào để tái tạo master key trong RAM.

```bash
vault status
# Sealed          true
#                  └─ ⭐ true = Vault ĐANG KHOÁ, MỌI API đều từ chối, kể cả với token hợp lệ
```

⇒ Đây là lớp bảo vệ: dù kẻ tấn công **lấy trộm được cả ổ đĩa** chứa dữ liệu Vault, dữ liệu vẫn **vô nghĩa** nếu không có đủ số khoá unseal (thường **thiết kế cần 3/5 khoá**, chia cho nhiều người giữ riêng — không ai một mình mở khoá được).

```bash
export VAULT_ADDR='https://vault.company.vn:8200'
#      └─ ⭐ BẮT BUỘC set biến này trước MỌI lệnh vault khác trong phiên làm việc,
#         nếu không vault mặc định tìm ở localhost:8200 -> báo lỗi kết nối refused

vault status                        # kiểm tra sealed/unsealed TRƯỚC khi làm gì khác
vault operator unseal
#                └─ nhập MỘT khoá unseal (chạy lệnh này NHIỀU LẦN với khoá KHÁC NHAU,
#                   tới khi đủ số khoá tối thiểu theo cấu hình, ví dụ 3/5)
```

**Xác thực — nhiều cách, `vault login` chỉ là một trong số đó:**

```bash
vault login                    # hỏi nhập TOKEN trực tiếp (hoặc mở trình duyệt tuỳ auth method cấu hình)
vault login -method=userpass username=kiennv     # xác thực bằng user/pass
vault token lookup             # ⭐ xem token HIỆN TẠI: hết hạn khi nào, thuộc policy nào
```

⚠️ **`vault token lookup`** rất đáng chạy trước một thao tác quan trọng — token Vault thường có **TTL (thời gian sống) ngắn** (đôi khi chỉ vài giờ), hết hạn giữa chừng thao tác sẽ khiến lệnh **đột ngột bị từ chối**.

⭐ **KV v1 vs KV v2 — khác biệt cú pháp quan trọng, gõ nhầm là lỗi khó hiểu:**

| | KV **v1** | KV **v2** (⭐ mặc định hiện nay) |
|---|---|---|
| Có lưu lịch sử version? | ❌ Không | ✅ **Có** — sửa đè không mất bản cũ |
| Cú pháp đường dẫn khi dùng API thô | `secret/myapp` | `secret/data/myapp` (⚠️ có thêm `data/`) |
| Cú pháp khi dùng `vault kv` CLI | Giống nhau | Giống nhau — **CLI tự lo phần `data/`** |

```bash
vault kv get secret/myapp                 # đọc TOÀN BỘ secret tại đường dẫn này
vault kv get -field=password secret/myapp # ⭐ chỉ lấy ĐÚNG MỘT field, gọn cho script
vault kv put secret/myapp password=abc123 user=admin
#                                          └─ ghi NHIỀU field cùng lúc, cách nhau bằng khoảng trắng
vault kv list secret/                      # liệt kê các path con (không phải nội dung secret)
```

🛑 **`vault kv put` GHI ĐÈ TOÀN BỘ, không phải chỉ thêm field mới:**

```bash
# Secret hiện có: {password: "old", user: "admin"}
vault kv put secret/myapp password=new
# 🛑 Kết quả: {password: "new"} -- field "user" đã BIẾN MẤT vì put GHI ĐÈ CẢ OBJECT
```

⇒ Muốn **chỉ sửa một field, giữ nguyên các field khác**, dùng `vault kv patch` (chỉ có ở KV v2):

```bash
vault kv patch secret/myapp password=new
#              └─ ⭐ CHỈ đổi field "password", các field KHÁC được GIỮ NGUYÊN
```

**Xem lịch sử version (chỉ có ở KV v2) — tính năng chính khiến v2 được ưa dùng:**

```bash
vault kv get -version=2 secret/myapp        # đọc PHIÊN BẢN CŨ số 2 (không phải bản mới nhất)
vault kv metadata get secret/myapp          # xem TOÀN BỘ lịch sử version + thời gian tạo mỗi bản
vault kv destroy -versions=1 secret/myapp   # xoá VĨNH VIỄN một version cụ thể (khác undelete được)
```

⚠️ **`vault kv delete` (xoá thường) khác `vault kv destroy` (xoá vĩnh viễn):** `delete` chỉ **đánh dấu ẩn**, vẫn khôi phục được bằng `vault kv undelete`; `destroy` xoá **thật sự, không hoàn tác**. Nhầm hai lệnh này là mất dữ liệu không cứu được.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa Sealed Secrets và SOPS — vì sao cần chúng để commit secret vào Git</b></summary>

⭐ **Tiền đề — vì sao K8s Secret THƯỜNG (đã học ở mục Namespace & Config phía trên) KHÔNG commit được vào Git?**

Nhắc lại mảnh ghép quan trọng: Secret của K8s chỉ **base64**, **KHÔNG mã hoá thật** — ai đọc được file YAML là đọc được secret gốc ngay lập tức (`base64 -d`). Với triết lý **GitOps** (mọi thứ nằm trong Git), commit thẳng Secret dạng này vào Git ⇒ **toàn bộ password/token lộ ra** cho bất kỳ ai có quyền đọc repo, kể cả khi đã xoá commit sau đó (lịch sử Git vẫn giữ).

⇒ **Sealed Secrets** và **SOPS** giải quyết đúng bài toán này: **mã hoá thật sự** secret trước khi commit, chỉ **giải mã được bởi đúng nơi cần dùng** (cluster / người có khoá).

**Sealed Secrets — mã hoá bằng khoá RIÊNG của cluster đích, chỉ cluster đó giải mã được:**

```bash
kubeseal --format yaml < secret.yaml > sealed-secret.yaml
#         │                │            └─ file MÃ HOÁ — AN TOÀN để commit vào Git
#         │                └────────────── file Secret GỐC (thường, base64) — KHÔNG commit file này
#         └─ ⭐ kubeseal GỌI SANG controller đang chạy TRONG cluster để lấy PUBLIC KEY mã hoá
#              (cần cluster đang chạy VÀ kubeseal-controller đã cài sẵn — không mã hoá offline hoàn toàn được)

kubectl apply -f sealed-secret.yaml
#              └─ controller trong cluster TỰ ĐỘNG giải mã và sinh ra Secret THƯỜNG bên trong cluster
#                 (bạn KHÔNG cần chạy lệnh giải mã tay — quá trình này TỰ ĐỘNG)
```

⭐ **Đặc điểm cốt lõi — mã hoá GẮN VỚI ĐÚNG một cluster cụ thể:**

`SealedSecret` được mã hoá bằng **public key riêng của controller trong cluster đích** — chỉ **chính cluster đó** (có đúng private key tương ứng) mới giải mã được. Copy file `sealed-secret.yaml` sang **cluster khác** (dev vs prod chẳng hạn) ⇒ cluster kia **không giải mã được**, báo lỗi.

⇒ Đây vừa là **ưu điểm bảo mật** (lộ file cũng không ai giải mã được nếu không có đúng cluster) vừa là **hạn chế thực tế** (không tái sử dụng file sealed giữa các môi trường — mỗi môi trường phải seal riêng).

⚠️ **Backup private key của controller là việc SỐNG CÒN.** Cluster bị xoá và dựng lại (mất private key cũ) ⇒ **mọi SealedSecret cũ trở nên vô dụng vĩnh viễn**, không giải mã lại được — phải seal lại từ đầu với secret gốc (nếu còn giữ ở nơi khác).

**SOPS — mã hoá theo TỪNG FIELD trong file, giữ nguyên cấu trúc YAML để đọc được:**

```bash
sops -e secrets.yaml > secrets.enc.yaml
#     └─ e = encrypt: mã hoá GIÁ TRỊ của từng field, GIỮ NGUYÊN tên field và cấu trúc YAML
#        (khác Sealed Secrets: SOPS vẫn ĐỌC ĐƯỢC cấu trúc, chỉ VALUE bị mã hoá)

sops -d secrets.enc.yaml
#     └─ d = decrypt: giải mã ra màn hình (KHÔNG tự ghi ra file — phải tự > ra file nếu cần)

sops secrets.enc.yaml
#     └─ ⭐ KHÔNG có -e/-d: TỰ ĐỘNG giải mã, MỞ EDITOR để sửa, rồi TỰ ĐỘNG mã hoá lại khi lưu
#        (một lệnh làm trọn quy trình sửa an toàn, không cần nhớ 2 bước riêng)
```

⭐ **SOPS khác Sealed Secrets ở chỗ NÀO — chọn công cụ nào tuỳ nhu cầu:**

| | Sealed Secrets | SOPS |
|---|---|---|
| Mã hoá theo | **Public key của MỘT cluster cụ thể** | Khoá PGP/**KMS** (AWS KMS, GCP KMS, age...) — độc lập cluster |
| `git diff` đọc được gì khi thay đổi 1 field? | ❌ Toàn bộ khối mã hoá đổi khác, không rõ field nào đổi | ⭐ Vẫn thấy **tên field** không đổi, chỉ giá trị (mã hoá) đổi |
| Cần cluster đang chạy để mã hoá? | ✅ Có (gọi tới controller) | ❌ Không — mã hoá offline được |
| Dùng cho | Chuyên biệt cho Secret của K8s | ⭐ Bất kỳ file YAML/JSON/ENV nào, không chỉ K8s |

⚠️ **Cả hai công cụ đều KHÔNG thay thế được RBAC.** Chúng bảo vệ secret **trong Git**, nhưng khi secret đã được **giải mã bên trong cluster** (thành K8s Secret thường), việc **ai đọc được secret đó trong cluster** vẫn phụ thuộc hoàn toàn vào RBAC (`kubectl get secret`) như đã nói ở mục Namespace & Config. Hai công cụ này bảo vệ **một khâu**, không phải toàn bộ vòng đời secret.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa systemctl nâng cao — enable vs is-enabled, và systemd-analyze</b></summary>

| Lệnh | Làm gì |
|---|---|
| `systemctl list-units --type=service` | Danh sách service **đang tải** (không nhất thiết đang chạy) |
| `systemctl list-units --state=failed` | ⭐ Chỉ service **đang fail** — dùng để rà soát nhanh cả hệ thống |
| `systemctl is-active` | Có đang **chạy** không? (trả `active`/`inactive`/`failed`) |
| `systemctl is-enabled` | ⚠️ Có được **đăng ký tự chạy khi boot** không — **KHÔNG liên quan** đang chạy hay không |
| `systemctl cat <s>` | In **nội dung file unit** thật, kể cả khi có nhiều file override chồng lên nhau |
| `systemctl show <s>` | **Mọi property** ở dạng key=value — chi tiết hơn `status` nhiều |
| `systemctl list-dependencies` | Cây phụ thuộc: service này **cần** những gì để chạy |
| `systemd-analyze blame` | Thời gian **boot** của từng service, sắp giảm dần |

⭐ **`is-active` vs `is-enabled` — HAI câu hỏi độc lập, đây là mảnh hay bị gộp làm một:**

| | `is-active` | `is-enabled` |
|---|---|---|
| Trả lời | *"BÂY GIỜ có đang chạy không?"* | *"Có được CÀI ĐẶT để tự chạy khi boot không?"* |
| Đổi bằng lệnh | `start` / `stop` | `enable` / `disable` |

⇒ **Bốn tổ hợp có thể xảy ra**, mỗi tổ hợp một tình huống thực tế khác nhau:

| `is-active` | `is-enabled` | Ý nghĩa thực tế |
|---|---|---|
| `active` | `enabled` | ✅ Bình thường: đang chạy, sẽ tự chạy lại sau reboot |
| `active` | `disabled` | ⚠️ Đang chạy NHƯNG **sẽ KHÔNG tự bật lại** sau khi reboot — dễ gây sự cố bất ngờ sau bảo trì |
| `inactive` | `enabled` | Đã tắt tạm thời (`stop`), nhưng **sẽ tự bật lại** ở lần reboot tới |
| `failed` | `enabled` | 🛑 Service **crash** khi khởi động, nhưng vẫn được cấu hình tự chạy — sẽ **crash lại** mỗi lần reboot cho tới khi sửa |

🛑 Tổ hợp `active` + `disabled` là bẫy hay gặp nhất: ai đó `systemctl start nginx` (chạy tạm để test) mà **quên `enable`** ⇒ mọi thứ trông ổn cho tới lần **reboot tiếp theo** — service **không tự bật lại**, gây sự cố "tự dưng mất dịch vụ sau khi bảo trì server".

⭐ **`enable --now` — làm cả hai việc trong MỘT lệnh, tránh quên mất một nửa:**

```bash
systemctl enable --now nginx
#                 └─ ⭐ vừa ĐĂNG KÝ tự chạy khi boot, VỪA BẬT ngay lập tức
#                    (thay vì phải gõ 2 lệnh riêng: enable rồi start)
```

**`systemctl cat` — công cụ điều tra khi "tôi sửa file unit mà hành vi không đổi":**

```bash
systemctl cat nginx.service
```

⭐ **Vì sao lệnh này quan trọng hơn tự mở file bằng `cat`/`vi`?** systemd cho phép **nhiều file override chồng lên nhau** (file gốc trong `/usr/lib/systemd/system/`, override trong `/etc/systemd/system/nginx.service.d/override.conf`). `systemctl cat` in ra **ĐÚNG những gì systemd THỰC SỰ dùng sau khi gộp tất cả các lớp** — tự mở một file bằng `vi` có thể chỉ thấy **một phần**, bỏ sót override đang có hiệu lực ở nơi khác.

```bash
systemctl show nginx --property=Restart,RestartSec,MemoryMax
#              └─ ⭐ lọc ĐÚNG property cần xem, thay vì cuộn qua hàng trăm dòng của `show` đầy đủ
```

⚠️ **Nhắc lại mảnh ghép đã nói ở mục Log hệ thống**: sửa file unit xong **BẮT BUỘC** `systemctl daemon-reload` rồi mới `restart` — thiếu bước reload, `cat`/`show` vẫn hiện đúng nội dung file trên đĩa, nhưng **systemd đang chạy trong RAM vẫn dùng bản CŨ** — đây chính là lý do "tôi sửa rồi mà không thấy đổi gì".

**`list-dependencies` — trả lời "service này cần gì mới chạy được":**

```bash
systemctl list-dependencies nginx
#                            └─ hiện cây: nginx.service PHỤ THUỘC vào network.target, ...
systemctl list-dependencies nginx --reverse
#                                  └─ ⭐ NGƯỢC LẠI: những service NÀO phụ thuộc VÀO nginx
#                                     (hữu ích để biết dừng nginx sẽ LÀM SẬP THEO những gì)
```

**`systemd-analyze blame` — tìm service làm CHẬM quá trình boot:**

```bash
systemd-analyze blame | head -10
#                        └─ 10 service CHIẾM NHIỀU THỜI GIAN NHẤT khi khởi động máy, sắp giảm dần
systemd-analyze critical-chain
#              └─ ⭐ khác blame: hiện CHUỖI phụ thuộc DÀI NHẤT quyết định TỔNG thời gian boot
#                 (một service chờ 5 lâu không đáng ngại nếu nó KHÔNG nằm trên đường găng của boot)
```

⚠️ `blame` liệt kê **thời gian riêng lẻ** của từng service, nhưng service chạy **song song** với nhau — thời gian boot tổng không phải tổng cộng dồn của `blame`. `critical-chain` mới cho biết **chuỗi nào thực sự quyết định** tổng thời gian khởi động — đáng tin hơn khi cần tối ưu tốc độ boot thật sự.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa Podman — vì sao "không daemon" là điểm khác biệt cốt lõi</b></summary>

⭐ **Tiền đề — sự khác biệt kiến trúc căn bản với Docker:**

| | Docker | Podman |
|---|---|---|
| Kiến trúc | Có **daemon** (`dockerd`) chạy nền, mọi lệnh `docker` gửi yêu cầu tới daemon đó | ⭐ **Không có daemon** — mỗi lệnh `podman` tự nó **là** tiến trình quản lý container luôn |
| Chạy container với quyền gì mặc định | daemon chạy bằng **root** ⇒ container cũng dễ có quyền root | ⭐ **Rootless theo mặc định** — container chạy bằng đúng quyền user gọi nó |
| Daemon chết thì sao | 🛑 **Mọi** container bị ảnh hưởng (daemon là điểm chết chung) | Không có điểm chết chung — mỗi container độc lập |

⇒ **Hệ quả bảo mật quan trọng**: Docker daemon chạy bằng root, và **ai thuộc nhóm `docker`** (đã nói ở mục User & Nhóm phía trên) **tương đương có quyền root** — vì họ ra lệnh được cho một tiến trình root. Podman **loại bỏ** đúng lỗ hổng này: không có daemon trung gian chạy root, container của user thường **thực sự** chỉ có quyền của user đó.

**Cú pháp — cố ý giống Docker gần như 100%, để chuyển đổi không phải học lại:**

```bash
podman ps / podman ps -a           # y hệt docker
podman run -d --name app <image>   # y hệt docker
podman images / podman pull <image>
podman logs -f <container>
podman exec -it <container> bash
podman build -t app:1.0 .
```

⭐ **`alias docker=podman` — dùng được vì tương thích cú pháp gần như tuyệt đối:**

```bash
alias docker=podman
#     └─ ⭐ vì cú pháp GẦN NHƯ GIỐNG HỆT, phần lớn script/Dockerfile CÓ SẴN chạy được
#        MÀ KHÔNG CẦN SỬA GÌ — đây là mục tiêu thiết kế của Podman
```

⚠️ **Không phải giống 100%** — vài khác biệt tinh vi có thể gây lỗi khi chuyển đổi vội vàng:
- `podman-compose` là **công cụ riêng** (không phải `docker compose`), cú pháp tương thích phần lớn nhưng **không đảm bảo mọi tính năng đều y hệt**.
- Mạng mặc định (`podman network`) có hành vi DNS nội bộ hơi khác đôi chỗ so với `docker network`.
- Volume mount rootless có thể gặp vấn đề **quyền UID/GID** khác Docker (do container không chạy bằng root thật).

**Pod — khái niệm mượn TỪ Kubernetes, không có ở Docker:**

```bash
podman pod create --name mypod
#           └─ ⭐ NHIỀU container CÙNG chia sẻ network namespace,
#              GIỐNG HỆT khái niệm "Pod" trong Kubernetes (đã học ở mục kubectl phía trên)
podman run -d --pod mypod nginx      # thêm container VÀO pod đã tạo
podman run -d --pod mypod busybox
#  => hai container này gọi nhau qua "localhost", CHIA SẺ CÙNG một địa chỉ IP -- y hệt cách Pod K8s hoạt động
```

⇒ Đây là điểm hay: **học Podman pod = ôn lại chính xác khái niệm Pod của K8s** — cùng một mô hình network namespace dùng chung.

⭐ **`podman generate kube` / `podman play kube` — cầu nối giữa Docker/Podman thường ngày và K8s thật:**

```bash
podman generate kube mypod > pod.yaml
#                └─ ⭐ XUẤT container/pod ĐANG CHẠY ra thành MANIFEST KUBERNETES chuẩn
#                   (dùng thử nhanh trên máy dev, rồi có sẵn YAML để triển khai lên K8s thật)

podman play kube pod.yaml
#            └─ NGƯỢC LẠI: đọc một manifest K8s, CHẠY THỬ ngay bằng Podman
#               (không cần dựng cả cluster K8s chỉ để test một YAML đơn giản)
```

⇒ Rất hữu ích khi phát triển local: viết thử YAML kiểu K8s, `play kube` để chạy thử nhanh trên máy cá nhân **trước khi** đưa lên cluster thật — rút ngắn vòng lặp thử-sửa.

⚠️ **Rootless Podman có giới hạn** không thể bỏ qua: không bind được **port dưới 1024** (80, 443) trừ khi cấu hình thêm (`sysctl net.ipv4.ip_unprivileged_port_start`), và một số tính năng cần capability đặc biệt của kernel sẽ **không hoạt động** như khi chạy bằng root — đây là đánh đổi cho việc **an toàn hơn theo mặc định**.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa crictl — công cụ dùng khi kubectl KHÔNG CÒN hoạt động</b></summary>

⭐ **Tiền đề — vì sao cần một công cụ RIÊNG ngoài `kubectl`?**

`kubectl` nói chuyện với **API server** — một thành phần **ở tầng cao**, phụ thuộc etcd, phụ thuộc mạng cluster hoạt động bình thường. Khi API server **sập**, hoặc node **mất kết nối** với control plane, `kubectl` **hoàn toàn không dùng được** — nhưng container **vẫn đang chạy thật sự trên node đó**, do **container runtime** (containerd/CRI-O) quản lý ở tầng thấp hơn nhiều, độc lập với API server.

```
kubectl  ──gọi──>  API server  ──gọi──>  kubelet (trên từng node)  ──gọi──>  container runtime
                        🛑 tầng NÀY sập                                      │
                                                                              └─ crictl NÓI CHUYỆN
                                                                                 TRỰC TIẾP VỚI ĐÂY,
                                                                                 BỎ QUA TOÀN BỘ tầng trên
```

⇒ **`crictl` chính là công cụ dùng khi mọi thứ ở tầng trên đã sập** — SSH thẳng vào node, dùng `crictl` để biết container **thật sự** đang chạy gì, bất kể API server sống hay chết.

| Lệnh | Tương đương `kubectl`/`docker` |
|---|---|
| `crictl ps` | `docker ps` — container đang chạy TRÊN NODE NÀY |
| `crictl ps -a` | Cả container đã dừng |
| `crictl pods` | ⭐ Liệt kê **sandbox** (Pod) — khái niệm crictl RIÊNG, xem giải thích dưới |
| `crictl images` | `docker images` |
| `crictl logs <id>` | `kubectl logs`, nhưng chạy **thẳng trên node**, không qua API server |
| `crictl exec -it <id> sh` | `kubectl exec` phiên bản node-local |
| `crictl inspect <id>` | `docker inspect` |
| `crictl stats` | `docker stats` |

⭐ **"sandbox" (`crictl pods`) — vì sao có khái niệm này mà Docker không có:**

Trong K8s, mỗi **Pod** cần một **network namespace dùng chung** cho mọi container bên trong nó (đây chính là lý do container cùng Pod gọi nhau qua `localhost` — đã nói ở mục Podman phía trên). Runtime tạo ra một **container ẩn đặc biệt** (thường dựa trên image `pause`) để **giữ chỗ** cho network namespace đó — gọi là **sandbox**. `crictl pods` liệt kê các sandbox này, khớp trực tiếp với khái niệm "Pod" mà `kubectl get pods` cho thấy ở tầng cao hơn.

**Kịch bản thực tế — node có container `CrashLoopBackOff` mà API server không phản hồi:**

```bash
# SSH thẳng vào node bị nghi có vấn đề
sudo crictl ps -a | grep -i myapp
#                    └─ tìm container theo TÊN, kể cả đã dừng (-a)

sudo crictl logs <container-id> --tail 100
#                                └─ ⭐ giống hệt tinh thần "kubectl logs --tail",
#                                   nhưng hoạt động NGAY CẢ KHI kube-apiserver đang sập

sudo crictl inspect <container-id> | jq '.status.reason, .status.exitCode'
#                                        └─ lý do dừng + mã thoát -- CHẨN ĐOÁN NGAY TẠI NODE
```

⚠️ **`crictl` gần như luôn cần `sudo`** — nó nói chuyện trực tiếp với socket của container runtime (`/run/containerd/containerd.sock` hoặc tương tự), vốn chỉ root/nhóm đặc quyền mới truy cập được.

⚠️ **`crictl` chỉ thấy container TRÊN ĐÚNG NODE nó đang chạy** — không có khái niệm "toàn cluster" như `kubectl get pods -A`. Muốn điều tra một Pod, phải **biết trước Pod đó đang chạy ở node nào** (từ `kubectl get pod -o wide` **trước khi** API server sập, hoặc từ thông tin đã ghi chú sẵn) rồi mới SSH đúng node đó.

**Cấu hình runtime endpoint — cần khi crictl không tự tìm thấy socket:**

```bash
crictl --runtime-endpoint unix:///run/containerd/containerd.sock ps
#      └─ chỉ định RÕ socket của containerd, cần khi có NHIỀU runtime cài trên cùng máy
#         gây nhầm lẫn, hoặc file cấu hình mặc định /etc/crictl.yaml chưa được thiết lập đúng
```

</details>

### nerdctl (CLI giống Docker cho containerd)
```bash
nerdctl ps                             # Liệt kê container
nerdctl run -d <image>                 # Chạy
nerdctl build -t app .                 # Build
nerdctl compose up -d                  # Hỗ trợ cả compose
nerdctl -n k8s.io ps                   # Xem container của k8s (namespace k8s.io)
```

<details>
<summary><b>Bấm xem: giải nghĩa nerdctl — và vì sao namespace k8s.io là chìa khoá</b></summary>

⭐ **Tiền đề — `nerdctl` khác `crictl` ở MỤC ĐÍCH, dù cùng chạy trên containerd:**

| | `crictl` | `nerdctl` |
|---|---|---|
| Sinh ra để | **Chẩn đoán/debug** (theo chuẩn CRI mà Kubernetes định nghĩa) | ⭐ **Thay thế trải nghiệm Docker CLI hằng ngày**, đầy đủ tính năng hơn |
| Cú pháp | Khác Docker khá nhiều | ⭐ **Gần như giống hệt** `docker` — chuyển đổi gần như không cần học lại |
| Hỗ trợ `compose` | ❌ Không | ✅ **Có** (`nerdctl compose`) |

⇒ Nếu `crictl` là "công cụ điều tra khi có sự cố", thì `nerdctl` là **"docker CLI thay thế"** cho containerd trong công việc **hằng ngày** — build, run, compose như bình thường.

```bash
nerdctl ps                    # y hệt docker ps
nerdctl run -d <image>        # y hệt docker run -d
nerdctl build -t app .        # y hệt docker build
nerdctl compose up -d         # ⭐ HỖ TRỢ compose — điều crictl KHÔNG có
```

⭐⭐ **`nerdctl -n k8s.io ps` — dòng lệnh QUAN TRỌNG NHẤT của mục này, cần hiểu SÂU vì sao có nó:**

**Tiền đề bắt buộc**: containerd tổ chức mọi container theo **namespace nội bộ của chính containerd** (khái niệm này **hoàn toàn khác** — dễ nhầm — với "namespace" của Kubernetes hay của Linux). Container do bạn tự chạy tay (`nerdctl run` trần) nằm ở namespace containerd tên **`default`**. Nhưng **container do Kubernetes/kubelet tạo ra** (Pod thật) lại nằm ở namespace containerd tên **`k8s.io`** — một namespace RIÊNG BIỆT, tách khỏi `default`.

```bash
nerdctl ps                # 🛑 CHỈ thấy container namespace "default" -> Pod K8s SẼ KHÔNG hiện ra ở đây!
nerdctl -n k8s.io ps       # ⭐ chỉ định RÕ namespace "k8s.io" -> MỚI thấy được container CỦA KUBERNETES
```

🛑 **Đây là bẫy gây bối rối lớn nhất khi mới học `nerdctl`**: SSH vào node K8s, gõ `nerdctl ps` thấy **trống trơn** hoặc chỉ vài container lạ, trong khi `kubectl get pods -o wide` rõ ràng báo node này đang chạy **10 pod** — không phải node hỏng, mà là container thật nằm ở namespace `k8s.io`, còn `nerdctl ps` (không cờ `-n`) mặc định chỉ nhìn vào `default`.

**Ba lệnh "namespace" khác nhau — dễ gộp lẫn, phải tách rõ để không hoang mang:**

| Loại "namespace" | Thuộc về | Ví dụ |
|---|---|---|
| Kubernetes namespace | Chia nhóm resource trong K8s | `default`, `ai-hub`, `kube-system` |
| Linux namespace (kernel) | Cô lập tiến trình (network, PID, mount...) | Cơ chế bên dưới container nói chung |
| **containerd namespace** | ⭐ Cách **containerd** tự phân vùng container do nó quản lý | `default` (do bạn tự tạo), `k8s.io` (do kubelet tạo) |

⇒ Ba khái niệm **cùng dùng chữ "namespace" nhưng hoàn toàn độc lập** — thấy chữ "namespace" trong lệnh `nerdctl -n k8s.io` thì **đừng liên tưởng** tới namespace của `kubectl get pods -n ai-hub`, chúng không liên quan gì tới nhau.

**Kịch bản thực tế — điều tra Pod K8s bằng nerdctl thay vì crictl:**

```bash
nerdctl -n k8s.io ps | grep myapp          # tìm container CỦA K8S theo tên
nerdctl -n k8s.io logs <container-id>      # xem log TRỰC TIẾP, không qua API server
nerdctl -n k8s.io inspect <container-id>   # chi tiết cấu hình runtime
```

⚠️ Về **mục đích chẩn đoán container của K8s khi API server sập**, `crictl` (chuẩn CRI chính thức) và `nerdctl -n k8s.io` (dành riêng cho containerd) làm được **việc tương tự nhau** — `crictl` có ưu điểm là **chuẩn chung** hoạt động với bất kỳ CRI runtime nào (containerd, CRI-O), còn `nerdctl` chỉ hoạt động khi runtime **chính là containerd**.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa Istio — sidecar injection và istioctl analyze</b></summary>

⭐ **Tiền đề — Service Mesh giải quyết bài toán gì, và "sidecar" là gì?**

Trong một cluster có hàng chục service gọi lẫn nhau, mỗi service muốn có **retry, timeout, mTLS, quan sát traffic** thì phải **tự viết logic đó trong code** — trùng lặp công sức, và mỗi ngôn ngữ lập trình lại làm khác nhau. **Service Mesh** giải quyết bằng cách: chèn một **proxy nhỏ** (Envoy) chạy **cạnh mỗi Pod** (gọi là **sidecar** — "xe bên hông xe máy") để **chặn và điều phối MỌI traffic ra/vào** của Pod đó — logic retry/mTLS/observability nằm ở proxy, **không cần sửa code ứng dụng**.

```
Không có mesh:  App A  ────gọi trực tiếp────>  App B
Có mesh:        App A -> [Envoy sidecar A] ──> [Envoy sidecar B] -> App B
                          └─ MỌI request đi QUA đây, App KHÔNG BIẾT có sidecar
```

| Lệnh | Làm gì |
|---|---|
| `istioctl version` | Version control plane + data plane (Envoy) — kiểm tra khớp nhau chưa |
| `istioctl install` | Cài Istio vào cluster |
| `istioctl analyze` | ⭐ **Phân tích cấu hình**, tìm lỗi mà `kubectl apply` KHÔNG báo |
| `istioctl proxy-status` | Đồng bộ config giữa control plane và **từng sidecar** |
| `istioctl proxy-config routes <pod>` | Route THẬT mà sidecar của pod đó đang dùng |

⭐ **Bật sidecar injection — bước BẮT BUỘC trước, nếu không mesh không hoạt động dù đã cài Istio:**

```bash
kubectl label namespace ai-hub istio-injection=enabled
#                                └─ ⭐ đánh dấu NAMESPACE này để Istio TỰ ĐỘNG chèn sidecar
#                                   vào MỌI Pod MỚI tạo ra trong namespace đó
```

🛑 **`istio-injection=enabled` CHỈ áp dụng cho Pod TẠO MỚI SAU KHI label được gắn.** Pod **đang chạy từ trước** đó **KHÔNG tự động** có sidecar — phải **tạo lại** (thường bằng `kubectl rollout restart deploy`) để injection có hiệu lực. Đây là nguyên nhân của "tôi đã label namespace mà sao Pod vẫn chỉ có 1 container" — cần restart, không phải chỉ label là xong.

```bash
kubectl get pod mypod -o jsonpath='{.spec.containers[*].name}'
#  Không có mesh:  myapp
#  ⭐ Có mesh:      myapp istio-proxy
#                          └─ ⭐ đây chính là SIDECAR — mỗi Pod trong mesh có ÍT NHẤT 2 container
```

⭐⭐ **`istioctl analyze` — CHẠY LỆNH NÀY TRƯỚC KHI DEBUG BẰNG TAY, vì nó bắt được lỗi `kubectl apply` bỏ qua:**

`kubectl apply` chỉ kiểm tra **cú pháp YAML đúng chuẩn K8s** — nó **KHÔNG BIẾT** ý nghĩa nghiệp vụ của các CRD Istio (VirtualService trỏ tới DestinationRule không tồn tại, Gateway không khớp Selector nào...). Config kiểu này **apply thành công, không báo lỗi gì**, nhưng traffic **âm thầm đi sai hướng hoặc rơi vào khoảng không**.

```bash
istioctl analyze -n ai-hub
#                 └─ ⭐ quét TOÀN BỘ cấu hình Istio TRONG namespace này,
#                    tìm những lỗi LOGIC mà kubectl apply đã bỏ qua hoàn toàn
```

⇒ **Quy tắc thực hành**: mọi lần sửa VirtualService/DestinationRule/Gateway, chạy `istioctl analyze` **ngay sau** `kubectl apply` — đừng đợi tới lúc traffic thật báo lỗi 503 mới đi tìm nguyên nhân.

⭐ **`istioctl proxy-status` — kiểm tra ĐỒNG BỘ giữa "ý định" (control plane) và "thực thi" (từng sidecar):**

```bash
istioctl proxy-status
# NAME                 CDS        LDS        EDS        RDS        SYNCED
# myapp-7d9f.ai-hub    SYNCED     SYNCED     SYNCED     SYNCED     ✅
# api-gw-x2k.ai-hub    STALE      SYNCED     SYNCED     SYNCED     ⚠️
#                       └─ ⭐ "STALE" = sidecar này ĐANG DÙNG CẤU HÌNH CŨ,
#                          chưa nhận được bản cập nhật mới nhất từ control plane
```

🛑 **`STALE` là nguyên nhân của "tôi đã sửa VirtualService rồi mà traffic vẫn đi theo rule cũ".** Sidecar chưa đồng bộ được config mới — thường do sidecar quá tải, hoặc mất kết nối tạm thời tới `istiod` (control plane). Kiểm tra log của chính sidecar đó khi gặp `STALE` kéo dài.

**Đào sâu route thật của MỘT pod cụ thể — khi cần biết CHÍNH XÁC nó đang định tuyến ra sao:**

```bash
istioctl proxy-config routes myapp-7d9f -n ai-hub
#                      │      └─ tên pod
#                      └─ routes: bảng ĐỊNH TUYẾN thật -- giống hệt tinh thần "kubectl describe"
#                         nhưng đào SÂU vào cấu hình BÊN TRONG sidecar, không phải object K8s

istioctl proxy-config cluster myapp-7d9f -n ai-hub
#                      └─ cluster (trong ngữ cảnh Envoy) = danh sách UPSTREAM mà sidecar này BIẾT tới,
#                         KHÁC hoàn toàn khái niệm "Kubernetes cluster" — dễ nhầm vì trùng chữ
```

⚠️ **Chữ "cluster" ở đây KHÔNG PHẢI Kubernetes cluster** — trong Envoy, "cluster" nghĩa là **một nhóm endpoint đích** (upstream) mà proxy có thể gửi traffic tới. Đọc lệnh `proxy-config cluster` mà liên tưởng tới "cụm Kubernetes" sẽ hiểu sai hoàn toàn kết quả trả về.

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa Linkerd — check trước, viz để quan sát</b></summary>

⭐ **Tiền đề — Linkerd cùng nhóm Service Mesh với Istio (đã giải thích ở mục trên), nhưng khác triết lý:** Linkerd chủ trương **đơn giản và nhẹ hơn** — ít tính năng cấu hình phức tạp hơn Istio, đổi lại **dễ vận hành hơn** và **sidecar tốn ít tài nguyên hơn**. Cùng khái niệm sidecar/mTLS/observability như đã học ở mục Istio, chỉ khác công cụ và triết lý thiết kế.

| Lệnh | Làm gì |
|---|---|
| `linkerd check` | ⭐ Kiểm tra sức khoẻ **TRƯỚC KHI** làm bất cứ điều gì khác |
| `linkerd install \| kubectl apply -f -` | Cài đặt (sinh YAML rồi apply) |
| `linkerd viz install` | Cài **thêm** bộ dashboard quan sát (tách riêng khỏi lõi) |
| `linkerd viz stat` | Thống kê success rate, RPS, latency |
| `linkerd viz tap` | ⭐ Xem **từng request LIVE**, thời gian thực |
| `linkerd viz top` | Route nào nhiều traffic nhất |

⭐⭐ **`linkerd check` — vì sao PHẢI chạy đầu tiên, không phải tuỳ chọn:**

```bash
linkerd check
```

Lệnh này kiểm tra **một chuỗi điều kiện tiên quyết** theo thứ tự: version CLI có khớp control plane không → cert TLS nội bộ còn hạn không → control plane có Healthy không → mọi thứ cấu hình đúng không. **Nếu bất kỳ bước nào trong đây fail, MỌI lệnh `linkerd viz` phía sau đều sẽ cho kết quả SAI LỆCH hoặc LỖI KHÓ HIỂU** — vì chúng dựa vào tiền đề "control plane đang khoẻ mạnh" mà `check` xác nhận.

⇒ **Quy tắc thực hành**: gặp bất kỳ hành vi lạ nào của Linkerd, **luôn `linkerd check` trước tiên** — đừng vội đi debug ứng dụng khi chính bản thân mesh đang có vấn đề nền tảng.

⚠️ Một lỗi hay gặp mà `check` bắt được sớm: **cert mTLS nội bộ của Linkerd có hạn sử dụng** (mặc định thường ~1 năm cho cert gốc). Cert hết hạn ⇒ **toàn bộ giao tiếp giữa các sidecar bị từ chối** ⇒ mesh "sập" đồng loạt trông như sự cố lớn, nhưng nguyên nhân gốc chỉ là **quên gia hạn chứng chỉ** — `linkerd check` cảnh báo trước khi việc này thành thảm hoạ.

**`viz` — bộ công cụ quan sát TÁCH RIÊNG khỏi phần lõi mesh:**

```bash
linkerd viz install | kubectl apply -f -
#      └─ ⭐ "viz" = visualization, một PHẦN MỞ RỘNG riêng biệt
#         (lõi mesh có thể chạy mà KHÔNG CẦN viz — viz chỉ để QUAN SÁT, không phải bắt buộc)
linkerd viz dashboard
#             └─ mở giao diện web quan sát traffic realtime trên trình duyệt
```

⭐⭐ **`linkerd viz tap` — công cụ mạnh nhất khi cần biết "request VỪA XẢY RA đi đâu, kết quả gì":**

```bash
linkerd viz tap deploy/myapp
#               └─ ⭐ xem TỪNG request LIVE, ngay lúc nó đang xảy ra — không phải số liệu tổng hợp
```

Kết quả in ra **từng dòng cho mỗi request**: method, đường dẫn, mã trạng thái trả về, độ trễ — **thời gian thực**, giống hệt tinh thần `tcpdump` (đã học ở mục Network debug) nhưng ở **tầng ứng dụng (Layer 7)** thay vì tầng gói tin thô.

⇒ Khi nghi ngờ *"service A có thực sự gọi được tới service B không, và trả về gì"* — thay vì đoán qua log ứng dụng (có thể log không đủ chi tiết), `tap` cho thấy **chính xác request đang chảy qua mesh**, độc lập với việc code ứng dụng có log tốt hay không.

⚠️ **`tap` tạo tải phụ đáng kể** khi bật trên deployment có traffic rất cao — nó chặn và nhân bản **mọi** request để hiển thị. Chỉ bật **trong thời gian ngắn** khi thực sự cần điều tra, giống nguyên tắc đã nói với `strace`/`tcpdump`.

**`viz stat` — con số tổng hợp, đối lập với `tap` (từng request riêng lẻ):**

```bash
linkerd viz stat deploy -n ai-hub
#                       └─ ⭐ success rate (%), RPS (request/giây), p50/p95/p99 latency
#                          THEO TỪNG DEPLOYMENT trong namespace — bức tranh TỔNG QUAN
```

⇒ Dùng `stat` để có **cái nhìn tổng thể** ("service nào đang có success rate thấp bất thường"), rồi dùng `tap` để **đào sâu vào TỪNG request** của đúng service nghi vấn đó — hai lệnh bổ trợ nhau theo hai mức độ chi tiết khác nhau.

```bash
linkerd viz top deploy/myapp
#               └─ giống "top" của Linux nhưng cho ROUTE: route nào NHIỀU TRAFFIC NHẤT, cập nhật liên tục
```

</details>

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

<details>
<summary><b>Bấm xem: giải nghĩa k9s, stern, kubectx/kubens, krew</b></summary>

**Tiền đề — vì sao cần công cụ ngoài `kubectl` thuần?** `kubectl get/describe/logs` là các lệnh **rời rạc, chạy một lần rồi thoát** — muốn theo dõi liên tục phải tự `watch` hoặc gõ lại nhiều lần. Nhóm công cụ này **bọc quanh `kubectl`** để có trải nghiệm **tương tác, realtime** hơn.

⭐ **`k9s` — TUI (Terminal UI) điều khiển cluster bằng bàn phím, không cần chuột:**

```bash
k9s                    # mở giao diện, mặc định context/namespace hiện tại
k9s -n ai-hub           # mở THẲNG vào namespace này, khỏi phải điều hướng
```

**Điều khiển cơ bản bên trong k9s** (không phải cờ dòng lệnh — đây là phím tắt TRONG giao diện):

| Phím | Làm gì |
|---|---|
| `:pods`, `:svc`, `:deploy` | Gõ dấu `:` rồi tên resource để **chuyển view** — thay cho gõ lại `kubectl get <resource>` |
| `/` | Lọc (filter) danh sách đang xem theo tên |
| `d` | **d**escribe — xem chi tiết + Events ngay tại chỗ, không cần gõ lệnh riêng |
| `l` | **l**ogs — xem log ngay, không cần rời màn hình |
| `s` | **s**hell — vào thẳng container, tương đương `kubectl exec -it` |
| `Ctrl+d` | Xoá resource đang chọn (⚠️ **có hỏi xác nhận**, nhưng vẫn nên cẩn trọng) |
| `Esc` | Quay lại view trước |

⭐ **Giá trị cốt lõi của k9s**: các thao tác `get` → `describe` → `logs` → `exec` mà bình thường phải gõ **4 lệnh `kubectl` riêng biệt với đúng tên pod mỗi lần**, trong k9s chỉ là **4 phím bấm** trên đúng dòng đang chọn — tốc độ điều tra sự cố nhanh hơn hẳn khi cần lướt qua nhiều pod liên tiếp.

⚠️ k9s là công cụ **tương tác** — không hợp dùng trong **script tự động hoá**. Với CI/CD hay script, vẫn phải dùng `kubectl` thuần (có thể parse output, có exit code rõ ràng).

⭐⭐ **`stern` — giải quyết đúng bài toán "xem log của NHIỀU pod cùng lúc, có màu phân biệt":**

🛑 **Vấn đề của `kubectl logs` thuần**: một Deployment có **5 replica** đang chạy — `kubectl logs <pod>` chỉ xem được **ĐÚNG MỘT pod tại một thời điểm**. Muốn xem log tổng hợp của cả 5, phải mở **5 terminal riêng**, mỗi cái gõ tên pod khác nhau — rất bất tiện khi cần theo dõi một sự cố đang xảy ra trên nhiều replica cùng lúc.

```bash
stern myapp
#     └─ ⭐ khớp theo TÊN (regex), tự động bám log của MỌI pod hiện có VÀ pod MỚI SINH RA sau đó
#        (khác kubectl logs: chỉ bám được đúng 1 pod cụ thể tại thời điểm bạn gõ lệnh)

stern myapp -n ai-hub          # trong 1 namespace
stern -l app=nginx             # ⭐ theo LABEL selector, không cần biết tên chính xác từng pod
stern myapp --since 10m        # chỉ log 10 phút gần đây (giống --since của journalctl/docker logs)
```

⭐ **Vì sao `stern` "tự bám pod mới sinh ra" là tính năng KHÔNG THỂ THAY THẾ bằng `kubectl logs`?** Trong lúc một **rollout** đang diễn ra (pod cũ bị thay pod mới liên tục), `kubectl logs <pod-cũ>` sẽ **dừng đột ngột** khi pod đó bị xoá — bạn phải tự tay lấy tên pod mới rồi chạy lại lệnh. `stern` với bộ lọc theo **tên pattern** hoặc **label** sẽ **tự động chuyển sang bám pod mới** ngay khi nó xuất hiện, không gián đoạn theo dõi.

Output của `stern` **tô màu riêng cho từng pod** ở đầu mỗi dòng — giúp phân biệt log của pod nào trong dòng log gộp chung, điều mà nối nhiều `kubectl logs -f &` chạy nền không làm được gọn gàng bằng.

⭐ **`kubectx` / `kubens` — thao tác nhanh hơn cú pháp `kubectl config` dài dòng đã học ở mục Context & Cluster:**

```bash
kubectx                # liệt kê MỌI context (tương đương kubectl config get-contexts, nhưng gọn hơn)
kubectx <context>      # chuyển NGAY (tương đương kubectl config use-context)
kubens                 # liệt kê namespace TRONG cluster hiện tại
kubens <namespace>     # đổi namespace mặc định (tương đương lệnh set-context dài đã học trước đó)
```

⇒ Đây thuần tuý là **bản rút gọn cú pháp** của các lệnh `kubectl config ...` đã giải thích chi tiết ở mục Context & Cluster phía trên — không có khái niệm mới, chỉ gõ nhanh hơn. Nhiều bản `kubectx` còn hỗ trợ **menu chọn bằng phím mũi tên** khi gõ trần không kèm tham số, thay vì phải nhớ chính xác tên context/namespace.

⭐ **`krew` — trình quản lý PLUGIN của kubectl, giống `apt`/`brew` nhưng dành riêng cho hệ sinh thái kubectl:**

```bash
kubectl krew install neat        # cài plugin "kubectl neat" (đã nhắc ở mục backup manifest phía trên)
kubectl krew list                # liệt kê plugin ĐÃ CÀI qua krew
kubectl krew search <keyword>    # tìm plugin trong kho plugin cộng đồng
kubectl krew upgrade             # nâng cấp TẤT CẢ plugin đã cài lên bản mới nhất
```

⇒ Sau khi cài bằng `krew`, mọi plugin tự động gọi được qua `kubectl <tên-plugin>` (ví dụ `kubectl neat`) — **liền mạch với `kubectl` gốc**, không cần nhớ thêm một chương trình riêng biệt để gọi.

⚠️ **`krew` cần tải plugin TỪ INTERNET** (kho plugin trên GitHub) — trên **VDI air-gapped**, bước cài đặt này thường **không thực hiện được** trực tiếp; cần tải file binary của plugin về qua máy có mạng rồi copy thủ công vào đúng đường dẫn `krew` quản lý.

</details>

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

> 📘 **Xem sâu hơn:** [`k8s-operations-playbook.md`](./k8s-operations-playbook.md) — playbook đầy đủ về
> sự cố thực tế (triệu chứng → chẩn đoán → xử lý → nguyên nhân gốc → phòng ngừa lâu dài),
> backup/DR chuyên sâu, reliability, observability, security, và lộ trình DevOps Lead.

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







