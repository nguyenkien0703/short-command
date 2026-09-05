# Risk Acceptance / Đề nghị ngoại lệ — template + ví dụ đã điền

> Tài liệu bạn dùng khi **không thể sửa ngay** một rủi ro, nhưng vẫn phải đi tiếp một cách chuyên nghiệp.
> Đây là thứ phân biệt rõ nhất một kỹ sư bảo mật **chín** với một người chỉ biết chặn build.
>
> Đọc kèm: [`../career-devsecops-finance.md`](../career-devsecops-finance.md) · [`threat-model-template.md`](./threat-model-template.md)

## Vì sao phải có văn bản này

```text
Không có quy trình ngoại lệ  ->  business sẽ VƯỢT MẶT bạn (tắt gate, deploy tay, xin đặc cách miệng)
                             ->  bạn mất quyền kiểm soát VÀ mất uy tín
                             ->  đến kỳ kiểm toán không ai biết còn tồn tại rủi ro gì

Có quy trình ngoại lệ        ->  rủi ro vẫn còn NHƯNG: được ghi nhận, có người chịu trách nhiệm,
                                 có biện pháp bù đắp, có hạn, và có kế hoạch xử lý dứt điểm
                             ->  bạn vẫn kiểm soát được bức tranh, kiểm toán chấp nhận được
```

**Nguyên tắc bất di bất dịch — thiếu một trong bốn thì đừng ký:**

| Nguyên tắc | Vì sao |
|---|---|
| **1. Có chủ sở hữu rủi ro** — người có thẩm quyền *kinh doanh*, không phải kỹ sư | Người hưởng lợi từ việc đi nhanh phải là người ký chịu rủi ro. Bảo mật **không** ký thay |
| **2. Có thời hạn và TỰ HẾT HIỆU LỰC** | Ngoại lệ không hạn = rủi ro vĩnh viễn được hợp thức hóa. Đây là cách nợ bảo mật tích tụ |
| **3. Có biện pháp giảm thiểu tạm thời ĐÃ triển khai** | "Sẽ làm sau" không phải giảm thiểu. Phải có cái gì đó đang chạy |
| **4. Có kế hoạch xử lý dứt điểm, có tên người và mốc** | Không có kế hoạch = đây không phải ngoại lệ, đây là đầu hàng |

> ⚠️ **Sai lầm kinh điển:** kỹ sư bảo mật tự ký chấp nhận rủi ro để "không cản trở anh em".
> Khi sự cố xảy ra, bạn là người chịu. Hãy để **đúng người** ký — đó không phải là né tránh,
> đó là đưa quyết định về đúng nơi có thẩm quyền quyết định.

## Khi nào dùng — và khi nào KHÔNG

```text
DÙNG khi:  CVE chưa có bản vá tương thích · vendor không hỗ trợ cấu hình an toàn
           · chuẩn mới ban hành, hệ thống cũ chưa kịp đáp ứng · thời điểm không cho phép
             (đóng băng thay đổi cuối năm) · chi phí khắc phục vượt xa giá trị rủi ro

KHÔNG DÙNG khi: chỉ vì "gấp quá" mà chưa thử tìm cách · rủi ro ở mức Nghiêm trọng và có
           đường khai thác thật từ Internet · vi phạm trực tiếp điều khoản pháp lý bắt buộc
           (không ai trong tổ chức có thẩm quyền "chấp nhận" việc vi phạm quy định NHNN)
```

> Điểm cuối rất quan trọng ở ngành tài chính: **có những thứ không được phép chấp nhận rủi ro.**
> Nếu điều khoản là bắt buộc theo luật/thông tư, con đường duy nhất là khắc phục hoặc dừng dịch vụ.
> Biết phân biệt "rủi ro có thể chấp nhận" và "vi phạm quy định" là dấu hiệu của Lead.

---

## Template (copy phần dưới)

