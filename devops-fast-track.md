# DevOps / Kubernetes Fast-Track — Kế hoạch phát triển nhanh nhất

> Lộ trình biến bạn từ "biết dùng" thành **người OWN toàn bộ k8s** trong ~90 ngày học có kỷ luật.
> Nguyên tắc xuyên suốt: **hiểu mental model → tự phá & tự sửa → own một thứ thật → dạy lại người khác.**
> Đọc kèm: [`k8s-operations-playbook.md`](./k8s-operations-playbook.md) · [`k8s-production-templates.md`](./k8s-production-templates.md) · [`cheatsheet.md`](./cheatsheet.md)

## 📑 Mục lục
- [Phần 0. Sự thật về "learn fast"](#phần-0-sự-thật-về-learn-fast)
- [Phần 1. Mental models (hiểu là tự suy ra được)](#phần-1-mental-models-hiểu-là-tự-suy-ra-được)
- [Phần 2. Break & Fix Labs (đòn bẩy #1)](#phần-2-break--fix-labs-đòn-bẩy-1)
- [Phần 3. Kế hoạch 90 ngày](#phần-3-kế-hoạch-90-ngày)
- [Phần 4. Capstone Project (chứng minh năng lực)](#phần-4-capstone-project-chứng-minh-năng-lực)
- [Phần 5. Kỹ năng biến Senior → Lead](#phần-5-kỹ-năng-biến-senior--lead)
- [Phần 6. Tự đánh giá — dấu hiệu bạn đã OWN](#phần-6-tự-đánh-giá--dấu-hiệu-bạn-đã-own)
- [Phần 7. Meta-skills: cách học nhanh gấp đôi](#phần-7-meta-skills-cách-học-nhanh-gấp-đôi)

---

## Phần 0. Sự thật về "learn fast"

Trước khi bắt đầu, 5 sự thật quyết định tốc độ của bạn:

1. **Tốc độ đến từ chiều sâu, không phải chiều rộng.** Học 1 thứ tới mức dạy được > lướt qua 10 thứ. Người "biết nhiều thứ nông" mãi là mid-level.
2. **Bạn học bằng cách phá, không phải bằng cách đọc.** Đọc = ảo giác hiểu. Chỉ khi cluster cháy trong tay bạn và bạn sửa được, kiến thức mới thành của bạn.
3. **Ownership = chịu trách nhiệm khi nó hỏng.** Nhận on-call cho một thứ thật là cú nhảy lớn nhất. Không có áp lực thật, không có tăng tốc thật.
4. **Feedback loop càng ngắn càng nhanh.** Local cluster (kind/k3s) reset trong 30 giây → bạn dám thử nghiệm điên rồ → học nhanh gấp 10.
5. **Dạy lại là nén 2 lần.** Viết blog/tài liệu/giảng cho đồng đội buộc bạn hiểu tới tận gốc. (Chính bộ tài liệu bạn đang xây là cách này.)

**Công thức mỗi ngày:** `1h đọc/xem lý thuyết → 2h lab tay → 15 phút viết lại điều học được.`

---

## Phần 1. Mental models (hiểu là tự suy ra được)

Nắm 5 mô hình này, bạn không cần học vẹt hành vi của từng resource nữa — bạn tự suy ra.

### 1.1 Reconciliation Loop — trái tim của K8s
```text
Bạn khai báo DESIRED STATE (YAML "tôi muốn 3 replica").
Controller chạy vòng lặp vô hạn:
   observe (thực tế đang mấy replica?) -> diff -> act (tạo/xóa cho khớp) -> lặp lại
```
Hệ quả bạn TỰ suy ra được:
- Xóa 1 pod của Deployment → nó mọc lại (controller kéo về desired). Muốn xóa hẳn phải sửa desired (scale 0 / delete deployment).
- `kubectl apply` = "đây là desired mới", controller tự làm phần còn lại. Đây là **declarative**, khác hẳn "chạy lệnh tuần tự" (imperative).
- Mọi thứ trong k8s đều là controller theo mẫu này: Deployment, ReplicaSet, HPA, cả ArgoCD. Hiểu 1 cái = hiểu tất cả.

### 1.2 Mọi thứ đi qua API Server + etcd
```text
kubectl / controller / kubelet  <->  API Server  <->  etcd (nguồn sự thật duy nhất)
```
- `kubectl get pod` = hỏi API Server, không hỏi node. → API Server chết thì `kubectl` mù, dù pod vẫn chạy.
- etcd = **nguồn sự thật duy nhất** của cả cluster. → Vì sao backup etcd là ưu tiên #1.
- Controller & kubelet dùng **watch** (không poll) để biết thay đổi tức thì.

### 1.3 Luồng một gói tin (networking — chỗ yếu của hầu hết mọi người)
```text
Pod A  --(CNI: mỗi pod 1 IP phẳng)-->  Pod B
Client --> Service (ClusterIP ảo) --> kube-proxy (iptables/IPVS) --> 1 pod backend bất kỳ
Internet --> Ingress Controller (nginx) --> Service --> Pod
DNS: tên "svc.ns.svc.cluster.local" -> CoreDNS -> ClusterIP
```
Hệ quả:
- Service là **ảo** (không có process) — chỉ là rule iptables do kube-proxy dựng. → "Service không kết nối" thường là endpoint rỗng (pod chưa Ready) hoặc iptables/CNI, không phải "service chết".
- Pod chưa Ready → không vào endpoint → Service không route tới. Readiness probe điều khiển việc này.

### 1.4 Scheduling — vì sao pod nằm ở node này
```text
Scheduler chọn node dựa trên: requests (đủ chỗ?) + affinity/taint/toleration + spread.
```
- `requests` là "đặt chỗ" scheduler dùng để tính, KHÔNG phải mức dùng thực. → Đặt sai requests là gốc của nửa số sự cố scheduling & lãng phí cost.

### 1.5 QoS & Eviction — ai bị giết trước khi thiếu tài nguyên
```text
Guaranteed (requests==limits)  > Burstable (có requests<limits)  > BestEffort (không đặt gì)
Node thiếu RAM/disk -> kubelet evict BestEffort trước, Guaranteed cuối cùng.
```
→ Muốn workload quan trọng "sống sót", đặt nó là Guaranteed.

> **Bài tập củng cố:** với mỗi mô hình trên, tự viết ra 3 sự cố mà nó giải thích. Nếu viết được là bạn đã thấm.

---

## Phần 2. Break & Fix Labs (đòn bẩy #1)

**Đây là phần quan trọng nhất của cả tài liệu.** Dựng cluster local, cố tình phá, tự chẩn đoán bằng playbook, tự sửa. Mỗi lab: gây lỗi → quan sát triệu chứng → tự chẩn đoán (KHÔNG xem đáp án ngay) → sửa → viết root cause.

### Setup môi trường (5 phút)
```bash
# kind = k8s trong Docker, reset trong 30s — lý tưởng để phá
kind create cluster --name lab                 # hoặc: k3d / minikube
kubectl cluster-info
# Deploy 1 app mẫu để có cái mà phá:
kubectl create deploy web --image=nginx --replicas=3
kubectl expose deploy web --port=80
```

### Lab 1 — CrashLoopBackOff
```bash
# Phá: cho container thoát ngay
kubectl create deploy crash --image=busybox -- /bin/sh -c "exit 1"
# Nhiệm vụ: dùng describe + logs --previous tìm ra vì sao, đọc exit code.
# Học được: quy trình chẩn đoán pod crash, ý nghĩa exit code.
```

### Lab 2 — OOMKilled
```bash
# Phá: đặt limit RAM tí xíu rồi ngốn RAM
kubectl run oom --image=polinux/stress --limits=memory=20Mi -- \
  stress --vm 1 --vm-bytes 100M --vm-hang 1
# Nhiệm vụ: xác nhận reason=OOMKilled, exit 137. Sửa: tăng limit đúng mức.
```

### Lab 3 — ImagePullBackOff
```bash
kubectl create deploy badimg --image=nginx:doesnotexist-9999
# Nhiệm vụ: đọc Events, phân biệt "not found" vs "unauthorized". Sửa: đúng tag.
```

### Lab 4 — Pod Pending (thiếu tài nguyên)
```bash
# Phá: yêu cầu nhiều CPU hơn cả node có
kubectl run huge --image=nginx --requests=cpu=100
# Nhiệm vụ: describe thấy "Insufficient cpu". Học: requests vs capacity node.
```

### Lab 5 — DNS hỏng (CoreDNS)
```bash
# Phá: scale CoreDNS về 0
kubectl scale deploy coredns -n kube-system --replicas=0
kubectl run t --rm -it --image=busybox:1.28 -- nslookup web   # sẽ fail
# Nhiệm vụ: chẩn đoán "vì sao resolve fail", tìm ra CoreDNS chết. Sửa: scale lại.
```

### Lab 6 — Service không kết nối (endpoint rỗng)
```bash
# Phá: đổi selector của service cho lệch label pod
kubectl patch svc web -p '{"spec":{"selector":{"app":"khong-ton-tai"}}}'
kubectl get endpoints web        # -> rỗng
# Nhiệm vụ: hiểu vì sao "service sống nhưng không route". Sửa: đúng selector.
```

### Lab 7 — Readiness fail (pod chạy nhưng 0/1)
```bash
# Phá: readiness trỏ tới path không tồn tại
kubectl set probe deploy/web --readiness --get-url=http://:80/khong-co --period=5 2>/dev/null \
  || kubectl edit deploy web   # thêm readinessProbe path sai
# Nhiệm vụ: pod Running nhưng READY 0/1, biến mất khỏi endpoints.
```

### Lab 8 — Rollout hỏng & rollback
```bash
kubectl set image deploy/web web=nginx:doesnotexist   # deploy bản lỗi
kubectl rollout status deploy/web                       # kẹt
kubectl rollout undo deploy/web                         # CẦM MÁU
# Học: phản xạ rollback trước, điều tra sau.
```

### Lab 9 — Node đầy disk / drain
```bash
kubectl cordon <node>                       # chặn schedule
kubectl drain <node> --ignore-daemonsets    # đẩy pod đi (xem PDB tác động)
kubectl uncordon <node>
# Học: quy trình bảo trì node an toàn.
```

### Lab 10 — etcd backup & restore (level cao, dùng kind/kubeadm)
```bash
# Thực hành đúng runbook trong k8s-production-templates.md mục H.
# Đây là bài tập "đáng giá nhất" — làm được là bạn tự tin cứu cluster thật.
```

### Lab 11 — Chaos có hệ thống
```bash
# Tự động phá ngẫu nhiên để kiểm tra khả năng chịu lỗi:
# - Cài Chaos Mesh hoặc dùng litmus
# - kill pod ngẫu nhiên, thêm network latency, ngốn CPU node
# Học: hệ thống có tự hồi phục không? PDB/replica/probe có đủ không?
```

> **Quy tắc vàng của lab:** KHÔNG google đáp án trong 15 phút đầu. Vật lộn tự chẩn đoán chính là lúc bạn học. Sau đó mới đối chiếu playbook.

---

## Phần 3. Kế hoạch 90 ngày

Mỗi tuần: mục tiêu rõ + lab tay + sản phẩm đầu ra (viết lại / commit code).

### Tháng 1 — Vững nền tảng & vận hành (Operate)
| Tuần | Trọng tâm | Lab / Sản phẩm |
|------|-----------|----------------|
| 1 | Mental models (Phần 1) + Linux/networking cơ bản | Viết lại 5 mô hình bằng lời của bạn |
| 2 | Core objects: Pod/Deployment/Service/Ingress/ConfigMap/Secret | Deploy 1 app 3-tier bằng YAML tay |
| 3 | **Break & Fix Lab 1–8** | Viết root cause cho từng lab |
| 4 | Storage (PV/PVC/StorageClass) + StatefulSet | Deploy Postgres có PVC, backup/restore dump |

### Tháng 2 — Xây dựng & Tự động hóa (Build)
| Tuần | Trọng tâm | Lab / Sản phẩm |
|------|-----------|----------------|
| 5 | **Helm** + **Kustomize** | Đóng gói app thành Helm chart |
| 6 | **GitOps ArgoCD** (app-of-apps) | Repo GitOps tự sync toàn bộ app |
| 7 | **CI/CD** (GitHub Actions): build→test→scan→push→deploy | Pipeline tự động deploy khi merge |
| 8 | **IaC Terraform**: dựng cluster + hạ tầng từ code | `terraform apply` ra 1 cluster thật |

### Tháng 3 — Tin cậy, Bảo mật & Quy mô (Reliability & Scale)
| Tuần | Trọng tâm | Lab / Sản phẩm |
|------|-----------|----------------|
| 9 | **Observability**: kube-prometheus-stack + Grafana + Loki | Dashboard + alert cho app |
| 10 | **Reliability**: probes/PDB/HPA/SLO + **Break & Fix 9–11** | Chaos test, app tự hồi phục |
| 11 | **Security**: RBAC/NetworkPolicy/PodSecurity/Trivy/kube-bench | Hardening + scan cluster |
| 12 | **Backup/DR + etcd restore (Lab 10)** + Capstone | DR drill thành công |

> Không nhất thiết đúng 90 ngày — tùy thời gian bạn có. Quan trọng là **thứ tự** (Operate → Build → Scale) và **mỗi tuần có sản phẩm**.

---

## Phần 4. Capstone Project (chứng minh năng lực)

Một dự án cá nhân "end-to-end" giá trị hơn mọi chứng chỉ khi phỏng vấn Lead. Mục tiêu: **dựng lại toàn bộ chỉ bằng git push.**

```text
Đề bài: "Platform tự phục vụ cho 1 app web + API + DB"

Yêu cầu (tick dần):
[ ] Cluster dựng bằng Terraform (không click tay) — EKS/GKE hoặc k3s trên VPS
[ ] App đóng gói Helm, cấu hình qua values theo môi trường (dev/staging/prod)
[ ] GitOps: ArgoCD app-of-apps, mọi thay đổi qua Git PR
[ ] CI/CD: push code -> build -> test -> scan (Trivy) -> sign -> ArgoCD tự deploy
[ ] Ingress + TLS tự động (cert-manager + Let's Encrypt)
[ ] Observability: Prometheus + Grafana dashboard + alert (5xx, latency, RAM)
[ ] Reliability: HPA, PDB, probes, NetworkPolicy default-deny
[ ] Backup: Velero theo lịch + CronJob dump DB -> S3, ĐÃ test restore
[ ] DR runbook: xóa sạch cluster -> dựng lại từ Git trong < 1h
[ ] README + kiến trúc + ADR giải thích lý do chọn từng thứ

Điểm cộng cực lớn: quay video "phá cluster rồi khôi phục từ Git".
```

Đây chính là bản mô tả công việc của một DevOps/Platform Lead. Làm xong = bạn đã own.

---

## Phần 5. Kỹ năng biến Senior → Lead

Giỏi kỹ thuật đưa bạn tới Senior. **Nhân bản năng lực ra cả team** mới đưa tới Lead.

- **Viết ADR (Architecture Decision Record):** mỗi quyết định lớn ghi lại *bối cảnh — lựa chọn — trade-off — hệ quả*. Giúp team hiểu "tại sao", tránh tranh cãi lại và người sau kế thừa được.
- **Runbook cho mọi alert:** alert → link runbook → các bước xử lý. Biến "chỉ mình bạn cứu được" thành "ai trực cũng xử lý được". Giảm bus-factor.
- **Incident command:** khi sự cố, ai điều phối, ai điều tra, ai liên lạc. Blameless postmortem → action item phòng ngừa có người & deadline.
- **Golden path / self-service:** dựng template + CI/CD sẵn để dev tự deploy an toàn, không phụ thuộc ops. Đây là công việc *platform*, đòn bẩy cao nhất.
- **Mentoring & review:** dạy lại, review thiết kế, nâng chuẩn cả team. Đo lường thành công bằng việc team làm được gì khi vắng bạn.
- **Giao tiếp ngược lên:** dịch rủi ro kỹ thuật thành ngôn ngữ business (rủi ro, chi phí, thời gian) để thuyết phục đầu tư đúng chỗ.
- **Cân đối 3 lực:** tốc độ (feature) ↔ độ tin cậy (SLO/error budget) ↔ chi phí (FinOps). Lead là người ra quyết định đánh đổi có cơ sở.

---

## Phần 6. Tự đánh giá — dấu hiệu bạn đã OWN

Đừng đo bằng "đọc bao nhiêu". Đo bằng những dấu hiệu này:

**Vận hành:**
- [ ] Có alert lúc 2h sáng, bạn biết mở runbook nào và xử lý không hoảng.
- [ ] Nhìn `kubectl get events` là đoán được chuyện gì, không cần google.
- [ ] Tự tin `drain` node production giữa giờ cao điểm (vì có PDB + replica).
- [ ] Đã từng **restore etcd/backup thật** và cluster sống lại.

**Xây dựng:**
- [ ] Xóa sạch cluster và dựng lại hoàn toàn từ Git/Terraform.
- [ ] Thêm 1 service mới end-to-end (code → CI → GitOps → prod) trong 1 buổi.

**Chiều sâu:**
- [ ] Giải thích được cho người mới: "gói tin đi từ browser tới pod qua những gì".
- [ ] Khi thấy sự cố lạ, bạn suy luận từ mental model chứ không thử mò.
- [ ] Dạy lại được — viết được tài liệu như bộ này cho người sau.

**Lead:**
- [ ] Team xử lý được sự cố khi bạn đi vắng (nhờ runbook bạn viết).
- [ ] Quyết định kỹ thuật của bạn có ADR, người khác hiểu và tin.

> Khi tick được đa số → bạn không còn "học k8s", bạn **là người người khác hỏi về k8s.**

---

## Phần 7. Meta-skills: cách học nhanh gấp đôi

- **Feedback loop cực ngắn:** luôn có 1 cluster local reset trong 30s để thử mọi thứ ngay.
- **Học theo sự cố, không theo mục lục:** thay vì đọc tuần tự, mỗi khi gặp lỗi → đào tận gốc lỗi đó. Kiến thức bám vào tình huống thật thì nhớ lâu.
- **Đọc source & docs gốc, không chỉ blog:** khi bí, đọc official docs + `kubectl explain <resource>` + code controller. Blog cho tốc độ, source cho chiều sâu.
- **Rubber-duck / dạy lại ngay:** sau mỗi lab, giải thích thành tiếng như đang dạy. Chỗ nào lắp bắp là chỗ chưa hiểu.
- **Spaced repetition cho lệnh/khái niệm:** lệnh hay quên → cho vào cheatsheet + alias, dùng thật nhiều lần.
- **Ghi lại mọi sự cố bạn xử lý:** một "incident journal" cá nhân. Sau 6 tháng đọc lại, bạn sẽ thấy mình đã trưởng thành tới đâu.
- **Vào cộng đồng:** đọc KEP (Kubernetes Enhancement Proposals), theo dõi CNCF, kubernetes slack. Ở gần người giỏi hơn để chuẩn của bạn tự nâng lên.
- **Kỷ luật > cảm hứng:** 2h mỗi ngày đều đặn trong 90 ngày thắng 20h dồn 1 ngày rồi bỏ. Đặt lịch cố định, coi như cuộc hẹn không hủy.

---

### Lời cuối
Không có đường tắt bỏ qua **hands-on** và **trách nhiệm thật**. Nhưng có đường nhanh: **hiểu mental model để không học vẹt, phá cluster để hiểu sâu, own một thứ thật để chịu áp lực, và dạy lại để nén kiến thức.** Làm đúng bốn điều đó, 90 ngày của bạn bằng 2 năm của người chỉ đọc.
