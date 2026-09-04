# Đầu tư & Tích lũy tài sản — Bộ skills và lộ trình bắt đầu

> Tài liệu tổng hợp: các bộ **Claude skill về tài chính/đầu tư** đáng dùng, cộng
> với một **lộ trình bắt đầu đầu tư khôn ngoan** cho người mới muốn hiểu thị
> trường và tích lũy tài sản từ sớm.
>
> ⚠️ **Miễn trừ:** Đây là tài liệu giáo dục, **không phải lời khuyên đầu tư**.
> Không công cụ nào dự đoán được giá. Tự kiểm chứng và cân nhắc tư vấn từ chuyên
> gia được cấp phép trước khi xuống tiền thật.

---

## 1. Các bộ skill đã cài sẵn trong repo này

16 skill starter đã được đưa vào [`.claude/skills/`](../.claude/skills/) — xem
[README của bộ skill](../.claude/skills/README.md) để biết danh sách đầy đủ và
cách dùng. Tất cả chạy được **không cần API trả phí**. Tóm tắt theo nhóm:

- **Nền tảng tính toán:** `time-value-of-money`, `return-calculations`,
  `statistics-fundamentals`.
- **Tài chính cá nhân & tích lũy sớm:** `emergency-fund`, `savings-goals`,
  `finance-psychology`.
- **Quản trị danh mục:** `asset-allocation`, `diversification`, `rebalancing`,
  `tax-efficiency`, `investment-policy`.
- **Hiểu thị trường & kỷ luật giao dịch:** `market-breadth-analyzer`,
  `uptrend-analyzer`, `position-sizer`, `trader-memory-core`, `signal-postmortem`.

## 2. Các nguồn skill/plugin tài chính đáng chú ý (khảo sát)

