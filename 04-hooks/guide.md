# 04 — Hooks（自動化觸發）

## 你要學什麼

讓特定動作發生時，自動執行你設定的指令，不用每次手動觸發。

---

## Hooks 是什麼

Hooks 是在 `settings.json` 裡設定的觸發規則。當 Claude 執行某個工具前後，自動跑你指定的 shell 指令。

```
PreToolUse   → 工具執行「之前」觸發
PostToolUse  → 工具執行「之後」觸發
```

**實際範例：**
- 每次 Claude 寫完檔案，自動跑 `npm run lint`
- 每次 Claude 執行 bash 指令，先印出 log
- Claude commit 之後，自動更新 changelog

---

## settings.json 在哪裡

有兩個位置：

```
~/.claude/settings.json          # 全域，所有專案都生效
你的專案/.claude/settings.json   # 只對這個專案生效
```

---

## 基本格式

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "echo '檔案已更新'"
          }
        ]
      }
    ]
  }
}
```

`matcher` 是工具名稱，常用的有：`Write`、`Edit`、`Bash`、`Read`。

**取得工具資訊：** Hook 執行時，Claude 會把工具的輸入/輸出以 JSON 格式傳給 stdin。用 `jq` 解析：

```bash
# Edit/Write → 取得檔案路徑
jq -r '.tool_input.file_path'

# Bash → 取得執行的指令
jq -r '.tool_input.command'
```

---

## 練習

在你的專案建立 `.claude/settings.json`，設定一個 PostToolUse hook：

**任務：每次 Claude 編輯檔案後，自動印出「已修改：[檔案路徑]」**

Hook 的指令會從 stdin 收到一個 JSON，用 `jq` 解析出你要的欄位：

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

設定完後，叫 Claude 修改任意一個檔案，觀察 hook 是否自動執行。

---

## 進階：實用的 hook 範例

**每次寫完 Java 檔案自動編譯：**
```json
{
  "matcher": "Write",
  "hooks": [
    {
      "type": "command",
      "command": "if echo $CLAUDE_TOOL_INPUT_FILE_PATH | grep -q '\\.java$'; then mvn compile -q; fi"
    }
  ]
}
```

**每次執行 bash 前記錄 log：**
```json
{
  "matcher": "Bash",
  "hooks": [
    {
      "type": "command",
      "command": "jq -r '.tool_input.command' >> ~/.claude/bash_history.log"
    }
  ]
}
```

---

## 完成條件

設定一個 hook，讓 Claude 執行特定工具後自動觸發你的 shell 指令，並親眼確認它有執行。
