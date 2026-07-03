# Practice Questions — kiểu đề thật, có giải thích

> Câu hỏi tình huống theo phong cách đề AWS (SAA/SysOps/DevOps/SAP). Tự trả lời TRƯỚC khi xem đáp án.
> Giá trị nằm ở phần **giải thích** — hiểu vì sao đúng và vì sao các lựa chọn khác sai.
> Map với các file: [00](./00-fundamentals.md) [01](./01-networking-vpc.md) [02](./02-compute.md) [03](./03-storage.md) [04](./04-databases.md) [05](./05-security-iam.md) [06](./06-management-devops.md) [07](./07-architecture-patterns.md)

---

## Networking (file 01)

**Q1.** Instance trong private subnet cần tải update từ internet nhưng KHÔNG được nhận kết nối từ internet. Cách nào?
- A. Gắn Elastic IP cho instance
- B. Đặt NAT Gateway ở public subnet, route 0.0.0.0/0 của private subnet trỏ tới NAT
- C. Thêm Internet Gateway vào private subnet
- D. Dùng VPC Peering

<details><summary>Đáp án</summary>

**B.** NAT Gateway cho outbound-only. A/C khiến instance nhận được inbound (sai yêu cầu). IGW không "thêm vào subnet" — subnet public là do route. D không liên quan internet.
</details>

**Q2.** SG của web server đã cho phép inbound 443, nhưng traffic vẫn bị chặn. SG outbound để mặc định (allow all). Nghi ngờ ở đâu?
- A. NACL của subnet đang chặn (nhớ NACL stateless → cần mở cả ephemeral ports outbound)
- B. IGW chưa gắn
- C. Route table thiếu local route
- D. SG cần thêm rule outbound 443

<details><summary>Đáp án</summary>

**A.** SG stateful nên reply tự thông; nghi ngờ NACL (stateless) chặn ephemeral ports (1024-65535) chiều outbound. C sai vì local route luôn tồn tại. D sai vì SG outbound đã allow all.
</details>

**Q3.** Công ty có 50 VPC ở nhiều account cần kết nối lẫn nhau và tới on-prem, quản lý tập trung. Chọn gì?
- A. VPC Peering full-mesh
- B. Transit Gateway
- C. Nhiều Internet Gateway
- D. PrivateLink cho từng cặp

<details><summary>Đáp án</summary>

**B.** Transit Gateway = hub-and-spoke cho nhiều VPC/account + on-prem, quản lý tập trung. A (peering) không bắc cầu, 50 VPC cần ~1225 kết nối — không khả thi. D chỉ expose 1 service.
</details>

**Q4.** Private subnet gọi S3 rất nhiều, chi phí NAT Gateway (theo GB) tăng cao. Giảm cách nào?
<details><summary>Đáp án</summary>

**Gateway VPC Endpoint cho S3** — traffic tới S3 đi trong mạng AWS, **miễn phí**, không qua NAT. (DynamoDB cũng có Gateway Endpoint.)
</details>

---

## Compute (file 02)

**Q5.** App cần load balancer định tuyến `/api/*` tới service A và `/img/*` tới service B trên cùng domain. Dùng gì?
- A. NLB   B. ALB   C. CLB   D. Global Accelerator
<details><summary>Đáp án</summary>

**B. ALB** — routing tầng 7 theo path/host. NLB là L4 (không đọc URL). GA tối ưu đường mạng, không route theo path.
</details>

**Q6.** Batch job xử lý ảnh, chạy hàng giờ, chịu được gián đoạn (tự resume). Tối ưu chi phí?
<details><summary>Đáp án</summary>

**Spot Instances** (qua ASG mixed hoặc AWS Batch) — rẻ tới ~90%, hợp workload chịu gián đoạn. Không dùng On-Demand (đắt) hay Reserved (cam kết dài, phí phạm cho batch).
</details>

**Q7.** API traffic bùng theo sự kiện, phần lớn thời gian idle, team muốn ít vận hành nhất. Chọn compute?
<details><summary>Đáp án</summary>

