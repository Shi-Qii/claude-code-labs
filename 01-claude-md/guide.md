# 01 — CLAUDE.md

## 你要學什麼

讓 Claude 在開啟你的專案時，自動知道這個專案是什麼、有什麼規則、你希望它怎麼行動。

---

## CLAUDE.md 是什麼

CLAUDE.md 是放在專案根目錄的說明檔。每次你在這個資料夾開啟 Claude Code，它都會自動讀取這份檔案。

**沒有 CLAUDE.md：**
```
你：我們的 API 用什麼驗證？
Claude：我不知道，你沒告訴我
```

**有 CLAUDE.md：**
```
你：我們的 API 用什麼驗證？
Claude：根據專案設定，你們用 JWT，token 存在 Authorization header
```

一份好的 CLAUDE.md 讓你每次開新對話不用重新介紹專案背景。

---

## CLAUDE.md 可以放什麼

沒有固定格式，但常見內容：

```markdown
# 專案名稱

## 技術棧
- 後端：Java Spring Boot
- 資料庫：PostgreSQL
- 部署：AWS EC2

## 開發規範
- 所有 API 回傳格式統一用 ApiResponse wrapper
- 測試用 JUnit 5，不用 mock framework

## 注意事項
- config/secret.yml 不能 commit
- 資料庫 migration 用 Flyway，不要手動改 schema
```

---

## 練習

在你自己的一個專案根目錄建立 `CLAUDE.md`，寫入以下內容（根據實際情況修改）：

```markdown
# [你的專案名稱]

## 這是什麼
[一句話描述這個專案]

## 技術棧
[列出主要技術]

## 我希望 Claude 知道的規範
[至少寫 2 條，例如命名規則、不能動的檔案、架構原則]
```

建完後，打開 Claude Code，直接問：「這個專案用什麼技術？」確認它能從 CLAUDE.md 回答你。

---

## 完成條件

Claude 能根據你的 CLAUDE.md 正確回答關於專案的問題，不需要你在對話裡重新說明。
