# 05 — Security & IAM (xuyên suốt mọi domain)

> Bảo mật là trụ cột có mặt trong MỌI câu hỏi. IAM là gốc. File này đi sâu IAM rồi tới mã hóa,
> bảo vệ mạng/ứng dụng, phát hiện mối đe dọa, và quản trị đa tài khoản.

## 1. IAM — sâu

### Các loại policy (phân biệt rõ — hay hỏi)
| Loại | Gắn vào | Vai trò |
|------|---------|---------|
| **Identity-based** | user/group/role | "danh tính này được làm gì" |
| **Resource-based** | resource (S3 bucket, SQS, KMS...) | "ai được động vào resource này" (có `Principal`) |
| **Permission Boundary** | user/role | trần quyền tối đa của danh tính (không cấp, chỉ giới hạn) |
| **SCP** (Organizations) | account/OU | trần quyền tối đa của cả account |
| **Session policy** | lúc assume role | thu hẹp quyền phiên tạm |

### Cấu trúc policy JSON
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject"],
    "Resource": "arn:aws:s3:::my-bucket/*",
    "Condition": { "IpAddress": { "aws:SourceIp": "10.0.0.0/16" } }
  }]
}
```

### Logic đánh giá quyền (THUỘC LÒNG)
```text
1. Mặc định: DENY (implicit).
2. Có Allow tường minh ở đâu đó? Nếu không -> Deny.
3. Có Deny tường minh (explicit Deny) ở BẤT KỲ policy nào? -> DENY (thắng tất cả).
Quyền cuối = (SCP) ∩ (Permission Boundary) ∩ (Identity/Resource policy), trừ mọi explicit Deny.
```
- **Cross-account access:** dùng **Role** — account A tạo role tin tưởng account B (trust policy), B assume role qua **STS** để lấy credential tạm.
- **IAM Roles for services:** EC2 (instance profile), Lambda (execution role), ECS (task role) — **không nhét access key**.
- **IRSA / Pod Identity** (EKS): gán IAM role cho pod theo ServiceAccount.
- **Identity federation:** SSO qua **IAM Identity Center** (kế nhiệm AWS SSO), SAML/OIDC, **Cognito** (user pool cho app end-user, identity pool cấp AWS creds tạm).

**Best practices IAM:** least privilege, role thay key, bật MFA (đặc biệt root), xoay key, dùng **IAM Access Analyzer** (phát hiện resource bị chia sẻ ra ngoài), **Credential Report** & **Access Advisor** (rà quyền thừa).

## 2. Mã hóa & quản lý khóa

### KMS — Key Management Service
```text
Quản lý khóa mã hóa. CMK/KMS key: AWS-managed, Customer-managed (bạn kiểm soát policy/xoay), hoặc AWS-owned.
Envelope encryption: KMS mã hóa "data key", data key mã hóa dữ liệu -> hiệu quả với data lớn.
```
- Tích hợp gần như mọi service (S3 SSE-KMS, EBS, RDS, Secrets...). **Key policy** + IAM kiểm soát ai dùng key.
- **Automatic key rotation** (hằng năm). **Multi-Region keys** cho DR. **CloudHSM** khi cần HSM chuyên dụng/tuân thủ (bạn toàn quyền, AWS không truy cập).

### Quản lý secret
- **Secrets Manager**: lưu secret + **tự động xoay** (rotation, tích hợp RDS) + phân quyền. Đắt hơn nhưng có rotation.
- **SSM Parameter Store**: lưu config/secret (SecureString qua KMS). Rẻ/miễn phí (standard), không tự rotate. Dùng cho config & secret đơn giản.

## 3. Bảo vệ mạng & ứng dụng
| Dịch vụ | Bảo vệ khỏi | Ghi chú |
|---------|-------------|---------|
| **Security Group / NACL** | truy cập mạng trái phép | tầng VPC (xem file 01) |
| **WAF** | tấn công tầng 7 (SQLi, XSS, bot) | gắn ALB/CloudFront/API GW; rule theo IP/rate/pattern |
| **Shield Standard** | DDoS L3/L4 cơ bản | miễn phí, tự động |
| **Shield Advanced** | DDoS lớn + hỗ trợ + hoàn phí | trả phí, có DRT team |
| **Firewall Manager** | quản lý WAF/SG/Shield tập trung đa account | qua Organizations |
| **Network Firewall** | tường lửa có state cấp VPC | filtering nâng cao |

## 4. Phát hiện & tuân thủ (Detection)
| Dịch vụ | Làm gì |
|---------|--------|
| **CloudTrail** | ghi lại **mọi API call** (ai làm gì, khi nào) — audit/forensic. Bật tổ chức + gửi S3 (immutable). |
| **GuardDuty** | phát hiện mối đe dọa bằng ML (traffic bất thường, port scan, crypto mining) từ log |
| **Inspector** | quét lỗ hổng EC2/ECR/Lambda (CVE) tự động |
| **Macie** | phát hiện dữ liệu nhạy cảm (PII) trong S3 bằng ML |
| **Security Hub** | tổng hợp findings + kiểm tra chuẩn (CIS/PCI) tập trung |
| **Detective** | điều tra nguyên nhân từ findings (graph) |
| **Config** | ghi lại & đánh giá **tuân thủ cấu hình** resource theo rule (drift, compliance) |
| **Audit Manager** | thu thập bằng chứng cho audit tuân thủ |

**Bộ đôi hay nhầm:** **CloudTrail = ai gọi API** (audit hành động) vs **Config = trạng thái/cấu hình resource có tuân thủ không** (drift). vs **CloudWatch = metric/log hiệu năng**.

## 5. Quản trị đa tài khoản (SAP nặng phần này)
- **Organizations + OU + SCP**: guardrail toàn tổ chức (vd chặn region, cấm tắt CloudTrail, cấm tạo IAM user).
- **Control Tower**: landing zone tự động (OU chuẩn, account factory, guardrails, log account tập trung).
- **IAM Identity Center**: SSO tập trung, gán permission set theo account.
- **Resource Access Manager (RAM)**: chia sẻ resource (subnet, TGW) giữa account.
- **Tagging + Tag Policies**: chuẩn hóa tag để quản trị/cost.

---

## Kiến trúc bảo mật mẫu (defense in depth)
```text
Edge:     CloudFront + WAF + Shield          (chặn tấn công L7/DDoS)
Network:  VPC private subnet + SG + NACL + Network Firewall
Identity: IAM least-privilege + roles + MFA + Identity Center (SSO)
Data:     KMS mã hóa at-rest + TLS in-transit + Secrets Manager
Detect:   CloudTrail + GuardDuty + Config + Security Hub (tổng hợp)
Govern:   Organizations + SCP + Control Tower (đa account)
```

## Tóm tắt điểm bẫy
- Explicit **Deny luôn thắng**; mặc định deny; quyền = giao của các lớp (SCP ∩ boundary ∩ policy).
- Cross-account/service → **Role + STS**, không dùng access key.
- SCP & Permission Boundary **giới hạn**, không cấp quyền.
- **Secrets Manager** (có rotation) vs **Parameter Store** (rẻ, không rotation).
- **CloudTrail** (API audit) vs **Config** (compliance cấu hình) vs **CloudWatch** (metric/log).
- ACM cert cho CloudFront phải ở **us-east-1**; KMS multi-region cho DR.