**Lambda** (+ API Gateway) — serverless, tự scale, trả theo dùng, không quản server. Hợp tải bùng/không đều. (Nếu chạy >15' hoặc cần trạng thái → cân nhắc Fargate.)
</details>

**Q8.** ASG thay instance khi EC2 status check fail, nhưng app bị treo (process sống, không phản hồi) mà instance không bị thay. Sửa?
<details><summary>Đáp án</summary>

Bật **ELB health check** trên ASG (thay vì chỉ EC2 health check). ELB kiểm tra ở tầng ứng dụng → phát hiện app treo → ASG thay instance.
</details>

---

## Storage (file 03)

**Q9.** Data truy cập nhiều trong 30 ngày đầu, sau đó hiếm khi đọc nhưng phải giữ 7 năm cho tuân thủ, chi phí thấp nhất. Thiết kế?
<details><summary>Đáp án</summary>

S3 **Lifecycle**: Standard (30 ngày) → chuyển **Glacier Deep Archive** (rẻ nhất) cho phần còn lại tới 7 năm. Cân nhắc **Object Lock (WORM)** cho compliance chống xóa.
</details>

**Q10.** Nhiều EC2 (Linux) ở các AZ khác nhau cần đọc/ghi CÙNG một bộ file chia sẻ. Dùng gì?
- A. EBS Multi-Attach   B. EFS   C. Instance Store   D. S3
<details><summary>Đáp án</summary>

**B. EFS** — NFS, nhiều EC2/nhiều AZ mount cùng lúc. EBS Multi-Attach chỉ trong 1 AZ + cần app cluster-aware. S3 không phải file system POSIX.
</details>

**Q11.** Cần ổ đĩa cho database tự quản trên EC2, đòi IOPS rất cao và ổn định. Chọn EBS loại nào?
<details><summary>Đáp án</summary>

**io2 (hoặc io2 Block Express)** — Provisioned IOPS SSD cho IOPS cao/ổn định. gp3 hợp phần lớn workload nhưng io2 cho DB IOPS-intensive.
</details>

---

## Databases (file 04)

**Q12.** RDS MySQL bị quá tải vì quá nhiều truy vấn ĐỌC (báo cáo). Cần scale đọc. Làm gì?
- A. Bật Multi-AZ   B. Thêm Read Replica   C. Tăng instance size   D. Bật backup
<details><summary>Đáp án</summary>

**B. Read Replica** — offload read (async). **Multi-AZ là để HA, standby KHÔNG phục vụ read** — đây là bẫy kinh điển. C có thể giúp tạm nhưng không scale ngang như replica.
</details>

**Q13.** Ứng dụng cần DB NoSQL scale khổng lồ, latency mili giây ổn định, không muốn quản server. Multi-region active-active. Chọn?
<details><summary>Đáp án</summary>

**DynamoDB + Global Tables** — serverless, latency ms, multi-region active-active. (Aurora Global thiên về đọc cross-region; DynamoDB Global Tables là active-active thật.)
</details>

**Q14.** Cần giảm tải DB bằng cache in-memory có HA và failover. Chọn?
<details><summary>Đáp án</summary>

**ElastiCache for Redis** (có replication + Multi-AZ failover). Memcached không có HA/persistence. Nếu cache cần bền như primary → MemoryDB.
</details>

---

## Security (file 05)

**Q15.** Lambda cần đọc secret DB, secret phải tự động xoay định kỳ. Dùng gì?
- A. Hardcode trong biến môi trường   B. Secrets Manager   C. Parameter Store standard   D. S3
<details><summary>Đáp án</summary>

**B. Secrets Manager** — có **rotation tự động** (tích hợp RDS). Parameter Store rẻ nhưng không tự rotate. Hardcode là anti-pattern.
</details>

**Q16.** Cần chặn TẤT CẢ account trong Organization tạo resource ngoài region ap-southeast-1, kể cả admin. Dùng gì?
<details><summary>Đáp án</summary>

**SCP (Service Control Policy)** với condition `aws:RequestedRegion` — guardrail cấp Organization, giới hạn cả admin. IAM policy lẻ không đủ (admin có thể tự sửa).
</details>

**Q17.** Cần vào shell EC2 trong private subnet để debug, nhưng chính sách bảo mật cấm mở port 22 và cấm bastion. Cách nào?
<details><summary>Đáp án</summary>

**SSM Session Manager** — shell qua IAM, không cần SSH/port 22/bastion, có log audit. (Cần SSM agent + instance có role + endpoint/NAT tới SSM.)
</details>

**Q18.** Muốn biết "ai đã xóa security group lúc 3h sáng". Dùng service nào?
<details><summary>Đáp án</summary>

**CloudTrail** — audit mọi API call (ai, khi nào, từ đâu). (Config cho biết cấu hình *thay đổi ra sao*; CloudTrail cho biết *ai gọi API nào*.)
</details>

---

## Management / DevOps (file 06)

**Q19.** CloudWatch không hiển thị metric RAM và disk-used của EC2. Vì sao và sửa thế nào?
<details><summary>Đáp án</summary>

EC2 **không phát metric RAM/disk-used mặc định** (hypervisor không thấy trong OS). Cài **CloudWatch Agent** để đẩy custom metric. Bẫy SysOps kinh điển.
</details>

**Q20.** Cần deploy phiên bản mới với khả năng **rollback tức thì** nếu lỗi, chấp nhận tốn gấp đôi tài nguyên tạm thời. Chiến lược?
<details><summary>Đáp án</summary>

**Blue/Green** — dựng môi trường mới song song, chuyển traffic; lỗi thì chuyển ngược ngay. (Canary/Linear dịch dần; In-place rollback chậm.)
</details>

**Q21.** Cần deploy 1 template hạ tầng ra 20 account và 3 region nhất quán. Dùng gì?
<details><summary>Đáp án</summary>

**CloudFormation StackSets** — deploy 1 template ra nhiều account/region tập trung (qua Organizations).
</details>

**Q22.** Khi GuardDuty phát hiện instance bị compromise, muốn TỰ ĐỘNG cô lập nó ngay. Kiến trúc?
<details><summary>Đáp án</summary>

**GuardDuty finding → EventBridge rule → Lambda/SSM Automation** đổi SG sang "isolation SG" (chặn hết) + snapshot để forensic. Đây là auto-remediation event-driven (DevOps Pro).
</details>

---

## Architecture / DR (file 07)

**Q23.** Business chịu được mất tối đa vài phút data và khôi phục trong vài phút khi mất cả region. Chi phí hợp lý. Chiến lược DR?
<details><summary>Đáp án</summary>

**Warm Standby** — bản thu nhỏ chạy sẵn ở region 2, scale up khi failover (RTO/RPO phút). Multi-site active-active tốt hơn nhưng đắt hơn nhiều; Pilot Light chậm hơn (phải khởi động).
</details>

**Q24.** Cần tách producer và consumer để chịu tải bùng và không mất message khi consumer chậm/lỗi. Dùng gì?
<details><summary>Đáp án</summary>

**SQS** (queue buffer) + **DLQ** cho message lỗi. Scale consumer (ASG) theo độ dài queue. Nếu cần fan-out tới nhiều service → **SNS → nhiều SQS**.
</details>

**Q25.** Web toàn cầu, cần giảm latency cho user khắp thế giới + giảm tải origin cho nội dung tĩnh. Dùng gì?
<details><summary>Đáp án</summary>

**CloudFront** (CDN cache tại edge) trước origin (S3/ALB). Kèm **Route 53 latency routing** nếu có nhiều region origin. WAF trên CloudFront cho bảo mật.
</details>

---

## Cách dùng bộ câu hỏi này
```text
1. Che đáp án, tự trả lời + tự giải thích LÝ DO.
2. Mở đáp án: nếu đúng nhưng lý do sai -> vẫn coi là chưa nắm.
3. Câu sai -> quay lại file tương ứng đọc lại mục đó -> note vào phần "điểm bẫy".
4. Lặp lại sau vài ngày (spaced repetition).
5. Sau đó chuyển sang bộ đề đầy đủ (Tutorials Dojo / AWS Official Practice).
```

> Mục tiêu không phải nhớ đáp án, mà là **nhận diện pattern**: đề cho từ khóa nào → map sang service/kiến trúc nào. Khi phản xạ được pattern, bạn đã sở hữu kiến thức.
