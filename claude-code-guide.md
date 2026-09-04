# Claude Code — Sổ tay lệnh, tính năng & tips dùng hiệu quả

> Tổng hợp toàn bộ command, tính năng và mẹo dùng Claude Code (CLI agent code của Anthropic)
> để tận dụng tối đa trong công việc: từ lệnh gõ tay hằng ngày, custom slash command,
> subagent, hook, MCP, đến quản lý chi phí và làm việc song song nhiều task.
>
> Đối tượng: dev/DevOps/Platform Engineer đã cài `claude` và muốn dùng thành thạo,
> không chỉ hỏi-đáp đơn giản.

---

## 📑 Mục lục

- [1. Claude Code là gì & cài đặt](#1-claude-code-là-gì--cài-đặt)
- [2. Cách khởi động & CLI flags quan trọng](#2-cách-khởi-động--cli-flags-quan-trọng)
- [3. Biến môi trường (env vars) hay dùng](#3-biến-môi-trường-env-vars-hay-dùng)
- [4. Slash command tích hợp sẵn (đầy đủ)](#4-slash-command-tích-hợp-sẵn-đầy-đủ)
- [5. Custom slash command — tự tạo lệnh riêng](#5-custom-slash-command--tự-tạo-lệnh-riêng)
- [6. CLAUDE.md — bộ nhớ ngữ cảnh dự án](#6-claudemd--bộ-nhớ-ngữ-cảnh-dự-án)
- [7. Subagent (Agent tool) — chia việc song song](#7-subagent-agent-tool--chia-việc-song-song)
- [8. Skills — gói kỹ năng tái sử dụng](#8-skills--gói-kỹ-năng-tái-sử-dụng)
- [9. Hooks — tự động hoá phản ứng theo sự kiện](#9-hooks--tự-động-hoá-phản-ứng-theo-sự-kiện)
- [10. MCP server — kết nối tool/hệ thống ngoài](#10-mcp-server--kết-nối-toolhệ-thống-ngoài)
- [11. Permission system & permission mode](#11-permission-system--permission-mode)
- [12. Plan Mode — đề xuất trước khi sửa code](#12-plan-mode--đề-xuất-trước-khi-sửa-code)
- [13. Git worktree — chạy nhiều task song song](#13-git-worktree--chạy-nhiều-task-song-song)
- [14. Headless mode, CI/CD & Claude Code trên web](#14-headless-mode-cicd--claude-code-trên-web)
- [15. Quản lý chi phí (cost management)](#15-quản-lý-chi-phí-cost-management)
- [16. Phím tắt & tính năng editor](#16-phím-tắt--tính-năng-editor)
- [17. IDE integration (VS Code / JetBrains)](#17-ide-integration-vs-code--jetbrains)
- [18. 🎯 Tips & Tricks — dùng Claude Code tạo giá trị thật](#18--tips--tricks--dùng-claude-code-tạo-giá-trị-thật)
- [19. Bảng tra nhanh cuối bài](#19-bảng-tra-nhanh-cuối-bài)

---

## 1. Claude Code là gì & cài đặt

Claude Code là CLI agent code chạy trong terminal, có thể đọc/sửa file, chạy lệnh shell,
tự lên kế hoạch nhiều bước (agentic loop), gọi công cụ ngoài qua MCP, và tích hợp vào
VS Code/JetBrains/GitHub Actions.

### Cài đặt

```bash
# macOS / Linux / WSL — cài native (khuyên dùng)
curl -fsSL https://claude.ai/install.sh | bash

# Windows PowerShell
irm https://claude.ai/install.ps1 | iex

# Qua npm (mọi hệ điều hành có Node)
npm install -g @anthropic-ai/claude-code

# macOS qua Homebrew
brew install --cask claude-code

# Linux qua apt
sudo install -d -m 0755 /etc/apt/keyrings
sudo curl -fsSL https://downloads.claude.ai/keys/claude-code.asc -o /etc/apt/keyrings/claude-code.asc
echo "deb [signed-by=/etc/apt/keyrings/claude-code.asc] https://downloads.claude.ai/claude-code/apt/stable stable main" \
  | sudo tee /etc/apt/sources.list.d/claude-code.list
sudo apt update && sudo apt install claude-code
```

Kiểm tra cài đặt & tự sửa lỗi cấu hình:

```bash
claude
/doctor
```

---

## 2. Cách khởi động & CLI flags quan trọng

```bash
claude                          # Vào phiên tương tác (interactive)
claude "sửa lỗi auth module"    # Vào phiên tương tác kèm prompt đầu tiên
claude -p "tóm tắt file log"    # Chế độ in kết quả 1 lần, không tương tác (headless)
cat build.log | claude -p "root cause là gì?"   # Pipe input vào
```

### Flags theo nhóm hay dùng nhất

**Model & độ "suy nghĩ" (effort):**
```bash
claude --model opus                  # sonnet | opus | haiku | fable
claude --model sonnet --effort high  # low | medium | high | xhigh | max
claude --fallback-model sonnet,haiku # tự chuyển model nếu model chính quá tải
```

**Quyền hạn (permission):**
```bash
claude --permission-mode plan               # default | plan | acceptEdits | auto | dontAsk | bypassPermissions
claude --dangerously-skip-permissions       # bỏ qua mọi xác nhận — CHỈ dùng trong container cô lập, không có mạng
claude --allowedTools "Bash,Edit,Read"      # cho phép sẵn các tool này, không hỏi lại
claude --disallowedTools "Bash(rm *)"       # cấm tuyệt đối pattern này
```

**Quản lý phiên làm việc (session):**
```bash
claude -c                          # tiếp tục phiên gần nhất (continue)
claude -r <session-id>             # resume 1 phiên cụ thể theo id/tên
claude --fork-session              # resume nhưng tạo session id mới (không ghi đè)
claude --worktree feature-auth     # mở phiên trong git worktree riêng
```

**Thư mục & phạm vi truy cập file:**
```bash
claude --add-dir ../shared-lib     # cho phép đọc/ghi thêm thư mục ngoài project hiện tại
```

**Đầu ra dạng script/CI:**
```bash
claude -p "..." --output-format json          # trả JSON có session_id, usage, structured_output
claude -p "..." --output-format stream-json    # trả từng sự kiện, dùng để pipe/xử lý realtime
```

Xem toàn bộ flags: `claude --help`.

---

## 3. Biến môi trường (env vars) hay dùng

```bash
ANTHROPIC_API_KEY=sk-...           # API key khi dùng trực tiếp Anthropic API
ANTHROPIC_BASE_URL=https://...     # đổi endpoint (proxy nội bộ, Bedrock, v.v.)
DISABLE_AUTOUPDATER=1              # tắt tự động update
CLAUDE_CONFIG_DIR=~/.my-claude     # đổi thư mục config mặc định (~/.claude)
ENABLE_TOOL_SEARCH=auto            # kiểm soát việc load tool MCP: true|false|auto|auto:N
MAX_MCP_OUTPUT_TOKENS=25000        # giới hạn token output từ MCP tool
```

---

## 4. Slash command tích hợp sẵn (đầy đủ)

Gõ trong phiên tương tác, bắt đầu bằng `/`. Nhóm theo mục đích sử dụng:

### Thiết lập & quản lý dự án
| Lệnh | Khi nào dùng |
|---|---|
| `/init` | Chạy 1 lần đầu ở project mới → Claude quét codebase, tự sinh `CLAUDE.md` |
| `/add-dir <path>` | Cho phép Claude đọc/ghi thêm 1 thư mục ngoài project (VD: thư viện dùng chung) |
| `/permissions` | Xem/sửa danh sách quyền allow-deny-ask |
| `/mcp` | Quản lý kết nối MCP server (bật/tắt/đăng nhập) |

### Model & hiệu năng
| Lệnh | Khi nào dùng |
|---|---|
| `/model [tên]` | Đổi model: `/model opus`, `/model sonnet` |
| `/effort [mức]` | Đổi độ sâu suy luận: `low/medium/high/xhigh/max` — việc đơn giản dùng thấp để tiết kiệm |
| `/fast [on\|off]` | Bật/tắt fast mode (Opus, output nhanh hơn) |

### Luồng làm việc & ngữ cảnh
| Lệnh | Khi nào dùng |
|---|---|
| `/plan` | Chuyển sang Plan Mode — Claude chỉ đề xuất, không sửa file (xem mục 12) |
| `/context [all]` | Xem biểu đồ dùng context window (bao nhiêu % bị chiếm bởi CLAUDE.md, history, tool...) |
| `/compact [chỉ thị]` | Tóm tắt hội thoại để giải phóng context, VD: `/compact chỉ giữ phần liên quan tới bug login` |
| `/batch <việc>` | Giao việc lớn, lặp lại trên nhiều file — Claude tự chia nhỏ & chạy song song |

### Review & chất lượng code
| Lệnh | Khi nào dùng |
|---|---|
| `/code-review [mức] [--fix] [--comment]` | Review diff hiện tại tìm bug; `--fix` để tự sửa, `--comment` để post lên PR |
| `/simplify` | Chỉ dọn code cho gọn/đơn giản hơn, không tìm bug |
| `/security-review` | Rà lỗ hổng bảo mật trong diff trước khi commit — **nên chạy trước mọi PR quan trọng** |

### Quản lý hội thoại
| Lệnh | Khi nào dùng |
|---|---|
| `/clear` | Xoá hết context, bắt đầu hội thoại mới (đổi sang task khác không liên quan) |
| `/resume` | Chọn lại 1 phiên cũ để tiếp tục |
| `/rewind` | Tua lại code & hội thoại về 1 checkpoint trước đó (nhấn Esc 2 lần cũng được) |
| `/branch [tên]` | Tạo nhánh hội thoại để thử hướng khác mà không mất hướng cũ |

### Debug & hỗ trợ
| Lệnh | Khi nào dùng |
|---|---|
| `/doctor` | Cài đặt bị lỗi, tool không chạy được → chạy lệnh này để chẩn đoán |
| `/status` | Xem trạng thái phiên, tài khoản đăng nhập |
| `/cost` hoặc `/usage` | Xem số token & chi phí ước tính đã dùng trong phiên |
| `/help` | Danh sách lệnh & trợ giúp nhanh |
| `/bug` | Gửi báo lỗi/feedback cho Anthropic |

### Tích hợp GitHub
| Lệnh | Khi nào dùng |
|---|---|
| `/review [số PR]` | Review nhanh 1 PR trên GitHub (read-only, 1 lượt) |
| `/install-github-app` | Cài GitHub App để Claude tự động review/fix CI trên PR |

### Nghiên cứu & phân tích
| Lệnh | Khi nào dùng |
|---|---|
| `/deep-research <câu hỏi>` | Tìm kiếm nhiều nguồn, đối chiếu chéo, tổng hợp báo cáo có trích dẫn |
| `/dataviz` | Gợi ý cách vẽ biểu đồ/dashboard đẹp, nhất quán |

### Hiển thị & tiện ích
| Lệnh | Khi nào dùng |
|---|---|
| `/config` | Mở màn hình cấu hình (theme, editor mode, model mặc định...) |
| `/export [tên file]` | Xuất hội thoại ra file text |
| `/memory` | Xem/sửa CLAUDE.md và bộ nhớ tự động |
| `/vim` | Bật chế độ soạn thảo kiểu Vim |
| `/theme` | Đổi theme màu terminal |
| `/ide` | Kết nối với VS Code/JetBrains đang mở |
| `/login` / `/logout` | Đăng nhập / đăng xuất tài khoản Anthropic |
| `/exit` | Thoát CLI |

### Chạy nền & tự động hoá
| Lệnh | Khi nào dùng |
|---|---|
| `/loop [phút] [prompt]` | Lặp lại 1 prompt/skill theo chu kỳ (mặc định 10 phút) — VD theo dõi trạng thái deploy |
| `/tasks` | Xem danh sách các subagent/task đang chạy nền trong phiên |

> **Mẹo:** có thể gõ nhiều slash command liền nhau trong 1 dòng, VD:
> `/code-review /security-review` để chạy tuần tự cả 2.

---

## 5. Custom slash command — tự tạo lệnh riêng

Tạo file Markdown trong:
- `.claude/commands/ten-lenh.md` → dùng chung cả team (commit vào git), lệnh là `/ten-lenh`
- `~/.claude/commands/ten-lenh.md` → chỉ cá nhân bạn, dùng ở mọi project

### Ví dụ: `/deploy` — kịch bản deploy có checklist

`.claude/commands/deploy.md`:
```markdown
---
description: Deploy ứng dụng lên môi trường chỉ định
argument-hint: [environment]
allowed-tools: Bash(npm run *), Bash(git *)
---

Deploy lên môi trường: $ARGUMENTS

1. Chạy `npm test`, dừng lại nếu fail
2. Chạy `npm run build`
3. Kiểm tra branch hiện tại đúng quy ước chưa (`git branch --show-current`)
4. Deploy: `npm run deploy -- --env=$ARGUMENTS`
5. Tóm tắt kết quả deploy
```

Dùng: `/deploy staging`

### Ví dụ: `/status-check` — tự động nhúng dữ liệu shell vào prompt

```markdown
---
description: Kiểm tra nhanh trạng thái repo trước khi review
---

Commit gần đây:
!`git log --oneline -n 5`

Trạng thái test:
!`npm test 2>&1 | tail -30`
```

Cú pháp `` !`lệnh` `` chạy shell command **trước** khi Claude nhận prompt, kết quả được
chèn thẳng vào — rất hữu ích để cung cấp ngữ cảnh "tươi" mỗi lần chạy lệnh.

### Frontmatter hay dùng
| Field | Ý nghĩa |
|---|---|
| `description` | Mô tả để Claude biết khi nào tự gọi lệnh này (nếu không tắt) |
| `disable-model-invocation: true` | Chỉ bạn gọi được, Claude không tự ý chạy |
| `allowed-tools` | Cho phép sẵn các tool nhất định khi chạy lệnh này, khỏi hỏi lại |
| `model` | Ép dùng model riêng cho lệnh này (VD lệnh nặng → `opus`) |
| `argument-hint` | Gợi ý tham số hiện khi gõ `/` |

---

## 6. CLAUDE.md — bộ nhớ ngữ cảnh dự án

`CLAUDE.md` là file Claude tự đọc mỗi khi mở phiên trong project — nơi khai báo build
command, kiến trúc, quy ước code, các "bẫy" hay gặp, để Claude không phải đoán mò mỗi lần.

### Cấp bậc (từ rộng → hẹp, đều được nạp cùng lúc)
1. Toàn tổ chức (do admin quản lý, chỉ đọc)
2. `~/.claude/CLAUDE.md` — cá nhân, áp dụng mọi project
3. `./CLAUDE.md` — theo project, commit chung với team
4. `./CLAUDE.local.md` — cá nhân, riêng cho project này (nên thêm vào `.gitignore`)

### Tạo nhanh
```bash
claude
/init
```
Claude sẽ quét codebase, phát hiện lệnh build/test, rồi sinh file — không ghi đè nếu đã có sẵn.

### Nội dung nên có (ví dụ áp dụng cho repo `short-command` này)
```markdown
# Lệnh hay dùng
- Không có build/test tự động — đây là repo tài liệu Markdown thuần.

# Quy ước
- Viết bằng tiếng Việt, giữ format bảng + code block như cheatsheet.md.
- Mỗi file mới cần được liệt kê lại trong README.md (mục lục & lộ trình).
- Không thêm heading emoji tuỳ tiện — theo đúng phong cách các file hiện có.
```

### Mẹo viết CLAUDE.md hiệu quả
- **Ngắn gọn, dưới ~200 dòng** — file càng dài, Claude càng dễ "lơ" phần cuối.
- Viết **cụ thể**: "chạy `npm test`" tốt hơn "nhớ test code trước khi commit".
- Dùng `.claude/rules/ten-file.md` với frontmatter `paths:` để chỉ nạp quy tắc khi
  đang đụng vào đúng thư mục đó (VD rules riêng cho `frontend/`, `api/`).

---

## 7. Subagent (Agent tool) — chia việc song song

Subagent là 1 phiên Claude con, chạy độc lập (context riêng), dùng để:
- Khám phá codebase lớn mà không tốn context của phiên chính (`Explore`)
- Lên kế hoạch/đề xuất kiến trúc (`Plan`)
- Chạy song song nhiều việc độc lập cùng lúc (review 3 module khác nhau, mỗi cái 1 agent)

### Loại có sẵn
- `general-purpose` — làm được mọi việc, dùng đủ tool
- `Explore` — chỉ đọc (Read/Grep/Glob/Bash read-only), tìm code cực nhanh
- `Plan` — như Explore, nhưng tối ưu cho việc phân tích & đề xuất giải pháp

### Tự định nghĩa subagent riêng

`.claude/agents/reviewer.md`:
```yaml
---
model: sonnet
tools: Read Grep Bash
---

Bạn là reviewer chuyên soát lỗi bảo mật & performance. Đọc diff, liệt kê vấn đề
theo mức độ nghiêm trọng, kèm ví dụ input gây lỗi cụ thể.
```

Dùng qua Agent tool khi cần: giao 1 câu hỏi khảo sát lớn ("tìm hết chỗ dùng thư viện X
trong repo") cho `Explore` để không "ngốn" context của phiên chính đang làm việc chính.

---

## 8. Skills — gói kỹ năng tái sử dụng

Khác custom slash command ở chỗ Skill có thể mang theo **script/file phụ trợ** và tự động
được Claude nhận diện để dùng đúng lúc (không cần bạn gõ lệnh).

Cấu trúc: `.claude/skills/ten-skill/SKILL.md` (kèm script nếu cần, VD `scripts/deploy.sh`).

Ví dụ skill có sẵn trong Claude Code: `/code-review`, `/security-review`, `/simplify`,
`/deep-research`, `/dataviz` — đều là Skill, không phải slash command "cứng".

**Khi nào tạo Skill thay vì custom command:** khi quy trình cần nhiều bước có điều kiện,
có thể tái dùng ở nhiều project, hoặc cần Claude tự nhận ra ngữ cảnh phù hợp để gọi
mà không cần bạn gõ `/` tường minh.

---

## 9. Hooks — tự động hoá phản ứng theo sự kiện

Hooks chạy lệnh/script tự động khi 1 sự kiện xảy ra — dùng để enforce rule cứng
(không phụ thuộc Claude "nhớ" làm hay không).

### Sự kiện hay dùng
- `PreToolUse` — trước khi 1 tool chạy (VD chặn `rm -rf`)
- `PostToolUse` — sau khi tool chạy xong (VD tự lint sau khi Edit file)
- `UserPromptSubmit` — trước khi Claude xử lý prompt của bạn
- `SessionStart` / `SessionEnd` — lúc mở/đóng phiên

### Ví dụ: tự lint sau mỗi lần sửa file

`.claude/settings.json`:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          { "type": "command", "command": "npm run lint", "async": true }
        ]
      }
    ]
  }
}
```

### Ví dụ: chặn lệnh nguy hiểm

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash(git push --force*)",
        "hooks": [
          { "type": "prompt", "prompt": "Force push có thể ghi đè lịch sử người khác. Có chắc chắn không?" }
        ]
      }
    ]
  }
}
```

Exit code `2` từ 1 command hook = chặn hành động, stderr là lý do hiển thị cho Claude.

---

## 10. MCP server — kết nối tool/hệ thống ngoài

MCP (Model Context Protocol) cho Claude Code gọi được tool của hệ thống ngoài: GitHub,
Sentry, database, Slack...

```bash
# Thêm MCP server qua HTTP (có OAuth)
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# Thêm MCP server chạy local qua stdio
claude mcp add --transport stdio my-db -- npx -y @bytebase/dbhub --dsn "postgresql://..."

# Quản lý
claude mcp list
claude mcp remove github
```

**Phạm vi (scope):**
- `--scope local` (mặc định) — riêng máy bạn, riêng project
- `--scope project` — lưu vào `.mcp.json`, chia sẻ qua git cho cả team
- `--scope user` — dùng ở mọi project trên máy bạn

Trong phiên: `/mcp` để bật/tắt, đăng nhập lại, xem danh sách server đang kết nối.

---

## 11. Permission system & permission mode

### Các mode
| Mode | Hành vi |
|---|---|
| `default` | Hỏi xác nhận mọi lệnh ghi/sửa (trừ lệnh read-only như `ls`, `cat`) |
| `plan` | Chỉ đọc & đề xuất, không sửa file/chạy lệnh ghi |
| `acceptEdits` | Tự động chấp nhận sửa file & thao tác file cơ bản (mkdir/mv/rm...) |
| `auto` | Bộ phân loại tự động duyệt hành động an toàn, chặn hành động rủi ro cao |
| `dontAsk` | Tự chối mọi thứ không nằm trong danh sách `allow` — dùng cho CI khoá chặt |
| `bypassPermissions` | Bỏ qua toàn bộ xác nhận — **chỉ dùng trong sandbox/container cô lập** |

Chuyển mode nhanh: nhấn **Shift+Tab** để cycle qua các mode trong phiên.

### Khai báo quyền trong `settings.json`
```json
{
  "permissions": {
    "allow": ["Bash(npm run *)", "Edit(src/**/*.ts)"],
    "deny": ["Bash(git push --force*)", "Read(~/.ssh/**)"],
    "additionalDirectories": ["~/shared-config"]
  }
}
```
Thứ tự ưu tiên: **deny > ask > allow** (khớp `deny` là chặn tuyệt đối, dù có khớp `allow`).

---

## 12. Plan Mode — đề xuất trước khi sửa code

Dùng khi:
- Refactor lớn, đụng nhiều file
- Đổi schema database
- Quyết định kiến trúc quan trọng, muốn xem trước khi cho Claude động vào code thật

```text
/plan
```
Claude sẽ đọc code, đề xuất phương án bằng lời/diff giả định, **không sửa file**.
Bạn duyệt xong thì chuyển sang `acceptEdits`/`auto` để Claude thực thi.

---

## 13. Git worktree — chạy nhiều task song song

Mỗi worktree là 1 thư mục làm việc riêng trỏ vào cùng repo nhưng khác branch —
cho phép chạy nhiều phiên Claude Code **song song, không đụng file nhau**.

```bash
# Terminal 1 — làm feature A
claude --worktree feature-auth

# Terminal 2 — làm bugfix B, độc lập hoàn toàn
claude --worktree bugfix-cache
```

Không có thay đổi → worktree & branch tự xoá khi xong. Có thay đổi → Claude hỏi giữ
lại hay xoá. Cũng có thể set `isolation: worktree` cho 1 subagent để nó tự làm trong
worktree tạm, xoá sau khi xong việc.

---

## 14. Headless mode, CI/CD & Claude Code trên web

### Headless (`-p`) — dùng trong script/CI

```bash
# Review code trong CI, trả JSON để parse tiếp
claude -p "Review diff này, liệt kê bug" --output-format json --allowedTools Read

# Nối tiếp 1 phiên đã có (giữ ngữ cảnh)
session=$(claude -p "Bắt đầu review" --output-format json | jq -r '.session_id')
claude -p "Tập trung vào query database" --resume "$session"
```

### GitHub Actions

```yaml
- uses: anthropics/claude-code@v1
  with:
    prompt: "Review PR này, liệt kê lỗi tiềm ẩn"
    permissions: "default"
```

### Claude Code trên web (claude.ai/code)

Chạy trên cloud, không cần cài local, gắn thẳng vào GitHub repo, có thể lưu & resume
phiên, và kéo phiên đó về terminal bằng lệnh `/teleport`. Phù hợp khi cần chạy task
dài hoặc chạy khi không ngồi máy (đây chính là kiểu phiên đang tạo ra tài liệu này).

---

## 15. Quản lý chi phí (cost management)

```text
/cost
```
Xem token đã dùng (input/output/cache) và chi phí ước tính của phiên hiện tại.

### Cách tiết kiệm token/chi phí
- **Đổi model theo việc**: `haiku` cho việc đơn giản, `sonnet` mặc định, `opus` chỉ khi
  cần suy luận sâu (kiến trúc, debug phức tạp).
- **`/effort low`** cho việc lặt vặt, `high/max` chỉ khi thực sự cần.
- **`/clear`** khi đổi sang task không liên quan — đừng giữ context cũ "ăn" token miễn phí.
- **`/compact <chỉ thị>`** khi hội thoại dài nhưng chưa xong việc — giữ lại đúng phần cần.
- **Giữ `CLAUDE.md` ngắn** (<200 dòng) — file này được nạp lại mỗi phiên.
- **Việc khảo sát lớn giao cho subagent `Explore`** thay vì để phiên chính tự đọc hết —
  đỡ tốn context của phiên chính đang cần dùng để sửa code.

---

## 16. Phím tắt & tính năng editor

| Phím | Chức năng |
|---|---|
| `Enter` | Gửi prompt |
| `Shift+Enter` | Xuống dòng, không gửi |
| `Shift+Tab` | Chuyển permission mode (default → acceptEdits → plan...) |
| `Esc` | Dừng Claude đang chạy / tạo checkpoint để `/rewind` |
| `Esc` x2 | Sửa lại tin nhắn trước đó |
| `Ctrl+R` | Tìm trong lịch sử prompt |
| `Ctrl+V` (Alt+V trên Windows/WSL) | Dán ảnh từ clipboard |
| `@ten-file` | Nhắc tới file/thư mục cụ thể trong prompt (autocomplete) |
| `!lệnh` | Chạy thẳng 1 lệnh bash ngay trong ô chat (không cần Claude xử lý) |

Bật Vim mode: `/config` → Editor mode → Vim. Tuỳ biến phím tắt: sửa file
`~/.claude/keybindings.json` (mở nhanh bằng `/keybindings`).

---

## 17. IDE integration (VS Code / JetBrains)

- **VS Code**: cài extension "Claude Code" (icon Spark trên Activity Bar). Có diff
  view song song, xem plan trước khi Claude áp dụng, tab nhiều phiên cùng lúc.
- **JetBrains** (IntelliJ, PyCharm, WebStorm...): cài plugin từ Marketplace, mở terminal
  tích hợp và gõ `claude`, sau đó `/ide` để kết nối 2 chiều (Claude thấy được diagnostics
  của IDE, IDE hiện diff của Claude).

---

## 18. 🎯 Tips & Tricks — dùng Claude Code tạo giá trị thật

1. **Viết CLAUDE.md ngay từ đầu project** — đầu tư 10 phút chạy `/init` và chỉnh tay,
   tiết kiệm hàng chục lần phải giải thích lại "chạy test bằng lệnh gì".
2. **Dùng Plan Mode cho việc rủi ro cao** — refactor lớn, đổi schema, quyết định kiến
   trúc: bắt Claude trình bày trước, bạn duyệt, rồi mới cho code chạy thật.
3. **Biến quy trình lặp lại thành custom slash command** — checklist review PR, kịch
   bản deploy, quy trình rotate secret... viết 1 lần trong `.claude/commands/`, cả team
   dùng chung nếu commit vào git.
4. **Dùng subagent `Explore` cho khảo sát lớn** — "tìm tất cả chỗ gọi API cũ trong
   repo" nên giao cho Explore để giữ context phiên chính sạch, tập trung vào việc sửa.
5. **Chạy `/security-review` trước khi mở PR quan trọng** — bắt được rò rỉ secret,
   SQL injection, lỗi auth mà review bằng mắt hay bỏ sót.
6. **Dùng hook để enforce rule cứng, đừng chỉ ghi trong CLAUDE.md** — VD chặn
   `git push --force` bằng `PreToolUse` hook thay vì hy vọng Claude "nhớ" đừng làm.
7. **Git worktree để chạy song song nhiều luồng việc** — vừa để Claude làm feature A,
   vừa tự tay debug feature B ở terminal khác, không sợ đụng file.
8. **Headless mode (`-p`) để nhúng Claude vào pipeline CI/CD** — lint tự động, review
   diff, sinh báo cáo — trả về JSON để script khác xử lý tiếp.
9. **Đổi model/effort theo độ khó việc** — đừng dùng Opus/max cho việc sửa 1 dòng
   config; dành model mạnh cho lúc thực sự cần suy luận sâu.
10. **`/compact` thay vì để hội thoại phình to vô hạn** — nhất là các phiên làm việc
    kéo dài cả ngày, compact định kỳ giữ Claude "tỉnh táo" và đỡ tốn phí.
11. **Dùng MCP để Claude thao tác trực tiếp trên hệ thống thật** (GitHub, Sentry,
    database) thay vì bạn copy-paste log/API response vào chat thủ công.
12. **Với repo tài liệu như `short-command` này**: dùng Claude Code để tự động hoá
    việc "viết lại điều đã học" — sau mỗi buổi làm việc thật, giao Claude tóm tắt lại
    thành 1 mục mới trong file phù hợp (cheatsheet, playbook, hoặc file này), đúng
    triết lý *học gắn việc thật · viết lại điều đã học* của repo.

---

## 19. Bảng tra nhanh cuối bài

```text
Bắt đầu project mới          → /init
Sửa việc rủi ro cao           → /plan  rồi mới acceptEdits
Đổi task không liên quan      → /clear
Hội thoại dài, chưa xong việc → /compact <chỉ thị>
Trước khi mở PR               → /code-review  →  /security-review
Khảo sát repo lớn              → giao subagent Explore
Chạy lặp lại theo lịch         → /loop <phút> <prompt>
Kiểm tra chi phí                → /cost
Cài đặt bị lỗi                  → /doctor
Muốn tự tạo lệnh riêng           → .claude/commands/ten-lenh.md
Muốn enforce rule cứng           → hooks trong .claude/settings.json
Kết nối GitHub/Sentry/DB          → claude mcp add ...
Chạy song song nhiều task          → claude --worktree <tên>
Dùng trong CI/CD                    → claude -p "..." --output-format json
```
