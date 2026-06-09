# Claude Code Labs

給同事自學 Claude Code 的實戰練習集。每個單元獨立，照順序做完就能掌握 Claude Code 的核心功能。

## 這是什麼

[Claude Code](https://claude.ai/code) 是 Anthropic 推出的 AI 編程工具，可以直接在終端機操作，幫你讀程式碼、改程式碼、執行指令、查資料。這份練習集從最基礎的設定開始，帶你學會 7 個讓開發效率大幅提升的功能。

## 需要準備什麼

| 項目 | 說明 |
|------|------|
| Claude Code | [下載安裝](https://claude.ai/code)，需要 Anthropic 帳號 |
| 終端機 | macOS Terminal、iTerm2、或 VS Code 內建終端機皆可 |
| 一個你自己的專案資料夾 | 任何語言都行，練習時會用到 |
| `jq`（04 Hooks 需要） | `brew install jq` |

## 練習地圖

| # | 主題 | 學到什麼 | 需要時間 |
|---|------|---------|---------|
| [01](./01-claude-md/guide.md) | CLAUDE.md | 讓 Claude 自動認識你的專案背景和規範 | ~15 分鐘 |
| [02](./02-skills/guide.md) | Skills（自訂指令） | 建立 `/你的指令`，讓重複動作一鍵完成 | ~20 分鐘 |
| [03](./03-memory/guide.md) | Memory | 讓 Claude 跨對話記住你的偏好和弱點 | ~15 分鐘 |
| [04](./04-hooks/guide.md) | Hooks | 工具執行前後自動觸發 shell 指令 | ~25 分鐘 |
| [05](./05-plan-mode/guide.md) | Plan Mode + Sub-agents | 複雜任務先規劃再執行，支援平行處理 | ~30 分鐘 |
| [06](./06-mcp/guide.md) | MCP 工具 | 串接外部服務（GitHub、資料庫、網頁）進 Claude | ~30 分鐘 |
| [07](./07-schedule/guide.md) | Schedule | 設定定時自動執行的 Claude 任務 | ~20 分鐘 |

## 怎麼開始

```bash
# 1. 安裝 Claude Code
# 前往 https://claude.ai/code 下載

# 2. 進入你的專案資料夾
cd 你的專案

# 3. 啟動 Claude Code
claude

# 4. 從第一個練習開始
# 打開 01-claude-md/guide.md，照著做
```

每個練習都有一個具體任務，做完就算過，不需要任何 AI 或 LLM 背景知識。

## 學完之後能做什麼

- 不用每次打開新對話都重新介紹專案
- 把常用工作流程變成自己的 slash 指令
- Claude 記住你的程式碼風格偏好
- 存檔、commit 時自動觸發 lint / 格式化
- 讓 Claude 直接查你的 GitHub issues 或資料庫
- 設定每天自動產生工作摘要
