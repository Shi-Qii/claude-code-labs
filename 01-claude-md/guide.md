# 01 — CLAUDE.md：讓 Claude 認識你的專案

## 你會學到什麼

- CLAUDE.md 的作用和載入機制
- 如何寫一份有效的 CLAUDE.md
- 多層 CLAUDE.md 的優先順序
- 讓 Claude 在不同專案有不同行為

預估時間：15 分鐘

---

## 核心概念

### CLAUDE.md 是什麼

CLAUDE.md 是放在專案根目錄的純文字說明檔（Markdown 格式）。每次你在這個資料夾開啟 Claude Code，它都會**自動讀取**這份檔案，作為這次對話的背景知識。

它解決的問題很簡單：**你不想每次開新對話都重新介紹一遍自己的專案**。

```
沒有 CLAUDE.md 的每次對話：
你：「幫我加一個 API」
Claude：「好的，你用什麼框架？資料庫是什麼？回傳格式有規範嗎？...」

有 CLAUDE.md 的每次對話：
你：「幫我加一個 API」
Claude：「好的，根據你的 Spring Boot 架構，我會在 UserController 加一個 endpoint，
         回傳 ApiResponse wrapper，用 JUnit 5 補測試...」
```

### 載入機制

Claude Code 啟動時會自動搜尋並載入：

```
1. 你的 home 目錄：~/.claude/CLAUDE.md         ← 所有專案共用的設定
2. 專案根目錄：   ./CLAUDE.md                   ← 這個專案的設定
3. 子資料夾：     ./src/CLAUDE.md               ← 特定模組的設定（如果有的話）
```

子資料夾的 CLAUDE.md 只在 Claude 處理該資料夾內的檔案時才會生效。越具體的設定優先度越高。

### CLAUDE.md 可以放什麼

沒有固定格式，放**對 Claude 有幫助的任何資訊**。常見的幾類：

**1. 專案基本資訊**
```markdown
## 這個專案是什麼
一個電商後台管理系統，Java Spring Boot + PostgreSQL，提供商品管理、訂單處理、報表功能。
```

**2. 技術棧**
```markdown
## 技術棧
- 語言：Java 17
- 框架：Spring Boot 3.2
- 資料庫：PostgreSQL 15，ORM 用 JPA/Hibernate
- 快取：Redis
- 部署：Docker + AWS ECS
- CI/CD：GitHub Actions
```

**3. 開發規範（這是最重要的部分）**
```markdown
## 開發規範
- 所有 API 回傳必須用 ApiResponse<T> 包裝
- Exception 統一在 GlobalExceptionHandler 處理，不要在 Controller 各自 catch
- 測試用 JUnit 5 + AssertJ，禁止用 Mockito mock 資料庫層
- 分支命名：feature/xxx、fix/xxx、chore/xxx
- Commit message 用中文，格式：feat/fix/chore: 說明
```

**4. 禁止事項（讓 Claude 不要踩雷）**
```markdown
## 注意事項
- config/secret.yml 絕對不能 commit
- 不要改 DatabaseMigrationConfig.java，這個檔案由 DBA 維護
- production profile 的任何設定改動要先問過 DevOps
- schema 變更只能透過 Flyway migration，不能手動執行 ALTER TABLE
```

**5. 常用路徑**
```markdown
## 重要檔案位置
- API 入口：src/main/java/com/example/api/
- 共用 DTO：src/main/java/com/example/dto/
- 資料庫 migration：src/main/resources/db/migration/
- 環境設定：src/main/resources/application-{profile}.yml
```

**6. 給 Claude 的角色定義**
```markdown
## Claude 的行為
- 修改程式碼前先說明打算怎麼做
- 如果我的要求和規範衝突，先提醒我再問要怎麼處理
- 產生的測試必須能實際執行，不要用假資料蒙混
```

---

## 知識點：為什麼 CLAUDE.md 比「在對話裡說」更好

| | 在對話裡說 | CLAUDE.md |
|---|---|---|
| 持續性 | 只在這次對話有效 | 每次開啟都自動載入 |
| 一致性 | 每次可能說不一樣 | 固定內容，不會忘 |
| 維護 | 散落在各次對話 | 集中管理，可以 git 版控 |
| 團隊共用 | 只有自己知道 | commit 進去大家都能用 |

---

## 知識點：CLAUDE.md 和系統提示（System Prompt）的關係

CLAUDE.md 本質上是一種**自動注入的 system prompt**。你寫在裡面的內容，等於每次對話開始時 Claude 都會先讀到。

這表示：
- 寫越具體越好，模糊的規範 Claude 不知道怎麼套用
- 不要寫過長（超過 2000 字效果會遞減），抓重點
- 可以用 Markdown 標題分類，讓 Claude 更容易找到相關資訊

---

## 練習

### 步驟 1：建立你的第一個 CLAUDE.md

在你工作的專案根目錄建立 `CLAUDE.md`，至少包含以下區塊：

```markdown
# [你的專案名稱]

## 這是什麼
[一句話描述這個專案做什麼]

## 技術棧
[列出主要語言、框架、資料庫]

## 開發規範
[至少 3 條：命名規則、架構原則、不能動的東西]

## 禁止事項
[至少 1 條：不能 commit 的檔案、不能隨便改的設定]
```

### 步驟 2：驗證它有效

打開 Claude Code，問以下這些問題，確認 Claude 能從 CLAUDE.md 回答：

```
1. 這個專案用什麼資料庫？
2. 我要加一個新 API，應該怎麼處理 Exception？
3. 有哪些檔案我不能隨便改？
```

### 步驟 3：加入全域設定（選做）

在 `~/.claude/CLAUDE.md` 加入你個人的習慣，讓所有專案都套用：

```markdown
## 我的個人偏好
- 回答用繁體中文
- 修改程式碼前先說明計畫
- 程式碼盡量不加不必要的註解
```

---

## 常見問題

**Q：CLAUDE.md 多長比較好？**
抓 500–1000 字。太短沒效果，太長 Claude 反而抓不到重點。

**Q：CLAUDE.md 要 commit 進去嗎？**
建議要，讓團隊成員開啟專案時 Claude 都有相同的背景知識。如果有些設定只想自己用，可以放在 `~/.claude/CLAUDE.md`。

**Q：可以即時修改 CLAUDE.md 然後馬上生效嗎？**
可以，下一個新對話就會載入最新版本。同一個對話內修改不會即時更新。

---

## 完成條件

Claude 能根據你的 CLAUDE.md 正確回答至少 3 個關於專案的問題，不需要你在對話裡重新說明。
