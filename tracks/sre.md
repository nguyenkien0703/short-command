# SRE — Site Reliability Engineering

> SRE = "vận hành như một bài toán phần mềm" (Google khởi xướng). Thay vì lo thủ công, SRE dùng
> **kỹ thuật + dữ liệu** để làm hệ thống đáng tin ở quy mô. Cốt lõi: **cân bằng độ tin cậy với tốc độ
> đổi mới bằng error budget** — biến "reliability" từ cảm tính thành con số quản trị được.

## 1. Mindset — điều phân biệt SRE với "Ops"

```text
Ops truyền thống: hệ thống hỏng -> con người sửa (toil tăng theo scale -> không bền).
SRE:              đo lường độ tin cậy -> tự động hóa -> loại toil -> hệ thống tự lành.
                  Chấp nhận "100% uptime là sai mục tiêu" -> đặt mục tiêu ĐỦ TỐT (SLO)
                  và dùng phần "được phép lỗi" (error budget) để đổi mới nhanh.
```
Nguyên tắc Google:
- **Toil budget**: giới hạn việc tay lặp lại < ~50% thời gian; phần còn lại làm engineering để giảm toil.
- **Error budget**: nếu còn budget → được release nhanh/mạo hiểm; hết budget → freeze feature, tập trung ổn định. Đây là **hợp đồng giữa Dev và Ops**, chấm dứt tranh cãi "nhanh vs ổn định".
- **Blameless postmortem**: sự cố là lỗi hệ thống, không phải người.

## 2. Concept lõi — SLI / SLO / SLA / Error Budget

```text
SLI (Indicator) : số ĐO thực tế. VD: tỷ lệ request thành công, latency p99, availability.
SLO (Objective) : MỤC TIÊU nội bộ cho SLI. VD: 99.9% request thành công trong 30 ngày.
SLA (Agreement) : cam kết với KHÁCH HÀNG (có phạt tiền). Luôn LỎNG hơn SLO.
Error Budget    : 100% - SLO. VD SLO 99.9% -> budget 0.1% = ~43 phút/tháng được phép lỗi.
```
**Chọn SLI tốt (điều nhiều người làm sai):** đo cái **user cảm nhận** (request thành công, đủ nhanh), không đo cái vô nghĩa với user (CPU %). Công thức phổ biến: `SLI = sự kiện tốt / tổng sự kiện`.

**Alert theo burn rate** (nâng cao): thay vì alert mỗi lỗi lẻ, alert khi **tốc độ đốt error budget** cao (vd đốt 2% budget trong 1h) → giảm nhiễu, cảnh báo đúng lúc quan trọng.

## 3. Các trụ cột kỹ năng SRE

| Trụ cột | Nội dung |
|---------|----------|
| **Observability** | metrics/logs/traces, dashboard, SLO tracking (Prometheus/Grafana, OpenTelemetry) |
| **Incident management** | phát hiện → triage (severity) → mitigate → resolve → **postmortem** blameless; on-call, runbook |
| **Capacity planning** | dự báo tải, load test, tránh cạn tài nguyên; autoscaling |
| **Reliability design** | redundancy, graceful degradation, circuit breaker, retry/backoff, bulkhead, blast radius nhỏ |
| **Toil reduction / automation** | tự động hóa việc lặp; self-healing |
| **Chaos engineering** | chủ động tiêm lỗi (kill pod/node, latency) để kiểm chứng khả năng chịu lỗi (Chaos Mesh, Litmus, Gremlin) |
| **Release engineering** | canary, progressive delivery, rollback tự động, feature flag |
| **Performance engineering** | tìm & xử lý bottleneck, tối ưu latency |

## 4. Kỹ năng: bạn ĐÃ CÓ vs CẦN THÊM
**Đã có (nền DevOps):** K8s, observability, CI/CD, IaC, automation, xử lý sự cố (playbook bạn đã viết chính là tư duy SRE!).

**Cần bổ sung:**
```text
- Tư duy SLO/error budget: định lượng reliability, làm việc với Product để đặt mục tiêu.
- Coding tốt hơn (SRE thiên về software): viết tool/automation/operator, không chỉ script.
- Capacity planning & performance engineering có phương pháp (không đoán).
- Chaos engineering: kiểm chứng giả định về khả năng chịu lỗi.
- Incident command: điều phối sự cố lớn, không chỉ tự sửa.
- Toán reliability cơ bản: MTBF/MTTR, phần trăm uptime <-> thời gian, tail latency.
```

## 5. Liên hệ AI/ML (vì bạn làm AI) — "SRE cho ML"
Hệ thống ML có bài toán reliability RIÊNG, rất cần SRE:
```text
- SLO cho ML: không chỉ latency/uptime mà cả CHẤT LƯỢNG (accuracy/quality) — model degrade
  là "sự cố" dù hệ thống vẫn "up".
- Serving reliability: GPU capacity, batching, cold start model lớn, timeout inference.
- Drift = incident: data/concept drift làm model kém dần -> cần monitor + alert + retrain.
- Chi phí GPU = ràng buộc capacity (GPU đắt -> autoscale/scale-to-zero thông minh).
- Rollback model: version model xấu -> rollback nhanh (giống rollback deploy).
```
→ Kết hợp SRE + MLOps = **"ML Reliability Engineer"**, hồ sơ rất hiếm.

## 6. Lộ trình học
```text
1. Đọc "SRE Book" + "SRE Workbook" (Google, miễn phí online) — nền tư duy.
2. Áp dụng SLO vào 1 service thật: chọn SLI, đặt SLO, dựng dashboard error budget.
3. Xây observability đầy đủ: metrics/logs/traces + alert theo burn rate.
4. Thực hành incident: viết runbook, tổ chức game day (chaos), viết postmortem.
5. Capacity & performance: load test (k6), tìm bottleneck, autoscale đúng.
6. Nâng coding: viết 1 operator/controller hoặc tool tự động hóa toil.
```

## 7. Thị trường & đòn bẩy
- SRE là chuẩn vàng vận hành ở công ty lớn (Google/Meta/Netflix và mọi scale-up).
- Chức danh: SRE, Reliability Engineer, Production Engineer, Platform/SRE.
- **Đòn bẩy Staff/Principal cao**: SLO & reliability là ngôn ngữ lãnh đạo kỹ thuật quan tâm (gắn tech với business: uptime = tiền/uy tín).
- Nền tảng tuyệt vời để lên **Principal SRE / Reliability Architect**.

## 8. Giao thoa với MLOps & DevSecOps
- **↔ MLOps:** reliability của serving/pipeline ML, SLO chất lượng model, drift-as-incident. Xem [mlops.md](./mlops.md).
- **↔ DevSecOps:** security incident cũng là reliability event; postmortem & incident process dùng chung; "resilience" bao gồm cả chịu lỗi lẫn chịu tấn công.

## Tóm tắt
- SRE biến reliability thành **con số quản trị được** (SLI/SLO/error budget), cân bằng ổn định ↔ tốc độ.
- Nền DevOps + playbook sự cố của bạn đã là hạt giống SRE; cần thêm tư duy SLO + coding + chaos + incident command.
- Với AI: "SRE cho ML" (SLO chất lượng, drift, GPU capacity) là ngách hiếm & giá trị.
