# Playbook Ngày Thi — CKA & CKS

> Đọc file này **trước khi học kiến thức**. Rất nhiều người trượt/hụt điểm không phải vì thiếu
> kiến thức mà vì: mất 50 phút check-in, không quen môi trường, không biết copy-paste, gõ YAML tay.
>
> Tổng hợp từ kinh nghiệm thi thật của cộng đồng VN (devops.vn, Viblo) + tài liệu chính chủ.

---

## 1. Trước ngày thi — Chuẩn bị vật lý

### Phòng thi
| Yêu cầu | Chi tiết |
|---|---|
| Phòng riêng | Không ai được vào trong suốt 2 tiếng. Proctor nhìn qua camera. |
| Bàn sạch | **Trống hoàn toàn**: không giấy, bút, sách, điện thoại, tai nghe, đồng hồ, cốc có hình. |
| Tường | Gỡ tranh/poster/giấy dán. Tường sơn trơn là OK. |
| Camera an ninh | ⚠️ **Có người bị bắt tháo camera an ninh trong phòng** mới cho thi. Kiểm tra trước. |
| Nước uống | Chỉ **cốc thủy tinh/nhựa trong suốt, không nhãn**. Uống trước khi thi cho chắc. |
| Ánh sáng | Đủ sáng để proctor thấy rõ mặt bạn. |

### Thiết bị
- **Chỉ 1 màn hình.** Dùng màn rời thì phải **gập màn laptop lại**. Màn ≥ 15", khuyến nghị ≥ 23"
  (đề chia đôi màn hình: bên trái câu hỏi, bên phải terminal — màn nhỏ rất khổ).
- **Webcam rời** tốt hơn camera laptop nhiều — check-in phải quay quanh phòng, gầm bàn, mặt bàn.
  Camera laptop khiến bạn phải bê cả máy đi vòng quanh (dễ mất kết nối).
- **Mạng dây (LAN)** thay vì WiFi. Có người bị rớt mạng giữa lúc proctor đang hướng dẫn.
- Kiểm tra **ổ điện** hoạt động — có người phải đổi phòng vì ổ điện hỏng.
- Chạy **system check** của PSI trước ít nhất 1 ngày.

### Giấy tờ
- **Hộ chiếu** hoặc **CCCD** — tên phải **khớp chính xác** với tên đăng ký thi. Sai một chữ là bị từ chối.

### Timeline ngày thi
```text
T-60 phút : Dọn phòng, test mạng/điện/camera, uống nước, đi WC.
T-30 phút : Mở link thi, bắt đầu check-in (hệ thống cho vào sớm 30 phút).
T-15..0   : Proctor kiểm tra ID + quay phòng. Dự phòng 30-50 phút cho khâu này.
T+0       : Đồng hồ 120 phút bắt đầu chạy — CHỈ khi bài thi thực sự mở.
```

> 🔴 **Bài học xương máu**: có người check-in 3 lần mới vào được, tốn 50+ phút.
> Đồng hồ 120 phút **không bị trừ** vì check-in, nhưng bạn sẽ mất bình tĩnh nghiêm trọng.
> Vào sớm 30 phút, coi như bảo hiểm.

---

## 2. Môi trường thi

```text
┌────────────────────────────┬───────────────────────────────────┐
│  KHUNG CÂU HỎI (trái)      │  REMOTE DESKTOP (phải)            │
│                            │                                   │
│  - Đề bài                  │  - Terminal (bash)                │
│  - Nút copy context cmd    │  - Firefox (tra docs)             │
│  - Nút flag câu hỏi        │  - Editor: vim / nano             │
│  - Notepad ghi chú         │                                   │
└────────────────────────────┴───────────────────────────────────┘
```

**Điểm cần biết:**
- Mỗi câu có **context riêng** (`kubectl config use-context ...`). Đề cho sẵn nút copy —
  **luôn bấm copy và chạy trước khi làm câu đó**. Làm sai context = 0 điểm dù đáp án đúng.
