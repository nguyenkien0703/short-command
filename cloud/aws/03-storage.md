# 03 — Storage (S3, EBS, EFS, FSx, Backup)

> Ba loại lưu trữ cốt lõi: **Object** (S3), **Block** (EBS), **File** (EFS/FSx). Biết chọn đúng loại là kỹ năng thi & thực chiến.

## 0. Ba mô hình lưu trữ
```text
Object (S3)  : lưu file như "vật thể" có key + metadata, truy cập qua API/HTTP. Không sửa 1 phần (ghi đè cả object). Vô hạn.
Block (EBS)  : ổ đĩa ảo gắn vào 1 EC2, như ổ cứng. Hệ điều hành format & mount. Gắn 1 AZ.
File (EFS/FSx): hệ thống file mạng (NFS/SMB) nhiều máy mount cùng lúc.
```

## 1. S3 — Simple Storage Service (đối tượng)

```text
Bucket (tên DUY NHẤT toàn cầu) -> chứa Object (key = "đường dẫn" + value + metadata).
Bucket thuộc 1 Region (data nằm ở region đó dù namespace tên là global).
Durability 99.999999999% (11 số 9) — gần như không mất data.
```

### Storage Classes (điểm thi rất nặng — chọn theo tần suất truy cập)
| Class | Dùng khi | Đặc điểm |
|-------|----------|----------|
| **Standard** | Truy cập thường xuyên | rẻ về truy cập, đắt lưu trữ; nhiều AZ |
| **Intelligent-Tiering** | Không đoán được pattern | tự chuyển tầng theo truy cập, có phí monitor nhỏ — an toàn |
| **Standard-IA** | Ít truy cập, cần ngay | rẻ lưu, phí truy cập/retrieval; nhiều AZ |
| **One Zone-IA** | Ít truy cập, tái tạo được | 1 AZ (rẻ hơn, kém bền hơn) |
| **Glacier Instant** | Archive nhưng cần ngay (ms) | rẻ, truy cập tức thì |
| **Glacier Flexible** | Archive, chờ phút–giờ | rất rẻ |
| **Glacier Deep Archive** | Archive lâu, chờ tới 12h | RẺ NHẤT — backup pháp lý dài hạn |

- **Lifecycle policy**: tự chuyển class theo tuổi (vd 30 ngày → IA, 90 ngày → Glacier, 365 → Deep Archive/xóa). Câu hỏi tối ưu chi phí kinh điển.

### Bảo mật S3
- **Mặc định private**; **Block Public Access** (bật ở account/bucket — chặn lộ dữ liệu).
- Kiểm soát truy cập: **IAM policy** (ai) + **Bucket policy** (resource-based, theo bucket) + ACL (cũ, tránh) + **Access Points** (nhiều điểm truy cập theo use case).
- **Mã hóa:** SSE-S3 (AWS quản key), **SSE-KMS** (dùng KMS, có audit/kiểm soát), SSE-C (bạn tự đưa key), client-side. Bật **mã hóa mặc định** cho bucket.
- **Pre-signed URL**: cấp quyền truy cập tạm 1 object (upload/download) không cần công khai.
- Truy cập private từ VPC → **Gateway Endpoint** (không qua internet, miễn phí).

### Tính năng S3 hay hỏi
- **Versioning**: giữ nhiều phiên bản, chống xóa/ghi đè nhầm. Bật để dùng với replication & MFA Delete.
- **Replication**: **CRR** (Cross-Region, DR/tuân thủ/độ trễ) & **SRR** (Same-Region, gộp log/khác account). Cần bật versioning.
- **S3 Object Lock / WORM**: chống xóa (compliance, giữ bất biến trong X ngày).
- **Static website hosting**: host web tĩnh (kèm CloudFront + OAC cho private origin).
- **Event Notifications**: trigger Lambda/SQS/SNS khi có object (vd upload ảnh → resize).
- **Transfer Acceleration**: upload nhanh qua edge; **Multipart upload**: file lớn chia phần song song.
- **S3 Select**: query 1 phần object (SQL) không tải cả file.
- **Storage Lens / Analytics**: nhìn tổng quan & tối ưu.

## 2. EBS — Elastic Block Store (block, cho EC2)