```markdown
# RA-<năm>-<số> — <tiêu đề ngắn: cái gì được miễn trừ, cho hệ thống nào>

| | |
|---|---|
| **Mã** | RA-YYYY-NNN |
| **Ngày lập** | YYYY-MM-DD |
| **Người lập** | <tên, chức danh> |
| **Hệ thống** | <tên> · **Cấp độ ATTT**: <1–5> · **Chủ sở hữu hệ thống**: <tên> |
| **Điều khoản/control bị miễn trừ** | <TT 09 mục X / PCI DSS 6.3.3 / chính sách nội bộ số Y> |
| **Trạng thái** | Chờ duyệt / Đang hiệu lực / Đã hết hạn / Đã xử lý dứt điểm |

## 1. Bối cảnh
<Chuyện gì đang xảy ra, vì sao không tuân thủ được ngay. Viết cho người không rành kỹ thuật đọc hiểu.>

## 2. Đánh giá rủi ro
| Hạng mục | Đánh giá |
|---|---|
| Kịch bản khai thác cụ thể | <mô tả bằng câu chuyện, không phải mô tả lỗ hổng> |
| Có đường khai thác từ Internet? | Có / Không — <bằng chứng xác minh> |
| Dữ liệu bị ảnh hưởng | <PII / dữ liệu thẻ / sinh trắc học / không có> |
| Khả năng xảy ra | Thấp / Trung bình / Cao — <lý do> |
| Ảnh hưởng nếu xảy ra | Thấp / Trung bình / Cao / Nghiêm trọng — <lý do, quy ra tiền/pháp lý nếu được> |
| **Rủi ro tồn dư sau giảm thiểu** | **<mức>** |

## 3. Biện pháp giảm thiểu tạm thời (ĐÃ triển khai — ghi rõ ngày)
| # | Biện pháp | Trạng thái | Bằng chứng |
|---|---|---|---|
| 1 | | Đã triển khai DD/MM | <link dashboard / ID rule / commit> |

## 4. Vì sao không khắc phục dứt điểm ngay
<Ràng buộc thật: kỹ thuật / vendor / thời gian / chi phí. Nêu cả phương án đã cân nhắc và loại bỏ.>

## 5. Kế hoạch xử lý dứt điểm
| Mốc | Việc | Người phụ trách | Hạn |
|---|---|---|---|
| Kiểm tra giữa kỳ | | | |
| Hoàn thành | | | |

## 6. Thời hạn ngoại lệ
Có hiệu lực đến **DD/MM/YYYY**. **Tự động hết hiệu lực, KHÔNG tự động gia hạn.**
Gia hạn (nếu cần) phải lập hồ sơ mới với đánh giá rủi ro cập nhật.

## 7. Điều kiện hủy ngoại lệ ngay lập tức
<Ví dụ: xuất hiện mã khai thác công khai; có dấu hiệu bị dò quét; thay đổi kiến trúc làm lộ ra Internet.>

## 8. Phê duyệt
| Vai trò | Tên | Ý kiến | Ngày |
|---|---|---|---|
| Người lập (tuyến 1) | | | |
| Chủ sở hữu hệ thống — **người chịu rủi ro** | | | |
| An toàn thông tin (tuyến 2) | | | |
| Quản lý rủi ro hoạt động | | | |
| <Cấp cao hơn nếu rủi ro tồn dư ≥ Cao> | | | |
```

---

## Ví dụ ĐÃ ĐIỀN

### RA-2026-014 — Ngoại lệ: `payment-report-svc` chạy thư viện có CVE mức High chưa có bản vá

| | |
|---|---|
| **Mã** | RA-2026-014 |
| **Ngày lập** | 2026-09-05 |
| **Người lập** | <tên>, Senior DevSecOps |
| **Hệ thống** | `payment-report-svc` · **Cấp độ ATTT**: 3 · **Chủ sở hữu**: Giám đốc Trung tâm Thanh toán |
| **Control bị miễn trừ** | Chính sách nội bộ ATTT-07: cổng CI chặn build khi có lỗ hổng ≥ High |
| **Trạng thái** | Đang hiệu lực |

#### 1. Bối cảnh
`payment-report-svc` sinh báo cáo đối soát nội bộ hằng ngày cho đội Vận hành thanh toán. Dịch vụ phụ thuộc
thư viện `libreport-x` v1.2 để xuất file. Phiên bản này có CVE mức High. Bản vá v2.0 thay đổi API không tương
thích ngược, cần sửa và kiểm thử lại toàn bộ luồng sinh báo cáo — ước tính 6 tuần. Cổng CI hiện chặn build,
khiến các bản vá **khác** của dịch vụ này cũng không lên được production.

#### 2. Đánh giá rủi ro
| Hạng mục | Đánh giá |
|---|---|
| Kịch bản khai thác cụ thể | Kẻ tấn công cần đưa được file đầu vào độc hại vào tiến trình sinh báo cáo để gây thực thi mã |
| Có đường khai thác từ Internet? | **Không** — dịch vụ chỉ nhận dữ liệu từ hàng đợi nội bộ, không có endpoint hướng ngoài. Đã xác minh bằng rà soát NetworkPolicy + Ingress ngày 04/09/2026 (xem SEC-1120) |
| Dữ liệu bị ảnh hưởng | Dữ liệu giao dịch tổng hợp (không chứa PAN, không chứa sinh trắc học); có mã khách hàng nội bộ |
| Khả năng xảy ra | **Thấp** — cần trước tiên chiếm được một dịch vụ nội bộ khác để đẩy dữ liệu vào hàng đợi |
| Ảnh hưởng nếu xảy ra | **Cao** — thực thi mã trong vùng ứng dụng, có thể dùng làm bàn đạp sang hệ thống khác |
| **Rủi ro tồn dư sau giảm thiểu** | **Trung bình** |

#### 3. Biện pháp giảm thiểu tạm thời (đã triển khai)
| # | Biện pháp | Trạng thái | Bằng chứng |
|---|---|---|---|
| 1 | NetworkPolicy giới hạn egress của pod chỉ tới hàng đợi + kho lưu báo cáo | Đã triển khai 03/09 | commit `a1b2c3d`, namespace `payments` |
| 2 | Rule SIEM cảnh báo khi tiến trình con bất thường được sinh ra trong pod này | Đã triển khai 04/09 | Rule ID `SIEM-2291` |
| 3 | Chuyển pod sang node pool riêng, không dùng chung với dịch vụ hướng Internet | Đã triển khai 04/09 | `nodeSelector: pool=internal-batch` |
| 4 | Ghi ACL chặt trên hàng đợi đầu vào: chỉ 2 dịch vụ được publish | Đã triển khai 05/09 | cấu hình MQ, ticket SEC-1121 |