- Câu hỏi có thể **flag để quay lại sau**. Dùng triệt để.
- Có **notepad** trong giao diện đề — ghi số câu chưa chắc chắn vào đó.
- Đề cho biết câu đó **bao nhiêu %** — làm câu điểm cao trước.

### Copy / Paste (hay bị vướng nhất)
| Ngữ cảnh | Phím |
|---|---|
| Copy trong terminal | `Ctrl + Shift + C` |
| Paste vào terminal | `Ctrl + Shift + V` |
| Paste vào Firefox | `Shift + Insert` (hoặc `Ctrl + V`) |
| Copy từ khung đề → terminal | Dùng nút copy sẵn có; nếu bôi đen thì `Ctrl+C` rồi `Ctrl+Shift+V` |

> ⚠️ Có kỳ thi báo cáo **Firefox trong remote desktop không dùng được `Ctrl+F`**.
> Không phải lúc nào cũng vậy, nhưng **hãy chuẩn bị tinh thần phải tra docs bằng cách nhớ đường**,
> không phụ thuộc tìm kiếm trong trang.

---

## 3. Setup 60 giây đầu tiên ⭐

Việc **đầu tiên** khi bài thi mở — gõ ngay khối này (thuộc lòng, gõ trong ~40 giây):

```bash
# 1. Alias & biến tiết kiệm thời gian
alias k=kubectl
export do="--dry-run=client -o yaml"      # tạo YAML mẫu
export now="--force --grace-period=0"     # xóa ngay lập tức
export ow="-o wide"

# 2. Autocomplete cho alias k (RẤT quan trọng)
source <(kubectl completion bash)
complete -o default -F __start_kubectl k

# 3. Vim: 2 space, không tab — YAML sai indent là hỏng hết
cat <<'EOF' > ~/.vimrc
set expandtab
set tabstop=2
set shiftwidth=2
set number
set ai
EOF
```

**Cách dùng:**
```bash
k run nginx --image=nginx $do > pod.yaml      # sinh YAML mẫu rồi sửa
k delete pod nginx $now                        # xóa không chờ
k get po $ow                                   # xem thêm IP/node
```

> 💡 `$do` và `$now` là quy ước chuẩn của cộng đồng CKA — gần như ai đi thi cũng dùng.
> Không viết YAML từ đầu bao giờ. Luôn `--dry-run=client -o yaml` rồi sửa.

### Vim survival kit (đủ để sống sót trong phòng thi)
| Việc | Phím |
|---|---|
| Vào insert | `i` · Thoát insert | `Esc` |
| Lưu & thoát | `:wq` · Thoát không lưu | `:q!` |
| Xóa dòng | `dd` · Xóa 5 dòng | `5dd` |
| Copy dòng / paste | `yy` / `p` |
| Undo / redo | `u` / `Ctrl+r` |
| Nhảy đầu / cuối file | `gg` / `G` |
| Tìm | `/text` → `n` (tiếp) |
| **Indent khối** | Bôi đen `V` + `j/k`, rồi `>` (thụt vào) hoặc `<` |
| Bật/tắt số dòng | `:set number` / `:set nonumber` |
| **Paste không bị lệch indent** | `:set paste` → paste → `:set nopaste` |

> 🔴 Bẫy kinh điển: paste YAML vào vim bị **auto-indent dồn bậc thang**.
> Luôn `:set paste` trước khi paste khối lớn.

### 3 tab terminal (chiến thuật của người thi 93/100)
```text
Tab 1: soạn/apply YAML
Tab 2: kiểm tra kết quả (k get / k describe / k logs)
Tab 3: ssh vào node (kubeadm, systemctl, /etc/kubernetes/...)
```
Nhớ `exit` khỏi SSH và quay về `candidate-node` sau mỗi câu — nếu không câu sau sẽ chạy nhầm máy.

---

## 4. Chiến thuật làm bài

### Quản lý thời gian
```text
120 phút / ~17 câu ≈ 7 phút/câu.

Phút 0-1    : setup alias/vim (mục 3)
Phút 1-5    : LƯỚT toàn bộ đề, đánh dấu câu dễ/khó, note % điểm
Phút 5-85   : làm câu dễ + điểm cao trước. Câu nào >8 phút chưa xong → FLAG, bỏ qua
Phút 85-110 : quay lại câu đã flag
Phút 110-120: KIỂM TRA LẠI. Đặc biệt: đúng namespace? đúng context? resource đã Running?
```

