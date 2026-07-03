# 07 — Architecture Patterns & Well-Architected (sâu)

> Ghép mọi service thành **giải pháp**. Đây là dạng câu hỏi chính của SAA/SAP: "chọn kiến trúc tốt nhất cho tình huống X".
> Học theo **pattern** + **trade-off**, không học vẹt.

## 1. Well-Architected — áp dụng thực chiến

Với mỗi bài toán, hỏi theo 6 trụ cột:
```text
Reliability   : chịu lỗi AZ? Region? tự hồi phục? backup/DR có RPO/RTO bao nhiêu?
Security      : least privilege? mã hóa? phân đoạn mạng? audit?
Cost          : right-size? Spot/Savings Plans? tier storage? serverless để trả theo dùng?
Performance    : đúng compute/DB? caching (CloudFront/ElastiCache)? async?
Operational   : IaC? monitor/alarm? tự động hóa? runbook?
Sustainability: region hiệu quả? tối ưu tài nguyên?
```
**Mẹo thi:** đọc đề, xác định **từ khóa ưu tiên** → chọn đáp án khớp:
- "most cost-effective" → rẻ nhất (Spot/serverless/S3 tier/right-size).
- "highly available / fault tolerant" → multi-AZ, ASG, health check.
- "disaster recovery / region failure" → multi-region, replication.
- "minimal operational overhead / managed" → serverless/managed (Lambda, Fargate, Aurora Serverless, DynamoDB).
- "decouple" → SQS/SNS/EventBridge.
- "real-time" → Kinesis. "least latency" → CloudFront/edge/Global Accelerator.

## 2. High Availability & Resilience
```text
Single point of failure -> loại bỏ bằng dư thừa:
  Compute : ASG + nhiều AZ + ELB health check
  DB      : RDS Multi-AZ / Aurora / DynamoDB (multi-AZ sẵn)
  Network : NAT mỗi AZ, subnet mỗi AZ, ALB nhiều AZ
  State   : đẩy state ra ngoài (S3/DynamoDB/ElastiCache) -> compute stateless -> scale/thay dễ
```
- **Stateless + externalized state** = nguyên tắc vàng để scale & HA.
- Health check ở mọi tầng (ELB, Route53, ASG).

## 3. Decoupling & Event-driven (giảm phụ thuộc)
| Dịch vụ | Mô hình | Dùng khi |
|---------|---------|----------|
| **SQS** | queue, 1 consumer group xử lý, buffer | tách producer/consumer, chịu tải bùng, retry, DLQ |
| **SNS** | pub/sub, fan-out 1→nhiều | thông báo nhiều subscriber cùng lúc |
| **EventBridge** | event bus + routing rule | event-driven phức tạp, tích hợp SaaS, schedule |
| **Step Functions** | state machine điều phối | workflow nhiều bước, có retry/branch/trạng thái |
| **Kinesis** | streaming realtime, nhiều consumer, replay | analytics realtime, ingest lớn |
- **Pattern kinh điển:** SNS **fan-out** → nhiều SQS (mỗi service 1 queue) → xử lý độc lập.
- **SQS + ASG**: scale worker theo độ dài queue (`ApproximateNumberOfMessagesVisible`).
- **DLQ** (Dead Letter Queue): hứng message xử lý lỗi nhiều lần → không mất, điều tra sau.

## 4. Kiến trúc mẫu (thuộc để phản xạ)

### a. Web app 3-tier co giãn, HA
```text
Route53 -> CloudFront (cache + WAF) -> ALB (nhiều AZ)
  -> ASG EC2/ECS (private subnet, nhiều AZ)
     -> ElastiCache (cache) + RDS Multi-AZ / Aurora (private)
  static assets: S3 (qua CloudFront, OAC)
```

### b. Serverless API (ít vận hành, trả theo dùng)
```text
Route53 -> CloudFront -> API Gateway -> Lambda -> DynamoDB
  auth: Cognito.  async: EventBridge/SQS.  file: S3 (+ presigned URL).
  Ưu: không quản server, tự scale, rẻ khi tải thấp. Nhược: cold start, giới hạn 15'.
```

