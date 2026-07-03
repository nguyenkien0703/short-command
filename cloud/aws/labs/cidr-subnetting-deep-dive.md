# Lab — CIDR & Subnetting đào sâu (làm được là sở hữu networking)

> Phần nhiều người sợ nhất nhưng là nền tảng của mọi thứ. Học tới mức **tự tính trong đầu**.
> Cấu trúc: hiểu nhị phân → công thức → bài mẫu có lời giải → tự luyện → thiết kế VPC thật.

---

## 1. Nền: IP là 32 bit nhị phân

```text
IPv4 = 4 octet × 8 bit = 32 bit.
10.0.5.12  =  00001010 . 00000000 . 00000101 . 00001100

Mỗi octet 8 bit, giá trị bit từ trái: 128 64 32 16 8 4 2 1
Ví dụ 5 = 00000101 (4+1),  200 = 11001000 (128+64+8).
```

**Bảng giá trị bit — thuộc lòng:**
```text
Bit thứ:   1    2    3    4    5   6   7   8
Giá trị: 128   64   32   16    8   4   2   1
```

## 2. CIDR /N nghĩa là gì

```text
/N  = N bit đầu (từ trái) là phần MẠNG (network) — CỐ ĐỊNH cho mọi host trong mạng.
      (32 - N) bit còn lại là phần HOST — thay đổi để đánh số từng máy.

Số địa chỉ trong khối = 2^(32-N).
```
**Bảng /N ↔ số địa chỉ (nhớ vài mốc, còn lại nhân/chia đôi):**
```text
/8  = 16,777,216      /24 = 256
/16 = 65,536          /25 = 128
/17 = 32,768          /26 = 64
/18 = 16,384          /27 = 32
/19 = 8,192           /28 = 16
/20 = 4,096           /29 = 8
/21 = 2,048           /30 = 4
/22 = 1,024           /31 = 2
/23 = 512             /32 = 1  (1 host duy nhất)
```
Quy tắc: **mỗi +1 vào /N → chia đôi. Mỗi -1 → gấp đôi.** `/24`=256, nên `/26`=256/4=64.

## 3. Subnet Mask — cách nhìn khác của /N

```text
/N   ->  N bit 1 rồi (32-N) bit 0  ->  đổi ra dạng thập phân = subnet mask
/24  =  11111111.11111111.11111111.00000000  =  255.255.255.0
/26  =  11111111.11111111.11111111.11000000  =  255.255.255.192
/16  =  11111111.11111111.00000000.00000000  =  255.255.0.0
```
**Octet "thú vị" (octet chứa ranh giới):** giá trị mask octet = 256 − (kích thước block trong octet đó).
- /26 → octet 4 có 2 bit mạng (128+64=192) → mask 255.255.255.**192**, block size = 256−192 = **64**.

## 4. Công thức tính 1 subnet (dùng cho mọi bài)

Cho một subnet CIDR, cần tìm: **Network address, dải host, Broadcast, block size**.

```text
Bước 1: block size = 2^(32-N) nhưng tính TRONG octet thú vị = 256 - mask_octet.
Bước 2: Network address = bội số của block size gần nhất <= IP (ở octet thú vị).
Bước 3: Broadcast = Network + block size - 1.
Bước 4: Host đầu = Network + 1,  Host cuối = Broadcast - 1.  (trừ AWS lấy thêm — xem mục 6)
```

## 5. Bài mẫu có lời giải

### Bài 1: `192.168.1.0/26` có bao nhiêu địa chỉ, dải nào?
```text
/26 -> 32-26 = 6 bit host -> 2^6 = 64 địa chỉ.
Octet thú vị = octet 4, mask = 255.255.255.192, block size = 256-192 = 64.
Các mạng /26 trong 192.168.1.x:  .0, .64, .128, .192
Với .0/26:  Network=192.168.1.0  Broadcast=192.168.1.63  Host=.1 -> .62
```

### Bài 2: IP `10.0.132.50/20` thuộc mạng nào?
```text
/20 -> octet thú vị là octet 3 (20 = 16 + 4 bit vào octet 3).
mask octet 3 = 11110000 = 240 -> block size = 256-240 = 16.
Bội số 16 gần nhất <= 132 -> 128.  => Network = 10.0.128.0/20
Broadcast = 10.0.143.255 (128 + 16 -1 = 143 ở octet 3). Dải: 10.0.128.0 - 10.0.143.255.
Vậy 10.0.132.50 nằm trong 10.0.128.0/20. ✔
```

### Bài 3: Cần subnet chứa ĐÚNG 500 host, dùng /mấy?
```text
Cần >= 500 địa chỉ dùng được. /23 = 512 (đủ), /24 = 256 (thiếu).
-> Chọn /23. (Nhớ trừ hao AWS 5 IP -> 512-5 = 507 vẫn đủ 500.)
```

### Bài 4: Chia `10.0.0.0/24` thành 4 subnet bằng nhau
```text
4 subnet -> cần thêm 2 bit mạng (2^2=4) -> /24 + 2 = /26. block size = 64.
  10.0.0.0/26   (.0   - .63)
  10.0.0.64/26  (.64  - .127)
  10.0.0.128/26 (.128 - .191)
  10.0.0.192/26 (.192 - .255)
```

## 6. AWS lấy 5 IP mỗi subnet (cực quan trọng)

