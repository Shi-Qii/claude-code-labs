# 02 — 自訂 Slash Commands 與 Skills

## 你會學到什麼

- 自訂 slash commands 的檔案結構和命名規則
- 如何寫出有效的指令內容
- 傳入參數給指令
- 讓指令讀檔、執行指令、組合多個動作
- 全域指令 vs 專案指令
- Skills（SKILL.md）與 slash commands 的差異

預估時間：20 分鐘

---

## 核心概念

### 自訂 Slash Commands 是什麼

自訂 slash commands 放在 `.claude/commands/` 資料夾，每個 `.md` 檔就是一個指令，**需要你手動輸入才會觸發**。

```
.claude/commands/review.md       →   /review
.claude/commands/daily.md        →   /daily
.claude/commands/deploy-check.md →   /deploy-check
```

**指令名稱 = 檔案名稱（不含 .md）**
**指令內容 = 檔案裡的文字（就是你告訴 Claude 要做什麼）**

### 兩種位置

```
~/.claude/commands/           ← 全域：在任何專案都能用
你的專案/.claude/commands/    ← 專案：只在這個專案有效
```

同名時，專案層級優先。

---

## 知識點：Skill 的內容怎麼寫

### 最簡單的形式：純自然語言

```markdown
說一句話問候我，並告訴我現在幾點。
```

這樣就夠了。輸入 `/hello` Claude 就會執行。

### 讀取檔案

```markdown
讀取 README.md，用一段話總結這個專案是什麼。
```

### 執行 shell 指令

```markdown
執行 git status 和 git log --oneline -10，
用中文說明目前分支的狀態。
```

### 傳入參數

在 skill 內容裡用 `$ARGUMENTS` 接收使用者輸入：

```markdown
搜尋專案裡所有包含 "$ARGUMENTS" 的檔案，
列出檔名和對應的行號。
```

使用方式：`/search UserService`

### 組合多個動作

```markdown
做以下三件事：
1. 執行 git diff HEAD 取得目前的改動
2. 執行 git log --oneline -5 取得最近的 commit
3. 根據以上資訊，用繁體中文寫一份 Pull Request 描述，
   包含：這次改了什麼、為什麼改、怎麼測試
```

### 讓 Claude 做決策

```markdown
讀取 git diff，判斷這次改動是 feat/fix/chore/refactor 哪種類型，
然後產生一個符合 Conventional Commits 格式的 commit message。
不要直接 commit，先讓我確認。
```

---

## 知識點：好的 Skill 設計原則

**1. 動作要具體**
```
❌ 「幫我看看程式碼」
✅ 「讀取 git diff，找出可能有安全疑慮的改動，例如 hardcode 的密碼、SQL injection 風險、未驗證的使用者輸入」
```

**2. 指定輸出格式**
```
✅ 「輸出格式：每個問題一行，格式為 [檔案:行號] 問題描述」
```

**3. 說清楚邊界條件**
```
✅ 「如果 git diff 是空的，說『目前沒有未 commit 的改動』」
```

**4. 可以引用 CLAUDE.md 的規範**
```
✅ 「根據 CLAUDE.md 裡的開發規範，檢查這次改動有沒有違反任何規則」
```

---

## 實用 Skill 範例集

### /gs — git 狀態速覽

`.claude/commands/gs.md`
```
執行以下指令並整合結果：
- git status（未 commit 的檔案）
- git log --oneline -5（最近 5 筆 commit）
- git stash list（如果有 stash 的話）

用中文輸出一份簡短的分支現況摘要。
```

### /pr — 產生 PR 描述

`.claude/commands/pr.md`
```
執行 git diff origin/main...HEAD 取得這個分支的所有改動，
再執行 git log origin/main...HEAD --oneline 取得 commit 清單。

用繁體中文產生一份 Pull Request 描述：

## 這次改了什麼
（一段話說明）

## 改動原因
（為什麼要這樣改）

## 測試方式
（怎麼驗證這次改動是正確的）

## 注意事項
（reviewer 需要特別留意的地方，如果沒有就省略）
```

### /check — commit 前檢查

`.claude/commands/check.md`
```
讀取 git diff，做以下檢查：

1. 安全性：有沒有 hardcode 的密碼、API key、token
2. 除錯碼：有沒有 console.log、System.out.println、debugger、breakpoint
3. 待辦：有沒有 TODO、FIXME、HACK 的註解
4. 規範：有沒有違反 CLAUDE.md 裡的開發規範

每個類別分別列出發現，沒問題就寫「✅ 通過」。
最後給一個整體評估：可以 commit / 需要先處理以上問題。
```

### /explain — 解釋選定的程式碼

`.claude/commands/explain.md`
```
讀取 $ARGUMENTS 這個檔案，
用繁體中文解釋：
1. 這個檔案的用途
2. 主要的函式/方法各做什麼
3. 有什麼值得注意的邏輯或設計決策
4. 和其他模組的關係（如果能判斷的話）
```

使用方式：`/explain src/service/UserService.java`

### /todo — 今日工作清單

`.claude/commands/todo.md`
```
讀取根目錄的 TODO.md，如果檔案不存在就建立一個空的。
顯示目前所有待辦事項，並問我今天要做哪些、有沒有要新增的。
根據我的回答更新 TODO.md。
```

---

## 練習

### 步驟 1：建立你的第一個 skill

```bash
mkdir -p .claude/commands
```

從上面的範例選一個，或自己設計，建立你的第一個 skill 檔案。

### 步驟 2：測試它

在 Claude Code 輸入 `/你的指令名稱`，確認它正確執行。

### 步驟 3：加入參數（進階）

修改你的 skill，或建立新的 skill，讓它接受 `$ARGUMENTS` 參數。
測試：`/你的指令 一些輸入`

---

## 常見問題

**Q：Skill 的 .md 檔案可以很長嗎？**
可以，但通常 5–15 行就夠了。太長反而讓 Claude 抓不到重點。

**Q：可以在 skill 裡呼叫另一個 skill 嗎？**
不行，skill 不能巢狀呼叫。但可以在 skill 裡描述多個步驟讓 Claude 依序執行。

**Q：skill 裡可以用 if/else 邏輯嗎？**
用自然語言描述條件即可，例如「如果 X 就做 A，否則做 B」。

---

---

## 進階：Skills（SKILL.md）

除了手動觸發的 slash commands，Claude Code 還有另一種機制叫 **Skills**。

**差異：**

| | 自訂 Slash Commands | Skills |
|---|---|---|
| 位置 | `.claude/commands/*.md` | `.claude/skills/<名稱>/SKILL.md` |
| 觸發方式 | 你輸入 `/指令名稱` | Claude 判斷任務相關時自動取用 |
| 適合 | 固定流程、需要刻意執行的工作 | 背景知識、Claude 需要主動參考的規範 |

**SKILL.md 格式：**

```markdown
---
name: code-review-rules
description: 程式碼審查的評分標準，在進行 code review 時參考
---

審查時重點關注：
1. 沒有 hardcode 的密碼或 API key
2. 所有公開函式都有明確的輸入驗證
3. 錯誤要被捕捉並記錄，不能靜默失敗
```

建立後，當你叫 Claude 做 code review，它會自動找到這個 skill 並參考裡面的標準，不需要你每次提醒。

---

## 完成條件

- 成功建立並執行至少一個自訂 slash 指令，且它確實做了你希望它做的事
- 能說清楚：slash commands 和 Skills 分別適合用在什麼情況
