# 00 — AWS Fundamentals

> Nền tảng phải nắm trước khi vào service: hạ tầng toàn cầu, tổ chức tài khoản, IAM cơ bản,
> mô hình trách nhiệm, và khung Well-Architected. Đây là "bối cảnh" mọi câu hỏi cert đều giả định.

## 1. Hạ tầng toàn cầu (Global Infrastructure)

```text
Region (vd us-east-1)  =  vùng địa lý, độc lập, tách biệt về mặt pháp lý/dữ liệu
  └── Availability Zone (AZ)  =  1+ data center, cách nhau vật lý nhưng nối bằng link tốc độ cao
        └── data center vật lý
Edge Location  =  điểm CDN/DNS (CloudFront, Route53) — nhiều hơn Region rất nhiều
```

- **Region**: mỗi region có mã (`us-east-1`, `ap-southeast-1`=Singapore). Chọn region dựa trên: **độ trễ tới user, giá, tuân thủ dữ liệu (data residency), và service có sẵn** (không phải region nào cũng có mọi service).
- **AZ**: ít nhất 3 AZ/region (đa số). Thiết kế **multi-AZ** = chịu lỗi 1 data center. AZ id (`use1-az1`) khác tên (`us-east-1a`) — tên được ánh xạ ngẫu nhiên theo account để cân tải.
- **Edge Location**: phục vụ CloudFront, Route 53, Global Accelerator, WAF — gần user để giảm latency.

**Điểm thi hay gặp:**
- High Availability → trải **nhiều AZ**. Disaster Recovery / thảm họa vùng → trải **nhiều Region**.
- Một số resource là **global** (IAM, Route53, CloudFront, WAF), số khác **regional** (VPC, S3 bucket namespace toàn cầu nhưng data ở 1 region), số khác **AZ-scoped** (EBS, subnet).

## 2. Mô hình trách nhiệm chung (Shared Responsibility Model)

```text
AWS chịu trách nhiệm: "Security OF the cloud"
   -> hạ tầng vật lý, phần cứng, mạng nền, hypervisor, managed service internals
Khách hàng chịu trách nhiệm: "Security IN the cloud"
   -> data, IAM/phân quyền, cấu hình SG/NACL, mã hóa, patch OS (với EC2), app
```
- Càng managed thì phần khách hàng lo càng ít: EC2 (lo cả OS) > RDS (AWS lo OS/DB engine patch) > S3/DynamoDB (AWS lo gần hết, bạn lo data + quyền) > Lambda (chỉ lo code + quyền).
- **Điểm bẫy:** với EC2, **patch OS là việc của bạn** (dùng SSM Patch Manager). Với RDS, patch DB là của AWS (bạn set maintenance window).

## 3. Tổ chức tài khoản (Account & Organizations)

- **Account** = ranh giới cô lập & billing. Best practice: **nhiều account** (prod/staging/dev/security tách riêng) thay vì 1 account khổng lồ.
- **AWS Organizations**: quản lý nhiều account tập trung.
  - **OU (Organizational Unit)**: nhóm account theo mục đích (Prod OU, Dev OU).
  - **SCP (Service Control Policy)**: giới hạn quyền TỐI ĐA của account/OU (guardrail). SCP **không cấp quyền**, chỉ **giới hạn** — quyền thực = giao của SCP ∩ IAM.
  - **Consolidated Billing**: gộp hóa đơn, hưởng volume discount & chia sẻ Reserved/Savings Plans.
- **Control Tower**: dựng landing zone multi-account theo best practice tự động (OU, SCP, logging, SSO sẵn).

**Điểm thi (SAP hay hỏi):** muốn cấm mọi account không được rời region cho phép, hoặc cấm tắt CloudTrail → dùng **SCP** ở cấp Organization.

## 4. IAM cơ bản (chi tiết ở [05-security-iam.md](./05-security-iam.md))