### c. Xử lý dữ liệu / ingest lớn
```text
Producers -> Kinesis Data Streams -> Lambda/KCL -> S3 (data lake)
  -> Firehose -> S3/Redshift/OpenSearch
  -> Athena/Glue query trên S3;  QuickSight dashboard.
```

### d. Microservices container
```text
ALB -> ECS/EKS (Fargate) nhiều service -> RDS/DynamoDB
  service discovery, X-Ray trace, Secrets Manager, CI/CD Code*/GitOps.
```

### e. Static website
```text
S3 (private) -> CloudFront (OAC) -> Route53 Alias.  ACM cert (us-east-1). Rẻ, scale vô hạn.
```

## 5. Disaster Recovery — 4 chiến lược (SAP thuộc lòng)
```text
                     RTO/RPO       Chi phí     Mô tả
Backup & Restore   : giờ–ngày      thấp nhất   backup ra S3/khác region, restore khi cần
Pilot Light        : chục phút     thấp        core (DB) chạy sẵn nhỏ ở region 2, scale khi cần
Warm Standby       : phút          trung bình  bản thu nhỏ CHẠY SẴN ở region 2, scale up khi failover
Multi-Site Active-Active: ~0        cao nhất    2 region cùng phục vụ, failover gần như tức thì
```
- **RPO** = mất tối đa bao nhiêu **data** (theo thời gian). **RTO** = khôi phục trong **bao lâu**.
- Chọn theo mức chịu đựng của business vs chi phí. Route53 failover/health check + replication (S3 CRR, Aurora Global, DynamoDB Global Tables) là công cụ chính.

## 6. Cost Optimization patterns
- **Compute:** right-size (Compute Optimizer), Savings Plans cho baseline, **Spot** cho phần co giãn/chịu gián đoạn, tắt môi trường dev ngoài giờ (Instance Scheduler), serverless để trả theo dùng.
- **Storage:** S3 Lifecycle → IA/Glacier, Intelligent-Tiering khi không đoán được, xóa snapshot/EBS mồ côi, gp3 thay gp2.
- **Network:** giảm NAT bằng **VPC Endpoint** (S3/DynamoDB), CloudFront giảm data transfer origin, tránh cross-AZ/cross-region thừa.
- **Data transfer** thường là chi phí ẩn lớn → thiết kế để giảm.

## 7. Migration (SAP) — 7 R's
```text
Rehost (lift-and-shift) · Replatform (chỉnh nhẹ) · Repurchase (đổi sang SaaS) ·
Refactor (viết lại cloud-native) · Retire (bỏ) · Retain (giữ on-prem) · Relocate.
Công cụ: Application Migration Service (MGN), DMS+SCT (database), DataSync, Snow Family, Migration Hub.
```

---

## Khung trả lời câu hỏi kiến trúc (áp dụng khi thi & khi thiết kế thật)
```text
1. Yêu cầu chính là gì? (HA? rẻ? nhanh? ít vận hành? bảo mật? DR?)
2. Loại bỏ đáp án vi phạm ràng buộc cứng (sai region, không HA, không bảo mật).
3. Trong số còn lại, chọn cái khớp TRỤ CỘT ưu tiên của đề.
4. Ưu tiên managed/serverless khi đề nhấn "ít vận hành".
5. Cảnh giác đáp án "quá mức cần thiết" (over-engineered) hoặc "rẻ nhưng không đạt yêu cầu".
```

## Tóm tắt điểm bẫy
- Đọc **từ khóa ưu tiên** của đề → map sang trụ cột Well-Architected.
- Stateless + đẩy state ra ngoài = chìa khóa scale/HA.
- Decouple bằng SQS/SNS/EventBridge; SNS fan-out → nhiều SQS.
- DR: nhớ 4 mức + phân biệt **RPO (data) vs RTO (thời gian)**.
- Giảm NAT & data transfer là nguồn tiết kiệm lớn hay bị bỏ qua.
- "Ít vận hành nhất" → chọn serverless/managed.
