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

## 🎯 適合什麼人？

- Claude Code 使用者，想把工具用到極致
- AI Agent 開發者，想了解架構設計
- 想把 AI 整合到工作流程的工程師
- 想開發自己的 AI 產品或 SaaS 的開發者

---

## 🚀 各單元說明

| 單元 | 主題 | 你會學到什麼 |
|------|------|------------|
| [01](./01-combo-workflow.md) | 🔗 Combo 工作流 | 把 Skills、Memory、Hooks 串起來，做到一個指令觸發整套自動化流程 |
| [02](./02-custom-mcp.md) | 🛠️ Custom MCP Server | 自己寫 MCP Server，把任何 API 變成 Claude 可以直接使用的工具 |
| [03](./03-claude-api.md) | 🤖 Claude API / Agent SDK | 使用 `@anthropic-ai/sdk` 開發自己的 AI 應用，而不是只透過 Claude Code 操作 |
| [04](./04-multi-agent.md) | 👥 Multi-Agent 系統 | 建立 Planner → Executor → Reviewer 架構，讓多個 Agent 分工合作 |
| [05](./05-cicd.md) | ⚙️ CI/CD 整合 | 把 Claude 接進 GitHub Actions，讓 PR 與程式碼審查流程自動化 |
| [06](./06-computer-use.md) | 🖥️ Computer Use | 讓 Claude 看懂畫面並操作滑鼠鍵盤，自動完成 UI 任務 |
| [07](./07-rag.md) | 📚 RAG + 知識庫 | 建立向量資料庫與文件檢索系統，讓 Claude 回答企業知識問題 |
| [08](./08-evaluation.md) | 📊 Evaluation Pipeline | 自動比較 Prompt 效果、打分數、產生評測報告 |
| [09](./09-prompt-optimization.md) | ✨ Prompt 進階優化 | 學會 Prompt Caching、Few-shot、Extended Thinking 等實戰技巧 |

---

## 📈 學習路線

根據你的目標選一條，不需要全部做完：

**想快速感受進階效果（不需要寫 API）**
```
01 Combo 工作流 → 09 Prompt 優化 → 05 CI/CD
```

**想打好 API 基礎再往上**
```
09 Prompt 優化 → 03 Claude API → 08 Evaluation → 07 RAG
```

**想做自己的 AI 工具或側專案**
```
03 Claude API → 02 Custom MCP → 04 Multi-Agent
```

**從頭到尾完整走一遍**
```
基礎篇 01-07
     ↓
01 Combo 工作流
     ↓
09 Prompt 優化
     ↓
03 Claude API / SDK       ← 大部分進階單元的起點
     ↙         ↘
02 MCP Server   08 Evaluation
     ↓               ↓
04 Multi-Agent  07 RAG
     ↓
05 CI/CD
     ↓
06 Computer Use           ← 最進階
```

從「會用 Claude」一路到「能開發 AI Agent 系統」。

---

## 難度與時間

| 單元 | 難度 | 時間 | 前置條件 |
|------|------|------|---------|
| 01 Combo 工作流 | ★★★☆☆ | 40 min | 基礎篇 02、03、04 |
| 02 Custom MCP Server | ★★★★☆ | 60 min | 基礎篇 06、Node.js |
| 03 Claude API | ★★★★☆ | 60 min | API key |
| 04 Multi-Agent | ★★★★★ | 90 min | 進階 03 |
| 05 CI/CD 整合 | ★★★☆☆ | 45 min | 基礎篇 02 |
| 06 Computer Use | ★★★★★ | 90 min | 進階 03、Docker |
| 07 RAG + 知識庫 | ★★★★☆ | 75 min | 進階 03、Python |
| 08 Evaluation Pipeline | ★★★☆☆ | 50 min | 進階 03 |
| 09 Prompt 進階優化 | ★★☆☆☆ | 30 min | 進階 03 |

> **難度說明：** ★★ 純技巧不寫碼 / ★★★ 少量程式碼 / ★★★★ 需要寫程式碼 / ★★★★★ 架構複雜或環境設定難

---

## 🔧 需要準備什麼

| 工具 | 需要的單元 | 安裝方式 |
|------|-----------|---------|
| Anthropic API key | 03、04、06、07、08 | [console.anthropic.com](https://console.anthropic.com) |
| Node.js | 02、03 | `brew install node` |
| Python | 07（推薦） | `brew install python` |
| Docker | 06 | [docker.com](https://www.docker.com) |
| jq | 01 | `brew install jq`（基礎篇 04 應已安裝）|
