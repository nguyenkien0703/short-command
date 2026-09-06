# CKS — Supply Chain Security (20%) 🔴

**Nội dung curriculum v1.34:**
- Minimize base image footprint
- Understand your supply chain (e.g. **SBOM**, CI/CD, artifact repositories)
- Secure your supply chain (**permitted registries**, sign and validate artifacts, etc.)
- Perform static analysis of user workloads and container images (e.g. **Kubesec, KubeLinter**)

---

## 1. Giảm base image footprint

**Nguyên tắc:** image càng nhỏ → càng ít package → càng ít CVE → attack surface càng nhỏ.

### Bậc thang image (từ to đến nhỏ)
```text
ubuntu:22.04         ~ 78 MB   — đủ shell, package manager  ⚠️ nhiều CVE
debian:bookworm-slim ~ 75 MB
alpine:3.20          ~  8 MB   — musl libc, có shell (busybox)
gcr.io/distroless/*  ~  2-20 MB — KHÔNG shell, không package manager
scratch              ~  0 MB   — rỗng hoàn toàn, chỉ chạy static binary
```

### Multi-stage build ⭐
```dockerfile
# --- Stage 1: build ---
FROM golang:1.23 AS builder
WORKDIR /src
COPY . .
RUN CGO_ENABLED=0 go build -o /app ./cmd/server

# --- Stage 2: runtime (chỉ copy binary) ---
FROM gcr.io/distroless/static-debian12:nonroot
COPY --from=builder /app /app
USER nonroot:nonroot
ENTRYPOINT ["/app"]
```

### Checklist Dockerfile an toàn — CKS hay bắt sửa
```dockerfile
FROM alpine:3.20                 # ✅ pin version cụ thể, KHÔNG dùng :latest
RUN adduser -D -u 1000 appuser   # ✅ tạo user thường
COPY --chown=appuser app /app
USER 1000                        # ✅ KHÔNG chạy root
EXPOSE 8080                      # ✅ không mở port thừa
ENTRYPOINT ["/app"]
```

| ❌ Sai | ✅ Đúng | Vì sao |
|---|---|---|
| `FROM ubuntu:latest` | `FROM alpine:3.20` | `latest` không tái lập được, ubuntu quá to |
| Không có `USER` | `USER 1000` | Mặc định chạy root |
| `RUN apt-get install curl wget vim` | Chỉ cài cái cần | Tool debug = tool cho attacker |
| `COPY . .` | `COPY app.jar .` + `.dockerignore` | Copy nhầm `.git`, secret |
| `ENV PASSWORD=123` | Dùng Secret của K8s | Secret nằm trong layer image, ai pull cũng đọc được |
| `ADD http://...` | `COPY` + verify checksum | `ADD` tự giải nén, tải URL — rủi ro |
| Nhiều `RUN` | Gộp `RUN a && b && rm -rf /var/lib/apt/lists/*` | Layer thừa vẫn chứa file đã xóa |

> 🔴 **Xóa file ở layer sau KHÔNG xóa nó khỏi image.** `RUN rm secret.txt` ở dòng dưới vẫn
> để lại file ở layer trên — `docker history` / `dive` đọc được.

---

## 2. Trivy — scan image tìm CVE ⭐⭐

```bash
# Scan cơ bản
trivy image nginx:1.27

# Chỉ HIGH và CRITICAL (dùng nhiều nhất trong đề)
trivy image --severity HIGH,CRITICAL nginx:1.27

# Chỉ lỗi đã có bản vá
trivy image --ignore-unfixed --severity HIGH,CRITICAL nginx:1.27

# Output gọn / máy đọc được
trivy image --format table nginx:1.27
trivy image --format json -o result.json nginx:1.27
trivy image --quiet --severity CRITICAL --format template \
  --template '{{ range . }}{{ range .Vulnerabilities }}{{ .VulnerabilityID }}{{"\n"}}{{ end }}{{ end }}' nginx:1.27

# Exit code khác 0 khi tìm thấy (dùng trong CI)
trivy image --exit-code 1 --severity CRITICAL nginx:1.27

# Scan các thứ khác
trivy fs /path/to/project            # source code + dependency
trivy config /path/to/Dockerfile     # sai cấu hình IaC
trivy k8s --report summary cluster   # scan cả cluster
trivy repo https://github.com/x/y

# Offline (đề thi thường không có internet)
trivy image --offline-scan --skip-db-update nginx:1.27
```

### Dạng bài kinh điển
> *"Có 4 image trong file `/opt/images.txt`. Tìm image nào KHÔNG có CVE mức CRITICAL
> và giữ lại Deployment dùng image đó; xóa các Deployment còn lại."*

