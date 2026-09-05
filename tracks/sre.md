# SRE — Site Reliability Engineering (🥈 trục bổ trợ)

> SRE = "vận hành như một bài toán phần mềm" (Google khởi xướng). Thay vì lo thủ công, SRE dùng
> **kỹ thuật + dữ liệu** để làm hệ thống đáng tin ở quy mô. Cốt lõi: **cân bằng độ tin cậy với tốc độ
> đổi mới bằng error budget** — biến "reliability" từ cảm tính thành con số quản trị được.
>
> *Cập nhật 09/2026 — đây là trục BỔ TRỢ, bồi sau khi đã vững [devsecops.md](./devsecops.md) (trục chính).
> Xem [tracks/README.md](./README.md) để biết vì sao thứ tự này.*

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

**Alert theo burn rate** — kỹ thuật đáng học nhất trong mảng này. Thay vì alert mỗi lỗi lẻ, alert khi
**tốc độ đốt error budget** vượt ngưỡng. Cấu hình phổ biến là *multi-window multi-burn-rate*:

```text
Burn rate = (tỷ lệ lỗi hiện tại) / (tỷ lệ lỗi cho phép theo SLO)
  burn 14.4x trong cửa sổ 1h  (+ xác nhận bằng cửa sổ 5m)  -> PAGE  (đốt hết ~2% budget/giờ: cháy nhà)
  burn 6x    trong cửa sổ 6h  (+ cửa sổ 30m)               -> PAGE  (chậm hơn nhưng vẫn nguy hiểm)
  burn 1x    trong cửa sổ 3 ngày                            -> TICKET (rò rỉ chậm, xử lý giờ hành chính)
=> Cửa sổ ngắn để phát hiện nhanh, cửa sổ dài để khỏi báo động giả. Đây cũng chính là thuốc chữa
   "alert fatigue" — dùng lại được cho alert bảo mật (xem devsecops.md §9).
```

## 3. Các trụ cột kỹ năng SRE

| Trụ cột | Nội dung |
|---------|----------|
| **Observability** | metrics/logs/traces, dashboard, SLO tracking — chuẩn hoá bằng **OpenTelemetry** (Prometheus/Grafana/Tempo/Loki hoặc SaaS) |
| **Incident management** | phát hiện → triage (severity) → mitigate → resolve → **postmortem** blameless; on-call, runbook, incident command |
| **Capacity planning** | dự báo tải, load test, tránh cạn tài nguyên; autoscaling; **cost** là ràng buộc thật |
| **Reliability design** | redundancy, graceful degradation, circuit breaker, retry/backoff (+ jitter), bulkhead, blast radius nhỏ |
| **Toil reduction / automation** | tự động hóa việc lặp; self-healing; đo tỷ lệ toil |
| **Chaos engineering** | chủ động tiêm lỗi (kill pod/node, latency) để kiểm chứng khả năng chịu lỗi (Chaos Mesh, Litmus, Gremlin) |
| **Release engineering** | canary, progressive delivery, rollback tự động, feature flag |
| **Performance engineering** | tìm & xử lý bottleneck, tail latency, profiling |

## 4. Kỹ năng: bạn ĐÃ CÓ vs CẦN THÊM
**Đã có (nền DevOps):** K8s, observability, CI/CD, IaC, automation, xử lý sự cố ([k8s-operations-playbook.md](../k8s-operations-playbook.md) bạn đã viết chính là tư duy SRE!).

**Cần bổ sung:**
```text
- Tư duy SLO/error budget: định lượng reliability, làm việc với Product để đặt mục tiêu.
- Coding tốt hơn (SRE thiên về software): viết tool/automation/operator, không chỉ script.
- Capacity planning & performance engineering có phương pháp (không đoán).
- Chaos engineering: kiểm chứng giả định về khả năng chịu lỗi.
- Incident command: điều phối sự cố lớn, không chỉ tự sửa.
- Toán reliability cơ bản: MTBF/MTTR, phần trăm uptime <-> thời gian, tail latency, phụ thuộc nối tiếp
  (3 dịch vụ 99.9% mắc nối tiếp -> chỉ còn ~99.7%).
```

