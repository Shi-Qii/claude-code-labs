# 04 · Multi-Agent 系統

**多個 Claude 實例分工合作，完成複雜任務**

---

## 你會做到什麼

設計一個三層 agent 系統：Planner（規劃）→ Executor（執行）→ Reviewer（審查），讓不同角色的 Claude 互相協作完成一個完整任務。

---

## 需要先完成

- 05-plan-mode（了解 sub-agents 概念）
- 03-claude-api（了解 API 呼叫）

---

## 你會學到什麼

- Orchestrator / Subagent 架構設計
- 如何讓 agent 之間傳遞上下文
- 平行執行 vs 序列執行的選擇
- 避免 agent 無限循環的方法
- Claude Code 內建的 Agent tool 用法

---

## 難度

★★★★★ 高（架構思維要求較高）

---

## 時間

約 90 分鐘

---

## 核心概念

**為什麼需要多個 agent？**

單一 Claude 做複雜任務有兩個問題：
1. Context 有限，任務太長會忘記前面的指示
2. 同一個模型同時扮演「規劃者」和「執行者」，容易自我說服跳過品質檢查

Multi-agent 的解法：讓不同的 Claude 實例各司其職，互相制衡。

```
單一 agent：
  Claude ─────── 規劃 + 執行 + 審查（容易妥協）

Multi-agent：
  Planner Claude  →  Executor Claude  →  Reviewer Claude
  （只想計畫）       （只管執行）         （只挑毛病）
```

**兩種架構模式：**

```
序列（Sequential）：
A 做完 → B 做 → C 做 → 結束
適合：有明確依賴關係的任務（必須先規劃才能執行）

平行（Parallel）：
A 做 ─┐
B 做 ─┼→ 彙整 → 結束
C 做 ─┘
適合：獨立的子任務（同時分析三個檔案）
```

**Agent 之間怎麼傳資料？**

最簡單的方式是**字串傳遞**：把上一個 agent 的輸出，當作下一個 agent 的輸入 prompt。

---

## 實作步驟

### Step 1：確認 03-claude-api 已完成

你需要能夠呼叫 Anthropic API。確認：

```bash
node -e "import('@anthropic-ai/sdk').then(m => console.log('OK'))"
```

### Step 2：建立基礎 agent 函式

建立 `agents.js`：

```javascript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

export async function runAgent(systemPrompt, userMessage, options = {}) {
  const response = await client.messages.create({
    model: options.model ?? "claude-sonnet-4-6",
    max_tokens: options.maxTokens ?? 2048,
    system: systemPrompt,
    messages: [{ role: "user", content: userMessage }],
  });

  return response.content.find((b) => b.type === "text")?.text ?? "";
}
```

### Step 3：實作 Planner Agent

```javascript
// planner.js
import { runAgent } from "./agents.js";

export async function planner(task) {
  const systemPrompt = `你是一個任務規劃師。
你的工作是把使用者的需求拆解成 3-5 個具體的執行步驟。
每個步驟要足夠具體，讓另一個人能夠直接執行。
輸出格式：
步驟 1：[具體動作]
步驟 2：[具體動作]
...
不要執行任何步驟，只負責規劃。`;

  const plan = await runAgent(systemPrompt, `任務：${task}`);
  console.log("📋 Planner 完成規劃：\n", plan);
  return plan;
}
```

### Step 4：實作 Executor Agent

```javascript
// executor.js
import { runAgent } from "./agents.js";

export async function executor(plan, context = "") {
  const systemPrompt = `你是一個任務執行者。
你會收到一份執行計畫，請逐步執行並回報每個步驟的結果。
執行時要具體，說明你做了什麼、結果是什麼。
如果某個步驟需要外部資源而你無法存取，說明你會如何處理。`;

  const input = context
    ? `計畫：\n${plan}\n\n背景資訊：\n${context}`
    : `計畫：\n${plan}`;

  const result = await runAgent(systemPrompt, input);
  console.log("⚙️ Executor 執行完成：\n", result);
  return result;
}
```

### Step 5：實作 Reviewer Agent

```javascript
// reviewer.js
import { runAgent } from "./agents.js";

export async function reviewer(originalTask, executionResult) {
  const systemPrompt = `你是一個品質審查員，態度嚴格。
你會收到原始任務需求和執行結果，評估執行結果是否符合需求。
評估標準：
1. 完整性：所有需求都有被處理到嗎？
2. 正確性：執行方向正確嗎？
3. 品質：有沒有明顯的問題或遺漏？

