# 進階練習

完成 01-07 基礎後的進階路線，共 9 個單元。每個單元獨立，可按興趣選擇順序，不需要全部做完。

---

## 這是什麼

01-07 基礎練習教你**設定** Claude Code：讓它認識你的專案、記住你的偏好、自動觸發工作流。

進階練習教你**擴展** Claude Code 的邊界：

- 把多個功能組合成更強大的自動化流程
- 自己寫工具讓 Claude 呼叫
- 用程式碼直接操控 Claude，做出自己的 AI 產品
- 把 Claude 接進 CI/CD、知識庫、瀏覽器

---

## 路線地圖

| # | 主題 | 你能做到什麼 | 難度 | 時間 |
|---|------|------------|------|------|
| [01](./01-combo-workflow.md) | **Combo 工作流** | 存檔自動 lint、commit 自動生成 PR 描述，Claude 記住你的格式偏好 | ★★★☆☆ | 40 min |
| [02](./02-custom-mcp.md) | **Custom MCP Server** | 把你自己的 API 或公司內部工具接進 Claude，讓 Claude 直接查詢 | ★★★★☆ | 60 min |
| [03](./03-claude-api.md) | **Claude API / Agent SDK** | 用程式碼呼叫 Claude，打造自己的 AI 工具和 agent | ★★★★☆ | 60 min |
| [04](./04-multi-agent.md) | **Multi-Agent 系統** | 多個 Claude 實例分工：一個規劃、一個執行、一個審查 | ★★★★★ | 90 min |
| [05](./05-cicd.md) | **CI/CD 整合** | PR 自動觸發 Claude review、安全掃描、changelog 自動生成 | ★★★☆☆ | 45 min |
| [06](./06-computer-use.md) | **Computer Use** | 讓 Claude 自己截圖、判斷畫面、點擊操作，完成整套瀏覽器流程 | ★★★★★ | 90 min |
| [07](./07-rag.md) | **RAG + 知識庫** | 把你的文件、筆記向量化，讓 Claude 直接查詢，突破 context 限制 | ★★★★☆ | 75 min |
| [08](./08-evaluation.md) | **Evaluation Pipeline** | 自動比較不同 prompt 版本的品質，讓 Claude 幫你打分數 | ★★★☆☆ | 50 min |
| [09](./09-prompt-optimization.md) | **Prompt 進階優化** | 用 caching 省下 90% 費用、用 few-shot 穩定輸出格式 | ★★☆☆☆ | 30 min |

---

## 各單元一句話說明

**01 Combo 工作流** — 把基礎篇學的三個功能（Skills、Memory、Hooks）串起來，讓它們互相配合，整套工作流一個指令觸發。

**02 Custom MCP Server** — MCP 是 Claude 和外部工具溝通的標準協定。基礎篇 06 教你用別人寫的 MCP，這裡教你自己寫一個，把任何 API 變成 Claude 可以直接呼叫的工具。

**03 Claude API / Agent SDK** — 不透過 Claude Code 介面，直接用 `@anthropic-ai/sdk` 寫程式碼呼叫 Claude。這是做自己 AI 產品的入口，後面 04、06、07、08 都以這個為基礎。

**04 Multi-Agent 系統** — 一個 Claude 做複雜任務容易出錯或偏離，拆成多個專責角色互相協作，輸出品質更穩。這裡實作 Planner → Executor → Reviewer 三層架構。

**05 CI/CD 整合** — Claude Code 有個 headless 模式（`claude -p "..."`），可以在沒有人在旁邊的情況下執行。把它放進 GitHub Actions，讓 PR 自動過一遍 Claude 的眼睛。

**06 Computer Use** — Claude 能看截圖、決定下一步要點哪裡、然後執行。用來自動化需要操作 UI 的任務，不需要網頁有 API。目前仍是 beta 功能。

**07 RAG + 知識庫** — Claude 的 context 有上限，塞不下整個文件庫。解法是把文件轉成向量，查詢時只取出相關的幾段。這裡實作從文件入庫到問答的完整流程。

**08 Evaluation Pipeline** — 不知道哪個 prompt 比較好？讓另一個 Claude 當評審，自動跑 A/B 比較、打分數、給理由。做 AI 產品必學。

**09 Prompt 進階優化** — 最輕量的一個單元。學會 prompt caching（重複的 system prompt 只算一次錢）、few-shot（給例子讓輸出格式穩定）、extended thinking（讓 Claude 多想幾步再回答）。

---

## 建議學習路徑

根據你的目標選一條：

**想快速感受進階效果**
```
01 Combo 工作流 → 09 Prompt 優化 → 05 CI/CD
```
不需要寫 API，在 Claude Code 環境內完成，建立在基礎篇的知識上。

**想打好 API 基礎再往上**
```
09 Prompt 優化 → 03 Claude API → 08 Evaluation → 07 RAG
```
循序漸進，每個單元都為下一個做準備。

**想做自己的 AI 工具或側專案**
```
03 Claude API → 02 Custom MCP → 04 Multi-Agent
```
從呼叫 API 開始，到寫工具，到設計多 agent 系統。

**想做最有視覺衝擊的東西**
```
03 Claude API → 06 Computer Use
```
看著 Claude 自己操控瀏覽器，比較震撼。

---

## 難度說明

| 星級 | 代表什麼 |
|------|---------|
| ★★☆☆☆ | 純設定或技巧，不需要寫新程式碼 |
| ★★★☆☆ | 需要修改設定檔或少量程式碼 |
| ★★★★☆ | 需要寫程式碼，有一定複雜度 |
| ★★★★★ | 架構設計要求高，或環境設定複雜 |

---

## 前置需求

大部分進階單元以 **03 Claude API** 為基礎，建議先做。

需要額外安裝的工具：

| 工具 | 需要的單元 | 安裝方式 |
|------|-----------|---------|
| Anthropic API key | 03, 04, 06, 07, 08 | [console.anthropic.com](https://console.anthropic.com) |
| Node.js | 02, 03 | `brew install node` |
| Python | 07（推薦） | `brew install python` |
| Docker | 06 | [docker.com](https://www.docker.com) |
| jq | 01 | `brew install jq`（基礎篇 04 應已安裝）|