## 5. Bối cảnh 2026 — mảng này đang đổi gì

**a) OpenTelemetry là mặc định.** Chuẩn instrumentation trung lập nhà cung cấp đã thắng: thu thập một
lần, đổi backend không phải viết lại code. Nếu học observability từ đầu ở 2026, **học OTel trước**, đừng
học theo agent riêng của một hãng.

**b) Từ "AI hỗ trợ" sang "tự động hoá phản ứng sự cố".** Mẫu hình đang chuẩn hoá: alert → agent đọc
metrics/log/trace + so với sự cố cũ → gợi ý nguyên nhân → với lớp lỗi đã biết thì **chạy runbook đã được
phê duyệt trước**. Gartner dự báo **85% doanh nghiệp dùng AI SRE tooling vào 2029** (dưới 5% năm 2025).
→ **Hệ quả cho bạn, rất thực tế:** giá trị dịch từ "người trực xử lý nhanh" sang **"hệ thống có telemetry
sạch + runbook máy đọc được + hành động remediation an toàn, có phê duyệt & audit"**. Runbook dạng văn
xuôi mơ hồ sẽ mất giá; runbook dạng *điều kiện → hành động → kiểm chứng* thì lên giá.

**c) DORA đổi hướng.** Báo cáo thường niên đổi tên thành *State of AI-assisted Software Development* và
mở rộng bộ 4 metric thành **5**; kết luận đáng nhớ: **AI khuếch đại — tăng throughput, nhưng nếu nền
tảng yếu thì đánh đổi bằng stability**. Số PR merge tăng mạnh mà chỉ số giao hàng của tổ chức không nhúc
nhích → nút thắt nằm ở **hệ thống kỹ thuật (platform, quy trình, review, môi trường test)**, chỗ SRE và
platform engineer làm chủ.

**d) Platform engineering.** Gartner dự báo **80% tổ chức kỹ thuật lớn có team platform trong 2026**.
SRE ngày càng gắn với việc xây **internal platform / paved road** — trùng đúng với triết lý paved road bên
DevSecOps, nên hai trục cộng hưởng rất tốt.

**e) Chi phí telemetry là ràng buộc mới.** Log/metric/trace phình theo scale; kỹ năng được đánh giá cao là
biết **giảm cardinality, sampling có chủ đích, phân tầng lưu trữ** mà vẫn trả lời được câu hỏi vận hành.

## 6. Lộ trình học
```text
1. Đọc "SRE Book" + "SRE Workbook" (Google, miễn phí online) — nền tư duy.
2. Áp dụng SLO vào 1 service thật: chọn SLI theo trải nghiệm user, đặt SLO, dựng dashboard error budget.
3. Observability chuẩn OTel: metrics/logs/traces + alert theo BURN RATE (multi-window, §2).
4. Thực hành incident: runbook dạng "điều kiện -> hành động -> kiểm chứng", tổ chức game day,
   viết postmortem blameless (và thực sự làm action item).
5. Capacity & performance: load test (k6), tìm bottleneck, autoscale đúng, tính cả chi phí.
6. Nâng coding: viết 1 operator/controller hoặc tool tự động hoá một loại toil cụ thể.
7. (Nối với trục chính) Áp SLO & incident process lên chính hệ thống bạn đã bảo mật: xem §8.
```

## 7. Bẫy thường gặp

