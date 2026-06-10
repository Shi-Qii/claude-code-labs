# Claude Code 速查表

---

## ⌨️ 對話框快捷鍵

| 快捷鍵 | 作用 |
|--------|------|
| `ESC` | 中斷 Claude 正在執行的操作 |
| `Shift+Tab` | 切換模式（一般 → auto-accept → plan） |
| `@` | 引用檔案或目錄（跳出選擇器） |
| `!` 開頭 | 直接執行 shell 指令，輸出進入對話上下文 |
| `↑` | 叫回上一條訊息 |

---

## 💬 Session 內 Slash 指令

| 指令 | 作用 |
|------|------|
| `/clear` | 清空對話，開始全新任務 |
| `/compact` | 壓縮對話，保留摘要，釋放 context 空間 |
| `/resume` | 開啟過往對話選單，選一個繼續 |
| `/model` | 查看或切換目前使用的模型 |
| `/doctor` | 自我診斷，找常見設定問題 |
| `/init` | 在當前專案自動產生 CLAUDE.md 草稿 |
| `/help` | 列出所有可用指令 |

---

## 🖥️ 終端機指令（啟動前）

| 指令 | 作用 |
|------|------|
| `claude` | 在目前資料夾啟動 Claude Code |
| `claude -c` / `claude --continue` | 直接接續最近一次對話 |
| `claude --resume` | 開啟過往對話選單，選一個繼續 |
| `claude --version` | 確認安裝版本 |
| `claude --permission-mode plan` | 啟動時直接進入 plan mode |

---

## 📁 設定檔位置

| 檔案 | 位置 | 用途 |
|------|------|------|
| `CLAUDE.md` | 專案根目錄 / `~/.claude/CLAUDE.md` | 給 Claude 的專案說明（自動載入） |
| `settings.json` | `.claude/settings.json` | 專案層級設定（可 commit 共用） |
| `settings.local.json` | `.claude/settings.local.json` | 個人設定（不 commit） |
| Slash commands | `.claude/commands/` | 自訂 `/指令`（可 commit 共用） |
| 個人全域設定 | `~/.claude/settings.json` | 跨所有專案生效 |

---

## 🔄 安全工作流（快速版）

```bash
git status          # 確認工作區乾淨再開始
# → 叫 Claude 做改動
git diff            # 先看它動了什麼
# → 確認沒問題
git checkout .      # 改壞了：還原全部
```
