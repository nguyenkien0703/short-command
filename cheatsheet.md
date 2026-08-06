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