| Bẫy | Vì sao chết | Cách tránh |
|-----|-------------|------------|
| Copy SLO "99.9%" cho mọi service | Không phản ánh nhu cầu user, tốn tiền vô ích | Hỏi ngược: mất bao lâu thì user/khách hàng thực sự đau? Đặt SLO từ đó |
| SLI đo CPU/memory | User không cảm nhận CPU | Đo request thành công, latency ở góc nhìn client |
| Alert theo ngưỡng tĩnh | Ồn, on-call chai lì | Burn-rate alerting (§2) |
| Dashboard nhiều, không trả lời được "có vi phạm SLO không" | Đẹp mà vô dụng | Mỗi service: 1 màn hình trả lời SLO + budget còn lại |
| Postmortem có blame ngầm | Người ta giấu sự cố | Tập trung vào cơ chế & rào chắn; action item có owner + hạn |
| Runbook viết xong bỏ đó | Lúc cháy nhà mở ra thì sai | Runbook chỉ sống nếu được dùng trong game day & sau mỗi sự cố |
| Chaos khi chưa có observability | Phá xong không biết chuyện gì xảy ra | Có SLO + telemetry trước, rồi mới chaos |
| Error budget không ai tôn trọng | SLO thành trang trí | Thoả thuận trước với Product: hết budget thì làm gì (freeze/ưu tiên ổn định) |

## 8. Giao thoa với DevSecOps (trục chính — xem [devsecops.md](./devsecops.md))

```text
Dùng chung quy trình : security incident = một loại sự cố production (severity, on-call,
                       incident command, blameless postmortem). Học một lần, dùng cho cả hai.
Dùng chung nền tảng  : audit log & threat detection ngồi trên chính stack metrics/log/trace của bạn.
Dùng chung tư duy    : error budget (SRE) ~ risk budget (Sec); assume failure ~ assume breach;
                       burn-rate alerting chữa alert fatigue cho cả hai bên.
Diễn tập             : game day (chaos) + purple team (giả lập tấn công) -> đo MTTD/MTTR.
Chiều SRE phục vụ Sec: MỌI security control là một dịch vụ production phải đáng tin —
                       admission webhook chết -> cả cụm không deploy được (failurePolicy, timeout,
                       fail-open vs fail-closed phải là quyết định có chủ đích, ghi vào ADR);
                       scanner treo -> pipeline đứng; agent runtime ngốn CPU -> ảnh hưởng workload.
                       Đây là chỗ nền SRE khiến bạn triển khai security KHÔNG gây sự cố — thứ phân biệt
                       người làm được thật với người chỉ bật tool.
```

## 9. Thị trường & đòn bẩy
- SRE là chuẩn vàng vận hành ở công ty lớn và mọi scale-up; DevOps/SRE nằm trong nhóm chuyên môn kỹ thuật
  tăng trưởng nhanh nhất, và đang mở thêm mảng vận hành hạ tầng AI/GPU.
- Chức danh: SRE, Reliability Engineer, Production Engineer, Platform/SRE.
- **Đòn bẩy Staff/Principal cao**: SLO & reliability là ngôn ngữ lãnh đạo kỹ thuật quan tâm (gắn tech với
  business: uptime = tiền/uy tín).
- Với định hướng hiện tại, SRE **không phải đích đến riêng** mà là lớp bồi làm cho hồ sơ DevSecOps của
  bạn nặng ký hơn: *người bảo mật được hệ thống mà không làm nó kém tin cậy đi.*

## Tóm tắt
- SRE biến reliability thành **con số quản trị được** (SLI/SLO/error budget), cân bằng ổn định ↔ tốc độ.
- Nền DevOps + playbook sự cố của bạn đã là hạt giống SRE; cần thêm tư duy SLO + burn-rate alert + coding
  + chaos + incident command.
- Bối cảnh 2026: OTel là mặc định, phản ứng sự cố đang tự động hoá (telemetry sạch + runbook máy đọc được
  lên giá), AI khuếch đại cả tốc độ lẫn rủi ro nếu nền yếu, platform engineering lên ngôi.
- Vai trò trong lộ trình của bạn: **lớp bổ trợ cho DevSecOps** — dùng chung incident process, và bảo đảm
  chính các security control cũng đáng tin.
