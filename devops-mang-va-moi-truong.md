# DevOps toàn cảnh: các mảng & ánh xạ theo môi trường

Tài liệu deep-dive: 10 mảng cốt lõi của DevOps, đi sâu vào **từng hạng mục con**, và với mỗi mảng trả lời câu hỏi thực chiến quan trọng nhất — **ở dev / test / staging / prod thì làm khác nhau ra sao, công cụ nào hợp môi trường nào**. Ghi chú bằng tiếng Việt để dễ tra cứu.

> **Nguyên tắc xuyên suốt:** Phần lớn trường hợp bạn dùng *cùng một công cụ* ở mọi môi trường. Thứ thay đổi là **cấu hình, quy mô, quyền truy cập và mức độ kiểm soát** — không phải bản thân công cụ. Mục tiêu tối thượng là **parity** (đồng nhất giữa các môi trường): khác biệt duy nhất nên nằm trong biến cấu hình và secrets.

## 📑 Mục lục

- [🌐 Mô hình môi trường (nền tảng)](#-mô-hình-môi-trường-nền-tảng)
- [01. 🌿 Source Control & Collaboration](#01--source-control--collaboration)
- [02. 🔄 CI/CD](#02--cicd)
- [03. 🏗️ Infrastructure as Code (IaC)](#03-️-infrastructure-as-code-iac)
- [04. 🐳 Containerization & Orchestration](#04--containerization--orchestration)
- [05. ☁️ Cloud & Networking](#05-️-cloud--networking)
- [06. 📊 Monitoring & Observability](#06--monitoring--observability)
- [07. 🔐 Security (DevSecOps)](#07--security-devsecops)
- [08. 🚑 Reliability & Operations (SRE)](#08--reliability--operations-sre)
- [09. 🧪 Testing & Quality](#09--testing--quality)
- [10. 🤝 Culture & Process](#10--culture--process)
- [⭐ Ma trận công cụ × môi trường](#-ma-trận-công-cụ--môi-trường)
- [🗺️ Lộ trình học](#️-lộ-trình-học)

---

## 🌐 Mô hình môi trường (nền tảng)

Trước khi vào từng mảng, cần thống nhất cách hiểu về môi trường vì **mỗi mảng bên dưới đều được lọc qua lăng kính này**.

| Môi trường | Mục đích | Dữ liệu | Tần suất đổi | HA / Scale | Quyền truy cập | Ai tự deploy |
|---|---|---|---|---|---|---|
| **dev** | Lập trình viên phát triển & thử nghiệm nhanh | Giả / mock, xoá bất kỳ lúc nào | Liên tục, mỗi commit | Không cần HA, 1 replica, máy nhỏ | Rộng — dev có quyền admin | Auto mỗi push |
| **test / QA** | Chạy automated test + QA kiểm thử thủ công | Bộ dữ liệu test cố định, tái lập được | Theo build / nightly | Giống dev, ổn định hơn | QA + dev | Auto sau CI xanh |
| **staging (stg / pre-prod / UAT)** | Bản sao **gần giống prod nhất**; UAT, perf test, smoke test trước release | Ẩn danh hoá từ prod hoặc dữ liệu thật thu nhỏ | Theo release candidate | Cấu hình = prod (quy mô có thể nhỏ hơn) | Hạn chế, gần giống prod | Auto có cổng phê duyệt |
| **non-prod** | *Thuật ngữ ô dù* gộp mọi môi trường không phải prod (dev + test + stg + sandbox + uat) | Không bao giờ chứa dữ liệu thật chưa xử lý | — | Tối ưu chi phí (tắt ngoài giờ, spot) | Nới lỏng hơn prod | — |
| **production (prod)** | Phục vụ người dùng thật, có SLA/SLO | Dữ liệu thật — nhạy cảm, có compliance | Qua change management, có phê duyệt | HA, multi-AZ, auto-scaling, DR | **Nghiêm ngặt nhất** — least privilege, break-glass | Manual approval / auto có kiểm soát |

> **Nguyên tắc vàng:** Config khác nhau, nhưng **công cụ và quy trình phải giống nhau**. Nếu staging build/deploy bằng đường khác prod thì staging vô nghĩa.

---

## 01. 🌿 Source Control & Collaboration

Nền móng của mọi thứ. Không có version control tốt thì không có CI/CD, không có IaC, không có audit. Đây là nơi "single source of truth" của code sinh ra.

### Các hạng mục con

- **Version control & nền tảng** — Git; GitHub, GitLab, Bitbucket, Azure Repos. Quản lý lịch sử, phân quyền, mirror.
- **Chiến lược nhánh (branching)** — Trunk-based (khuyên dùng cho CI/CD), GitHub Flow, Git Flow (release dài), release branches.
- **Code review & PR** — Pull/Merge Request, required reviewers, `CODEOWNERS`, branch protection, merge queue.
- **Repo structure** — Monorepo (Nx, Turborepo, Bazel) vs polyrepo. Ảnh hưởng trực tiếp cách build & deploy.
- **Commit hygiene** — Conventional Commits, signed commits (GPG/SSH), pre-commit hooks, semantic versioning & tags.
- **Từ Git kích hoạt** — Webhooks, GitOps (thay đổi = commit), protected tags kích hoạt release pipeline.

### Ánh xạ theo môi trường

Bản thân Git không "theo môi trường", nhưng **nhánh/tag nào deploy vào môi trường nào** là quyết định thiết kế cốt lõi.

| Môi trường | Nhánh nguồn (trunk-based) | Branch protection | Ai được merge |
|---|---|---|---|
| **dev** | feature branch / `main` | Không hoặc nhẹ | Bất kỳ dev |
| **test** | `main` sau CI xanh | CI phải pass | Auto sau merge PR |
| **staging** | tag `rc-*` hoặc `release/*` | ≥1 reviewer + CI | Maintainer |
| **prod** | tag `v*` (semver) đã ký | CODEOWNERS + signed + approval | Release manager / auto có gate |

**Công cụ:** Mọi môi trường dùng chung một nền tảng Git. Khác biệt nằm ở *branch protection rules* và *environment approvals* (GitHub Environments, GitLab Protected Environments) — chính chúng gắn nhánh/tag với môi trường và bắt buộc phê duyệt cho prod.

---

## 02. 🔄 CI/CD

Trái tim của DevOps. **CI** = tự động build + test mỗi thay đổi. **CD** = tự động đưa artifact đã kiểm chứng qua các môi trường tới người dùng.

### Các hạng mục con

- **Build & Test tự động (CI)** — Compile, unit/integration test, lint, đóng gói artifact. Chạy trên mọi PR/commit.
- **Artifact & Registry** — Build một lần, promote qua các môi trường. Lưu ở Artifactory, Nexus, GHCR, ECR.
- **Promotion pipeline (CD)** — Cùng một artifact đi dev → test → stg → prod, chỉ đổi config. **Không rebuild giữa chừng.**
- **Chiến lược deploy** — Rolling, Blue-Green, Canary, Feature Flags. Chọn theo mức rủi ro của môi trường.
- **Cổng phê duyệt (gate)** — Manual approval, automated gate (test pass, security scan), change window cho prod.
- **Rollback & recovery** — Tự động rollback khi health check fail, giữ N phiên bản trước để quay lui nhanh.

> **Nguyên tắc cốt lõi — Build once, deploy many:** Artifact **build đúng một lần** ở CI rồi được *promote* qua từng môi trường. Nếu mỗi môi trường build lại thì "cái chạy ở prod" không phải "cái đã test ở staging" — mất toàn bộ giá trị của pipeline.

### Chiến lược deploy & kiểm soát theo môi trường

| Môi trường | Trigger | Chiến lược deploy | Cổng kiểm soát | Rollback |
|---|---|---|---|---|
| **dev** | Auto mỗi push | Recreate / rolling đơn giản | Không | Push lại |
| **test** | Auto sau CI xanh | Rolling | Test suite phải pass | Auto nếu smoke test fail |
| **staging** | Auto lên RC, có gate | **Giống hệt prod** (blue-green/canary) để tập dượt | QA/UAT sign-off, perf test | Diễn tập rollback như thật |
| **prod** | Manual approval / auto có gate | Canary → progressive, Blue-Green, feature flags | Approval + change window + tất cả gate | **Tự động** theo SLO/health, một cú click quay lui |

### Công cụ theo vai trò

| Vai trò | Công cụ phổ biến | Ghi chú theo môi trường |
|---|---|---|
| CI engine | GitHub Actions, GitLab CI, Jenkins, CircleCI, Azure Pipelines, Buildkite | Cùng engine cho mọi môi trường; dùng *environment/stage* để phân biệt |
| CD / GitOps | Argo CD, Flux, Spinnaker, GitLab CD | GitOps rất mạnh cho stg+prod: trạng thái mong muốn = Git, tự đồng bộ |
| Progressive delivery | Argo Rollouts, Flagger, LaunchDarkly, Unleash | Canary/feature-flag chủ yếu ở **prod** (tập ở staging) |
| Artifact registry | Artifactory, Nexus, GHCR, ECR/GCR/ACR | Một registry, promote qua tag: `-dev` → `-rc` → `-release` |

---

## 03. 🏗️ Infrastructure as Code (IaC)

Hạ tầng được mô tả bằng code, version hoá, review được và tái tạo được. Công cụ số một để đạt "environment parity": cùng một code tạo ra cả 4 môi trường, chỉ khác biến đầu vào.

### Các hạng mục con

- **Provisioning** — Tạo hạ tầng cloud (VPC, VM, DB, cluster): Terraform, OpenTofu, Pulumi, CloudFormation, Bicep.
- **Configuration management** — Cấu hình bên trong máy/OS: Ansible, Chef, Puppet, SaltStack.
- **State & module** — Remote state (S3+DynamoDB, TF Cloud), state locking, module tái sử dụng, workspace.
- **Policy as Code** — OPA/Conftest, Sentinel, Checkov, tfsec — chặn cấu hình sai *trước* khi apply.
- **Image building** — Packer tạo golden image / AMI; Dockerfile cho container base images.
- **Drift detection** — Phát hiện hạ tầng thực đã lệch khỏi code (ai đó sửa tay trên console).

> **Chìa khoá parity:** Dùng **cùng một module Terraform**, khác nhau chỉ ở file biến: `dev.tfvars`, `staging.tfvars`, `prod.tfvars`. Cách tổ chức: dùng *workspaces* hoặc thư mục theo môi trường (khuyên dùng — cô lập state rõ ràng hơn).

### Ánh xạ theo môi trường

| Yếu tố | dev | test | staging | prod |
|---|---|---|---|---|
| Quy mô tài nguyên | Nhỏ nhất (t3.micro) | Nhỏ | ~ bằng prod | Full, multi-AZ |
| State backend | Remote state riêng | Remote state riêng | Remote + **locking** | Remote + **locking bắt buộc** |
| Ai được apply | Dev tự chạy | CI | CI có review | **Chỉ qua pipeline + approval**, cấm apply tay |
| Chi phí | Tắt ngoài giờ, spot, auto-destroy | Spot | On-demand | On-demand/reserved, luôn bật |
| Policy gate | Cảnh báo | Cảnh báo | Chặn | **Chặn cứng** (encryption, tag, no public) |

**Công cụ:** *Provision:* Terraform/OpenTofu (đa cloud, phổ biến nhất), Pulumi (dùng ngôn ngữ lập trình), CloudFormation/Bicep (khoá theo AWS/Azure). *Config:* Ansible (agentless, dễ bắt đầu). *Policy:* Checkov/tfsec/OPA trong CI, siết dần từ "warn" ở non-prod lên "block" ở prod.

---

## 04. 🐳 Containerization & Orchestration

Đóng gói ứng dụng + dependency thành đơn vị chạy nhất quán ở mọi nơi — chính là cách kỹ thuật để "chạy được ở máy tôi" cũng chạy ở prod. Orchestration lo scheduling, scaling, self-healing cho hàng loạt container.

### Các hạng mục con

- **Container runtime** — Docker/containerd, image, multi-stage build, distroless, image scanning & signing.
- **Orchestration** — Kubernetes (chuẩn de-facto), ECS, Nomad. Scheduling, self-healing, HPA/VPA.
- **K8s packaging** — Helm, Kustomize, Jsonnet. Cùng chart, khác `values-<env>.yaml`.
- **Service mesh & ingress** — Istio, Linkerd, Cilium; Ingress/Gateway API; mTLS nội bộ.
- **Serverless / PaaS** — Lambda, Cloud Run, Fargate, Knative — khi không muốn quản cluster.
- **Image registry** — ECR/GCR/ACR/GHCR/Harbor; quét lỗ hổng, ký (cosign), retention policy.

### Ánh xạ theo môi trường

| Yếu tố | dev | test | staging | prod |
|---|---|---|---|---|
| Cluster | Local: kind, k3d, minikube | Cluster non-prod chung (namespace riêng) | Cluster giống prod | Cluster prod chuyên dụng, multi-AZ |
| Cô lập | Namespace / máy cá nhân | Namespace | Cluster/Account riêng | **Account/Project riêng biệt** |
| Replicas & autoscale | 1, không HPA | 1–2 | Giống prod (thu nhỏ) | HPA/VPA, PodDisruptionBudget, multi-AZ |
| Image tag | `:dev-sha`, mutable | `:test-sha` | `:rc-x` | **Digest cố định + đã ký** (immutable) |
| Cấu hình | Cùng base | Cùng base | Cùng base | **Cùng Helm chart / Kustomize base**, khác nhau ở `values-<env>.yaml` và overlay |

**Công cụ:** *Local dev:* Docker Compose, kind/k3d, Tilt/Skaffold (hot-reload vào cluster). *Test→prod:* managed K8s (EKS/GKE/AKS) + Helm/Kustomize + Argo CD. *Nhẹ hơn K8s:* ECS/Cloud Run/Fargate cho đội nhỏ. Nguyên tắc: local có thể khác, nhưng test/stg/prod phải cùng loại cluster để parity.

---

## 05. ☁️ Cloud & Networking

Nơi mọi thứ chạy. Bao gồm chọn nhà cung cấp, thiết kế mạng, và — cực kỳ quan trọng khi có nhiều môi trường — cách **cô lập** chúng để dev không bao giờ đụng được vào prod.

### Các hạng mục con

- **Cloud provider** — AWS, GCP, Azure; hybrid/multi-cloud; region & availability zone.
- **Networking** — VPC, subnet, route table, security group/NACL, VPN, Transit Gateway, PrivateLink.
- **Edge & traffic** — DNS (Route53), Load Balancer, CDN (CloudFront), WAF, API Gateway, TLS/cert.
- **Account isolation** — Multi-account (AWS Organizations), landing zone, tách billing theo môi trường.
- **FinOps / chi phí** — Budget alert, tagging, right-sizing, reserved/spot, dọn tài nguyên orphan.
- **Managed data services** — RDS, S3, cache (Redis), queue (SQS/Kafka) — cấu hình khác nhau rõ theo môi trường.

### Ánh xạ theo môi trường

| Yếu tố | dev / test / non-prod | staging | prod |
|---|---|---|---|
| Cô lập | Account/Project **non-prod** riêng, tách hẳn prod | Có thể chung account pre-prod | **Account/Project prod tách biệt hoàn toàn** |
| Mạng | VPC đơn giản, có thể public subnet | VPC giống prod | Private subnet, NAT, WAF, DDoS protection |
| DNS | `*.dev.cty.com` | `*.stg.cty.com` | `cty.com` (prod), cert quản lý chặt |
| HA | Single-AZ | Multi-AZ (tập dượt) | Multi-AZ, đôi khi multi-region + DR |
| Chi phí | Spot, auto-stop ngoài giờ, budget cứng | On-demand | Reserved/Savings Plan, capacity planning |

> ⚠️ **Cảnh báo cô lập:** Sai lầm phổ biến & nguy hiểm nhất là để dev và prod **chung một account/VPC**. Một script test lỡ tay có thể xoá dữ liệu prod. Tách account/project cho non-prod và prod là ranh giới an toàn cứng — kèm SCP/Org Policy chặn cross-account.

---

## 06. 📊 Monitoring & Observability

Khả năng trả lời "hệ thống đang thế nào, và tại sao?". Ba trụ cột: **metrics** (số liệu), **logs** (sự kiện), **traces** (hành trình request). Cộng thêm alerting để biến tín hiệu thành hành động.

### Các hạng mục con

- **Metrics** — Prometheus, VictoriaMetrics, Grafana, CloudWatch, Datadog. RED/USE method.
- **Logging** — ELK/OpenSearch, Loki, Fluent Bit, structured JSON logs, log level theo môi trường.
- **Distributed tracing** — OpenTelemetry (chuẩn), Jaeger, Tempo, Zipkin. Theo dõi request qua nhiều service.
- **Alerting & on-call** — Alertmanager, PagerDuty, Opsgenie. Alert dựa trên **SLO**, không phải mọi spike.
- **Dashboard & SLO** — Grafana dashboard, SLI/SLO, error budget burn-rate alerts.
- **APM & RUM** — Datadog APM, New Relic, Sentry (error tracking), Real User Monitoring.

### Ánh xạ theo môi trường

| Yếu tố | dev | test | staging | prod |
|---|---|---|---|---|
| Log level | DEBUG | DEBUG/INFO | INFO | INFO/WARN (tránh log dữ liệu nhạy cảm) |
| Retention | Vài ngày | Vài ngày | 2–4 tuần | **Dài (30–90+ ngày)**, có thể theo compliance |
| Alerting | Tắt / dev tự xem | Chỉ báo pipeline | Alert nhưng không gọi on-call | **On-call 24/7**, paging theo SLO |
| Trace sampling | 100% | 100% | Cao | Sampling (1–10%) để tiết kiệm chi phí |
| Chi tiết dashboard | Cơ bản | Test-focused | Giống prod | Đầy đủ: SLO, business metrics, capacity |

**Công cụ:** Dùng **cùng một stack quan sát** cho mọi môi trường (vd Prometheus + Grafana + Loki + OTel/Tempo), phân biệt bằng label `env=`. Staging trở thành nơi kiểm chứng alert trước khi bật ở prod. Trọng tâm đầu tư (retention dài, on-call, APM trả phí) dồn vào **prod**.

---

## 07. 🔐 Security (DevSecOps)

"Shift left": đưa bảo mật vào sớm và xuyên suốt pipeline, không phải một bước kiểm tra cuối. Quét ở non-prod (rẻ, sớm) nhưng siết chặt kiểm soát ở prod (dữ liệu thật).

### Các hạng mục con

- **SAST / SCA** — Static analysis (SonarQube, Semgrep, CodeQL) + dependency scan (Snyk, Dependabot, Trivy).
- **DAST & container scan** — OWASP ZAP; quét image (Trivy, Grype); scan IaC (Checkov, tfsec).
- **Secrets management** — Vault, AWS/GCP Secrets Manager, SOPS, External Secrets. Secret scanning (gitleaks).
- **Identity & Access** — IAM, RBAC, least privilege, SSO/OIDC, MFA, break-glass account.
- **Supply chain security** — SBOM, ký artifact (cosign/sigstore), SLSA, provenance, admission control.
- **Policy & Compliance** — OPA/Kyverno, CIS Benchmark, SOC2/ISO/PCI, audit logging, network policy.

### Ánh xạ theo môi trường

| Kiểm soát | dev / test / non-prod | staging | prod |
|---|---|---|---|
| Scan (SAST/SCA/image) | Chạy trong CI, **fail build nếu critical** | Như prod | Bắt buộc pass toàn bộ gate |
| Secrets | Secret riêng, giá trị giả/thấp rủi ro | Gần prod, ẩn danh | **Vault + rotation + audit**, ít người truy cập |
| Quyền truy cập | Rộng để dev tự làm | Hạn chế | **Least privilege + JIT + MFA + break-glass** |
| Dữ liệu | Không bao giờ dữ liệu thật | Ẩn danh hoá / masked | Dữ liệu thật, mã hoá at-rest & in-transit |
| Audit log | Cơ bản | Đầy đủ | **Bất biến, lưu lâu, phục vụ compliance** |

**Nguyên tắc:** Việc *quét* nên giống nhau ở mọi môi trường (shift-left, rẻ nhất khi bắt lỗi sớm). Việc *kiểm soát truy cập & bảo vệ dữ liệu* thì tăng dần theo độ nhạy: lỏng ở dev, nghiêm ngặt nhất ở prod. **Không bao giờ** đưa dữ liệu thật xuống non-prod.

---

## 08. 🚑 Reliability & Operations (SRE)

Giữ hệ thống chạy đáng tin cậy và biết cách phục hồi khi hỏng. Mảng này gần như **chỉ thực sự áp dụng cho prod** — nhưng staging là nơi tập dượt mọi kịch bản.

### Các hạng mục con

- **SLI / SLO / SLA** — Định nghĩa "đủ tốt", error budget quyết định tốc độ release.
- **Backup & DR** — Backup tự động, restore test định kỳ, RTO/RPO, multi-region failover.
- **Incident management** — On-call rotation, runbook, severity levels, postmortem *blameless*.
- **Scaling & capacity** — Auto-scaling, load balancing, capacity planning, performance budget.
- **Chaos engineering** — Chaos Monkey, Litmus, Gremlin — chủ động tiêm lỗi để kiểm chứng khả năng chịu lỗi.
- **Automation / chống toil** — Tự động hoá việc lặp lại, self-healing, auto-remediation.

### Ánh xạ theo môi trường

| Hoạt động | dev / test | staging | prod |
|---|---|---|---|
| SLO / error budget | Không | Theo dõi để so chuẩn | **Bắt buộc**, chi phối tốc độ deploy |
| Backup & DR | Không / tối thiểu | Test quy trình restore | Backup tự động + **diễn tập DR định kỳ** |
| On-call | Không | Không (giờ hành chính) | **24/7 rotation**, runbook, escalation |
| Chaos testing | Không | **Nơi lý tưởng để chạy** chaos an toàn | Game day có kiểm soát (nâng cao) |
| Load / perf test | Đơn vị nhỏ | **Load test chính thức** (k6, JMeter, Gatling) | Chỉ khi cần, có kiểm soát |

> **Vai trò của staging:** Tồn tại chính là để **tập dượt cho prod** — chạy load test, thử chaos, diễn tập rollback và restore, kiểm chứng runbook. Mọi thứ đau đớn nên xảy ra lần đầu ở staging, không phải prod.

---

## 09. 🧪 Testing & Quality

Chất lượng được kiểm chứng tự động ở từng tầng. Kim tự tháp test quyết định cái gì chạy ở đâu: nhiều test nhanh & rẻ ở dev/CI, ít test đắt & thực tế ở staging.

### Kim tự tháp test — loại test chạy ở đâu

| Loại test | Mục đích | Chạy chủ yếu ở | Công cụ |
|---|---|---|---|
| **Unit** | Logic từng hàm/lớp | dev / CI (mỗi commit) | Jest, JUnit, pytest, Go test |
| **Integration** | Nhiều thành phần + DB/queue | CI · test | Testcontainers, Postman/Newman |
| **Contract** | Giao kèo giữa các service | test | Pact |
| **E2E / UI** | Luồng người dùng thật | test · staging | Playwright, Cypress, Selenium |
| **Performance / Load** | Chịu tải, độ trễ | staging | k6, JMeter, Gatling, Locust |
| **Security** | Lỗ hổng | CI · staging | OWASP ZAP, Snyk, Trivy |
| **Smoke / Sanity** | Kiểm tra nhanh sau deploy | staging · prod (post-deploy) | Health check, synthetic monitoring |
| **Chaos / Resilience** | Chịu lỗi hạ tầng | staging | Litmus, Gremlin, Chaos Mesh |

> **Quy tắc kim tự tháp:** Càng xuống môi trường gần prod, test càng *thực tế nhưng càng đắt và chậm* → chạy càng ít. Bắt lỗi càng sớm (dev/CI) càng rẻ. Ở prod chỉ nên có smoke test / synthetic monitoring nhẹ sau deploy, **tuyệt đối không** chạy load/chaos vô kiểm soát trên người dùng thật.

---

## 10. 🤝 Culture & Process

Mảng "vô hình" nhưng nền tảng nhất — DevOps sinh ra là một **văn hoá** trước khi là công cụ. Không có văn hoá đúng thì công cụ xịn đến mấy cũng thất bại.

### Các hạng mục con

- **Cộng tác Dev–Ops** — Phá bỏ silo, "you build it, you run it", đội cross-functional sở hữu trọn vòng đời.
- **Automation-first** — Bất cứ việc làm tay lần thứ ba đều nên tự động hoá. Chống toil.
- **Feedback loop nhanh** — Rút ngắn thời gian từ commit đến biết kết quả; deploy nhỏ & thường xuyên.
- **DORA metrics** — Deploy frequency, Lead time, Change failure rate, MTTR — thước đo sức khoẻ DevOps.
- **Blameless culture** — Postmortem không đổ lỗi, học từ sự cố, tâm lý an toàn để báo cáo lỗi.
- **Docs & knowledge** — Runbook, ADR, wiki, onboarding — kiến thức không nằm trong đầu một người.

### 4 chỉ số DORA — la bàn đo trưởng thành DevOps

| Chỉ số | Ý nghĩa | Elite team |
|---|---|---|
| **Deployment Frequency** | Tần suất deploy lên prod | Nhiều lần/ngày (on-demand) |
| **Lead Time for Changes** | Từ commit đến chạy ở prod | < 1 giờ |
| **Change Failure Rate** | % deploy gây sự cố | 0–15% |
| **MTTR** | Thời gian phục hồi sau sự cố | < 1 giờ |

> **Vì sao quan trọng nhất:** Toàn bộ mô hình môi trường ở trên chỉ hoạt động khi có **văn hoá** tôn trọng nó — không ai "sửa nóng" thẳng lên prod, không ai bỏ qua staging vì "gấp", mọi thay đổi đều qua pipeline. Công cụ ép được một phần, nhưng phần lớn là kỷ luật tập thể.

---

## ⭐ Ma trận công cụ × môi trường

Bảng "một trang" để tra nhanh. **Ghi nhớ:** cột công cụ thường giống nhau — thứ đổi là cấu hình, quy mô và kiểm soát ở hai cột phải.

| Mảng | Công cụ tiêu biểu (dùng chung mọi env) | Non-prod (dev/test) | Prod |
|---|---|---|---|
| Source Control | Git, GitHub/GitLab | Branch protection nhẹ, dev tự merge | Signed tag, CODEOWNERS, approval |
| CI/CD | GitHub Actions/GitLab CI, Argo CD | Auto deploy, không gate | Canary/blue-green, approval, auto-rollback |
| IaC | Terraform/OpenTofu, Ansible | Apply linh hoạt, tài nguyên nhỏ, policy = warn | Chỉ qua pipeline, policy = block, locking |
| Container/K8s | Docker, Kubernetes, Helm | kind/k3d, 1 replica, tag mutable | Managed K8s, HPA, digest đã ký, multi-AZ |
| Cloud/Network | AWS/GCP/Azure, Terraform | Account non-prod, single-AZ, spot | Account prod tách biệt, multi-AZ, WAF, reserved |
| Observability | Prometheus, Grafana, Loki, OTel | Log DEBUG, retention ngắn, không on-call | Sampling, retention dài, on-call 24/7, SLO alert |
| Security | Trivy, Snyk, Vault, OPA | Scan trong CI, secret giả, quyền rộng | Vault+rotation, least-privilege, audit bất biến |
| Reliability | PagerDuty, k6, Litmus | Không SLO/DR (staging: load & chaos) | SLO, DR drill, on-call, error budget |
| Testing | pytest/Jest, Playwright, k6 | Unit/integration/E2E theo kim tự tháp | Smoke + synthetic monitoring sau deploy |
| Culture | DORA, postmortem, runbook | Thử nghiệm tự do | Change management, blameless postmortem |

---

## 🗺️ Lộ trình học

Đây *thực sự* là một chuỗi có thứ tự — mỗi bước là nền cho bước sau.

1. **Git & CI/CD cơ bản** — Nền tảng của mọi tự động hoá. Bắt đầu với GitHub Actions.
2. **Docker** — Đóng gói nhất quán, điều kiện để có parity.
3. **Một cloud (AWS)** — Hiểu compute, network, IAM, managed services.
4. **IaC (Terraform)** — Tạo cả 4 môi trường từ cùng một code.
5. **Kubernetes** — Orchestration ở quy mô lớn, Helm, GitOps.
6. **Observability & Security** — Prometheus/Grafana + shift-left security.

> **Lời khuyên cuối:** Đừng cố học hết cùng lúc. Dựng một project nhỏ rồi thêm dần từng mảng: Git → CI test → Docker → deploy lên 1 môi trường → tách dev/prod → thêm monitoring. Học qua làm dính hơn học qua đọc rất nhiều.
