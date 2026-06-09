# 06 — MCP（串接外部工具）

## 你要學什麼

讓 Claude 能直接操作外部服務，例如查 GitHub issue、搜尋文件、存取資料庫。

---

## MCP 是什麼

MCP（Model Context Protocol）是一個標準，讓 Claude 可以呼叫外部工具。安裝 MCP server 後，Claude 就能直接用那個工具，不需要你複製貼上資料。

**沒有 MCP：**
```
你：幫我查一下 GitHub 上有哪些 open issue
你：（自己去 GitHub 複製 issue 列表貼進來）
Claude：好，根據你貼的內容...
```

**有 MCP：**
```
你：幫我查一下 GitHub 上有哪些 open issue
Claude：（直接呼叫 GitHub API）這是目前 12 個 open issue...
```

---

## 怎麼安裝 MCP Server

MCP server 設定在 `~/.claude/settings.json`：

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "你的 token"
      }
    }
  }
}
```

重啟 Claude Code 後就能使用。

---

## 常用 MCP Server

| Server | 用途 | 安裝套件 |
|--------|------|---------|
| GitHub | 查 issue、PR、repo | `@modelcontextprotocol/server-github` |
| Filesystem | 讀寫指定目錄的檔案 | `@modelcontextprotocol/server-filesystem` |
| Fetch | 抓網頁內容 | `@modelcontextprotocol/server-fetch` |
| Postgres | 查資料庫 | `@modelcontextprotocol/server-postgres` |

完整清單：https://github.com/modelcontextprotocol/servers

---

## 練習

安裝 `Fetch` MCP server（不需要 token，最好上手）：

**1. 更新 `~/.claude/settings.json`：**
```json
{
  "mcpServers": {
    "fetch": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-fetch"]
    }
  }
}
```

**2. 重啟 Claude Code**

**3. 測試：**
```
用 fetch 工具抓取 https://httpbin.org/json 的內容，解釋這個 API 回傳了什麼
```

確認 Claude 真的去抓了網頁內容，而不是從訓練資料回答。

---

## 完成條件

成功安裝一個 MCP server，並讓 Claude 透過它取得外部資料回答你的問題。
