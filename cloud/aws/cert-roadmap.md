# Cert Roadmap — SAA · SAP · SysOps (CloudOps) · DevOps Pro

> Lộ trình ôn & chiến lược thi từng cert. Map với các file trong folder này.

## Bức tranh tổng thể
```text
Foundational : Cloud Practitioner (CLF-C02)   — tùy chọn, nền tảng nhẹ
Associate    : SAA-C03 (Architect) · SysOps (SOA-C02) · Developer (DVA-C02)
Professional : SAP-C02 (Architect Pro) · DOP-C02 (DevOps Pro)
Specialty    : Security, Networking, ML, Database...

Gợi ý thứ tự cho mục tiêu của bạn:
   SAA  ->  SysOps  ->  DevOps Pro  ->  SAP
   (nền kiến trúc -> vận hành -> tự động hóa đỉnh -> kiến trúc đỉnh)
```

Nguyên tắc chung mọi cert: **hiểu concept + làm tay Free Tier + học theo use case + luyện đề đọc kỹ giải thích.**

---

## 1. SAA-C03 — Solutions Architect Associate

**Bản chất:** chọn kiến trúc tốt nhất theo tình huống. Rộng, không quá sâu. Nền cho mọi cert sau.

**4 domain (trọng số):**
```text
1. Design Secure Architectures        ~30%   -> file 05, 01
2. Design Resilient Architectures      ~26%   -> file 07, 02, 04 (Multi-AZ, ASG, DR)
3. High-Performing Architectures        ~24%   -> file 02,03,04 (caching, đúng service)
4. Cost-Optimized Architectures         ~20%   -> file 03,07 (S3 tier, Spot, Savings Plans)
```
**Phải cực vững:** VPC/networking (01), S3 storage classes (03), RDS Multi-AZ vs Read Replica (04), ELB/ASG (02), IAM (05), decoupling SQS/SNS (07), Well-Architected (00/07).

**Chiến lược:** ~80% câu là "chọn giải pháp tốt nhất" — luyện đọc từ khóa ưu tiên. Loại đáp án sai ràng buộc trước.

**Thời gian ôn tham khảo:** 4–8 tuần. Ôn theo file 00→07 rồi luyện đề.

---

## 2. SysOps / CloudOps — SOA-C02

**Bản chất:** **vận hành** hệ thống trên AWS — monitor, tự động hóa, bảo mật vận hành, triển khai, tối ưu, khắc phục sự cố. **Hands-on nặng** (có thể có lab thực hành).

**Domain:**
```text
Monitoring/Logging/Remediation    -> CloudWatch, EventBridge, Config (file 06)
Reliability & BCP                 -> backup, Multi-AZ, ASG, DR (07,02,04)
Deployment/Provisioning/Automation-> CloudFormation, SSM (06)
Security & Compliance             -> IAM, KMS, Config, CloudTrail (05)
Networking & Content Delivery     -> VPC, Route53, CloudFront (01)
Cost & Performance Optimization   -> Cost Explorer, right-size (06,07)
```
**Đặc trưng SysOps (hay thi):**
- CloudWatch Agent cho RAM/disk metric; alarm & EC2 recovery.
- SSM Session Manager/Patch/Run Command.
- CloudFormation troubleshooting, drift.
- Service Quotas gây sự cố.
- Config rules + auto-remediation.
- Backup/restore, snapshot lifecycle.

**Chiến lược:** thực hành thật nhiều trên console. Nắm chắc CloudWatch/SSM/CloudFormation.

---

## 3. DOP-C02 — DevOps Engineer Professional

**Bản chất:** **tự động hóa** vòng đời — CI/CD, IaC, monitoring, incident response, ở quy mô. Cần nền Associate (SysOps/Developer) vững.

