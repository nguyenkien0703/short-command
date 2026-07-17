# Các Skill Claude Code để nắm bắt tin tức nhanh (cập nhật 07/2026)

Tổng hợp các skill/plugin giúp Claude Code tự động research tin tức mới, trending, và tổng hợp báo cáo — kiểu như `last30days`.

---

## 1. last30days — skill "quốc dân" cho research tin mới nhất ⭐ Nên cài đầu tiên

- **Repo**: https://github.com/mvanhorn/last30days-skill (tác giả Matt Van Horn, ~47k stars)
- **Phiên bản mới nhất**: v3.16.0 (07/2026) — cập nhật rất đều, 15 release chỉ trong 2 tháng
- **Làm gì**: Research bất kỳ chủ đề nào trong 30 ngày gần nhất trên **Reddit, X/Twitter, YouTube (kèm transcript), Hacker News, Polymarket, GitHub, arXiv, Techmeme, TikTok, Instagram, LinkedIn, Bluesky, Threads, StockTwits** + web search, xếp hạng theo engagement thật (upvote/like), rồi tổng hợp thành báo cáo có trích dẫn nguồn.
- **Tính năng mới đáng chú ý (v3.11 → v3.16)**:
  - **Discovery mode**: tự tìm chủ đề đang trending (không cần biết trước cần hỏi gì)
  - Nguồn miễn phí arXiv + Techmeme (CLI tự cài)
  - Reddit scoring theo upvote thật, "best comments" gom từ mọi nguồn
  - Chạy được trên Claude Code, Cursor, Copilot, Gemini CLI, Claude Desktop, Codex và 50+ host khác

**Cài đặt (Claude Code):**
```bash
# Cách 1 — marketplace (khuyến nghị, tự auto-update):
/plugin marketplace add mvanhorn/last30days-skill

# Cách 2 — Agent Skills CLI:
npx skills add mvanhorn/last30days-skill -g
```

**API key**: KHÔNG bắt buộc. Reddit, HN, Polymarket, GitHub chạy ngay không cần key. Tùy chọn thêm: `XAI_API_KEY` (X/Twitter), `yt-dlp` (YouTube), Brave Search (2.000 query/tháng miễn phí), ScrapeCreators (TikTok/Instagram/LinkedIn — 10.000 call miễn phí), Perplexity.

**Cách dùng:**
```
/last30days AI agent frameworks
/last30days what's trending in devops
```

---

## 2. tech-digest — điểm tin tech hằng ngày, zero dependencies

- **Repo**: https://github.com/camilleroux/tech-digest
- **Làm gì**: Gõ `/digest` → chạy script Python fetch song song RSS từ **Hacker News, Lobste.rs** và nhiều nguồn khác, lọc theo ngày, xếp hạng theo score, khử trùng lặp → trả về danh sách bài đáng đọc.
- **Điểm mạnh**: không cần API key, không dependencies, cực nhẹ — hợp làm "morning coffee digest".

```bash
npx skills add camilleroux/tech-digest -g
```

---

## 3. news-aggregator-skill — tổng hợp 28 nguồn tin (có cả nguồn Trung Quốc)

- **Trang**: https://claudemarketplaces.com/skills/cclank/news-aggregator-skill/news-aggregator-skill (hoặc https://clawhub.ai/cclank/skills/news-aggregator-skill)
- **Làm gì**: Kéo tin từ **28 nguồn**: Hacker News, GitHub Trending, Hugging Face papers, và các nền tảng Trung Quốc (Juejin, Zhihu...) → hợp nếu muốn theo dõi cả hệ sinh thái AI Trung Quốc.

---

## 4. rss-digest — biến RSS feed của bạn thành bản tin chọn lọc

- **Trang**: https://claudemarketplaces.com/skills/odysseus0/feed/rss-digest
- **Làm gì**: Đưa danh sách RSS feed bạn theo dõi → skill kéo bài mới, lọc "signal vs noise", tóm tắt những bài đáng đọc. Hợp nếu bạn đã có sẵn bộ feed (blog DevOps, K8s, AWS...).

---

## 5. AI-News-Fetcher — tin AI + GitHub trending dạng giao diện tương tác

- **Repo**: https://github.com/MohamedMamdouh18/AI-News-Fetcher
- **Làm gì**: Fetch tin AI mới + repo GitHub đang trending, render thành **React artifact tương tác** ngay trong chat.

---

## 6. hacker-news-reader — đọc HN trực tiếp

- **Trang**: https://mcpmarket.com/tools/skills/hacker-news-reader
- **Làm gì**: Lấy top stories, comments, user data từ Hacker News API real-time ngay trong Claude Code.

---

## Có sẵn trong Claude Code (không cần cài)

- **`deep-research`** — research đa nguồn sâu, fan-out search + verify chéo + báo cáo có trích dẫn. Hợp khi cần phân tích kỹ một chủ đề thay vì điểm tin nhanh.
- **`/morning`** — morning brief: bản tin buổi sáng dạng HTML, có thể đặt lịch chạy tự động các ngày trong tuần.

---

## Gợi ý combo cho nhu cầu "nắm tin nhanh"

| Nhu cầu | Dùng gì |
|---|---|
| "Chủ đề X có gì mới 30 ngày qua?" | `last30days` |
| Điểm tin tech mỗi sáng | `tech-digest` hoặc `/morning` |
| Theo dõi feed riêng của mình | `rss-digest` |
| Tìm chủ đề đang hot (chưa biết hỏi gì) | `last30days` discovery mode |
| Nghiên cứu sâu 1 chủ đề | `deep-research` (có sẵn) |

## Nơi khám phá thêm skill mới

- https://www.awesomeskills.dev — directory skill lớn
- https://claudemarketplaces.com — marketplace skill/plugin
- https://skillsplayground.com/best-claude-code-skills/ — top 25 skill 2026
- https://github.com/aradotso/trending-skills — skill đang trending