```bash
# Cách làm
for img in $(cat /opt/images.txt); do
  echo "=== $img"
  trivy image --severity CRITICAL --quiet $img | grep -c CRITICAL
done

# Hoặc dùng exit-code
for img in $(cat /opt/images.txt); do
  if trivy image --exit-code 1 --severity CRITICAL --quiet $img >/dev/null 2>&1; then
    echo "$img : SẠCH"
  else
    echo "$img : CÓ CRITICAL"
  fi
done
```

> *"Tìm mọi Pod trong ns `prod` dùng image có CVE CRITICAL và xóa chúng."*
```bash
for p in $(k get po -n prod -o jsonpath='{.items[*].metadata.name}'); do
  img=$(k get po $p -n prod -o jsonpath='{.spec.containers[0].image}')
  trivy image --exit-code 1 --severity CRITICAL -q $img >/dev/null 2>&1 \
    || { echo "XOA $p ($img)"; k delete po $p -n prod; }
done
```

---

## 3. SBOM — Software Bill of Materials

SBOM = danh sách đầy đủ mọi thành phần/thư viện trong một artifact.
Dùng để trả lời nhanh: *"Log4Shell mới ra — image nào của tôi dính?"*

```bash
# Sinh SBOM bằng Trivy
trivy image --format cyclonedx -o sbom.json nginx:1.27
trivy image --format spdx-json -o sbom.spdx.json nginx:1.27

# Scan CVE TỪ SBOM (không cần image nữa)
trivy sbom sbom.json

# Bằng syft (tool chuyên SBOM)
syft nginx:1.27 -o cyclonedx-json > sbom.json
syft nginx:1.27 -o spdx-json
grype sbom:./sbom.json                # grype quét CVE từ SBOM
```

**2 định dạng chuẩn:**
| | SPDX | CycloneDX |
|---|---|---|
| Tổ chức | Linux Foundation | OWASP |
| Thiên về | Compliance, license | Bảo mật, chuỗi cung ứng |

---

## 4. Static analysis — Kubesec & KubeLinter ⭐

### Kubesec — chấm điểm bảo mật manifest
```bash
# Local
kubesec scan pod.yaml

# Qua API online
curl -sSX POST --data-binary @pod.yaml https://v2.kubesec.io/scan

# Qua Docker
docker run -i kubesec/kubesec:v2 scan /dev/stdin < pod.yaml
```

**Output:**
```json
[{
  "object": "Pod/myapp.default",
  "valid": true,
  "score": -30,
  "scoring": {
    "critical": [
      {"id": "Privileged", "selector": "containers[] .securityContext .privileged == true",
       "reason": "Privileged containers can allow almost completely unrestricted host access"}
    ],
    "advise": [
      {"id": "ApparmorAny", "selector": ".metadata .annotations .container.apparmor.security.beta.kubernetes.io/nginx"},
      {"id": "SeccompAny", "selector": ".metadata .annotations .seccomp.security.alpha.kubernetes.io/pod"},
      {"id": "ServiceAccountName", "selector": ".spec .serviceAccountName"}
    ]
  }
}]
```
> **Điểm âm = có vấn đề nghiêm trọng.** Đề hay bảo: *"Sửa manifest để kubesec score > 0"*.
> Cách nâng điểm nhanh: `runAsNonRoot`, `readOnlyRootFilesystem`, `drop ALL caps`,
> `allowPrivilegeEscalation: false`, đặt `resources.limits`, `seccompProfile`.

### KubeLinter
```bash
kube-linter lint pod.yaml
kube-linter lint ./manifests/
kube-linter lint --format json deployment.yaml
kube-linter checks list                        # xem mọi check có sẵn
kube-linter lint --include no-read-only-root-fs pod.yaml
```

### So sánh 3 tool static analysis
| Tool | Quét gì | Output |
|---|---|---|
| **Kubesec** | Manifest K8s | Điểm số + gợi ý |
| **KubeLinter** | Manifest K8s + Helm chart | Danh sách vi phạm |
| **Trivy config** | Dockerfile, Terraform, K8s YAML | Misconfiguration |

---

## 5. Permitted registries — chỉ cho phép registry tin cậy ⭐⭐

### Cách 1 — ImagePolicyWebhook (cách "chính thống" của CKS)

**Bước 1 — file cấu hình admission:**
```yaml
# /etc/kubernetes/admission/admission-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: ImagePolicyWebhook
  configuration:
    imagePolicy:
      kubeConfigFile: /etc/kubernetes/admission/kubeconf.yaml
      allowTTL: 50
      denyTTL: 50
      retryBackoff: 500
      defaultAllow: false        # ← webhook chết thì TỪ CHỐI (an toàn).
                                 #    true = cho qua (không an toàn)
```