#### 4. Vì sao không khắc phục dứt điểm ngay
- Nâng lên `libreport-x` v2.0 phá vỡ API sinh file; cần viết lại lớp xuất báo cáo + đối chiếu kết quả với
  báo cáo cũ trong ít nhất 2 chu kỳ EOD để đảm bảo số liệu không lệch.
- **Đã cân nhắc và loại bỏ:** (a) vá thủ công thư viện — không có bản vá backport, tự vá tạo nhánh riêng
  khó bảo trì; (b) đổi sang thư viện khác — thay đổi lớn hơn nâng cấp; (c) hạ ngưỡng cổng CI toàn tổ chức —
  không chấp nhận được, ảnh hưởng mọi dịch vụ.
- Thời điểm: tháng 12 là mùa đóng băng thay đổi cuối năm tài chính, nên phải hoàn thành trước 30/11.

#### 5. Kế hoạch xử lý dứt điểm
| Mốc | Việc | Người phụ trách | Hạn |
|---|---|---|---|
| M1 | Viết lại lớp xuất báo cáo trên v2.0, chạy song song môi trường kiểm thử | <tên dev lead> | 15/10/2026 |
| M2 | Đối chiếu số liệu 2 chu kỳ EOD, nghiệp vụ xác nhận | <tên đội đối soát> | 05/11/2026 |
| M3 | Triển khai production, gỡ ngoại lệ | <tên> | 20/11/2026 |

#### 6. Thời hạn ngoại lệ
Có hiệu lực đến **30/11/2026**. Tự động hết hiệu lực. Không tự động gia hạn.
*(Chọn 30/11 thay vì 31/12 để còn đệm trước kỳ đóng băng thay đổi cuối năm.)*

#### 7. Điều kiện hủy ngoại lệ ngay lập tức
- Xuất hiện mã khai thác công khai cho CVE này, **hoặc**
- SIEM ghi nhận cảnh báo `SIEM-2291` kích hoạt, **hoặc**
- Kiến trúc thay đổi khiến dịch vụ nhận dữ liệu từ nguồn ngoài vùng tin cậy hiện tại.

#### 8. Phê duyệt
| Vai trò | Tên | Ý kiến | Ngày |
|---|---|---|---|
| Người lập (tuyến 1) | <tên>, Senior DevSecOps | Đề nghị phê duyệt với 4 biện pháp bù đắp | 05/09/2026 |
| Chủ sở hữu hệ thống | Giám đốc TT Thanh toán | | |
| An toàn thông tin (tuyến 2) | <tên>, Phòng ATTT | | |
| Quản lý rủi ro hoạt động | <tên> | | |

---

## Vận hành sổ ngoại lệ (việc của Lead)

Một ngoại lệ đơn lẻ là chuyện của Senior. **Quản trị cả tập ngoại lệ là chuyện của Lead.**

```text
1. SỔ TẬP TRUNG — mọi ngoại lệ ở một chỗ, có trạng thái, có hạn. Đừng để rải rác trong email/chat.
2. TỰ ĐỘNG NHẮC — cảnh báo trước hạn 30 ngày cho người phụ trách và chủ sở hữu rủi ro.
3. TỰ ĐỘNG CHẶN LẠI — hết hạn thì cổng CI/policy siết lại, không cần ai nhớ. Đây là điểm mấu chốt:
   ngoại lệ phải hết hiệu lực bằng CƠ CHẾ, không bằng thiện chí.
4. BÁO CÁO HẰNG QUÝ — số ngoại lệ đang mở, số quá hạn, xu hướng, top 3 rủi ro tồn dư lớn nhất.
   Đưa lên hội đồng rủi ro. Đây là cách bảo mật có tiếng nói ở cấp cao.
5. CHỈ SỐ CẦN THEO DÕI:
   - Số ngoại lệ QUÁ HẠN  -> mục tiêu: 0. Đây là chỉ số kỷ luật của tổ chức.
   - Tuổi trung bình của ngoại lệ đang mở -> tăng dần = nợ bảo mật đang tích tụ.
   - Tỉ lệ ngoại lệ được gia hạn -> cao = ước lượng kế hoạch không thực tế, cần chấn chỉnh.
```

> **Mẹo thực chiến:** khi trình bày với lãnh đạo, đừng nói "chúng ta có 23 ngoại lệ".
> Hãy nói: *"23 ngoại lệ đang mở, 2 quá hạn — cả 2 thuộc hệ thống kênh số, chủ sở hữu là anh X,
> tôi đề xuất đưa vào ưu tiên quý này."* Cụ thể, có người, có đề xuất — đó là ngôn ngữ của Lead.
