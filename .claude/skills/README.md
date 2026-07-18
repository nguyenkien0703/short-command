# Finance & Investing Skills

Bộ **16 Claude skills** phục vụ mục tiêu: hiểu thị trường, đầu tư khôn ngoan và
tích lũy tài sản từ sớm. Tất cả đều chạy được **không cần API trả phí** (một vài
skill dùng CSV công khai / dữ liệu bạn tự nhập).

## Cách dùng

Các skill nằm ở `.claude/skills/<tên-skill>/SKILL.md`. Trong một phiên Claude
Code chạy tại repo này, Claude sẽ tự động nạp skill khi câu hỏi khớp mô tả
(`description`) của nó — ví dụ hỏi "tôi nên để quỹ dự phòng bao nhiêu tháng?" sẽ
kích hoạt `emergency-fund`. Bạn cũng có thể gọi thẳng tên skill.

Nhiều skill kèm `scripts/*.py` (tính toán thuần) và `references/*.md` (kiến thức
nền). Chạy script bằng `python3 <path>` khi cần con số cụ thể.

## Danh mục skill

### 1. Nền tảng tính toán (bắt đầu từ đây)
| Skill | Dùng khi |
|-------|----------|
| `time-value-of-money` | Chiết khấu dòng tiền, PV/FV/NPV/IRR, sức mạnh lãi kép khi đầu tư sớm |
| `return-calculations` | Đo lợi nhuận thật (TWR, IRR theo dòng tiền, CAGR), so sánh các cách tính |
| `statistics-fundamentals` | Phân phối lợi nhuận, tương quan, hồi quy, kiểm định — nền cho quản trị rủi ro |

### 2. Xây nền tài chính cá nhân & tích lũy sớm
| Skill | Dùng khi |
|-------|----------|
| `emergency-fund` | Quỹ dự phòng nên bao nhiêu tháng, để ở đâu (làm TRƯỚC khi đầu tư) |
| `savings-goals` | Lập & theo dõi mục tiêu: hưu trí, mua nhà, học phí; tỉ lệ tiết kiệm cần thiết |
| `finance-psychology` | Nhận diện & tránh thiên kiến hành vi (sợ mất, tự tin thái quá, tâm lý đám đông) |

### 3. Đầu tư khôn ngoan & quản trị danh mục
| Skill | Dùng khi |
|-------|----------|
| `asset-allocation` | Phân bổ vốn theo lớp tài sản, glide path, mean-variance, risk parity |
| `diversification` | Đa dạng hóa qua tương quan, biên hiệu quả, đa dạng theo nhân tố |
| `rebalancing` | Khi nào & cách tái cân bằng (theo lịch/ngưỡng), tối ưu thuế & phí |
| `tax-efficiency` | Tối đa lợi nhuận sau thuế: asset location, thu hoạch lỗ, thứ tự rút tiền |
| `investment-policy` | Viết Tuyên bố Chính sách Đầu tư (IPS): mục tiêu, khẩu vị rủi ro, ràng buộc |

### 4. Hiểu thị trường & kỷ luật giao dịch (không cần API trả phí)
| Skill | Dùng khi |
|-------|----------|
| `market-breadth-analyzer` | Điểm sức khỏe độ rộng thị trường 0–100 từ CSV công khai |
| `uptrend-analyzer` | Chẩn đoán môi trường thị trường (breadth, luân chuyển ngành, động lượng) |
| `position-sizer` | Tính số cổ phiếu nên mua theo rủi ro mỗi lệnh (fixed %, ATR, Kelly) |
| `trader-memory-core` | Ghi & theo dõi luận điểm đầu tư từ ý tưởng → đóng lệnh + postmortem |
| `signal-postmortem` | Rút bài học sau mỗi lệnh: false positive, cơ hội bỏ lỡ, sai môi trường |

## Lộ trình gợi ý cho người mới
1. **Tuần 1–2 — Nền móng:** `emergency-fund` → `savings-goals` → `time-value-of-money`
   (hiểu vì sao bắt đầu sớm quan trọng nhờ lãi kép).
2. **Tuần 3–4 — Thiết kế danh mục:** `asset-allocation` → `diversification` →
   `investment-policy` (viết ra quy tắc của chính bạn).
3. **Duy trì:** `rebalancing` + `tax-efficiency` định kỳ; `return-calculations`
   để chấm điểm trung thực.
4. **Học thị trường an toàn:** `market-breadth-analyzer` / `uptrend-analyzer` để
   đọc bối cảnh; luyện `position-sizer` + `trader-memory-core` + `signal-postmortem`
   ở chế độ *paper trading* trước khi dùng tiền thật.
5. **Chống tự phá hoại:** đọc `finance-psychology` xuyên suốt.

## Nguồn & giấy phép (attribution)
Các skill được tuyển chọn (vendored) từ hai dự án mã nguồn mở **giấy phép MIT**:

- **finance_skills** — Joel Lewis (MIT). Cung cấp: `return-calculations`,
  `time-value-of-money`, `statistics-fundamentals`, `emergency-fund`,
  `savings-goals`, `asset-allocation`, `diversification`, `rebalancing`,
  `finance-psychology`, `tax-efficiency`, `investment-policy`.
  https://github.com/JoelLewis/finance_skills
- **claude-trading-skills** — TraderMonty (MIT). Cung cấp:
  `market-breadth-analyzer`, `uptrend-analyzer`, `position-sizer`,
  `trader-memory-core`, `signal-postmortem`.
  https://github.com/tradermonty/claude-trading-skills

Đây chỉ là *tập con starter*. Repo gốc còn nhiều skill nâng cao (DCF, VaR,
Black-Litterman, backtest, options, screener CANSLIM/VCP...). Cài đầy đủ:

```bash
# finance_skills (marketplace / npx)
npx skills add JoelLewis/finance_skills

# claude-trading-skills: tải .skill trong thư mục skill-packages/ rồi upload,
# hoặc copy thư mục skill vào .claude/skills/
```

> **Lưu ý quan trọng:** Đây là công cụ giáo dục và phân tích, **không phải lời
> khuyên đầu tư**. Không skill nào dự đoán được giá. Luôn tự kiểm chứng và cân
> nhắc tham vấn chuyên gia tài chính được cấp phép trước khi xuống tiền thật.
