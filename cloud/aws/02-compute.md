# 02 — Compute (EC2, ASG, ELB, Serverless, Containers)

> Khối tính toán: từ máy ảo EC2 tới serverless Lambda và container ECS/EKS. Trọng tâm SAA/SAP/DevOps.

## 1. EC2 — máy ảo

```text
Instance = 1 VM. Định nghĩa bởi: AMI (image) + Instance Type (CPU/RAM) + Network (VPC/subnet/SG) + Storage (EBS) + IAM Role.
```

### Instance families (nhớ theo chữ cái đầu)
| Họ | Tối ưu cho | Ví dụ dùng |
|----|-----------|-----------|
| **T** (t3, t4g) | Burstable, rẻ | web nhỏ, dev, traffic thấp/bùng |
| **M** (m5, m6i) | Cân bằng | app tổng quát |
| **C** (c5, c6g) | Compute (CPU) | batch, HPC, game server |
| **R** (r5) | RAM lớn | in-memory DB, cache, analytics |
| **X** | RAM cực lớn | SAP HANA, in-memory khổng lồ |
| **I** / **D** | Storage/IO cao (NVMe local) | DB tự quản, data warehouse |
| **P** / **G** / **Inf** | GPU / ML | training, inference, đồ họa |
- Hậu tố: `g`=ARM Graviton (rẻ hơn ~20%, hiệu năng/giá tốt), `n`=network cao, `d`=NVMe local.

