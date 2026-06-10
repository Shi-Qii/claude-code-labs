# 06 — MCP：串接外部工具

## 你會學到什麼

- MCP（Model Context Protocol）是什麼
- MCP Server 的安裝和設定方式
- 常用的 MCP Server 和各自的用途
- 怎麼驗證 MCP Server 有在運作
- 自訂 MCP Server 的概念

預估時間：30 分鐘

前置需求：Node.js（`node --version` 確認已安裝）

---

## 核心概念

### MCP 是什麼

MCP（Model Context Protocol）是 Anthropic 制定的開放標準，讓 Claude 可以呼叫外部工具和服務。安裝 MCP Server 後，Claude 就像多了一雙手，能直接操作那個服務，不需要你在中間複製貼上。

**沒有 MCP：**
```
你：「幫我整理 GitHub 上這週的 open issue」
你：（手動去 GitHub 複製 issue 列表）
你：（貼進 Claude 的對話）
Claude：「根據你貼的內容，這週有 8 個 issue...」
```

**有 GitHub MCP：**
```
你：「幫我整理 GitHub 上這週的 open issue」
Claude：（直接呼叫 GitHub API，取得 issue 列表）
Claude：「這週有 8 個 open issue，分類如下...」
```

### MCP 的運作機制

```
你 → Claude → MCP Server → 外部服務（GitHub、資料庫、網頁...）
                 ↑
            在本機跑的一個程式
            負責把 Claude 的請求轉換成服務的 API 呼叫
```

MCP Server 是在你電腦本機執行的程式。Claude 發出請求，MCP Server 幫你打 API，把結果回傳給 Claude。

---

## 知識點：設定方式

Claude Code 用 `claude mcp add` 指令管理 MCP Server。

**方法一：指令新增（推薦）**
```bash
# 新增 stdio 類型的 MCP Server
claude mcp add 伺服器名稱 -- 執行指令 [參數...]

# 查看已安裝的清單
claude mcp list

# 移除
claude mcp remove 伺服器名稱
```

**方法二：.mcp.json（適合團隊共用）**

在專案根目錄建立 `.mcp.json`，可以 commit 進 repo 讓團隊共用：

```json
{
  "mcpServers": {
    "伺服器名稱": {
      "command": "執行指令",
      "args": ["參數"],
      "env": {
        "環境變數": "值"
      }
    }
  }
}
```

**驗證安裝成功**

在 Claude Code 輸入 `/mcp`，可以看到所有連線中的 MCP Server 狀態，比問 Claude「你有哪些工具」更直接準確。

> ⚠️ 注意：`settings.json` 裡的 `mcpServers` 是 **Claude Desktop** 的設定格式，不適用於 Claude Code。兩者使用相同的 server 程式，但管理方式不同。

---

## 常用 MCP Server 一覽

### Fetch — 抓取網頁內容

```bash
claude mcp add fetch -- uvx mcp-server-fetch
```

**用途：** 讓 Claude 直接抓取任何網頁或 API 的內容
**不需要 token，最好上手**

使用範例：
```
用 fetch 抓取 https://api.github.com/repos/microsoft/vscode/releases/latest
告訴我最新版的 VS Code 版本號和發布日期
```

> 需要先安裝 `uv`：`brew install uv`

---

### GitHub — 操作 GitHub

GitHub 官方 MCP Server 使用 remote 連線方式：

```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp/
```

**如何取得 token：**
GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens → 選擇需要的權限（Issues: Read，Contents: Read 通常夠用）

**用途：** 查 issue、PR、檔案內容、commit 歷史

使用範例：
```
查看 [你的 repo] 最近 7 天新增的 issue，
按照 label 分類列出來
```

---

### Filesystem — 讀寫指定目錄

```bash
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem /Users/你的名字/Documents
```

**用途：** 讓 Claude 存取 Claude Code 預設範圍以外的目錄（例如你的文件資料夾）

---

## 知識點：安裝 MCP Server 的流程

```
1. 找到你想用的 MCP Server
   → 官方列表：https://github.com/modelcontextprotocol/servers
   → 或搜尋：「[服務名稱] MCP server」

2. 用 claude mcp add 安裝

3. 在 Claude Code 輸入 /mcp 確認連線狀態

4. 實際測試：叫 Claude 用那個工具做一件事
```

---

## 知識點：MCP 工具的安全性

MCP Server 會用你的帳號和權限去呼叫外部服務。使用時注意：

- **GitHub token 只給必要的權限**，不要用 classic token 給 full access
- **資料庫連線字串包含密碼**，不要 commit 進去，用環境變數或本機設定
- **Filesystem MCP 只給需要的目錄**，不要給根目錄
- 定期 rotate token，尤其是出現在設定檔裡的

---

## 練習

### 步驟 1：安裝 Fetch MCP（不需要 token）

先確認 `uv` 已安裝：
```bash
brew install uv
```

然後新增 MCP Server：
```bash
claude mcp add fetch -- uvx mcp-server-fetch
```

在 Claude Code 輸入 `/mcp` 確認 fetch 出現在連線清單。

### 步驟 2：驗證它有在運作

```
用 fetch 工具抓取 https://httpbin.org/json 的內容，
解釋這個 JSON 回傳了什麼欄位
```

確認 Claude 真的去抓了，而不是從訓練資料回答。

### 步驟 3：安裝一個你實際需要的 MCP（選做）

根據你的工作需求，從上面的清單選一個安裝：
- 有用 GitHub？裝 GitHub MCP
- 有本機資料庫？裝 PostgreSQL MCP
- 需要查即時資訊？裝 Brave Search MCP

---

## 常見問題

**Q：重啟 Claude Code 後 MCP Server 沒有出現怎麼辦？**
先確認 `node` 和 `npx` 有安裝（`node --version`），再檢查 settings.json 的 JSON 格式有沒有語法錯誤。

**Q：npx 每次都要下載嗎？**
第一次會下載並快取，之後直接用快取版本，很快。

**Q：可以同時裝多個 MCP Server 嗎？**
可以，在 `mcpServers` 物件裡加多個 key 即可。

**Q：有 Claude Desktop 版的 MCP Server 可以用在 Claude Code 嗎？**
Server 本身通用，但設定方式不同。Claude Desktop 是在 `claude_desktop_config.json` 裡設定，Claude Code 是用 `claude mcp add` 指令或 `.mcp.json`。把 Desktop 的設定直接貼進 Claude Code 的 settings.json 不會生效。

---

## 完成條件

安裝至少一個 MCP Server，成功讓 Claude 透過它取得外部資料並回答你的問題。
