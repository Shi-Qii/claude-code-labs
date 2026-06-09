# 02 — Skills（自訂 slash 指令）

## 你要學什麼

建立你自己的 `/指令`，讓重複做的事變成一個字搞定。

---

## Skills 是什麼

Skills 是放在 `.claude/commands/` 資料夾的 `.md` 檔案，每個檔案就是一個自訂指令。

```
.claude/commands/deploy-check.md   →   /deploy-check
.claude/commands/review.md         →   /review
.claude/commands/daily.md          →   /daily
```

檔案內容就是你要 Claude 做的事，可以是任何自然語言指令。

---

## 怎麼寫一個 Skill

**最簡單的形式：**

`.claude/commands/hello.md`
```
說一句話問候我，並告訴我現在是幾點。
```

輸入 `/hello` 就會執行。

**實用的形式（帶參數）：**

`.claude/commands/pr-summary.md`
```
讀取目前 git diff 的內容，用繁體中文寫一份 PR 描述：
- 第一段：這次改了什麼（一句話）
- 第二段：為什麼這樣改
- 第三段：測試了哪些情境
```

**可以讀檔、執行指令、做任何 Claude 能做的事。**

---

## 練習

在你目前的專案裡建立 `.claude/commands/` 資料夾，然後建立一個對你有用的指令。

以下是幾個範例，選一個或自己設計：

**選項 A：git 狀態摘要**
```
讀取 git status 和最近 5 筆 git log，
用中文摘要目前分支的狀態：有什麼未 commit 的改動、最近做了什麼。
```

**選項 B：今日 TODO**
```
讀取專案根目錄的 TODO.md（如果沒有就說「尚未建立 TODO.md」），
條列出今天要做的事，並問我有沒有要新增的項目。
```

**選項 C：code review 前檢查**
```
讀取目前 git diff，檢查：
1. 有沒有 console.log / System.out.println 沒清掉
2. 有沒有 hardcode 的密碼或 API key
3. 有沒有 TODO 留在程式碼裡
列出所有發現，沒問題就說「ready for review」。
```

建完後輸入 `/你的指令名稱` 確認它正確執行。

---

## 完成條件

成功建立並執行至少一個自訂 slash 指令。
