# AWS Mastery — Tổng hợp Service, Concept & Lộ trình Cert

> Bộ tài liệu học AWS **từ nền tảng tới kiến trúc**, đủ sâu để thi và **sở hữu** các cert:
> **SAA** (Solutions Architect Associate) · **SAP** (Solutions Architect Professional) ·
> **SysOps/CloudOps** · **DevOps Engineer Professional**.
> Mỗi file: concept → cách hoạt động → use case → kiến trúc → điểm hay gặp trong đề thi.

## 📚 Cấu trúc tài liệu

| File | Nội dung | Cert liên quan |
|------|----------|----------------|
| [00-fundamentals.md](./00-fundamentals.md) | Global infra, Account, IAM cơ bản, Well-Architected, Billing | Tất cả |
| [01-networking-vpc.md](./01-networking-vpc.md) | **CIDR, VPC, Subnet, Route, IGW/NAT, SG/NACL, Peering, TGW, Endpoint, Route53, CloudFront** | SAA, SAP, SysOps |
| [02-compute.md](./02-compute.md) | EC2, AMI, ASG, ELB, Lambda, ECS/EKS/Fargate | SAA, SAP, DevOps |
| [03-storage.md](./03-storage.md) | S3, EBS, EFS, FSx, Storage Gateway, Backup | SAA, SysOps |
| [04-databases.md](./04-databases.md) | RDS, Aurora, DynamoDB, ElastiCache, Redshift | SAA, SAP |
| [05-security-iam.md](./05-security-iam.md) | IAM sâu, KMS, Secrets, WAF/Shield, GuardDuty, Org/SCP | Tất cả, SAP |
| [06-management-devops.md](./06-management-devops.md) | CloudWatch, CloudTrail, Config, CloudFormation, Code* CI/CD, SSM | SysOps, DevOps |
| [07-architecture-patterns.md](./07-architecture-patterns.md) | Use case & kiến trúc mẫu, Well-Architected sâu, DR, cost | SAA, SAP |
| [labs/cidr-subnetting-deep-dive.md](./labs/cidr-subnetting-deep-dive.md) | **Lab CIDR/subnetting** — toán nhị phân, chia subnet, thiết kế VPC (có bài + lời giải) | SAA, SysOps |
| [practice-questions.md](./practice-questions.md) | **Câu hỏi luyện thi kiểu đề thật** có giải thích, theo domain | Tất cả |
| [cert-roadmap.md](./cert-roadmap.md) | Lộ trình ôn từng cert, domain, chiến lược thi | Tất cả |

## 🎯 Thứ tự học đề xuất

```text
1. Fundamentals (00)         -> hiểu account, region/AZ, IAM, mô hình trách nhiệm
2. Networking (01)           -> XƯƠNG SỐNG. Không nắm VPC thì mọi thứ mù mờ.
3. Compute (02) + Storage (03) + Databases (04)   -> khối service chính
4. Security (05)             -> xuyên suốt mọi domain
5. Management & DevOps (06)  -> vận hành, tự động hóa
6. Architecture (07)         -> ghép tất cả thành giải pháp
7. Cert roadmap              -> luyện đề theo cert mục tiêu
```

## 🗺️ Lộ trình cert (tổng quan)

```text
                   ┌─────────────────────────────┐
                   │  Cloud Practitioner (tùy chọn) │  ← nền, không bắt buộc
                   └──────────────┬──────────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              ▼                                        ▼
   ┌──────────────────────┐              ┌──────────────────────────┐
   │ Solutions Architect  │              │  SysOps Administrator     │
   │ Associate (SAA-C03)  │              │  (CloudOps) Associate     │
   └──────────┬───────────┘              └────────────┬─────────────┘
              │                                        │
              ▼                                        ▼
   ┌──────────────────────┐              ┌──────────────────────────┐
   │ Solutions Architect  │              │ DevOps Engineer           │
   │ Professional (SAP)   │              │ Professional (DOP-C02)    │
   └──────────────────────┘              └──────────────────────────┘

Gợi ý cho bạn: SAA -> SysOps -> DevOps Pro -> SAP.
(SAA cho nền kiến trúc, SysOps cho vận hành, DevOps Pro & SAP là đỉnh.)
```

## 💡 Cách dùng bộ tài liệu để "sở hữu" cert

1. **Đọc hiểu concept** (file này) — không học vẹt, hiểu *tại sao* service tồn tại & giải quyết vấn đề gì.
2. **Làm tay trên Free Tier** — mỗi service tự tạo/xóa ít nhất 1 lần. Kiến thức chỉ thật khi bấm nút.
3. **Học theo use case & kiến trúc** — đề thi hỏi "chọn giải pháp tốt nhất cho tình huống X", không hỏi định nghĩa.
4. **Luyện đề** (Tutorials Dojo / official practice) — đọc kỹ giải thích cả đáp án đúng lẫn sai.
5. **Ghi chú điểm bẫy** — mỗi lần sai, note lại vào chính file tương ứng.

> ⚠️ Lưu ý: giá cả, giới hạn (limits/quotas) và tính năng AWS thay đổi liên tục. Tài liệu này là
> nền tảng concept & kiến trúc (ổn định lâu dài); luôn đối chiếu **AWS Docs** cho con số cụ thể mới nhất.
