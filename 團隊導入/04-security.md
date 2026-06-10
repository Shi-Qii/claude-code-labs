# 04 · 公司環境注意事項

> **免責聲明：** 這份文件提供一般性原則和建議起點，各條規則都應依你們公司的資安政策調整。導入前請先跟資安團隊確認。

---

## 哪些資訊不應該進入對話

避免把以下內容貼進 Claude Code 對話：

| 類型 | 例子 |
|------|------|
| 憑證和金鑰 | API key、database password、OAuth secret |
| 個人識別資料 | 用戶的真實姓名、email、身分證字號 |
| 客戶資料 | 訂單內容、帳號餘額、交易紀錄 |
| 公司機密 | 未公開的產品計畫、財務數字、合約內容 |
| 內部系統資訊 | 內網 IP、VPN 憑證、資料庫連線字串 |

**實務做法：** 如果要讓 Claude 看含有敏感資料的程式碼，先把真實值替換成假的（`password = "REDACTED"`），再傳給 Claude。

---

## Secrets 管理

**開發環境：**
```
# 用 .env 管理，確認 .gitignore 有這行：
.env
.env.local
.env.*.local
```

**不要做：**
- 把 API key 硬寫在程式碼裡
- 把含有真實憑證的 `.env` commit 進 repo
- 在對話裡複製貼上 production 環境的連線字串

---

## 新手保守設定：先不開 auto-accept

剛開始用的人建議保持預設的「需要確認」模式，不要立刻打開 auto-accept：

```
Shift+Tab 循環模式：
  一般模式（預設）→ auto-accept → plan mode

建議先習慣「一般模式」幾週，每個操作都確認，
熟悉 Claude 的行為模式後再視需要開 auto-accept。
```

---

## settings.json 保守設定

在 `.claude/settings.json` 加上禁止規則，擋掉高風險指令：

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force)",
      "Bash(git push --force-with-lease)",
      "Bash(DROP TABLE*)",
      "Bash(kubectl delete *)"
    ]
  }
}
```

**原則：** 寧可保守多擋，之後有需要再開，比出了問題再補救容易。

---

## MCP 最小權限

設定 MCP server 時，token 只給它需要的權限：

| MCP 用途 | 建議權限 |
|----------|---------|
| GitHub（讀 issue/PR） | `repo:read` 就夠，不需要 `write` |
| GitHub（要能開 PR） | `repo:write`，但不要 `admin` |
| 資料庫（查詢） | 只讀帳號，不給 DML 權限 |
| Slack（查訊息） | `channels:read`，不需要 `chat:write` |

**新增 MCP 前問自己：** 「這個 token 如果洩漏，最壞情況是什麼？」  
如果答案很嚴重，就需要更小的權限範圍或另外評估。

---

## 對話資料與訓練

預設情況下，Anthropic 可能使用對話內容改善模型。

如果你們有資料不外流的要求：
1. 在 Claude Console 的 Enterprise 設定關閉訓練用途（需要 Team / Enterprise 方案）
2. 或在 `~/.claude/settings.json` 查詢目前的隱私設定

企業合規需求請洽 Anthropic 企業銷售，確認 BAA（Business Associate Agreement）或資料處理合約。

---

[← 回到團隊導入首頁](./README.md)
