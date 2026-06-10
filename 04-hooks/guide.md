# 04 — Hooks：工具執行時自動觸發動作

## 你會學到什麼

- Hooks 的觸發時機和種類
- settings.json 的設定格式
- 用 jq 解析 Hook 傳入的工具資訊
- 常用的 Hook 實際應用場景
- 全域 hook vs 專案 hook

預估時間：25 分鐘

前置需求：`brew install jq`

---

## 核心概念

### Hooks 是什麼

Hooks 讓你在 Claude 執行工具的**前後**，自動跑你指定的 shell 指令。不需要你手動觸發，只要設定一次就永遠自動執行。

```
你告訴 Claude：「幫我修改 UserService.java」

Claude 準備執行 Edit 工具
    ↓
[PreToolUse Hook 觸發] → 你的 shell 指令跑一次
    ↓
Claude 執行 Edit（修改檔案）
    ↓
[PostToolUse Hook 觸發] → 你的 shell 指令再跑一次
    ↓
Claude 繼續對話
```

### Hook 觸發時機

```
PreToolUse       → 工具執行之前
PostToolUse      → 工具執行之後
UserPromptSubmit → 你送出訊息時
Notification     → Claude 發出通知時
Stop             → Claude 完成回應時
SubagentStop     → Sub-agent 完成時
PreCompact       → 對話記憶壓縮之前
SessionStart     → 對話開始時
SessionEnd       → 對話結束時
```

### Hook 能做什麼

- 執行任意 shell 指令（lint、格式化、編譯、測試）
- 寫入 log 檔
- 發送通知
- 備份檔案
- 檢查條件、決定是否繼續

---

## 知識點：settings.json 在哪裡

```
~/.claude/settings.json           ← 全域：所有專案都生效
你的專案/.claude/settings.json    ← 專案：只對這個專案生效
```

兩個都有時，設定會合併，同名 key 以專案層級優先。

---

## 知識點：基本格式

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "工具名稱",
        "hooks": [
          {
            "type": "command",
            "command": "你的 shell 指令"
          }
        ]
      }
    ]
  }
}
```

**matcher 可以用的工具名稱：**

| 工具 | 觸發時機 |
|------|---------|
| `Write` | Claude 建立新檔案時 |
| `Edit` | Claude 修改現有檔案時 |
| `Bash` | Claude 執行 shell 指令時 |
| `Read` | Claude 讀取檔案時 |
| `.*` | 任何工具（用正規表示式） |

---

## 知識點：Hook 怎麼知道 Claude 在操作哪個檔案

這是最重要的知識點。**Hook 不是透過環境變數接收資訊，而是透過 stdin 收到一個 JSON。**

Claude 在觸發 hook 時，會把工具的輸入輸出用 JSON 格式傳給你的指令的 stdin：

```json
{
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "/path/to/UserService.java",
    "old_string": "...",
    "new_string": "..."
  }
}
```

用 `jq` 從這個 JSON 取出你要的欄位：

```bash
# 取得被修改的檔案路徑
jq -r '.tool_input.file_path'

# 取得被執行的 bash 指令
jq -r '.tool_input.command'

# 取得工具名稱
jq -r '.tool_name'
```

**所以 hook 的指令通常長這樣：**

```bash
# 讀取 stdin（JSON），解析出 file_path，再做你要做的事
jq -r '.tool_input.file_path' | xargs -I{} echo "修改了：{}"
```

---

## 實用 Hook 範例集

### 每次修改檔案後印出路徑

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '\"已修改：\" + .tool_input.file_path'"
          }
        ]
      }
    ]
  }
}
```

### 每次寫入 JS/TS 檔案後自動跑 ESLint

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | grep -E '\\.(ts|js|tsx|jsx)$' | xargs -r npx eslint --fix"
          }
        ]
      }
    ]
  }
}
```

### 每次 Claude 執行 bash 指令時記錄 log

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.command' >> ~/.claude/bash_history.log"
          }
        ]
      }
    ]
  }
}
```

### 每次修改 Java 檔案後自動編譯

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | grep -q '\\.java$' && mvn compile -q 2>&1 | tail -5 || true"
          }
        ]
      }
    ]
  }
}
```

### 每次修改檔案後自動用 Prettier 格式化

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path // .tool_response.filePath' | xargs -r prettier --write 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

---

## 知識點：Hook 的執行結果

- Hook 的 stdout 會顯示在 Claude Code 的輸出裡
- Hook 失敗（exit code 非 0）預設**不會**中斷 Claude 的操作
- 如果你想讓 **PreToolUse** hook 阻止工具執行，讓 hook 以 **exit code 2** 結束，Claude 會收到 stderr 的內容並停止執行該工具
- 也可以讓 hook 輸出 JSON `{"decision": "block", "reason": "說明原因"}` 達到相同效果

---

## 練習

### 步驟 1：建立最基本的 hook

在你的專案建立 `.claude/settings.json`：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '\"[Hook] 已修改：\" + .tool_input.file_path'"
          }
        ]
      }
    ]
  }
}
```

### 步驟 2：觸發它

叫 Claude 修改任意一個檔案，例如：
```
在 README.md 最後加一行「updated」
```

確認在輸出裡看到 `[Hook] 已修改：README.md`。

### 步驟 3：加入一個對你實際有用的 hook

根據你的技術棧，從上面的範例選一個實際有用的 hook 加進去，或自己設計：
- 有用 JavaScript/TypeScript？加 ESLint
- 有用 Java？加自動編譯
- 想記錄 Claude 的操作？加 log

---

## 常見問題

**Q：hook 設定好之後要重啟 Claude Code 嗎？**
需要，修改 settings.json 後要重新開啟 Claude Code 才會載入。

**Q：hook 裡可以執行多個指令嗎？**
可以，用 `&&` 或 `;` 串連，或是呼叫一個 shell script。

**Q：我怎麼知道 hook 有沒有在執行？**
在 hook 指令裡加一個 `echo`，輸出會顯示在 Claude Code 介面上。

---

## 完成條件

設定一個 hook，叫 Claude 修改一個檔案後，hook 自動執行並在介面上顯示輸出。
