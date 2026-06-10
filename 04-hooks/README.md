# 04 · Hooks

設定 Claude 修改檔案後自動觸發 lint / 編譯，不用手動執行。

**預估時間：** 25 分鐘  
**前置需求：** `brew install jq`

---

## 你會做到什麼

- 設定一個 PostToolUse hook，讓 Claude 修改 JS/TS 檔案後自動跑 ESLint
- 用 `jq` 從 hook 的 stdin JSON 取出被修改的檔案路徑
- 了解 PreToolUse hook 如何以 exit code 2 阻止工具執行

---

[→ 開始學習](./guide.md)

---

[← 上一個：03 · Memory](../03-memory/README.md) ｜ [回到首頁](../README.md) ｜ [下一個：05 · Plan Mode →](../05-plan-mode/README.md)
