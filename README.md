> 內容驗證日期：2026-06（依 Fable 5 review 修正）

# Claude Code Labs

幫助工程師從零學會 Claude Code 和 Claude API 的實戰練習集。

---

## 這是什麼

[Claude Code](https://claude.ai/code) 是 Anthropic 推出的 AI 編程工具，直接在終端機操作，能幫你讀程式碼、寫程式碼、執行指令、查外部資料。

這個倉庫分兩條路線：

- **基礎篇**：把 Claude Code 設定成真正懂你專案的開發夥伴
- **進階篇**：擴展邊界，做出自己的 AI 工具和系統

---

## 🗺️ 學習路線

```
00 基本使用  →  基礎篇（01-07）  →  進階篇（01-09）
  30 min          約 3 小時            約 9 小時（可選做）
```

需要速查快捷鍵和指令？[→ CHEATSHEET](./CHEATSHEET.md)

---

## 📘 基礎篇

完全新手從這裡開始，00 是必看的第一課，01-07 基本上互相獨立，少數範例會用到前面單元的成果。

| 單元 | 主題 | 你會做到什麼 | 時間 |
|------|------|------------|------|
| [→ 開始](./00-basics/guide.md) | **00 · 基本使用** ⭐ 新手第一課 | 學會日常核心操作：@ 引用、ESC 中斷、上下文管理、git 安全工作流 | ~30 min |
| [→ 開始](./01-claude-md/guide.md) | **01 · CLAUDE.md** | 讓 Claude 自動認識你的專案，不用每次重新介紹 | ~15 min |
| [→ 開始](./02-skills/guide.md) | **02 · Slash Commands & Skills** | 建立自己的 `/指令`，一行搞定重複工作流程 | ~20 min |
| [→ 開始](./03-memory/guide.md) | **03 · Memory** | 讓 Claude 跨對話記住你的偏好，關掉重開還記得 | ~15 min |
| [→ 開始](./04-hooks/guide.md) | **04 · Hooks** | 存檔後自動 lint / 編譯，不用手動執行 | ~25 min |
| [→ 開始](./05-plan-mode/guide.md) | **05 · Plan Mode** | 複雜任務先規劃再動手，多個子任務平行執行 | ~30 min |
| [→ 開始](./06-mcp/guide.md) | **06 · MCP 工具** | 讓 Claude 直接查 GitHub、資料庫，不用你複製貼上 | ~30 min |
| [→ 開始](./07-schedule/guide.md) | **07 · Schedule** | 設定雲端排程，每天自動產生工作摘要 | ~20 min |

**基礎篇學完你能做到什麼：**

| 之前 | 之後 |
|------|------|
| 每次開新對話都要重新解釋專案 | Claude 自動知道你的技術棧和規範 |
| 常用工作流程每次手動做 | `/pr`、`/check` 一個指令完成 |
| Claude 不記得你上次說的話 | 記得你的程式碼風格偏好 |
| 查 GitHub issue 要自己複製貼上 | Claude 直接查，不需要你當中間人 |

---

## 🚀 進階篇

完成基礎篇後，根據興趣選擇路線，不需要全部做完。

| 單元 | 主題 | 你會做到什麼 | 難度 |
|------|------|------------|------|
| [→ 開始](./進階練習/01-combo-workflow.md) | **01 · Combo 工作流** | Skills + Hooks + Memory 三合一自動化 | ★★★☆☆ |
| [→ 開始](./進階練習/02-custom-mcp.md) | **02 · Custom MCP** | 自己寫 MCP Server，把任何 API 接進 Claude | ★★★★☆ |
| [→ 開始](./進階練習/03-claude-api.md) | **03 · Claude API** | 用程式碼呼叫 Claude，打造自己的 AI 工具 | ★★★★☆ |
| [→ 開始](./進階練習/04-multi-agent.md) | **04 · Multi-Agent** | Planner → Executor → Reviewer 三層架構 | ★★★★★ |
| [→ 開始](./進階練習/05-cicd.md) | **05 · CI/CD** | PR 自動觸發 Claude review | ★★★☆☆ |
| [→ 開始](./進階練習/06-computer-use.md) | **06 · Computer Use** | 讓 Claude 操控瀏覽器完成 UI 任務 | ★★★★★ |
| [→ 開始](./進階練習/07-rag.md) | **07 · RAG** | 建立知識庫，讓 Claude 查詢你的文件 | ★★★★☆ |
| [→ 開始](./進階練習/08-evaluation.md) | **08 · Evaluation** | 自動比較 Prompt 版本，LLM-as-judge | ★★★☆☆ |
| [→ 開始](./進階練習/09-prompt-optimization.md) | **09 · Prompt 優化** | Caching 省 90% 費用、Few-shot 穩定輸出 | ★★★☆☆ |

[查看進階篇完整說明 →](./進階練習/README.md)

---

## 🔧 需要準備什麼

| 項目 | 說明 | 需要的單元 |
|------|------|-----------|
| Claude Code | 見下方安裝說明 | 全部 |
| 終端機 | macOS Terminal、iTerm2、VS Code 終端機皆可 | 全部 |
| 一個你的專案資料夾 | 任何語言都行 | 全部 |
| uv | `brew install uv`（fetch MCP 需要） | 06 |
| Node.js | `node --version` 確認（filesystem MCP、進階 02/03 需要） | 06、進階 02/03 |
| jq | `brew install jq` | 04 |
| Anthropic API key | [console.anthropic.com](https://console.anthropic.com)（進階篇 03 之後需要） | 進階 03-09 |

---

## 💰 費用說明

**基礎篇 01-07：** Claude Pro 或 Max 訂閱即可，無額外費用。

**進階篇 03 之後：** 需要 Anthropic API key，API 用量**另外計費**，與訂閱方案分開。全部做完約數美元等級，依模型選擇和測試次數而異。

---

## 安裝 Claude Code

```bash
# 安裝
npm install -g @anthropic-ai/claude-code

# 驗證安裝成功
claude --version

# 進入你的專案，啟動 Claude Code
cd 你的專案資料夾
claude
```

---

## 怎麼開始

打開 [`01-claude-md/guide.md`](./01-claude-md/guide.md)，照著做。每個單元結尾都有明確的**完成條件**，做到那個條件就算過，再進下一個。

---

## 🏢 要帶團隊導入？

基礎篇和進階篇是個人學習課綱。如果你想把 Claude Code 推廣給整個開發團隊，[團隊導入/](./團隊導入/) 有完整的工具包：

- 共用設定教學（新同事 clone 即可用）
- 90 分鐘工作坊腳本
- 學習進度追蹤範本
- 公司環境資安注意事項
- Java/Spring Boot 實戰案例集（適合 demo 素材）

一般學習者不需要進去，主線課程完全不受影響。

---

## 遇到問題

- **自我診斷：** 在 Claude Code 內執行 `/doctor`
- **官方文件：** [docs.claude.com](https://docs.claude.com)
- **回報問題：** [開 repo issue](../../issues)

---

## License

MIT © Shiqi — 詳見 [LICENSE](./LICENSE)
