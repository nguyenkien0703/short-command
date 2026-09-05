# Nguồn tài liệu ôn CKA & CKS

> Tổng hợp nguồn học, ưu tiên **cộng đồng Việt Nam** (devops.vn, Viblo) rồi tới quốc tế.
> Kèm nhận xét thực tế: cái nào đáng tiền, cái nào bỏ qua được.

---

## 1. Chính chủ CNCF / Linux Foundation ⭐ (đọc đầu tiên)

| Nguồn | Link | Dùng để |
|---|---|---|
| **Curriculum PDF** | [github.com/cncf/curriculum](https://github.com/cncf/curriculum) | **Nguồn sự thật duy nhất** về domain & trọng số. File hiện hành: `CKA_Curriculum_v1.35.pdf`, `CKS_Curriculum v1.34.pdf` |
| Trang cert CKA | [cncf.io/training/certification/cka](https://www.cncf.io/training/certification/cka/) | Giá, thời lượng, chính sách |
| Trang cert CKS | [cncf.io/training/certification/cks](https://www.cncf.io/training/certification/cks/) | Điều kiện CKA, chính sách |
| **Candidate Handbook** | Link trong trang cert | ⭐ Danh sách **domain docs được phép mở** trong phòng thi — đọc lại trước ngày thi vì hay đổi |
| Exam FAQ | Link trong trang cert | Version K8s của môi trường thi, quy định phòng thi |
| K8s Docs | [kubernetes.io/docs](https://kubernetes.io/docs/) | Tài liệu duy nhất được mở khi thi |
| CVE feed chính thức | [kubernetes.io/docs/reference/issues-security/official-cve-feed](https://kubernetes.io/docs/reference/issues-security/official-cve-feed/) | CKS — biết version nào có lỗ hổng gì |

> 🔴 **Curriculum PDF quan trọng hơn mọi khóa học.** Trọng số trên trang web CNCF đôi khi
> lệch với PDF (ví dụ CKS: web ghi Cluster Setup 10%, PDF v1.34 ghi 15%). **Tin PDF.**

---

## 2. Cộng đồng Việt Nam 🇻🇳

### devops.vn
| Bài | Link | Nội dung chính |
|---|---|---|
| **Chia sẻ lộ trình thực tế chinh phục CKA** | [devops.vn/posts/chinh-phuc-chung-chi-cka-certified-kubernetes-administrator](https://devops.vn/posts/chinh-phuc-chung-chi-cka-certified-kubernetes-administrator/) | Lộ trình cá nhân, nguồn học, mẹo học nhanh (tóm tắt transcript video bằng AI), nhấn mạnh **kiểm tra kỹ** — tác giả được 78% dù làm hết câu |
| **Làm thế nào tôi chinh phục Golden Kubestronaut?** | [devops.vn/posts/lam-the-nao-toi-chinh-phuc-golden-kubestronaut](https://devops.vn/posts/lam-the-nao-toi-chinh-phuc-golden-kubestronaut/) | Toàn cảnh 5 cert Kubestronaut + các cert mở rộng. Nhận xét CKS là "khó nhất nhưng thú vị nhất". Chi phí bundle |
| **Các chứng chỉ Kubernetes** | [devops.vn/posts/cac-chung-chi-kubernetes](https://devops.vn/posts/cac-chung-chi-kubernetes/) | So sánh CKA/CKAD/CKS: đối tượng, thời lượng, điểm đậu, giá |
| Articles hub | [devops.vn/articles](https://devops.vn/articles/) | Bài mới về hạ tầng/DevOps/Cloud |
| Learning Hub | [devops.vn/learning-hub](https://devops.vn/learning-hub/) | Series học theo chủ đề |
| Weekly newsletter | [devops.vn/weekly](https://devops.vn/weekly/) | Cập nhật hệ sinh thái |

**Rút ra từ devops.vn:**
- Người có lab thật cần **~1 tháng** cho CKA; người mới cần 2–3 tháng.
- Chỉ dùng **1 khóa chính + luyện đề**, đừng ôm nhiều tài liệu.
- CKS: phải **tự dựng kịch bản tấn công** mới hiểu, không chỉ học lệnh.
- KCSA (nếu định lấy) đừng coi thường phần lý thuyết — tỉ lệ trượt cao.

### Viblo
| Bài | Link | Nội dung chính |
|---|---|---|
| **Series: Luyện thi chứng chỉ CKA** | [viblo.asia/s/3kY4gDbkJAe](https://viblo.asia/s/3kY4gDbkJAe) | Series nhiều phần: review kỳ thi, review nguồn tài liệu, **các dạng bài kèm gợi ý lời giải** |
| **Review kỳ thi CKA và kinh nghiệm xương máu** | [viblo.asia/p/cka-review-ky-thi-cka-va-kinh-nghiem-xuong-mau-rut-ra-sau-khi-thi-2oKLn2oaLQO](https://viblo.asia/p/cka-review-ky-thi-cka-va-kinh-nghiem-xuong-mau-rut-ra-sau-khi-thi-2oKLn2oaLQO) | ⭐ Bài đáng đọc nhất về **môi trường thi**: check-in 3 lần mất 50', ổ điện hỏng, camera đơ, Firefox không dùng được Ctrl+F, phím copy-paste, alias `$do`/`$now` |
| **Hành trình chinh phục chứng chỉ CKA 2025** | [viblo.asia/p/hanh-trinh-chinh-phuc-chung-chi-cka-2025-1j4lQe3DJwl](https://viblo.asia/p/hanh-trinh-chinh-phuc-chung-chi-cka-2025-1j4lQe3DJwl) | ⭐ **Thay đổi CKA 2025**: Helm, Kustomize, Gateway API, CRD, CNI/DNS/Ingress nâng cao. Khuyến nghị hoàn thành trong 70–75' để có thời gian dò |
| **Chia sẻ kinh nghiệm thi CKA năm 2024** | [viblo.asia/p/chia-se-kinh-nghiem-thi-chung-chi-cka-nam-2024-EbNVQwy0JvR](https://viblo.asia/p/chia-se-kinh-nghiem-thi-chung-chi-cka-nam-2024-EbNVQwy0JvR) | Đạt **93/100** lần đầu. Mẹo: 3 tab terminal (soạn YAML / verify / SSH node), alias, mua voucher sale 50% |
| **Tôi đã chinh phục bài thi CKA ngay lần đầu như thế nào?** | [viblo.asia/p/chinh-phuc-bai-thi-cka-ngay-lan-dau-nhu-the-nao-yZjJYjzgLOE](https://viblo.asia/p/chinh-phuc-bai-thi-cka-ngay-lan-dau-nhu-the-nao-yZjJYjzgLOE) | ⭐ Chi tiết nhất về **chiến thuật phòng thi**: quét đề trước, không viết YAML từ đầu, tìm docs bằng `kind:`, quy định phòng thi, săn Cyber Monday |

### blog.devopsviet.com
| Bài | Link | Nội dung chính |
|---|---|---|
| **Hướng dẫn học chứng chỉ CKS A–Z** | [blog.devopsviet.com/2023/12/17/huong-dan-hoc-chung-chi-certified-kubernetes-security-specialist-a-z](https://blog.devopsviet.com/2023/12/17/huong-dan-hoc-chung-chi-certified-kubernetes-security-specialist-a-z/) | ⭐ Tài liệu CKS tiếng Việt đầy đủ nhất: lộ trình 12 tuần, review từng nguồn học, quy trình check-in, chi phí thực tế |

> ⚠️ Bài viết cộng đồng có thể **lỗi thời** về trọng số/giá/curriculum
> (ví dụ: bài 2023–2024 chưa có Gateway API, ghi hiệu lực 3 năm thay vì 2).
> Dùng chúng cho **kinh nghiệm thi**, đối chiếu **curriculum PDF** cho nội dung.

---

## 3. Khóa học & lab (quốc tế)

### Bắt buộc — chọn 1 khóa chính
| Nguồn | Giá | Nhận xét |
|---|---|---|
| **Udemy: CKA with Practice Tests** (Mumshad Mannambeth / KodeKloud) | ~$15 khi sale | ⭐ Khóa được cộng đồng VN khuyên nhiều nhất. Lab bám sát đề thi. **Kiểm tra bản đã cập nhật Helm/Kustomize/Gateway API chưa** |
| **KodeKloud CKA/CKS** | ~$200/năm | Lab trên trình duyệt, 3 mock test mỗi cert. Đắt hơn nhưng lab tốt hơn Udemy |
| **Linux Foundation LFS258 (CKA) / LFS260 (CKS)** | Thường bán bundle với exam | Chính chủ, đầy đủ nhưng khô. Mua bundle course+exam đôi khi rẻ hơn mua exam riêng |

### Bắt buộc — luyện đề
| Nguồn | Giá | Nhận xét |
|---|---|---|
| **killer.sh** | ⭐ **Miễn phí** khi mua exam (2 session × 36h) | **Khó hơn đề thật rõ rệt**. Session 1 được 60% là bình thường. Đọc kỹ solution — đó mới là giá trị chính |
| **Killercoda** | Miễn phí | [killercoda.com/killer-shell-cka](https://killercoda.com/killer-shell-cka) · [/cks](https://killercoda.com/killer-shell-cks) — ~40 scenario theo chủ đề, làm mỗi cái **2–3 lần** |
| **KodeKloud CKS Challenges** | Miễn phí | Sát đề CKS nhất trong các nguồn miễn phí |

### Tham khảo thêm
| Nguồn | Nhận xét |
|---|---|
| [github.com/techiescamp/cka-certification-guide](https://github.com/techiescamp/cka-certification-guide) | Syllabus + lệnh tổng hợp |
| [github.com/techiescamp/cks-certification-guide](https://github.com/techiescamp/cks-certification-guide) | Tương tự cho CKS |
| [github.com/walidshaari/Kubernetes-Certified-Administrator](https://github.com/walidshaari/Kubernetes-Certified-Administrator) | Kho link tổng hợp lâu đời |
| [k8s-security.geek-kb.com](https://k8s-security.geek-kb.com/) | Theo dõi **thay đổi curriculum CKS** theo thời gian |
| [devopscube.com/cks-exam-guide-tips](https://devopscube.com/cks-exam-guide-tips/) | Hướng dẫn CKS chi tiết |

> 💡 Lời khuyên nhất quán từ cộng đồng VN: **đừng ôm quá nhiều tài liệu.**
> 1 khóa chính + Killercoda + killer.sh là đủ để đạt điểm cao.

---

## 4. Docs của tool (CKS)

| Tool | Docs | Ghi chú |
|---|---|---|
| **Trivy** | [trivy.dev/latest/docs](https://trivy.dev/latest/docs/) | Có trong danh sách được phép của CKS |
| **Falco** | [falco.org/docs](https://falco.org/docs/) | Được phép |
| **AppArmor** | [gitlab.com/apparmor/apparmor/-/wikis/Documentation](https://gitlab.com/apparmor/apparmor/-/wikis/Documentation) | Được phép |
| **etcd** | [etcd.io/docs](https://etcd.io/docs/) | Được phép |
| **Cilium** | [docs.cilium.io](https://docs.cilium.io/) | Được phép |
| kube-bench | [github.com/aquasecurity/kube-bench](https://github.com/aquasecurity/kube-bench) | Tool tự in remediation |
| kubesec | [kubesec.io](https://kubesec.io/) | |
| KubeLinter | [docs.kubelinter.io](https://docs.kubelinter.io/) | |
| Kyverno | [kyverno.io/docs](https://kyverno.io/docs/) | |
| OPA Gatekeeper | [open-policy-agent.github.io/gatekeeper](https://open-policy-agent.github.io/gatekeeper/) | |
| gVisor | [gvisor.dev/docs](https://gvisor.dev/docs/) | |

> ⚠️ Danh sách domain được phép **thay đổi theo đợt**. Kiểm tra **Candidate Handbook**
> ngay trước ngày thi, đừng tin danh sách này (hay bất kỳ blog nào) một cách tuyệt đối.

---

## 5. Môi trường lab

| Cách | Ưu | Nhược | Đủ cho |
|---|---|---|---|
| **2 VM Ubuntu + kubeadm** ⭐ | Giống môi trường thi nhất | Cần 4 vCPU / 8GB tổng | **CKA + CKS đầy đủ** |
| Killercoda | Miễn phí, không cài gì | Session 1 giờ, mất khi hết | Luyện scenario nhanh |
| kind (`kind create cluster`) | Nhẹ, dựng trong 1 phút | ❌ Không có kubeadm upgrade, etcd restore thật, AppArmor | Workload, Service, Storage |
| minikube | Dễ dùng | ❌ 1 node, thiếu nhiều thứ | Workload cơ bản |
| Cloud VM (EC2/GCE) | Mạnh, linh hoạt | Tốn tiền | CKA + CKS đầy đủ |

**Cluster kind 3 node (cho phần workload/network):**
```yaml
# 3-node.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
networking:
  disableDefaultCNI: false
```
```bash
kind create cluster --name cka --config 3-node.yaml
```

**Cluster kubeadm (bắt buộc cho CKA §Cluster Architecture và toàn bộ CKS):**
xem [cka/01 §3](./cka/01-cluster-architecture.md#3-kubeadm--dựng--join-cluster).

---

## 6. Chi phí & săn voucher 💰

| Khoản | Giá niêm yết | Giá thực tế nếu săn sale |
|---|---|---|
| CKA exam | $445 | ~$220–310 |
| CKS exam | $445 | ~$220–310 |
| Udemy course | $80–100 | **~$15** (Udemy sale gần như liên tục) |
| KodeKloud | ~$200/năm | Có sale Black Friday |
| Killercoda | Miễn phí | — |
| killer.sh | Miễn phí kèm exam | — |

**Mẹo tiết kiệm:**
1. **Cyber Monday** (thứ Hai cuối tháng 11) — Linux Foundation giảm sâu nhất năm (~30–50%).
   Cộng đồng VN ghi nhận có năm mua CKA chỉ ~$198.
2. **Mua voucher trước, thi sau** — voucher thường có hạn 12 tháng. Không cần sẵn sàng mới mua.
3. **Bundle course + exam** đôi khi rẻ hơn mua exam đơn lẻ.
4. **Bundle Kubestronaut / Golden Kubestronaut** nếu định lấy nhiều cert.
5. Mỗi exam đã **bao gồm 1 lần thi lại miễn phí** — đừng mua thêm.

---

## 7. Thứ tự đọc đề xuất

```text
1. Curriculum PDF (CNCF)                 ← biết chính xác thi cái gì
2. exam-day-playbook.md (repo này)       ← biết luật chơi
3. Viblo: "Review kỳ thi CKA..."          ← biết môi trường thi thật ra sao
4. Viblo: "Hành trình CKA 2025"           ← biết cái gì mới
5. Khóa học chính (Udemy/KodeKloud)      ← học kiến thức
6. certs/k8s/cka/*.md (repo này)          ← hệ thống hóa + tra nhanh
7. Killercoda scenarios                   ← lab theo chủ đề
8. practice-questions.md (repo này)       ← tự kiểm tra
9. killer.sh ×2                           ← chốt hạ
```

---

## 8. Ghi chú của riêng bạn

> Mỗi lần đọc được mẹo hay, hoặc thi/lab gặp bẫy mới → thêm vào đây.
> Đây là phần **giá trị nhất** của file này về lâu dài.

| Ngày | Nguồn | Ghi chú |
|---|---|---|
| | | |