**Mục tiêu: xong lượt 1 trong 70–75 phút**, để dành 45 phút dò lại.
Người đạt 93/100 xong 17 câu còn dư 50 phút. Người đạt 78/100 làm hết câu nhưng
*"không kiểm tra kỹ"* — chênh lệch nằm ở khâu verify, không phải kiến thức.

### Nguyên tắc vàng
1. **Đọc kỹ namespace.** Đề rất hay bảo tạo trong namespace lạ. Quên `-n` = mất trọn điểm câu đó.
2. **Chạy context command trước mỗi câu.** Luôn luôn. Không có ngoại lệ.
3. **Không viết YAML tay.** `$do` sinh mẫu → sửa. Nhanh hơn 3–4 lần và không sai cú pháp.
4. **Không cầu toàn.** Điểm đậu 66% (CKA) / 67% (CKS). Bỏ hẳn 3–4 câu vẫn đậu.
   Đừng chết chìm ở 1 câu khó trong khi 3 câu dễ chưa làm.
5. **Verify sau mỗi câu** — 20 giây thôi:
   ```bash
   k get <resource> -n <ns>        # có tồn tại không
   k describe <res> | tail -20     # Events có lỗi không
   k get po -n <ns>                # Running hay CrashLoop
   ```
6. **Câu nào không làm được thì để nguyên**, đừng xóa/phá cluster — có thể ảnh hưởng câu khác.
7. **Task nhiều phần → làm được phần nào chấm phần đó** (chấm theo tiêu chí). Luôn làm phần dễ trước.

### Tra docs hiệu quả
- Được phép: **`kubernetes.io/docs`, `kubernetes.io/blog`** và subdomain (CKA).
  CKS mở thêm: **Trivy, Falco, AppArmor, etcd, Cilium** docs (xem [cks/00-exam-guide.md](./cks/00-exam-guide.md)).
- ❌ **Kustomize docs (`kubectl.docs.kubernetes.io`) KHÔNG được phép** trong CKA →
  phải **thuộc** cú pháp `kustomization.yaml`.
- Mẹo tìm nhanh: search theo **`kind:`** — ví dụ gõ `kind: NetworkPolicy` để nhảy thẳng tới ví dụ YAML,
  thay vì đọc cả trang lý thuyết.
- Chỉ mở **1 tab**, tránh lạc sang domain khác (proctor có thể cảnh cáo).
- **Trước ngày thi, bookmark trong đầu** đường đi tới 10 trang hay dùng nhất
  (xem checklist ở cuối file).

---

## 5. Cứu hộ khi có sự cố

| Sự cố | Xử lý |
|---|---|
| Rớt mạng giữa chừng | Đồng hồ **vẫn chạy**. Kết nối lại ngay, chat với proctor. Đây là lý do phải dùng LAN. |
| Camera đơ | Restart trình duyệt PSI. Proctor sẽ chờ. |
| Terminal treo | Mở tab terminal mới; session cluster không mất. |
| Proctor gọi giữa lúc đang gõ | Dừng, trả lời qua chat. Đừng phớt lờ — có thể bị hủy bài thi. |
| Không hiểu đề | Đọc lại **2 lần**. Đề viết ngắn gọn nhưng chính xác từng chữ ("create" vs "update", "expose" vs "create service"). |
| Làm hỏng cluster | Đa số câu độc lập context. Nếu phá 1 cluster, chuyển context sang câu khác làm tiếp. |

---

## 6. Sau khi thi

- Kết quả trả trong **~24 giờ** (trước đây là 36h) qua email + portal Linux Foundation.
- **Có 1 lần thi lại miễn phí** kèm theo lệ phí. Trượt lần 1 không phải thảm họa —
  nhưng lần 2 nên cách ít nhất 2–3 tuần để vá lỗ hổng thật.
