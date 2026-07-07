# 01 — Networking & VPC (đào sâu)

> **Xương sống của AWS.** Không nắm chắc VPC thì mọi kiến trúc đều mù mờ. File này đi từ
> CIDR (toán địa chỉ IP) → VPC/Subnet → định tuyến → cổng ra vào → bảo mật → kết nối lai/liên vùng → DNS/CDN.
> Đây là phần nặng nhất trong SAA/SAP/SysOps.

## 1. CIDR — nền tảng địa chỉ IP (phải thấm)

**CIDR** (Classless Inter-Domain Routing) = cách viết một dải IP: `10.0.0.0/16`.

```text
IPv4 = 32 bit, chia 4 octet: 10.0.0.0  =  00001010.00000000.00000000.00000000
/N  = N bit đầu là phần MẠNG (cố định), 32-N bit sau là phần HOST (thay đổi được)

/16 -> 16 bit host -> 2^16 = 65,536 địa chỉ   (10.0.0.0 -> 10.0.255.255)
/24 -> 8  bit host -> 2^8  = 256 địa chỉ      (10.0.1.0 -> 10.0.1.255)
/28 -> 4  bit host -> 2^4  = 16 địa chỉ
```

**Quy tắc nhớ nhanh:** /N nhỏ = mạng to. Mỗi +1 vào /N → chia đôi số địa chỉ.
```text
/16 = 65536 | /17 = 32768 | /18 = 16384 | ... | /24 = 256 | /25 = 128 | ... | /28 = 16
```

**AWS lấy mất 5 IP mỗi subnet** (không dùng được cho instance):
```text
Subnet 10.0.1.0/24 (256 địa chỉ) -> chỉ 251 dùng được:
  .0   = Network address
  .1   = VPC router
  .2   = AWS DNS
  .3   = dành riêng (tương lai)
  .255 = Broadcast (AWS không hỗ trợ broadcast nhưng vẫn giữ chỗ)
```

