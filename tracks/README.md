# Specialization Tracks — SRE · DevSecOps · MLOps

> Phân tích sâu 3 hướng chuyên môn hoá từ DevOps, và **điểm hội tụ** dành riêng cho người đang làm
> **DevOps cho dự án AI** (như bạn). Đọc để chọn hướng có chủ đích.

## 📂 Trong folder này
- [mlops.md](./mlops.md) — Vận hành hệ thống ML/AI (gần bạn nhất)
- [sre.md](./sre.md) — Site Reliability Engineering (độ tin cậy ở quy mô)
- [devsecops.md](./devsecops.md) — Bảo mật dịch chuyển vào toàn vòng đời
- **[roadmap-ai-platform-engineer.md](./roadmap-ai-platform-engineer.md)** — 🎯 **Lộ trình phân lớp** MLOps→SRE→DevSecOps + LLMOps vs MLOps (bắt đầu ở đây để định hình)
- [ml-serving-k8s.md](./ml-serving-k8s.md) — 🔧 Kỹ thuật: **serving ML trên K8s (KServe/GPU)** — thế mạnh khác biệt của bạn
- [llmops-end-to-end.md](./llmops-end-to-end.md) — 🔧 Kỹ thuật: **LLMOps end-to-end** (RAG + vLLM + eval + guardrails)

## So sánh nhanh 3 hướng

| Tiêu chí | **SRE** | **DevSecOps** | **MLOps** |
|----------|---------|---------------|-----------|
| Câu hỏi cốt lõi | "Hệ thống có đáng tin không?" | "Hệ thống có an toàn không?" | "Model có được huấn luyện/triển khai/giám sát đúng không?" |
| Đơn vị quan tâm | SLO, error budget, incident | Threat, vulnerability, compliance | Data, model, experiment, drift |
| Nền tảng gốc | Software eng + Ops | Security + DevOps | DevOps + Data/ML |
| Độ hiếm nhân lực | Trung bình | Cao | **Rất cao (đang bùng nổ)** |
| Đòn bẩy với nền DevOps của bạn | Cao | Cao | **Rất cao** |
| Độ liên quan tới AI của bạn | Gián tiếp | Gián tiếp (đang nổi lên: AI security) | **Trực tiếp** |

## Điểm mấu chốt cho BẠN (DevOps cho AI)

Bạn không cần chọn "một trong ba" theo kiểu loại trừ. Với bối cảnh làm AI, ba hướng **hội tụ thành một vị thế hiếm và giá trị cao**:

```text
        MLOps (lõi - bạn đang gần nhất)
       /                              \
  SRE cho ML                     Security cho ML
  (reliability của              (MLSecOps: model/data/
   hệ thống ML,                  LLM security, governance)
   serving, GPU, drift)
       \                              /
        =>  "AI Platform / ML Infrastructure Engineer"
            (hồ sơ cực hiếm, lương cao, đúng làn sóng)
```

**Đề xuất lộ trình cho bạn:**
1. **Lấy MLOps làm trục chính** (gần bạn nhất, liên quan trực tiếp AI, hiếm nhất) — [mlops.md](./mlops.md).
2. **Bổ sung SRE** cho phần vận hành ML ở quy mô (serving, GPU, latency, drift = reliability) — [sre.md](./sre.md).
3. **Thêm DevSecOps** ở khía cạnh AI (model/data governance, LLM security, supply chain) — [devsecops.md](./devsecops.md).

Kết quả: bạn trở thành người **xây và vận hành platform AI end-to-end đáng tin & an toàn** — thứ mọi công ty làm AI đang thiếu trầm trọng.

> Ba file dưới đây: mỗi file có mindset → concept lõi → kỹ năng (đã có vs cần) → tool → **liên hệ AI** → lộ trình → thị trường/đòn bẩy → giao thoa với 2 hướng kia.