- Đậu → nhận cert PDF + badge Credly. Hiệu lực **2 năm**.
- CKA đậu → mở khóa quyền đăng ký CKS.

---

## 7. Checklist "thuộc lòng trước khi thi"

### CKA — phải gõ được không cần tra docs
- [ ] `kubeadm upgrade plan` / `apply` — quy trình upgrade control-plane rồi node
- [ ] Backup & restore **etcd** (`etcdctl snapshot save/restore` + 3 cờ cert)
- [ ] Sinh mọi resource bằng `$do`: pod, deploy, svc, job, cronjob, cm, secret, sa
- [ ] RBAC: `k create role/rolebinding/clusterrole/clusterrolebinding` bằng lệnh imperative
- [ ] `k drain` / `cordon` / `uncordon` + `--ignore-daemonsets --delete-emptydir-data`
- [ ] NodeSelector, node affinity, taint/toleration
- [ ] PV / PVC / StorageClass — accessModes, reclaimPolicy
- [ ] NetworkPolicy — cú pháp `podSelector` + `ingress`/`egress` (viết được từ đầu)
- [ ] **Gateway API**: `GatewayClass` / `Gateway` / `HTTPRoute`
- [ ] **Helm**: `repo add/update`, `install`, `upgrade`, `template`, `-f values.yaml`
- [ ] **Kustomize**: `kustomization.yaml` (resources, namespace, images, patches) — ⚠️ docs bị chặn
- [ ] `k top node/pod`, `k logs --previous`, `k get events --sort-by=.lastTimestamp`
- [ ] JSONPath & custom-columns cơ bản

### CKS — phải gõ được không cần tra docs
- [ ] NetworkPolicy default-deny ingress **và** egress
- [ ] Pod Security Admission — 3 label trên namespace (`enforce/audit/warn` × `privileged/baseline/restricted`)
- [ ] `securityContext` đầy đủ: `runAsNonRoot`, `runAsUser`, `readOnlyRootFilesystem`,
      `allowPrivilegeEscalation: false`, `capabilities.drop: [ALL]`
- [ ] AppArmor: profile + annotation/field trên Pod
- [ ] seccomp: `RuntimeDefault` + custom profile ở `/var/lib/kubelet/seccomp/profiles/`
- [ ] Audit policy YAML + 4 cờ `--audit-*` trên kube-apiserver
- [ ] Falco: đọc/sửa rule, tìm log
- [ ] `trivy image --severity HIGH,CRITICAL <image>`
- [ ] gVisor/Kata: `RuntimeClass` + `runtimeClassName`
- [ ] ImagePolicyWebhook: AdmissionConfiguration + `--admission-control-config-file`
- [ ] `kube-bench run --targets=master` và đọc `[FAIL]`
- [ ] Sửa manifest static pod trong `/etc/kubernetes/manifests/` và biết chờ apiserver restart

---

## 8. Sai lầm khiến mất điểm oan (tổng hợp từ người đã thi)

| # | Sai lầm | Cách tránh |
|---|---|---|
| 1 | Quên `-n <namespace>` | Đọc đề gạch chân namespace trước khi gõ |
| 2 | Quên chạy context command | Bấm nút copy đề cho, dán, Enter — thành phản xạ |
| 3 | Viết YAML từ đầu → sai indent, hết giờ | Luôn `$do` |
| 4 | Paste vào vim bị dồn indent | `:set paste` trước khi paste |
| 5 | Sửa static pod manifest xong không chờ apiserver lên | `watch k get po -n kube-system` |
| 6 | SSH vào node xong quên `exit` | Câu sau chạy nhầm máy → làm sai hết |
| 7 | Làm xong không verify | 20 giây `k get`/`describe` sau mỗi câu |
| 8 | Sa lầy 20 phút vào 1 câu | Quá 8 phút → flag, đi tiếp |
| 9 | Không đọc kỹ "create" vs "modify" | Đọc đề 2 lần |
| 10 | Không set alias/autocomplete đầu giờ | 40 giây đầu tư, tiết kiệm 15 phút |
