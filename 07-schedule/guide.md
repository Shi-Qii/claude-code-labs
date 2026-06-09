# 07 — Schedule（定時自動執行）

## 你要學什麼

設定 Claude 在指定時間自動執行任務，不需要你手動觸發。

---

## Schedule 是什麼

Claude Code 的 `/schedule` 功能讓你建立定時任務（cron job），在雲端自動執行指定的 Claude 任務。

**適合的使用場景：**
- 每天早上自動整理昨天的 git log 並寄送摘要
- 每週產生專案進度報告
- 定時檢查 CI 是否有失敗的 build
- 每天提醒你今天要做什麼

---

## 怎麼設定

在 Claude Code 裡輸入 `/schedule`，然後描述你要做的事和頻率：

```
/schedule 每天早上 9 點，讀取最近一天的 git log，
用繁體中文整理成今日工作摘要，格式是：
- 昨天完成了什麼
- 目前有哪些未 commit 的改動
```

Claude 會幫你設定好 cron job，自動在指定時間執行。

---

## 練習

設定一個對你實際有用的定時任務。以下是幾個選項：

**選項 A：每日工作啟動提醒**
```
/schedule 每個工作日早上 9:30，
顯示今天的日期，並問我今天的工作重點是什麼，
幫我記錄到 daily_log.md
```

**選項 B：每週程式碼品質檢查**
```
/schedule 每週五下午 5 點，
掃描專案裡所有 TODO、FIXME、HACK 的註解，
列出清單並統計數量
```

**選項 C：定時備份提醒**
```
/schedule 每天下午 6 點，
檢查目前 git status，
如果有未 commit 的改動就提醒我記得 commit
```

---

## 查看和管理排程

```
/schedule list      # 查看所有排程
/schedule delete    # 刪除排程
```

---

## 完成條件

成功建立一個定時任務，並確認它出現在 `/schedule list` 裡。