```text
Ổ đĩa ảo gắn vào EC2, nằm trong 1 AZ (chỉ gắn EC2 CÙNG AZ). Bền (replicate trong AZ).
Snapshot -> lưu vào S3 (incremental), dùng để backup/copy/migrate qua AZ/Region.
```
### Loại volume
| Loại | Bản chất | Dùng cho |
|------|----------|----------|
| **gp3 / gp2** | SSD tổng quát | mặc định, boot, hầu hết workload (gp3 tách IOPS/throughput khỏi size) |
| **io2 / io1** | SSD IOPS cao (Provisioned IOPS) | DB đòi IOPS lớn, io2 Block Express cực cao |
| **st1** | HDD throughput | big data, log tuần tự (không boot) |
| **sc1** | HDD lạnh, rẻ | truy cập hiếm |

- **gp3** thường tốt nhất về giá/hiệu năng cho phần lớn workload.
- **EBS Multi-Attach** (io1/io2): 1 volume gắn nhiều EC2 cùng AZ (cần app cluster-aware).
- **Encryption** (KMS): mã hóa at-rest + snapshot; bật mặc định được.
- **Instance Store** (khác EBS): ổ NVMe **vật lý gắn liền instance** — cực nhanh nhưng **ephemeral** (mất khi stop/terminate). Dùng cache/scratch, không lưu data quan trọng.

## 3. EFS — Elastic File System (file, NFS, Linux)
```text
NFS được nhiều EC2 (và Lambda/ECS) ở NHIỀU AZ mount CÙNG LÚC. Tự co giãn dung lượng.
```
- Dùng khi: nhiều instance chia sẻ file (web content, CMS, shared home), cần POSIX.
- **Storage classes**: Standard & **IA** (lifecycle chuyển file ít dùng → rẻ). One Zone cho rẻ hơn.
- **Performance/Throughput mode**: General Purpose / Max IO; Bursting / Provisioned / Elastic.
- Khác EBS: EBS = 1 AZ, gắn 1 (hoặc ít) EC2; EFS = nhiều AZ, nhiều máy, đắt hơn.

## 4. FSx — file system chuyên dụng
| FSx | Dùng cho |
|-----|----------|
| **FSx for Windows File Server** | SMB, Active Directory — workload Windows |
| **FSx for Lustre** | HPC, ML, xử lý dữ liệu tốc độ cực cao (tích hợp S3) |
| **FSx for NetApp ONTAP** | tính năng NetApp (multi-protocol, snapshot) |
| **FSx for OpenZFS** | ZFS |

## 5. Data transfer & Hybrid storage
- **Storage Gateway**: cầu nối on-prem ↔ AWS storage (File/Volume/Tape Gateway) — cache local, backup lên S3/Glacier.
- **DataSync**: đồng bộ khối lượng lớn on-prem ↔ AWS (nhanh, có lịch).
- **Snow Family** (Snowcone/Snowball/Snowmobile): chuyển petabyte offline bằng thiết bị vật lý khi mạng không đủ.
- **AWS Backup**: quản lý backup tập trung cho EBS/EFS/RDS/DynamoDB/FSx... theo policy + cross-region/cross-account (điểm thi SysOps/SAP).

---

## Cây quyết định chọn storage
```text
Lưu file/object truy cập qua API, web, backup, data lake  -> S3 (chọn class theo tần suất)
Ổ đĩa cho 1 EC2 (boot, DB tự quản)                        -> EBS (gp3 mặc định, io2 nếu IOPS cao)
Nhiều Linux EC2 mount chung, nhiều AZ                      -> EFS
Windows/SMB + AD                                          -> FSx for Windows
HPC/ML tốc độ cực cao                                     -> FSx for Lustre
Cache tốc độ tối đa, chấp nhận mất khi stop                -> Instance Store
Backup tập trung nhiều service                            -> AWS Backup
Chuyển petabyte offline                                   -> Snow Family
```

## Tóm tắt điểm bẫy
- S3 durability 11 số 9; chọn **class theo tần suất truy cập** + **Lifecycle** để tối ưu chi phí.
- Deep Archive = rẻ nhất, chờ tới 12h.
- EBS gắn **1 AZ**; snapshot để qua AZ/Region. Instance Store là **ephemeral**.
- EFS = nhiều máy/nhiều AZ (Linux NFS); FSx Windows = SMB/AD.
- Versioning bắt buộc cho Replication; Object Lock cho WORM/compliance.
- Private S3 từ VPC → Gateway Endpoint (miễn phí, không qua NAT).