```text
Subnet 10.0.1.0/24 (256 địa chỉ) -> dùng được 251:
  10.0.1.0   Network address        (không dùng)
  10.0.1.1   VPC router
  10.0.1.2   Amazon DNS
  10.0.1.3   Dự phòng tương lai
  10.0.1.255 Broadcast              (không dùng)
```
→ Khi tính "subnet đủ cho X instance", nhớ **trừ 5**. Ví dụ /28 = 16 địa chỉ → chỉ **11** instance.

## 7. Thiết kế VPC thật (bài lớn — làm bằng tay)

### Đề: Thiết kế VPC cho app 3-tier, HA 3 AZ, chừa dư mở rộng.
Yêu cầu: mỗi AZ có 1 public (ALB/NAT) + 1 private-app + 1 private-db. Tổng 9 subnet.

**Lời giải mẫu — VPC `10.0.0.0/16` (65536 địa chỉ):**
```text
Chọn mỗi subnet /20 (4096 địa chỉ - rộng rãi, dễ đọc). /16 có 16 khối /20.

AZ-a:  public-a      10.0.0.0/20    (10.0.0.0   - 10.0.15.255)
       private-app-a 10.0.16.0/20   (10.0.16.0  - 10.0.31.255)
       private-db-a  10.0.32.0/20   (10.0.32.0  - 10.0.47.255)
AZ-b:  public-b      10.0.48.0/20
       private-app-b 10.0.64.0/20
       private-db-b  10.0.80.0/20
AZ-c:  public-c      10.0.96.0/20
       private-app-c 10.0.112.0/20
       private-db-c  10.0.128.0/20

Đã dùng tới 10.0.143.255. Còn 10.0.144.0 - 10.0.255.255 (>28k địa chỉ) CHỪA cho mở rộng. ✔
```
> Bài học thiết kế: **chọn block đều, dễ đọc, chừa dư lớn**. Đừng chia sát nút — CIDR không thu nhỏ được, mở rộng chỉ thêm secondary CIDR (phức tạp hơn).

### Biến thể tiết kiệm (subnet nhỏ hơn):
```text
Nếu app nhỏ, dùng /24 mỗi subnet (256 địa chỉ, dùng được 251):
  public: /24, private-app: /23 (512, cho ASG scale nhiều), private-db: /24.
Nguyên tắc: subnet chứa workload co giãn (ASG) nên rộng hơn.
```

## 8. Ràng buộc CIDR khi kết nối (peering/TGW/VPN/DX)

```text
- CIDR các VPC/on-prem KẾT NỐI với nhau KHÔNG được chồng lấn.
  VPC A 10.0.0.0/16 và VPC B 10.0.0.0/16 -> KHÔNG peer được (trùng).
  -> Quy hoạch dải riêng cho mỗi VPC/môi trường ngay từ đầu:
       Prod    10.0.0.0/16
       Staging 10.1.0.0/16
       Dev     10.2.0.0/16
       On-prem 172.16.0.0/16
- Route phải cụ thể; longest prefix match thắng.
```

## 9. Bài tự luyện (tự giải, đáp án cuối file)

```text
Q1. 172.16.0.0/22 có bao nhiêu địa chỉ? Dải từ đâu tới đâu?
Q2. IP 10.10.10.10/27 thuộc network nào? Broadcast? Host cuối?
Q3. Cần subnet cho tối đa 30 instance EC2 (nhớ AWS lấy 5). Dùng /mấy?
Q4. Chia 192.168.0.0/24 thành 8 subnet bằng nhau — liệt kê network của từng cái.
Q5. VPC 10.0.0.0/16. Bạn đã có subnet 10.0.0.0/17. Subnet /17 thứ hai (không chồng) là gì?
Q6. Muốn peer 2 VPC: A=10.0.0.0/16, B=10.0.0.0/24. Có được không? Vì sao?
```

---

## Đáp án bài tự luyện
```text
Q1. /22 = 2^10 = 1024 địa chỉ. 172.16.0.0 - 172.16.3.255 (block size octet 3 = 4).
Q2. /27 block size = 32. Bội số 32 <= 10 là 0 -> Network 10.10.10.0/27.
    Broadcast = 10.10.10.31. Host cuối = 10.10.10.30.
Q3. 30 instance + 5 AWS = cần >= 35 địa chỉ. /27=32 (thiếu), /26=64 (đủ). -> /26.
Q4. 8 subnet -> +3 bit -> /27, block size 32:
    192.168.0.0, .32, .64, .96, .128, .160, .192, .224  (đều /27).
Q5. 10.0.128.0/17 (khối /17 thứ hai: 10.0.0.0/17 và 10.0.128.0/17).
Q6. KHÔNG. B (10.0.0.0/24) nằm TRONG dải A (10.0.0.0/16) -> chồng lấn -> không route được.
```

## Checklist "đã sở hữu CIDR"
```text
[ ] Nhìn /N là biết ngay số địa chỉ (không cần tra bảng).
[ ] Tính được network/broadcast/dải host của 1 subnet trong đầu.
[ ] Chia 1 khối thành N subnet bằng nhau nhanh.
[ ] Thiết kế được sơ đồ VPC nhiều AZ có chừa dư.
[ ] Nhớ AWS lấy 5 IP và luôn trừ khi tính sức chứa.
[ ] Biết vì sao 2 CIDR chồng lấn thì không peer được.
```