**Ràng buộc CIDR của VPC:**
- VPC CIDR khối chính: từ **/16 tới /28** (đề nghị dùng dải private RFC1918: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`).
- **Subnet không được chồng lấn** nhau trong VPC.
- **Không đổi/thu nhỏ** CIDR chính sau khi tạo (chỉ **thêm secondary CIDR** được). → **Kế hoạch CIDR từ đầu**, chừa dư để mở rộng.
- Khi peering/kết nối nhiều VPC/on-prem: **CIDR các bên KHÔNG được trùng** (nếu trùng thì không route được).

**Bài tập bắt buộc:** chia `10.0.0.0/16` thành 3 AZ × (1 public + 1 private) subnet /20. → Tự tính dải từng subnet. Làm được là bạn nắm CIDR.

## 2. VPC — mạng riêng ảo

```text
VPC = mạng riêng của bạn trong 1 Region, cô lập logic khỏi VPC khác.
 - Có CIDR (vd 10.0.0.0/16), trải trên nhiều AZ của region.
 - Chứa subnet, route table, gateway, SG, NACL...
 - Default VPC: AWS tạo sẵn (mọi subnet public) — tiện test nhưng production nên tự tạo.
```
- **Tenancy**: default (shared hardware) hoặc dedicated (phần cứng riêng, đắt).
- Bật **DNS resolution / DNS hostnames** để instance có tên DNS.

## 3. Subnet — chia VPC theo AZ

```text
Subnet = 1 dải con của VPC CIDR, NẰM TRONG ĐÚNG 1 AZ (không trải nhiều AZ).
 -> Muốn HA: tạo subnet ở nhiều AZ.
```

**Public vs Private — khác biệt DUY NHẤT là route:**
```text
Public subnet  = route table có đường 0.0.0.0/0 -> Internet Gateway (IGW)
                 + instance có Public IP / Elastic IP
Private subnet = KHÔNG có đường trực tiếp ra IGW
                 (ra internet phải qua NAT Gateway)
```
> Bản thân subnet không có thuộc tính "public/private" — nó public **vì** route của nó trỏ ra IGW. Đây là điểm bẫy kinh điển.

- **Auto-assign public IP**: bật ở public subnet để instance tự có IP public.
- Kiến trúc chuẩn 3-tier: public subnet (ALB) → private subnet (app/EC2) → private subnet (DB/RDS).

## 4. Route Table — định tuyến

```text
Mỗi subnet gắn ĐÚNG 1 route table (nếu không gắn -> dùng main route table của VPC).
Route = "đích (CIDR) -> target (gateway/interface)". Chọn route KHỚP CỤ THỂ NHẤT (longest prefix match).
```
Ví dụ route table của public subnet:
```text
Destination      Target
10.0.0.0/16      local            <- traffic nội bộ VPC (luôn có, không xóa được)
0.0.0.0/0        igw-xxxx         <- ra internet
```
Private subnet:
```text
10.0.0.0/16      local
0.0.0.0/0        nat-xxxx         <- ra internet 1 chiều qua NAT
```

## 5. Cổng ra/vào Internet

| Thành phần | Vai trò | Ghi chú |
|-----------|---------|---------|
| **Internet Gateway (IGW)** | Cho VPC ra/vào internet 2 chiều | 1 IGW/VPC, gắn vào VPC; cần route + public IP |
| **NAT Gateway** | Cho private subnet ra internet **1 chiều** (outbound) | Managed, HA trong 1 AZ, **tính phí theo giờ + GB**; đặt ở **public** subnet |
| **NAT Instance** | (cũ) EC2 tự làm NAT | Rẻ hơn nhưng tự quản lý, tự HA — ít dùng |
| **Egress-only IGW** | Như NAT nhưng cho **IPv6** outbound-only | IPv6 vốn public nên cần cái này để chặn inbound |

**Điểm thi quan trọng:**
- NAT Gateway đặt ở **public subnet**, private subnet route `0.0.0.0/0 -> nat`. 
- **HA cho NAT:** đặt 1 NAT Gateway **mỗi AZ** (NAT chỉ sống trong 1 AZ; nếu AZ đó chết, subnet AZ khác vẫn ra được internet của riêng nó).
- NAT Gateway **tốn tiền** → workload nặng traffic ra ngoài cân nhắc **VPC Endpoint** (xem mục 8) để tới S3/DynamoDB không qua NAT.

## 6. Security Group (SG) vs Network ACL (NACL) — điểm bẫy kinh điển

| | **Security Group** | **Network ACL** |
|--|--------------------|-----------------|
| Cấp độ | **Instance** (ENI) | **Subnet** |
| Stateful? | **Stateful** — cho phép inbound thì reply outbound tự động được | **Stateless** — phải mở CẢ inbound VÀ outbound rời nhau |
| Rule | Chỉ **Allow** (mặc định deny hết) | Có cả **Allow và Deny** |
| Đánh giá | Đánh giá **tất cả** rule (union) | Theo **thứ tự số rule**, gặp match đầu tiên là áp dụng |
| Mặc định | Deny inbound, allow outbound | Default NACL: allow hết; NACL tự tạo: deny hết |

**Nhớ:** SG = tường lửa quanh **instance** (stateful, chỉ allow). NACL = tường lửa quanh **subnet** (stateless, có deny, theo thứ tự).
- SG có thể tham chiếu **SG khác** làm nguồn (vd: "cho phép từ SG của ALB") — rất mạnh cho micro-segmentation.
- **Ephemeral ports:** vì NACL stateless, reply đi ra dùng cổng tạm (1024-65535) → phải mở outbound ephemeral ports thủ công. Đây là bẫy hay gặp khi "SG đúng rồi mà vẫn không thông" → kiểm tra NACL.

## 7. Kết nối nhiều VPC / On-premises (hybrid)

```text
VPC Peering        : nối 1-1 hai VPC. KHÔNG bắc cầu (A-B, B-C không cho A-C).
                     Không chồng CIDR. Đơn giản, rẻ, nhưng rối khi nhiều VPC (n² kết nối).
Transit Gateway    : hub-and-spoke, nối HÀNG TRĂM VPC + on-prem tập trung. Có bắc cầu (route table riêng).
                     -> Lựa chọn cho mạng lớn/nhiều account (điểm thi SAP).
VPN (Site-to-Site) : nối on-prem <-> VPC qua internet, mã hóa IPsec. Rẻ, nhanh dựng, latency/băng thông
                     phụ thuộc internet.
Direct Connect (DX): đường cáp riêng vật lý tới AWS. Băng thông ổn định, latency thấp, đắt, dựng lâu (tuần/tháng).
                     -> Kết hợp VPN làm backup cho DX.
PrivateLink        : expose 1 service qua Endpoint riêng, không mở cả mạng (xem mục 8).
```

**Cây quyết định (điểm thi):**
- 2-3 VPC đơn giản → **Peering**.
- Nhiều VPC / nhiều account / cần scale → **Transit Gateway**.
- On-prem cần nối, chấp nhận qua internet, cần nhanh/rẻ → **Site-to-Site VPN**.
- On-prem cần băng thông ổn định + latency thấp + traffic lớn → **Direct Connect** (+ VPN backup).
- Chỉ cần expose 1 service (không mở cả VPC) → **PrivateLink**.

## 8. VPC Endpoints — tới AWS service không qua internet

```text
Gateway Endpoint    : cho S3 và DynamoDB. Thêm route vào route table. MIỄN PHÍ.
Interface Endpoint  : cho hầu hết service khác (dùng PrivateLink/ENI riêng trong subnet). Tính phí/giờ + GB.
```
**Vì sao quan trọng:** private subnet muốn gọi S3/DynamoDB thường phải đi qua **NAT (tốn tiền + qua internet công cộng của AWS)**. Dùng **Gateway Endpoint** → traffic đi trong mạng AWS, **miễn phí, an toàn hơn**. Đây là câu hỏi tối ưu chi phí/bảo mật rất hay gặp.

## 9. DNS — Route 53

- **Registrar + DNS service + health check**. TTL, các loại record: A, AAAA, CNAME, **Alias** (đặc thù AWS — trỏ tới ALB/CloudFront/S3, miễn phí, dùng được ở zone apex khác CNAME).
- **Routing policies (điểm thi):**
  | Policy | Dùng khi |
  |--------|----------|
  | Simple | 1 record đơn giản |
  | **Weighted** | chia traffic theo % (canary, A/B) |
  | **Latency** | route tới region có latency thấp nhất cho user |
  | **Failover** | active-passive (primary chết → chuyển sang secondary) |
  | **Geolocation** | route theo vị trí địa lý user (tuân thủ/nội dung theo vùng) |
  | Geoproximity | route theo khoảng cách + dịch chuyển bias |
  | Multivalue | trả nhiều IP + health check (kiểu round-robin có kiểm tra sức khỏe) |
- **Private Hosted Zone**: DNS nội bộ cho VPC.
- **Resolver (inbound/outbound endpoint):** phân giải DNS lai giữa on-prem ↔ VPC.

## 10. CDN & tăng tốc — CloudFront & Global Accelerator

- **CloudFront**: CDN, cache nội dung tại edge, giảm latency, giảm tải origin. Tích hợp **WAF, Shield, OAC** (chỉ CloudFront được đọc S3 private), TLS. Dùng cho web/static/streaming.
- **Global Accelerator**: 2 IP anycast tĩnh, route traffic qua **mạng backbone AWS** tới endpoint tốt nhất (TCP/UDP, không chỉ HTTP). Dùng cho app cần IP tĩnh, failover nhanh liên vùng, game/VoIP. → Khác CloudFront (CloudFront cache HTTP; GA tối ưu đường mạng cho mọi protocol).

## 11. Load Balancing (chi tiết ở [02-compute.md](./02-compute.md))
- **ALB** (L7, HTTP/HTTPS, path/host routing), **NLB** (L4, TCP/UDP, cực nhanh, IP tĩnh), **GWLB** (chèn appliance bảo mật). Đặt ở nhiều AZ để HA.

## 12. Giám sát mạng (SysOps hay hỏi)
- **VPC Flow Logs**: ghi lại traffic IP (ACCEPT/REJECT) → S3/CloudWatch. Debug "vì sao không thông" (thấy REJECT là do SG/NACL).
- **Reachability Analyzer**: phân tích đường đi logic giữa 2 điểm, chỉ ra chỗ chặn.
- **Traffic Mirroring**: copy traffic để phân tích sâu (IDS).

---

## Kiến trúc VPC mẫu (3-tier, HA)

```text
VPC 10.0.0.0/16, 2 AZ:
  AZ-a: public-a 10.0.0.0/24  (ALB, NAT-a)   | private-app-a 10.0.10.0/24 | private-db-a 10.0.20.0/24
  AZ-b: public-b 10.0.1.0/24  (ALB, NAT-b)   | private-app-b 10.0.11.0/24 | private-db-b 10.0.21.0/24

  Internet -> IGW -> ALB (public, 2 AZ)
                       -> EC2/ASG (private-app, 2 AZ)  --ra internet--> NAT (mỗi AZ) -> IGW
                             -> RDS Multi-AZ (private-db)
  private-app -> S3/DynamoDB qua Gateway Endpoint (không qua NAT)
```

## Tóm tắt điểm bẫy hay gặp
- Subnet "public" vì **route trỏ IGW** + có public IP, không phải thuộc tính riêng.
- AWS lấy **5 IP** mỗi subnet.
- CIDR không chồng lấn; peering/kết nối cần CIDR khác nhau; không thu nhỏ CIDR sau khi tạo.
- SG **stateful, chỉ allow, cấp instance**; NACL **stateless, có deny, cấp subnet, theo thứ tự** → nhớ ephemeral ports.
- NAT Gateway ở **public subnet**, HA cần **1 NAT/AZ**, và **tốn tiền** → dùng Gateway Endpoint cho S3/DynamoDB.
- Nhiều VPC/account → **Transit Gateway**; expose 1 service → **PrivateLink**.
- Route 53 **Alias** khác CNAME (dùng được ở apex, miễn phí, trỏ resource AWS).