```text
Principal (user/role/service)  --xin-->  Action trên Resource
   -> đánh giá bởi Policy (JSON):  Effect(Allow/Deny) + Action + Resource + Condition
```
- **IAM User**: danh tính người/ứng dụng lâu dài (có access key/password). Hạn chế dùng cho app.
- **IAM Role**: danh tính **tạm thời**, không có credential cố định — được "assume" để lấy quyền tạm (STS). **Ưu tiên role** cho EC2/Lambda/cross-account thay vì nhét access key.
- **IAM Group**: gom user để gán policy chung.
- **Policy**: Managed (AWS quản lý / customer quản lý) hoặc Inline. Quy tắc đánh giá: **mặc định Deny → cần Allow tường minh → mọi Deny tường minh luôn thắng.**
- **Root user**: chỉ dùng cho vài tác vụ bắt buộc, **bật MFA**, không dùng hằng ngày.

**Nguyên tắc vàng:** Least Privilege + dùng Role + MFA + không bao giờ hardcode key.

## 5. Cách gọi AWS (Access methods)

- **Console** (web), **CLI** (`aws ...`), **SDK** (code), **CloudFormation/CDK/Terraform** (IaC).
- **CLI cấu hình:** `aws configure` (access key + region) hoặc **SSO/profile**. Trên EC2/Lambda → dùng **instance profile/role**, không cần key.

## 6. AWS Well-Architected Framework (6 trụ cột)

Khung tư duy AWS dùng để đánh giá kiến trúc — **cực kỳ quan trọng cho SAA/SAP**:

| Trụ cột | Ý nghĩa | Ví dụ dịch vụ/thực hành |
|---------|---------|-------------------------|
| **Operational Excellence** | Vận hành & cải tiến liên tục | IaC, CloudWatch, runbook, automation |
| **Security** | Bảo vệ data & hệ thống | IAM least-privilege, KMS, WAF, GuardDuty |
| **Reliability** | Chịu lỗi & tự hồi phục | Multi-AZ, ASG, health check, backup/DR |
| **Performance Efficiency** | Dùng tài nguyên hiệu quả | đúng instance type, caching, serverless |
| **Cost Optimization** | Tối ưu chi phí | Right-sizing, Savings Plans, S3 tiering |
| **Sustainability** | Bền vững (giảm carbon) | region hiệu quả, tối ưu tài nguyên |

**Tư duy thi:** với mỗi tình huống, xác định đề đang ưu tiên trụ cột nào (rẻ nhất? tin cậy nhất? nhanh nhất?) → chọn đáp án khớp trụ cột đó.

## 7. Billing & Cost (nền tảng)

- **Free Tier**: 12 tháng đầu (một số always-free). Dùng để luyện tay.
- **Pricing models EC2:** On-Demand (linh hoạt, đắt) > Reserved Instances / **Savings Plans** (cam kết 1-3 năm, rẻ 40-72%) > **Spot** (thừa, rẻ tới 90%, có thể bị thu hồi) > Dedicated Host.
- **Công cụ:** Cost Explorer (phân tích), Budgets (cảnh báo ngưỡng), Cost & Usage Report (chi tiết), Compute Optimizer (gợi ý right-size), Trusted Advisor (kiểm tra best practice + tiết kiệm).

**Điểm thi:** workload chạy 24/7 lâu dài → Savings Plans/RI. Workload chịu gián đoạn (batch, CI) → Spot. Không đoán được, ngắn hạn → On-Demand.

---

## Tóm tắt điểm bẫy hay gặp
- Global vs Regional vs AZ-scoped resource — nhớ phân loại.
- SCP **giới hạn**, không cấp quyền; explicit Deny luôn thắng.
- HA = multi-AZ; DR khỏi thảm họa vùng = multi-Region.
- Patch OS của EC2 là việc của bạn; RDS engine patch là của AWS.
- Ưu tiên Role + STS (tạm thời) thay cho access key cố định.
