# Claude Code Labs

給同事自學 Claude Code 的實戰練習集。7 個單元，每個獨立，照順序做完就能掌握 Claude Code 的核心功能。

---

## 這是什麼

[Claude Code](https://claude.ai/code) 是 Anthropic 推出的 AI 編程工具，直接在終端機操作，能幫你讀程式碼、寫程式碼、執行指令、查外部資料。

這份練習集不教你「怎麼跟 Claude 聊天」，而是教你**怎麼把 Claude Code 設定成一個真正懂你專案的開發夥伴**——從讓它認識你的專案，到自動化你的工作流程，到串接外部工具。

---

## 需要準備什麼

| 項目 | 說明 |
|------|------|
| Claude Code | [下載安裝](https://claude.ai/code)，需要 Anthropic 帳號 |
| 終端機 | macOS Terminal、iTerm2、VS Code 內建終端機皆可 |
| 一個你的專案資料夾 | 任何語言都行，練習時會用到 |
| Node.js | `node --version` 確認已安裝（06 MCP 需要） |
| jq | `brew install jq`（04 Hooks 需要） |

---

## 練習地圖

> 點擊下方「開始練習」進入各單元的詳細說明。

| 單元 | 主題 | 你會做到什麼 | 時間 |
|------|------|------------|------|
| [→ 開始練習](./01-claude-md/guide.md) | **01 · CLAUDE.md** | 寫一份讓 Claude 自動認識你專案的設定檔，之後不用每次重新介紹 | ~15 分鐘 |
| [→ 開始練習](./02-skills/guide.md) | **02 · Skills** | 建立你自己的 `/指令`，一行搞定重複的工作流程（例如 `/pr`、`/check`） | ~20 分鐘 |
| [→ 開始練習](./03-memory/guide.md) | **03 · Memory** | 讓 Claude 跨對話記住你的偏好，關掉重開還記得 | ~15 分鐘 |
| [→ 開始練習](./04-hooks/guide.md) | **04 · Hooks** | 設定 Claude 修改檔案後自動觸發 lint / 編譯，不用手動執行 | ~25 分鐘 |
| [→ 開始練習](./05-plan-mode/guide.md) | **05 · Plan Mode + Sub-agents** | 複雜任務先讓 Claude 給計畫再動手，多個搜尋任務平行執行 | ~30 分鐘 |
| [→ 開始練習](./06-mcp/guide.md) | **06 · MCP 工具** | 讓 Claude 直接查 GitHub issues、資料庫、網頁，不用你中間複製貼上 | ~30 分鐘 |
| [→ 開始練習](./07-schedule/guide.md) | **07 · Schedule** | 設定每天自動產生工作摘要、每週掃描 TODO 清單，不用手動觸發 | ~20 分鐘 |

總時間約 **2.5 小時**，可以分幾天做，每個單元獨立不互相依賴。

---

## 怎麼開始

**1. 安裝 Claude Code**

前往 [claude.ai/code](https://claude.ai/code) 下載安裝，登入 Anthropic 帳號。

**2. 進入你的專案，啟動 Claude Code**

```bash
cd 你的專案資料夾
claude
```

**3. 從第一個單元開始**

打開 [`01-claude-md/guide.md`](./01-claude-md/guide.md)，照著做。每個單元結尾都有明確的完成條件，做到那個條件就算過。

---

## 七個單元學完你能做到什麼

**之前：**
- 每次開新對話都要重新解釋專案背景
- 常用的工作流程（產生 PR 描述、commit 前檢查）每次手動做
- Claude 不記得你上次說的話
- 查 GitHub issue 要自己複製貼上給 Claude

**之後：**
- Claude 自動知道你的技術棧、規範、禁止事項
- `/pr`、`/check`、`/gs` 一個指令完成整套流程
- Claude 記得你的程式碼風格偏好，不用重複說
- 存檔後自動 lint，commit 前自動檢查安全性問題
- Claude 直接查你的 GitHub、資料庫，不需要你當中間人
- 每天早上自動收到昨日工作摘要

---

## 每個單元的結構

每個 `guide.md` 都包含：

- **你會學到什麼** — 明確列出知識點
- **核心概念** — 搭配對比例子說明原理
- **知識點** — 深入的技術細節和注意事項
- **實用範例** — 可以直接複製使用的範本
- **練習** — 具體的操作步驟
- **常見問題** — 預先回答你可能卡住的地方
- **完成條件** — 明確知道什麼時候算做完
