# 02 · Custom MCP Server

**不只用別人的工具，自己寫一個讓 Claude 呼叫**

---

## 你會做到什麼

用 Node.js 寫一個 MCP server，把你自己的 API（或公司內部工具）接進 Claude，讓 Claude 直接查詢而不需要你複製貼上。

---

## 需要先完成

- 06-mcp（了解 MCP 是什麼）
- Node.js 基礎

---

## 你會學到什麼

- MCP 協定的運作原理
- 用 `@modelcontextprotocol/sdk` 建立 server
- 定義 tools、resources、prompts
- 把 local server 接進 Claude Code

---

## 難度

★★★★☆ 中高（需要寫程式碼）

---

## 時間

約 60 分鐘

---

## 核心概念

基礎篇 06 教你「接別人寫好的 MCP server」。這一關教你自己寫一個。

**MCP 是什麼協定？**

MCP（Model Context Protocol）是 Claude 和外部工具溝通的標準介面。概念就像 USB：只要符合 USB 規格，任何裝置都能接上電腦。只要符合 MCP 規格，任何工具都能接進 Claude。

```
沒有 MCP：
你 → 複製資料 → 貼給 Claude → Claude 回答 → 你複製結果 → 貼到工具

有 MCP：
你 → 跟 Claude 說「幫我查 XX」→ Claude 直接呼叫工具 → 直接回答
```

**MCP Server 能提供三種東西：**

| 類型 | 說明 | 例子 |
|------|------|------|
| `tools` | Claude 可以主動呼叫的函式 | 查資料庫、呼叫 API、執行指令 |
| `resources` | Claude 可以讀取的資料來源 | 文件、設定檔、即時資料 |
| `prompts` | 預設的 prompt 模板 | 固定格式的查詢指令 |

這一關專注在 **tools**，這是最常用也最實用的類型。

---

## 實作步驟

### Step 1：建立專案

```bash
mkdir my-mcp-server
cd my-mcp-server
npm init -y
npm install @modelcontextprotocol/sdk
```

### Step 2：寫最簡單的 MCP Server

建立 `index.js`：

```javascript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "my-tools",
  version: "1.0.0",
});

// 定義一個 tool
server.tool(
  "get_weather",
  "查詢指定城市的天氣",
  {
    city: z.string().describe("城市名稱，例如 Taipei"),
  },
  async ({ city }) => {
    // 這裡換成你真正的 API 呼叫
    const mockData = {
      Taipei: "28°C，晴天",
      Tokyo: "22°C，多雲",
    };

    const weather = mockData[city] ?? "查無資料";

    return {
      content: [
        {
          type: "text",
          text: `${city} 目前天氣：${weather}`,
        },
      ],
    };
  }
);

// 啟動 server
const transport = new StdioServerTransport();
await server.connect(transport);
```

在 `package.json` 加入：

```json
{
  "type": "module"
}
```

### Step 3：測試 Server 能跑起來

```bash
node index.js
```

沒有報錯、也沒有輸出就對了（它在等 stdin 輸入）。`Ctrl+C` 停止。

### Step 4：接進 Claude Code

編輯 `~/.claude/settings.json`（全域設定）或專案的 `.claude/settings.json`：

```json
{
  "mcpServers": {
    "my-tools": {
      "command": "node",
      "args": ["/絕對路徑/my-mcp-server/index.js"]
    }
  }
}
```

### Step 5：重啟 Claude Code 並測試

重新開啟 Claude Code，輸入：

```
幫我查 Taipei 的天氣
```

Claude 應該會自動呼叫 `get_weather` 這個 tool。

### Step 6：加入第二個實用的 tool

練習用：把本地 JSON 檔案當作「資料庫」來查詢：

```javascript
import { readFileSync } from "fs";

server.tool(
  "search_contacts",
  "在聯絡人清單中搜尋",
  {
    keyword: z.string().describe("搜尋關鍵字"),
  },
  async ({ keyword }) => {
    const contacts = JSON.parse(readFileSync("./contacts.json", "utf-8"));
    const results = contacts.filter(
      (c) =>
        c.name.includes(keyword) ||
        c.email.includes(keyword)
    );

    return {
      content: [
        {
          type: "text",
          text: results.length
            ? JSON.stringify(results, null, 2)
            : "找不到符合的聯絡人",
        },
      ],
    };
  }
);
```

建立 `contacts.json`：

```json
[
  { "name": "王小明", "email": "ming@example.com", "dept": "工程" },
  { "name": "李美華", "email": "hua@example.com", "dept": "設計" }
]
```

---

## 程式碼範本

完整可運行的雙 tool MCP Server：

```javascript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";
import { readFileSync, existsSync } from "fs";

const server = new McpServer({
  name: "my-tools",
  version: "1.0.0",
});

server.tool(
  "read_file",
  "讀取專案中的檔案內容",
  {
    path: z.string().describe("相對路徑，例如 src/index.js"),
  },
  async ({ path }) => {
    if (!existsSync(path)) {
      return { content: [{ type: "text", text: `檔案不存在：${path}` }] };
    }
    const content = readFileSync(path, "utf-8");
    return { content: [{ type: "text", text: content }] };
  }
);

server.tool(
  "list_files",
  "列出資料夾內的檔案",
  {
    dir: z.string().describe("資料夾路徑，預設為當前目錄").default("."),
  },
  async ({ dir }) => {
    const { readdirSync } = await import("fs");
    const files = readdirSync(dir);
    return { content: [{ type: "text", text: files.join("\n") }] };
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

---

## 常見問題

**Q：`npm install @modelcontextprotocol/sdk` 裝不到？**
確認 Node.js 版本 >= 18。執行 `node --version` 確認。

**Q：Claude Code 重啟後看不到我的 tool？**
確認 `settings.json` 裡的路徑是**絕對路徑**，不是相對路徑。用 `pwd` 取得當前目錄的完整路徑。

**Q：Claude 沒有自動呼叫 tool，只是用文字回答？**
把 tool 的 description 寫更明確，讓 Claude 知道什麼情況該用它。例如「當使用者詢問天氣時呼叫此 tool」。

**Q：Tool 回傳錯誤，Claude 怎麼處理？**
在 handler 裡 throw Error，Claude 會收到錯誤訊息並告訴你。不要讓 server crash，用 try/catch 包住。

**Q：可以在 tool 裡呼叫需要認證的 API 嗎？**
可以，把 API key 放在環境變數裡，server 啟動時讀取 `process.env.MY_API_KEY`。在 `settings.json` 的 `env` 欄位設定：`"env": { "MY_API_KEY": "your-key" }`。

---

## 完成條件

做到以下三件事即算完成：

1. 寫好一個有至少兩個 tool 的 MCP server，能用 `node index.js` 啟動不報錯
2. 接進 Claude Code 後，Claude 能成功呼叫其中一個 tool 並回傳結果
3. 能說清楚：你的 tool 做了什麼事、它的 input schema 長什麼樣
