# 01 · Combo 工作流

**把 Skills + Hooks + Memory 組合成真正自動化的流程**

---

## 你會做到什麼

建立一套「存檔 → 自動 lint → 自動生成 commit message → PR 描述一鍵完成」的完整工作流，Claude 全程記得你的格式偏好。

---

## 需要先完成

- 02-skills
- 03-memory
- 04-hooks

---

## 你會學到什麼

- 多個功能組合時的設計思維
- 讓 Memory 裡的偏好被 Skills 引用
- Hooks 觸發後把結果餵給下一個 Skill
- 除錯組合流程的方法

---

## 難度

★★★☆☆ 中等（概念不難，設定細節多）

---

## 時間

約 40 分鐘

---

## 核心概念

基礎篇的三個功能各自獨立運作：

```
Skills  →  你下指令，Claude 執行
Memory  →  Claude 記住你的偏好
Hooks   →  檔案變動時自動執行 shell 指令
```

Combo 的核心是讓它們**互相感知**：

```
你存檔
  → Hook 觸發 lint（shell）
  → lint 結果寫入暫存檔
  → /commit 這個 Skill 讀取 lint 結果 + Memory 裡的 commit 格式偏好
  → 自動生成符合你風格的 commit message
```

關鍵問題：**三個工具之間怎麼傳資料？**

答案是**檔案**。Hook 把結果寫到 `.claude/tmp/`，Skill 去讀那個檔案。Memory 儲存偏好，Skill 在 prompt 裡引用它。

---

## 實作步驟

### Step 1：確認基礎設定已就位

```bash
# 確認 hooks 設定存在
cat .claude/settings.json

# 確認有 skills 資料夾
ls .claude/commands/
```

### Step 2：建立暫存資料夾

```bash
mkdir -p .claude/tmp
echo ".claude/tmp/" >> .gitignore
```

### Step 3：設定 PostToolUse Hook 捕捉 lint 結果

在 `.claude/settings.json` 加入：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint \"$CLAUDE_TOOL_INPUT_FILE_PATH\" --format json > .claude/tmp/lint-result.json 2>&1 || true"
          }
        ]
      }
    ]
  }
}
```

### Step 4：在 Memory 裡記錄 commit 格式偏好

開啟 Claude Code，輸入：

```
請記住我的 commit message 格式偏好：
- 第一行：type(scope): 簡短說明（不超過 50 字）
- type 只用：feat / fix / refactor / docs / chore
- 用繁體中文寫說明
- 不要加 emoji
```

### Step 5：建立 /commit Skill

新增 `.claude/commands/commit.md`：

```markdown
執行以下步驟：

1. 讀取 `.claude/tmp/lint-result.json`，確認沒有 error（warning 可以忽略）
2. 執行 `git diff --staged` 取得變更內容
3. 根據我在 Memory 裡記錄的 commit message 格式偏好，生成一條 commit message
4. 顯示給我確認，等我說 ok 再執行 `git commit -m "..."`

如果 lint 有 error，先告訴我哪個檔案哪一行出錯，不要繼續 commit。
```

### Step 6：建立 /pr Skill

新增 `.claude/commands/pr.md`：

```markdown
執行以下步驟：

1. 執行 `git log main..HEAD --oneline` 取得這個 branch 的所有 commits
2. 執行 `git diff main...HEAD --stat` 取得變更摘要
3. 根據以上資訊生成 PR 描述，格式：
   - ## 這個 PR 做了什麼（2-3 句話）
   - ## 主要變更（bullet points）
   - ## 測試方式
4. 輸出完整的 `gh pr create` 指令，讓我直接複製執行
```

### Step 7：測試整套流程

```bash
# 修改任意一個 JS 檔案後存檔
# 觀察 lint 是否自動執行
cat .claude/tmp/lint-result.json

# 暫存變更
git add .

# 呼叫 combo skill
/commit
```

---

## 程式碼範本

完整的 `.claude/settings.json` 範本：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT_FILE_PATH\" | grep -qE '\\.(js|ts|jsx|tsx)$'; then npx eslint \"$CLAUDE_TOOL_INPUT_FILE_PATH\" --format json > .claude/tmp/lint-result.json 2>&1 || true; fi"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT_COMMAND\" | grep -q 'git commit'; then echo 'Reminder: run /commit skill instead of raw git commit'; fi"
          }
        ]
      }
    ]
  }
}
```

---

## 常見問題

**Q：Hook 觸發了但 lint-result.json 是空的？**
先手動跑 `npx eslint yourfile.js --format json` 確認 eslint 有安裝。如果沒有，`npm install -D eslint`。

**Q：/commit Skill 說找不到 lint-result.json？**
確認 `.claude/tmp/` 資料夾存在，且你至少存過一次 JS/TS 檔案讓 Hook 跑過一次。

**Q：Memory 裡的格式偏好沒有被 Skill 使用？**
在 Skill 的 prompt 明確寫「根據我在 Memory 裡記錄的 commit 格式偏好」，Claude 才會主動去查。

**Q：不用 ESLint，用其他 linter 可以嗎？**
可以，把 `npx eslint` 換成 `npx prettier --check` 或 `go vet ./...` 等任何 CLI 工具都行，只要輸出重導向到 `.claude/tmp/` 就好。

**Q：這套流程在沒有 JS 專案的情況下要怎麼練習？**
可以建一個簡單的 Node.js 專案：`mkdir test-project && cd test-project && npm init -y && npm install -D eslint`。

---

## 完成條件

做到以下三件事即算完成：

1. 修改一個 JS 檔案存檔後，`.claude/tmp/lint-result.json` 自動更新
2. 執行 `/commit` 後，Claude 生成的 commit message 符合你在 Memory 裡設定的格式
3. 執行 `/pr` 後，輸出完整可用的 `gh pr create` 指令
