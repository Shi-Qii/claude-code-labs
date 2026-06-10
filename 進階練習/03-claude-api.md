# 03 · Claude API / Agent SDK

**直接用程式碼呼叫 Claude，打造自己的 AI 工具**

---

## 你會做到什麼

用 `@anthropic-ai/sdk` 寫一個小型 agent，能自動完成多步驟任務（例如：讀資料夾 → 分析程式碼 → 輸出報告）。

---

## 需要先完成

- Node.js 或 Python 基礎
- Anthropic 帳號（有 API key）

---

## 你會學到什麼

- Messages API 基本結構
- Tool use（讓 Claude 呼叫你定義的函式）
- 串接多輪對話
- Streaming 即時輸出
- Token 計算與費用估算

---

## 難度

★★★★☆ 中高

---

## 時間

約 60 分鐘

---

## 核心概念

Claude Code 是幫你**操作開發工具**的介面。Claude API 是讓你**把 Claude 的能力放進自己的程式**。

```
Claude Code：你 → 對話框 → Claude → 執行工具
Claude API ：你的程式 → API → Claude → 回傳結果 → 你的程式繼續跑
```

**什麼時候要用 API 而不是 Claude Code？**

- 想做一個給別人用的工具（不需要他們有 Claude Code）
- 需要在固定時間自動執行（排程、CI/CD）
- 需要處理大量資料（批次分析 1000 筆）
- 想做自己的聊天介面或 SaaS 產品

**Messages API 的基本結構：**

```
你送出：
  - model（用哪個模型）
  - max_tokens（最多生成幾個 token）
  - messages（對話歷史，role: user / assistant 交替）

Claude 回傳：
  - content（回答內容）
  - usage（用了多少 token）
  - stop_reason（為什麼停止）
```

---

## 實作步驟

### Step 1：取得 API Key