**Bước 2 — kubeconfig trỏ tới webhook:**
```yaml
# /etc/kubernetes/admission/kubeconf.yaml
apiVersion: v1
kind: Config
clusters:
- name: image-checker
  cluster:
    certificate-authority: /etc/kubernetes/admission/ca.crt
    server: https://image-checker.default.svc:1323/image_policy
contexts:
- name: default
  context:
    cluster: image-checker
    user: api-server
current-context: default
users:
- name: api-server
  user:
    client-certificate: /etc/kubernetes/admission/apiserver-client.crt
    client-key: /etc/kubernetes/admission/apiserver-client.key
```

**Bước 3 — bật trên apiserver:**
```yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml
- --enable-admission-plugins=NodeRestriction,ImagePolicyWebhook
- --admission-control-config-file=/etc/kubernetes/admission/admission-config.yaml
# + mount volume:
    volumeMounts:
    - name: admission
      mountPath: /etc/kubernetes/admission
      readOnly: true
  volumes:
  - name: admission
    hostPath:
      path: /etc/kubernetes/admission
      type: DirectoryOrCreate
```

> 🔴 3 bẫy của ImagePolicyWebhook:
> 1. **Quên mount volume** → apiserver không lên.
> 2. **`defaultAllow: true`** → webhook chết là mọi image lọt qua. Đề thường bắt đổi thành `false`.
> 3. **Quên thêm `ImagePolicyWebhook` vào `--enable-admission-plugins`** → file config vô nghĩa.

### Cách 2 — OPA Gatekeeper
```yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata: {name: k8sallowedrepos}
spec:
  crd:
    spec:
      names: {kind: K8sAllowedRepos}
      validation:
        openAPIV3Schema:
          type: object
          properties:
            repos: {type: array, items: {type: string}}
  targets:
  - target: admission.k8s.gatekeeper.sh
    rego: |
      package k8sallowedrepos
      violation[{"msg": msg}] {
        container := input.review.object.spec.containers[_]
        satisfied := [good | repo = input.parameters.repos[_]; good = startswith(container.image, repo)]
        not any(satisfied)
        msg := sprintf("container <%v> has an invalid image repo <%v>", [container.name, container.image])
      }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sAllowedRepos
metadata: {name: repo-must-be-trusted}
spec:
  match:
    kinds: [{apiGroups: [""], kinds: ["Pod"]}]
    namespaces: [prod]
  parameters:
    repos: ["registry.internal.com/", "gcr.io/my-project/"]
```

### Cách 3 — Kyverno (đơn giản hơn nhiều, đang phổ biến)
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata: {name: restrict-registries}
spec:
  validationFailureAction: Enforce        # Enforce | Audit
  background: true
  rules:
  - name: validate-registry
    match:
      any:
      - resources:
          kinds: [Pod]
    validate:
      message: "Chỉ được dùng image từ registry.internal.com"
      pattern:
        spec:
          containers:
          - image: "registry.internal.com/*"
---
# Cấm tag :latest
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata: {name: disallow-latest-tag}
spec:
  validationFailureAction: Enforce
  rules:
  - name: require-image-tag
    match: {any: [{resources: {kinds: [Pod]}}]}
    validate:
      message: "Image phải có tag cụ thể, không được dùng :latest"
      pattern:
        spec:
          containers:
          - image: "!*:latest"
```

```bash
k get clusterpolicy
k get policyreport -A
k describe clusterpolicy restrict-registries
```

### Cách 4 — AlwaysPullImages (bổ trợ)
```yaml
- --enable-admission-plugins=AlwaysPullImages,...
```
Ép mọi Pod phải pull image mới → Pod không thể dùng image đã cache trên node mà không có credential.

---

## 6. Ký & verify image (sign/validate artifacts)

```bash
# --- cosign (sigstore) ---
cosign generate-key-pair                        # sinh cosign.key + cosign.pub

cosign sign --key cosign.key registry.io/app:v1
cosign verify --key cosign.pub registry.io/app:v1

# Keyless (dùng OIDC identity, không cần giữ key)
cosign sign registry.io/app:v1
cosign verify registry.io/app:v1 \
  --certificate-identity=user@example.com \
  --certificate-oidc-issuer=https://accounts.google.com

# Attestation (đính kèm SBOM đã ký)
cosign attest --key cosign.key --predicate sbom.json registry.io/app:v1
```

**Bắt buộc verify bằng Kyverno:**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata: {name: verify-image-signature}
spec:
  validationFailureAction: Enforce
  rules:
  - name: verify-signature
    match: {any: [{resources: {kinds: [Pod]}}]}
    verifyImages:
    - imageReferences: ["registry.io/*"]
      attestors:
      - entries:
        - keys:
            publicKeys: |-
              -----BEGIN PUBLIC KEY-----
              ...
              -----END PUBLIC KEY-----
```