**Domain:**
```text
SDLC Automation (CI/CD)           ~22%  -> CodePipeline/Build/Deploy, chiến lược deploy (06)
Configuration Mgmt & IaC          ~17%  -> CloudFormation/CDK, StackSets, SSM (06)
Resilient Cloud Solutions         ~15%  -> HA/DR multi-region, auto-heal (07)
Monitoring & Logging              ~15%  -> CloudWatch, EventBridge, X-Ray (06)
Incident & Event Response         ~14%  -> EventBridge auto-remediation, alarm (06,05)
Security & Compliance (automation)~17%  -> Config, GuardDuty, Secrets rotation (05)
```
**Phải cực vững:**
- CodeDeploy Blue/Green & Canary; rollback tự động theo CloudWatch alarm.
- CI/CD pipeline đa account/region; artifact & approval.
- CloudFormation nâng cao (nested, StackSets, hooks, custom resource).
- Tự động phản ứng: EventBridge → Lambda/SSM Automation remediation.
- Blue/Green cho ECS/Lambda; ASG lifecycle hooks.

**Chiến lược:** câu hỏi tình huống dài, phức tạp. Hiểu **luồng end-to-end** của pipeline & tự động hóa, không học lẻ từng service.

---

## 4. SAP-C02 — Solutions Architect Professional

**Bản chất:** thiết kế giải pháp **phức tạp, đa account, hybrid, migration, quy mô enterprise**. Câu hỏi dài, nhiều ràng buộc mâu thuẫn (rẻ NHƯNG phải HA NHƯNG ít vận hành...).

**Domain:**
```text
Design for Organizational Complexity  -> Organizations, SCP, Control Tower, RAM, multi-account (05)
Design for New Solutions              -> ghép mọi service, Well-Architected sâu (07)
Continuous Improvement                -> tối ưu hệ thống đang chạy (perf/cost/security)
Migration & Modernization             -> 7R, DMS/SCT/MGN, DataSync, Snow (07,03,04)
```
**Phải cực vững:**
- Multi-account governance (Organizations/SCP/Control Tower/Identity Center).
- Hybrid networking: Direct Connect, VPN, Transit Gateway, PrivateLink (01).
- DR chiến lược (4 mức, RPO/RTO), multi-region (Aurora Global, DynamoDB Global, Route53).
- Migration lớn & cost optimization ở quy mô.

**Chiến lược:** đọc kỹ đề dài, gạch từ khóa ràng buộc, loại đáp án vi phạm, chọn cái **cân bằng** đúng ưu tiên. Quản lý thời gian (câu dài).

---

## Kế hoạch ôn mẫu (áp dụng cho cert bất kỳ)
```text
Tuần 1-2 : Đọc hiểu concept (các file trong folder) + làm tay Free Tier từng service.
Tuần 3-4 : Học theo use case & kiến trúc (file 07) + ghi chú điểm bẫy.
Tuần 5   : Luyện đề bộ 1 -> chấm -> ĐỌC KỸ GIẢI THÍCH cả đúng lẫn sai -> note lỗ hổng.
Tuần 6   : Ôn lỗ hổng + luyện đề bộ 2,3 tới khi ổn định > 80%.
Trước thi: đọc lại phần "Tóm tắt điểm bẫy" cuối mỗi file.
```

## Tài nguyên tham khảo
- **AWS Official**: Skill Builder, Exam Guide + Sample Questions, Whitepapers (Well-Architected, DR, Security Best Practices), FAQ từng service.
- **Luyện đề**: Tutorials Dojo (Jon Bonso), AWS Official Practice.
- **Hands-on**: Free Tier, AWS Workshops, qwiklabs.
- **Đào sâu**: tài liệu trong folder này + đọc AWS Docs cho con số/limit mới nhất.

## Lời khuyên "sở hữu" thay vì chỉ "qua"
> Đừng học để đậu rồi quên. Với mỗi service, tự hỏi: *nó giải quyết vấn đề gì, khi nào KHÔNG dùng nó,
> nó đánh đổi cái gì.* Làm được project thật (như Capstone trong `devops-fast-track.md`) áp dụng
> các service này thì cert chỉ là hệ quả — và kiến thức là của bạn mãi mãi.