前往 [console.anthropic.com](https://console.anthropic.com) → API Keys → Create Key。

把 key 存到環境變數，**不要直接寫在程式碼裡**：

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
# 或加到 ~/.zshrc 讓每次開 terminal 都有
```

### Step 2：建立專案並安裝 SDK

```bash
mkdir my-claude-agent
cd my-claude-agent
npm init -y
npm install @anthropic-ai/sdk
```

在 `package.json` 加入：

```json
{
  "type": "module"
}
```

### Step 3：最簡單的 Hello World

建立 `hello.js`：

```javascript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic(); // 自動讀取 ANTHROPIC_API_KEY

const response = await client.messages.create({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  messages: [
    { role: "user", content: "用一句話解釋什麼是 API" }
  ],
});

console.log(response.content[0].text);
console.log(`\n使用 tokens：${response.usage.input_tokens} in / ${response.usage.output_tokens} out`);
```

```bash
node hello.js
```

### Step 4：加入 Tool Use

Tool use 讓 Claude 在回答時「呼叫」你定義的函式：

```javascript
import Anthropic from "@anthropic-ai/sdk";
import { readdirSync, statSync } from "fs";

const client = new Anthropic();

// 定義 tools
const tools = [
  {
    name: "list_files",
    description: "列出指定資料夾的檔案清單",
    input_schema: {
      type: "object",
      properties: {
        path: {
          type: "string",
          description: "資料夾路徑",
        },
      },
      required: ["path"],
    },
  },
];

// 實作 tool 的真正邏輯
function listFiles(path) {
  const files = readdirSync(path);
  return files
    .map((f) => {
      const stat = statSync(`${path}/${f}`);
      return `${f} (${stat.isDirectory() ? "資料夾" : stat.size + " bytes"})`;
    })
    .join("\n");
}

// Agent 主迴圈
async function runAgent(userMessage) {
  const messages = [{ role: "user", content: userMessage }];

  while (true) {
    const response = await client.messages.create({
      model: "claude-sonnet-4-6",
      max_tokens: 2048,
      tools,
      messages,
    });

    // 把 Claude 的回應加入對話歷史
    messages.push({ role: "assistant", content: response.content });

    // 只在 stop_reason === "tool_use" 時繼續處理工具，其他情況（end_turn、max_tokens 等）一律結束
    if (response.stop_reason !== "tool_use") {
      const textBlock = response.content.find((b) => b.type === "text");
      return textBlock?.text ?? "";
    }

    // 處理 tool calls
    const toolResults = [];
    for (const block of response.content) {
      if (block.type === "tool_use") {
        let result;
        if (block.name === "list_files") {
          result = listFiles(block.input.path);
        }
        toolResults.push({
          type: "tool_result",
          tool_use_id: block.id,
          content: result,
        });
      }
    }

    // 把 tool 結果回傳給 Claude
    messages.push({ role: "user", content: toolResults });
  }
}

// 執行
const result = await runAgent("幫我列出 . 資料夾有哪些檔案，然後告訴我哪個最大");
console.log(result);
```

### Step 5：加入 Streaming

大量輸出時用 streaming，即時顯示結果：

```javascript
const stream = client.messages.stream({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  messages: [{ role: "user", content: "寫一首關於程式碼的短詩" }],
});

for await (const chunk of stream) {
  if (
    chunk.type === "content_block_delta" &&
    chunk.delta.type === "text_delta"
  ) {
    process.stdout.write(chunk.delta.text);
  }
}
console.log(); // 換行
```

---

## 程式碼範本

**完整的多輪對話 agent（含 tool use + streaming）：**

```javascript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const tools = [
  {
    name: "calculate",
    description: "執行數學計算",
    input_schema: {
      type: "object",
      properties: {
        expression: { type: "string", description: "數學算式，例如 2 + 2 * 3" },
      },
      required: ["expression"],
    },
  },
];

function calculate(expression) {
  // 注意：eval 在生產環境有安全風險，這裡僅作示範
  return String(eval(expression));
}

async function chat(history, userMessage) {
  history.push({ role: "user", content: userMessage });

  while (true) {
    const response = await client.messages.create({
      model: "claude-sonnet-4-6",
      max_tokens: 1024,
      system: "你是一個數學助教，需要計算時使用 calculate tool，不要自己算。",
      tools,
      messages: history,
    });

    history.push({ role: "assistant", content: response.content });

    // 只在 tool_use 時繼續迴圈，避免 max_tokens 等情況造成無限循環
    if (response.stop_reason !== "tool_use") {
      const text = response.content.find((b) => b.type === "text")?.text ?? "";
      return { text, history };
    }

    const results = response.content
      .filter((b) => b.type === "tool_use")
      .map((b) => ({
        type: "tool_result",
        tool_use_id: b.id,
        content: calculate(b.input.expression),
      }));

    history.push({ role: "user", content: results });
  }
}

// 多輪對話
const history = [];
const r1 = await chat(history, "123 * 456 等於多少？");
console.log("Claude：", r1.text);

const r2 = await chat(history, "再乘以 2 呢？");
console.log("Claude：", r2.text);
```

---

## 常見問題

**Q：`ANTHROPIC_API_KEY` 設好了但還是說找不到？**
重新開一個 terminal（環境變數要在新 shell 才生效）。或直接在程式碼 `new Anthropic({ apiKey: "sk-ant-..." })` 測試，確認 key 本身沒問題。

**Q：報錯 `overloaded_error`？**
Anthropic API 有時流量高會拒絕請求，等幾秒再試。正式專案要加 retry 邏輯。

**Q：Tool use 之後 Claude 沒有繼續，直接結束了？**
確認你的迴圈有把 `tool_result` 送回去。Claude 需要收到 tool 的執行結果才會繼續回答。

**Q：怎麼知道這次呼叫花了多少錢？**
`response.usage` 裡有 `input_tokens` 和 `output_tokens`。claude-sonnet-4-6 的費率查 [anthropic.com/pricing](https://www.anthropic.com/pricing)。

**Q：System prompt 應該放什麼？**
放**不會隨對話改變**的固定指示：角色設定、回答格式、限制條件。每輪對話都會重新送，所以越短越省錢。

---

## 完成條件

做到以下三件事即算完成：

1. 成功呼叫 API 拿到回覆，並印出使用的 token 數
2. 實作一個有 tool use 的 agent，Claude 能正確呼叫你的函式
3. 用 streaming 模式輸出一段較長的回答，觀察文字一個個出現的效果