**Pin image theo digest** — cách đơn giản nhất để đảm bảo bất biến:
```yaml
# ❌ tag có thể bị đẩy đè
image: nginx:1.27
# ✅ digest không đổi được
image: nginx@sha256:28402db69fec7c17e179ea87882667f1e054391138f77ffaf0c3eb388efc3ffb
```
```bash
# Lấy digest
crictl inspecti nginx:1.27 | grep -i digest
k get po <pod> -o jsonpath='{.status.containerStatuses[*].imageID}'
```

---

## 7. Hiểu chuỗi cung ứng — bức tranh

```text
Dev code ──► Git ──► CI build ──► Registry ──► K8s cluster
    │          │         │            │             │
    ▼          ▼         ▼            ▼             ▼
 SAST      branch    scan dep     scan image    admission
 secret    protect   (trivy fs)   (trivy)       (Kyverno/OPA/
 scan      signed    SBOM         sign(cosign)   ImagePolicyWebhook)
           commit    lint         private repo   PSA
                     (kubesec)                   runtime (Falco)
```

**Mỗi mắt xích có thể bị tấn công:**
| Mắt xích | Tấn công | Phòng thủ |
|---|---|---|
| Source | Commit độc hại, dependency confusion | Signed commit, branch protection, lock file |
| CI/CD | Runner bị chiếm, secret lộ trong log | Ephemeral runner, OIDC thay static key, mask secret |
| Dependency | Typosquatting, package bị chiếm | SBOM, `trivy fs`, pin version + hash |
| Registry | Push image độc hại, tag bị đè | Private registry, ký image, dùng digest |
| Deploy | Manifest bị sửa | GitOps + review, admission policy |
| Runtime | Container escape | PSA, seccomp/AppArmor, Falco |

---

## 8. Dạng bài hay ra

| # | Đề bài | Hướng làm |
|---|---|---|
| 1 | Scan 3 image, tìm cái có CVE CRITICAL, xóa Deployment dùng nó | Mục 2 |
| 2 | Sửa Dockerfile: bỏ root, pin version, gộp RUN | Mục 1 |
| 3 | Chạy kubesec trên manifest, sửa để hết critical | Mục 4 |
| 4 | Bật ImagePolicyWebhook với `defaultAllow: false` | Mục 5.1 |
| 5 | Tạo Kyverno policy chặn image ngoài registry cho phép | Mục 5.3 |
| 6 | Sinh SBOM cho image và lưu ra file | `trivy image --format cyclonedx -o` |
| 7 | Sửa Deployment dùng image theo digest thay vì tag | Mục 6 |
| 8 | Bật admission plugin `AlwaysPullImages` | Mục 5.4 |
| 9 | Tìm Pod nào dùng tag `:latest` và sửa | `k get po -A -o jsonpath` + grep |
| 10 | Xóa mọi Pod dùng image từ registry không tin cậy | Loop + `k get po -o jsonpath` |

**Câu 9/10 — script:**
```bash
# Tìm Pod dùng :latest hoặc không có tag
k get po -A -o jsonpath='{range .items[*]}{.metadata.namespace}{"/"}{.metadata.name}{"\t"}{.spec.containers[*].image}{"\n"}{end}' \
  | grep -E ':latest|\s[^:]+$'

# Tìm Pod dùng registry lạ
k get po -A -o jsonpath='{range .items[*]}{.metadata.namespace}{"/"}{.metadata.name}{"\t"}{.spec.containers[*].image}{"\n"}{end}' \
  | grep -v 'registry.internal.com'
```

---

## 9. Bẫy tổng kết

1. **ImagePolicyWebhook: quên mount volume → apiserver chết.** Backup manifest trước.
2. **`defaultAllow: false`** là câu trả lời an toàn (đề gần như luôn hỏi cái này).
3. **Phải thêm `ImagePolicyWebhook` vào `--enable-admission-plugins`**, không chỉ tạo file config.
4. **Trivy trong phòng thi thường không có internet** — dùng `--skip-db-update` hoặc DB đã cache sẵn.
5. **`trivy image --severity HIGH,CRITICAL`** — viết liền, phân cách bằng dấu phẩy, **không có space**.
6. **Xóa file ở layer sau không xóa khỏi image.**
7. **Secret trong `ENV` của Dockerfile nằm vĩnh viễn trong layer.**
8. **kubesec điểm âm = có vấn đề critical.**
9. **`:latest` là anti-pattern** — CKS luôn coi là lỗi.
10. **Digest (`@sha256:`) bất biến, tag thì không.**
11. **Kyverno `validationFailureAction: Enforce`** mới thực sự chặn; `Audit` chỉ ghi log.