輸出格式：
## 審查結果
[通過 / 需要修改]

## 優點
- [列出做得好的地方]

## 問題
- [列出需要改善的地方，如果有的話]

## 建議
[下一步建議]`;

  const review = await runAgent(
    systemPrompt,
    `原始任務：${originalTask}\n\n執行結果：\n${executionResult}`
  );
  console.log("🔍 Reviewer 審查完成：\n", review);
  return review;
}
```

### Step 6：組裝主流程

建立 `main.js`：

```javascript
import { planner } from "./planner.js";
import { executor } from "./executor.js";
import { reviewer } from "./reviewer.js";

async function runPipeline(task) {
  console.log(`\n🚀 開始處理任務：${task}\n${"─".repeat(50)}`);

  // 序列執行
  const plan = await planner(task);
  console.log("─".repeat(50));

  const result = await executor(plan);
  console.log("─".repeat(50));

  const review = await reviewer(task, result);
  console.log("─".repeat(50));

  console.log("\n✅ Pipeline 完成");
  return { plan, result, review };
}

await runPipeline(
  "分析一個 Node.js 專案的程式碼品質，找出潛在的問題並提出改善建議"
);
```

```bash
node main.js
```

### Step 7：改為平行執行（進階）

當任務可以拆成獨立子任務時，用 `Promise.all` 平行跑：

```javascript
import { runAgent } from "./agents.js";

async function parallelAnalysis(files) {
  const analyzeFile = (filename) =>
    runAgent(
      "你是程式碼審查員，分析這個檔案的主要功能和潛在問題。",
      `檔案名稱：${filename}`
    );

  // 同時分析多個檔案
  const results = await Promise.all(files.map(analyzeFile));

  // 彙整結果
  const summary = await runAgent(
    "你是技術主管，根據各檔案的分析結果，給出整體評估。",
    results
      .map((r, i) => `${files[i]} 分析結果：\n${r}`)
      .join("\n\n")
  );

  return summary;
}

const summary = await parallelAnalysis([
  "src/index.js",
  "src/utils.js",
  "src/api.js",
]);
console.log(summary);
```

---

## 程式碼範本

**最小可運行的 3-agent 序列 pipeline：**

```javascript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

async function ask(role, message) {
  const res = await client.messages.create({
    model: "claude-sonnet-4-6",
    max_tokens: 1024,
    system: role,
    messages: [{ role: "user", content: message }],
  });
  return res.content[0].text;
}

const task = "寫一個 JavaScript function，計算陣列的平均值";

const plan = await ask(
  "你是規劃師，把任務拆成具體步驟，不要執行",
  task
);

const code = await ask(
  "你是工程師，根據計畫輸出可運行的程式碼，只輸出程式碼",
  `計畫：${plan}`
);

const review = await ask(
  "你是 code reviewer，找出程式碼的問題，沒問題就說通過",
  `程式碼：${code}`
);

console.log("計畫：\n", plan);
console.log("\n程式碼：\n", code);
console.log("\n審查：\n", review);
```

---

## 常見問題

**Q：Agent 之間傳遞的內容越來越長，token 費用很高？**
每個 agent 只傳它需要的資訊，不要把整個對話歷史傳下去。Planner 的輸出給 Executor，但不需要把 Planner 的 system prompt 也一起傳。

**Q：Executor 做了和 Planner 計畫不一樣的事？**
在 Executor 的 system prompt 強調「嚴格按照計畫執行，不要自行修改計畫內容」。

**Q：Reviewer 每次都說通過，沒什麼用？**
讓 Reviewer 更嚴格：在 system prompt 加「你的職責是找問題，如果找不到問題才算通過，輕易說通過是失職」。

**Q：平行執行時，哪個 agent 先完成不確定，會不會亂掉？**
`Promise.all` 會等所有 promise 都 resolve 才繼續，結果順序和輸入一致，不會亂。

**Q：怎麼讓系統在 Reviewer 說「需要修改」時自動重跑 Executor？**
加一個迴圈，檢查 review 結果是否包含「需要修改」，如果有就重跑，設定最多重試 3 次避免無限迴圈。

---

## 完成條件

做到以下三件事即算完成：

1. 跑通完整的 Planner → Executor → Reviewer 三層 pipeline，每個 agent 的輸出都有明顯不同的風格
2. 把其中一個 agent 換成平行執行（例如讓 Executor 同時處理兩個子任務）
3. 能說清楚：為什麼這個任務適合用 multi-agent，而不是一個 agent 就好