### Pricing (đã nêu ở fundamentals, nhắc lại vì hay hỏi)
On-Demand (linh hoạt) · Reserved/**Savings Plans** (cam kết 1-3 năm, rẻ) · **Spot** (thừa, rẻ ~90%, bị thu hồi 2 phút báo trước) · Dedicated Host/Instance (compliance/licensing).
- **Spot** hợp: batch, CI/CD, xử lý chịu gián đoạn, worker stateless. Kết hợp **Spot + On-Demand** trong ASG (mixed instances policy).

### AMI & bootstrap
- **AMI** = ảnh máy (OS + cấu hình). Golden AMI (build sẵn bằng **EC2 Image Builder** / Packer) → khởi động nhanh, nhất quán.
- **User Data**: script chạy lúc boot lần đầu (cài phần mềm).
- **Instance Metadata (IMDS)**: `http://169.254.169.254/...` — lấy thông tin instance & credential của role. **Dùng IMDSv2** (chống SSRF).

### Placement Groups
- **Cluster** (cùng 1 rack, latency thấp, HPC) · **Spread** (mỗi instance 1 phần cứng, HA tối đa) · **Partition** (nhóm phân vùng, cho HDFS/Kafka lớn).

### Storage cho EC2 → xem [03-storage.md](./03-storage.md) (EBS vs Instance Store)

## 2. Auto Scaling Group (ASG) — co giãn & tự hồi phục

```text
ASG giữ số instance theo mong muốn, trải nhiều AZ, thay instance chết tự động.
 - Launch Template (định nghĩa instance) + min/desired/max + subnet(nhiều AZ) + health check.
 - Scaling policies:
     Target Tracking  : giữ 1 metric ở mức mục tiêu (vd CPU 50%) — đơn giản, khuyên dùng
     Step / Simple    : theo bậc alarm CloudWatch
     Scheduled        : theo lịch (giờ cao điểm biết trước)
     Predictive       : ML dự báo tải
```
- **Health check**: EC2 (phần cứng) + **ELB** (ứng dụng) → nên bật ELB health check để thay instance "sống nhưng app hỏng".
- **Lifecycle hooks**: chèn hành động lúc launch/terminate (drain kết nối, cài đặt).
- **Cooldown / warm-up**: tránh scale dồn dập.
- ASG = trụ cột **Reliability** + **Cost** (scale-in khi rảnh).

## 3. Elastic Load Balancing (ELB)

| Loại | Tầng | Dùng cho | Đặc điểm |
|------|------|----------|----------|
| **ALB** | L7 (HTTP/HTTPS) | web, microservice | routing theo **path/host/header**, WebSocket, redirect, auth (Cognito/OIDC), target = instance/IP/**Lambda** |
| **NLB** | L4 (TCP/UDP/TLS) | throughput cực cao, latency thấp | **IP tĩnh/EIP mỗi AZ**, giữ source IP, hàng triệu req/s |
| **GWLB** | L3 | chèn appliance bảo mật (firewall/IDS) | dùng GENEVE |
| (CLB) | cũ | legacy | tránh dùng cho thiết kế mới |

- **Target Group**: nhóm đích + health check. **Sticky sessions** (cookie) khi cần.
- **Cross-zone load balancing**: phân bổ đều qua AZ (ALB bật mặc định; NLB tính phí khi bật).
- **SSL/TLS**: terminate ở ALB (dùng **ACM** cấp cert miễn phí). SNI cho nhiều domain.
- **Điểm thi:** cần routing theo URL/host → ALB. Cần IP tĩnh + TCP + tốc độ cực cao → NLB.

## 4. Serverless — Lambda

```text
Chạy code theo sự kiện, không quản server, trả tiền theo số lần gọi + thời gian chạy (GB-giây).
 - Trigger: API Gateway, S3, DynamoDB Streams, EventBridge, SQS, SNS, ALB...
 - Giới hạn: timeout tối đa 15 phút, RAM tới ~10GB (CPU tỉ lệ RAM), gói deploy có hạn.
 - Cold start: lần đầu/khi scale — giảm bằng Provisioned Concurrency.
 - Concurrency: mặc định giới hạn/account; đặt Reserved Concurrency để bảo vệ/giới hạn.
```
- Dùng khi: event-driven, tải bùng/không đều, glue giữa service, cron (EventBridge). Không hợp: chạy lâu >15', cần trạng thái lớn, độ trễ cực thấp ổn định.
- **Lambda@Edge / CloudFront Functions**: chạy logic tại edge (chỉnh request/response gần user).
- Kèm **API Gateway** (REST/HTTP/WebSocket) + **Step Functions** (điều phối workflow nhiều bước, có retry/trạng thái).

## 5. Containers — ECS / EKS / Fargate

```text
ECS  : orchestrator của AWS, đơn giản, tích hợp sâu AWS. Task/Service/Cluster.
EKS  : Kubernetes managed (control plane do AWS chạy). Chọn khi cần chuẩn K8s/đa cloud/hệ sinh thái.
Fargate : chế độ SERVERLESS cho container (ECS & EKS) — không quản EC2, trả theo vCPU/RAM task dùng.
EC2 launch type : bạn tự quản node EC2 (rẻ hơn ở quy mô lớn, kiểm soát nhiều hơn).
ECR  : registry lưu image (private), tích hợp scan.
```
**Cây quyết định (điểm thi):**
- Muốn đơn giản, ở trong AWS → **ECS**. Cần Kubernetes/đa cloud → **EKS**.
- Không muốn quản server → **Fargate**. Cần tối ưu chi phí ở quy mô lớn / cần GPU/đặc thù → **EC2 launch type**.
- CI/CD container: build → push **ECR** → deploy ECS/EKS.

## 6. Các compute khác (biết để nhận diện)
- **Elastic Beanstalk**: PaaS — deploy app (web) nhanh, AWS lo hạ tầng (EC2+ASG+ELB). Dev không rành hạ tầng.
- **Batch**: chạy job batch quy mô lớn (tự quản queue + compute, hợp Spot).
- **Lightsail**: VPS đơn giản giá phẳng, cho project nhỏ.
- **Outposts / Wavelength / Local Zones**: đưa compute AWS về on-prem/biên/gần user.

---

## Kiến trúc compute mẫu
```text
Web co giãn, HA:   Route53 -> CloudFront -> ALB(2AZ) -> ASG EC2(2AZ) -> RDS Multi-AZ
Serverless API:    Route53 -> API Gateway -> Lambda -> DynamoDB   (+ S3 cho static qua CloudFront)
Microservices:     ALB -> ECS/EKS (Fargate) -> service mesh -> RDS/DynamoDB
Batch chịu gián đoạn: SQS -> ASG Spot workers / AWS Batch
```

## Tóm tắt điểm bẫy
- Spot cho workload chịu gián đoạn; Savings Plans cho chạy lâu dài.
- ASG bật **ELB health check** để thay instance app-hỏng; trải **nhiều AZ**.
- ALB = L7 routing; NLB = L4 + IP tĩnh + tốc độ.
- Lambda tối đa **15 phút**; cold start giảm bằng Provisioned Concurrency.
- Fargate = không quản node; EKS = cần Kubernetes; ECS = đơn giản trong AWS.
- Cert (ACM) miễn phí gắn ở ALB/CloudFront (ACM cho CloudFront phải ở **us-east-1**).
