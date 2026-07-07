# 06 — Management, Monitoring & DevOps

> Trọng tâm cho **SysOps (CloudOps)** và **DevOps Professional**: quan sát, tự động hóa, IaC, CI/CD, vận hành.

## 1. CloudWatch — quan sát (metrics, logs, alarms)
```text
Metrics : số đo theo thời gian (CPU, latency...). Custom metric đẩy từ app.
Logs    : log tập trung (Log Group -> Log Stream). Logs Insights = query log bằng cú pháp riêng.
Alarms  : ngưỡng trên metric -> hành động (SNS notify, ASG scale, EC2 recover).
Dashboards, Events (EventBridge), Synthetics (canary), RUM, ServiceLens/X-Ray (trace).
```
- **Metric mặc định** của EC2 KHÔNG có RAM & disk-used → cần cài **CloudWatch Agent** (custom metric). Điểm thi kinh điển.
- **Alarm states**: OK / ALARM / INSUFFICIENT_DATA. Composite alarm gộp nhiều alarm.
- **EC2 recovery**: alarm `StatusCheckFailed_System` → tự recover instance (giữ IP/EBS).
- **Log retention** cấu hình được (mặc định giữ vô hạn → tốn tiền, nên đặt retention).
- **Metric filter**: biến pattern trong log thành metric (vd đếm "ERROR").

## 2. CloudTrail & Config (đã ở file 05, nhắc vai trò vận hành)
- **CloudTrail**: audit API — điều tra "ai đổi cái gì". Gửi tới S3 + CloudWatch Logs.
- **Config**: theo dõi thay đổi cấu hình + đánh giá rule tuân thủ + **auto-remediation** (SSM Automation).

## 3. EventBridge — sự kiện & tự động hóa
```text
Event bus: nhận event (từ AWS service, app, SaaS) -> rule khớp -> target (Lambda, SNS, SQS, Step Functions...).
Scheduler: cron/rate cho tác vụ định kỳ (thay CloudWatch Events cũ).
```
- Dùng để: phản ứng tự động (vd GuardDuty finding → Lambda cô lập instance), điều phối event-driven, cron serverless.

## 4. Systems Manager (SSM) — vận hành EC2/hybrid (SysOps nặng)
| Tính năng | Làm gì |
|-----------|--------|
| **Session Manager** | shell vào EC2 **không cần SSH/bastion/port 22** (qua IAM, có log) — bảo mật hơn |
| **Run Command** | chạy lệnh trên nhiều instance cùng lúc |
| **Patch Manager** | vá OS theo lịch/baseline (nhớ: patch OS EC2 là việc của bạn) |
| **Parameter Store** | lưu config/secret |
| **State Manager** | giữ cấu hình mong muốn |
| **Automation** | runbook tự động hóa (remediation, tác vụ vận hành) |
| **Inventory** | kiểm kê phần mềm/cấu hình |
- **Session Manager** thay bastion host → không mở SSH ra internet (câu hỏi bảo mật hay gặp).

## 5. IaC — CloudFormation & CDK
```text
CloudFormation: mô tả hạ tầng bằng YAML/JSON (declarative). Stack = tập resource quản lý cùng nhau.
 - Template -> Stack. Change Sets (xem trước thay đổi). Drift detection (phát hiện sửa tay).
 - StackSets: deploy 1 template ra NHIỀU account/region.
 - Nested stacks, Parameters, Mappings, Outputs, Conditions.
 - Rollback tự động khi tạo lỗi. DeletionPolicy: Retain/Snapshot để bảo vệ data.
CDK: viết hạ tầng bằng ngôn ngữ lập trình (TS/Python...) -> tổng hợp ra CloudFormation. Tái sử dụng, logic.
SAM: framework serverless (CloudFormation rút gọn cho Lambda/API).
```
- Terraform (đa cloud) cũng phổ biến — DevOps thực tế hay dùng; cert AWS thiên về CloudFormation/CDK.
- **Điểm thi:** cần deploy nhiều account/region nhất quán → **StackSets**. Bảo vệ DB khi xóa stack → **DeletionPolicy: Retain/Snapshot**.

## 6. CI/CD — bộ Code* (DevOps Professional lõi)
```text
CodeCommit   : Git repo managed (giống GitHub private). [đang giảm ưu tiên — thực tế hay dùng GitHub]
CodeBuild    : build & test managed (buildspec.yml). Trả theo phút build.
CodeDeploy   : deploy ra EC2/ASG/Lambda/ECS với chiến lược:
                 - EC2/ASG: In-place hoặc Blue/Green
                 - Lambda/ECS: Canary / Linear / All-at-once (dịch traffic dần)
                 - appspec.yml + hooks (BeforeInstall, AfterInstall...)
CodePipeline : điều phối pipeline (Source -> Build -> Test -> Deploy), tích hợp approval thủ công.
CodeArtifact : lưu package (npm/pip/maven). CodeGuru: review code + profiler bằng ML.
```
### Chiến lược deploy (điểm thi)
| Chiến lược | Cơ chế | Rủi ro |
|-----------|--------|--------|
| **In-place / Rolling** | cập nhật lần lượt | rẻ, có downtime nhẹ, rollback chậm |
| **Blue/Green** | dựng môi trường mới song song, chuyển traffic | an toàn, rollback tức thì, tốn gấp đôi tài nguyên |
| **Canary** | chuyển % nhỏ traffic trước, theo dõi, rồi tăng | phát hiện lỗi sớm |
| **Linear** | tăng traffic đều theo bước | |
- **Rollback tự động** khi CloudWatch alarm kích hoạt trong lúc deploy → best practice DevOps.

## 7. Cost & Optimization (SysOps)
- **Cost Explorer / Budgets / CUR** (chi tiết), **Compute Optimizer** (right-size), **Trusted Advisor** (best practice + tiết kiệm + service limits), **Savings Plans/RI recommendations**.
- **Tagging** để phân bổ chi phí theo team/project.

## 8. Vận hành khác
- **Trusted Advisor**: kiểm tra 5 nhóm (cost, performance, security, fault tolerance, service limits).
- **Health Dashboard**: sự cố AWS ảnh hưởng account bạn.
- **Service Quotas**: xem & xin tăng limit (nhiều sự cố do đụng quota — điểm SysOps).
- **Resource Groups + Tag Editor**: nhóm & quản lý resource theo tag.

---

## Pipeline CI/CD mẫu (DevOps)
```text
Dev push -> CodeCommit/GitHub
  -> CodePipeline:
       Source  -> CodeBuild (test + build image -> ECR + scan)
               -> Manual Approval (với prod)
               -> CodeDeploy (Blue/Green ra ECS/Lambda, canary 10% -> 100%)
       Rollback tự động nếu CloudWatch alarm (5xx/latency) kích hoạt.
Observability: CloudWatch (metric/log/alarm) + X-Ray (trace) + EventBridge (tự động phản ứng).
IaC: toàn bộ hạ tầng bằng CloudFormation/CDK, deploy đa account bằng StackSets.
```

## Tóm tắt điểm bẫy
- EC2 **RAM/disk metric cần CloudWatch Agent** (không có mặc định).
- **Session Manager** thay bastion (không mở SSH).
- **StackSets** cho đa account/region; **DeletionPolicy** bảo vệ data khi xóa stack.
- CodeDeploy Blue/Green = rollback tức thì; Canary/Linear dịch traffic dần + rollback theo alarm.
- **CloudTrail** (ai gọi API) vs **Config** (tuân thủ cấu hình) vs **CloudWatch** (metric/log).
- Nhiều sự cố do **Service Quotas** — kiểm tra limit trước khi scale.