| Nguồn | Nội dung | Ghi chú |
|-------|----------|---------|
| **Plugin `Finance`** (marketplace `knowledge-work-plugins`) | Plugin tài chính chính thức trong workspace của bạn | **Đã bật sẵn** — dùng được ngay |
| **[JoelLewis/finance_skills](https://github.com/JoelLewis/finance_skills)** | 84 skill / 7 plugin domain: return, TVM, thống kê, quản lý tài sản (DCF, VaR, Black-Litterman, risk parity), tuân thủ, tư vấn, vận hành giao dịch | MIT. Cài: `npx skills add JoelLewis/finance_skills` |
| **[tradermonty/claude-trading-skills](https://github.com/tradermonty/claude-trading-skills)** | 50+ skill cho nhà đầu tư/trader cá nhân: độ rộng thị trường, screener (CANSLIM, VCP), position sizing, nhật ký giao dịch, backtest | MIT. Có lộ trình "5 skill không cần API" |
| **[Claude Cookbook — Financial applications](https://platform.claude.com/cookbook/skills-notebooks-02-skills-financial-applications)** | Ví dụ chính thức của Anthropic: dashboard tài chính, phân tích danh mục qua Excel/PPT/PDF | Tài liệu học tốt |
| **[Anthropic financial modeling skill]** (đóng gói production) | Định giá / mô hình tài chính chuyên sâu cho công việc nghiêm túc | Cho phân tích định giá |
| **[claudemarketplaces.com/skills/category/finance](https://claudemarketplaces.com/skills/category/finance)** | Danh mục cộng đồng: crypto, phân tích cổ phiếu, DeFi, danh mục | Duyệt thêm |

## 3. Lộ trình bắt đầu đầu tư khôn ngoan (thứ tự ưu tiên)

Nguyên tắc: **an toàn nền tảng trước, thị trường sau; kỷ luật trước, lợi nhuận sau.**

### Bước 0 — Trước khi đầu tư một đồng nào
- **Trả nợ lãi cao** (thẻ tín dụng, vay tiêu dùng) trước — không kênh đầu tư nào
  ăn chắc bằng việc xóa khoản nợ lãi 20–30%/năm.
- **Quỹ dự phòng** 3–6 tháng chi phí, để nơi thanh khoản cao (→ skill `emergency-fund`).
- **Xác định mục tiêu & thời gian** (hưu trí 20 năm? mua nhà 5 năm?) → skill `savings-goals`.

### Bước 1 — Hiểu sức mạnh của "bắt đầu sớm"
- Chạy skill `time-value-of-money` để thấy lãi kép: mỗi năm bắt đầu sớm hơn có
  thể chênh lệch rất lớn ở đích. **Thời gian trong thị trường > canh thời điểm.**

### Bước 2 — Chọn cách đầu tư phù hợp người mới
Khôn ngoan nhất cho phần lớn người mới **không phải** chọn cổ phiếu riêng lẻ, mà là:
- **DCA (đầu tư đều đặn hàng tháng)** một số tiền cố định vào **quỹ chỉ số / ETF
  chi phí thấp, đa dạng rộng** — giảm rủi ro canh thời điểm, kỷ luật tự động.
- **Phân bổ tài sản** theo khẩu vị rủi ro & thời gian (→ `asset-allocation`), rồi
  **đa dạng hóa** (→ `diversification`). Nguyên tắc chung: càng xa mục tiêu, càng
  chịu được nhiều cổ phiếu; càng gần, càng tăng phần an toàn (trái phiếu/tiền mặt).
- Ưu tiên **chi phí thấp** và **hiệu quả thuế** (→ `tax-efficiency`) — phí và
  thuế bào mòn lợi nhuận dài hạn nhiều hơn bạn nghĩ.

### Bước 3 — Viết ra quy tắc của chính bạn
- Dùng `investment-policy` lập một IPS ngắn: mục tiêu lợi nhuận, mức rủi ro chịu
  được, tỉ trọng mục tiêu, quy tắc mua/bán. Có luật trước → không quyết định theo
  cảm xúc lúc thị trường biến động.

### Bước 4 — Duy trì & chấm điểm trung thực
- **Tái cân bằng** định kỳ (theo lịch hoặc theo ngưỡng) → `rebalancing`.
- Đo lợi nhuận thật bằng `return-calculations` (đừng tự lừa mình bằng con số cảm tính).

### Bước 5 — Học thị trường mà không mạo hiểm tiền thật
- Đọc bối cảnh vĩ mô bằng `market-breadth-analyzer` / `uptrend-analyzer`.
- Nếu muốn thử giao dịch chủ động: **paper trading** trước (ví dụ Alpaca paper
  miễn phí), luyện `position-sizer` (rủi ro ≤1% mỗi lệnh), ghi nhật ký bằng
  `trader-memory-core`, và rút bài học bằng `signal-postmortem`. Chỉ chuyển sang
  tiền thật khi đã có quy trình ổn định qua nhiều tháng.

### Bước 6 — Chống tự phá hoại
- Đọc `finance-psychology` xuyên suốt: sợ mất mát, tự tin thái quá, tâm lý đám
  đông là nguyên nhân thua lỗ phổ biến hơn cả chọn sai tài sản.

## 4. Gợi ý "bắt đầu ngay hôm nay"

Một cách khởi động cụ thể, ít rủi ro nhất cho người mới:

1. **Hôm nay:** mở phiên Claude Code tại repo này, hỏi
   *"quỹ dự phòng của tôi nên bao nhiêu tháng và để ở đâu?"* → kích hoạt
   `emergency-fund`. Hoàn thiện quỹ này trước.
2. **Tuần này:** dùng `savings-goals` + `time-value-of-money` để đặt mục tiêu và
   thấy tác động của việc bắt đầu sớm; chốt một **số tiền DCA hàng tháng** vừa sức.
3. **Tháng này:** dùng `asset-allocation` + `diversification` chọn một **danh mục
   ETF chỉ số chi phí thấp** đơn giản (ví dụ phối hợp cổ phiếu toàn cầu + trái
   phiếu theo khẩu vị), rồi tự động hóa việc mua đều hàng tháng.
4. **Song song (không tiền thật):** mở tài khoản **paper trading** để luyện đọc
   thị trường và kỷ luật sizing với bộ skill trading — tách hoàn toàn khỏi danh
   mục dài hạn ở bước 3.

Cách này giúp bạn **tích lũy tài sản đều đặn từ sớm** (phần cốt lõi, thụ động,
chi phí thấp) trong khi **học hiểu thị trường an toàn** (phần thực hành, không
mạo hiểm) — đúng mục tiêu bạn đặt ra.

---

## Nguồn tham khảo
- [Claude Skills for financial applications — Claude Cookbook](https://platform.claude.com/cookbook/skills-notebooks-02-skills-financial-applications)
- [Finance & Trading Skills — Claude Marketplaces](https://claudemarketplaces.com/skills/category/finance)
- [Top 8 Claude Skills for Finance and Quantitative Developers — Snyk](https://snyk.io/articles/top-claude-skills-finance-quantitative-developers/)
- [Portfolio Analysis Skill for Claude Code — Matt Stockton](https://mattstockton.com/2026/02/12/portfolio-analysis-skill-for-claude-code.html)
- [JoelLewis/finance_skills — GitHub](https://github.com/JoelLewis/finance_skills)
- [tradermonty/claude-trading-skills — GitHub](https://github.com/tradermonty/claude-trading-skills)
- [Investment Analysis Claude Code Skill — mcpmarket.com](https://mcpmarket.com/tools/skills/investment-analysis-financial-valuation)
- [Claude Skills for Investing — digitally-create.com](https://www.digitally-create.com/claude-skills/investing)
