# 05 — Plan Mode + Sub-agents：規劃後再動手

## 你會學到什麼

- Plan Mode 的作用和使用時機
- 如何進入和離開 Plan Mode
- 讀懂 Claude 產生的計畫並給出有效回饋
- Sub-agents 是什麼、怎麼運作
- 什麼情況適合用 Sub-agents
- Plan Mode + Sub-agents 的組合用法

預估時間：30 分鐘

---

## Part 1：Plan Mode

### 為什麼需要 Plan Mode

Claude 的預設行為是「邊想邊做」——你說「加一個 API」，它馬上就開始改程式碼。大部分時候沒問題，但當任務複雜的時候，你會想**先看它打算怎麼做，確認之後才讓它動手**。

```
沒有 Plan Mode：
你：「把登入流程改成支援 OAuth」
Claude：（直接開始改 UserController、SecurityConfig、AuthService...）
你：「等等，你動了 SecurityConfig？我們有幾個地方依賴舊的設定...」
（已經改了 7 個檔案，要 revert 很麻煩）

有 Plan Mode：
你：「把登入流程改成支援 OAuth，先規劃」
Claude：「我打算這樣做：
  1. 新增 OAuthController（新檔案）
  2. 修改 SecurityConfig，加入 OAuth provider 設定
  3. 修改 UserService，支援 OAuth 使用者建立
  4. 更新 application.yml，加入 OAuth credentials
  預計影響 4 個檔案，有沒有我沒考慮到的地方？」
你：「SecurityConfig 那個先不要動，我們用另一個方式」
Claude：「了解，我調整計畫...」
（確認後才動手）
```

### 怎麼進入 Plan Mode

**方法一：直接用語言要求**
```
先不要改程式碼，告訴我你打算怎麼做
先列出計畫，等我確認再執行
```

**方法二：輸入 `/plan`**
在對話框輸入 `/plan`，Claude 就會進入只規劃不執行的模式。

**方法三：用 `--plan` flag 啟動**
```bash
claude --plan
```

### 讓計畫更有用：怎麼問出好計畫

**要求 Claude 列出影響範圍：**
```
在實作之前，先列出：
1. 會改到哪些檔案
2. 每個檔案改什麼
3. 有什麼風險或我需要注意的地方
```

**要求 Claude 考慮邊界條件：**
```
先規劃怎麼加這個功能，並且考慮：
- 如果使用者輸入是空的怎麼處理
- 併發的情況下會不會有問題
- 向下相容性
```

**要求 Claude 評估複雜度：**
```
評估一下這個需求的複雜度，
哪個部分最難，哪個部分最容易出錯？
```

---

## 知識點：讀懂計畫並給出有效回饋

Claude 的計畫通常長這樣：

```
我打算這樣實作：

1. 建立 EmailService.java
   - 加入 sendVerificationEmail() 方法
   - 使用 JavaMailSender 發信

2. 修改 UserService.java
   - 在 register() 方法裡呼叫 EmailService

3. 修改 application.yml
   - 加入 SMTP 設定

需要注意：目前 UserService 的測試可能需要更新，
因為 register() 的行為會改變。
```

**給回饋的方式：**

```
「好，但第 3 點 SMTP 設定放在 application-dev.yml 就好，
 不要放主設定檔，production 的設定由 DevOps 維護」

「第 1 點和第 2 點 OK，第 3 點跳過，我自己來設定」

「整個計畫沒問題，開始執行吧」
```

---

## Part 2：Sub-agents

### Sub-agents 是什麼

Sub-agents 讓 Claude 把工作拆給多個獨立的 agent **同時處理**。每個 sub-agent 有自己的上下文，執行完後把結果回報給主 agent 整合。

```
沒有 Sub-agents：
Claude 一件一件做：
搜尋 UserController → 搜尋 UserService → 搜尋 UserRepository → 整合結果
（順序執行，花的時間是 3 件事的總和）

有 Sub-agents：
Claude 同時派出 3 個 agent：
[Agent 1] 搜尋 UserController
[Agent 2] 搜尋 UserService      ← 三個同時跑
[Agent 3] 搜尋 UserRepository
整合三個結果
（花的時間約等於最慢的那一個）
```

### 什麼時候用 Sub-agents

**適合：**
- 要同時搜尋多個不相關的地方
- 要平行執行幾個獨立的分析任務
- 任務很大，不想讓主對話的上下文被大量搜尋結果塞滿

**不適合：**
- 任務 B 依賴任務 A 的結果（順序相依）
- 簡單的單一任務（overkill）

### 怎麼用 Sub-agents

直接在對話裡說，Claude 會自己決定要不要用 sub-agents：

```
同時搜尋以下三個問題，然後整合結果：
1. 這個專案有哪些 Controller 類別？
2. 哪些 Service 有 @Transactional 標注？
3. 有哪些檔案有 TODO 還沒處理？
```

也可以明確要求：

```
用 sub-agents 平行執行以下分析，
不要順序執行：
...
```

---

## 知識點：Plan Mode + Sub-agents 組合技

```
步驟 1：用 Plan Mode 把大任務分解成幾個獨立模組
步驟 2：每個獨立模組可以用 sub-agent 平行探索
步驟 3：確認計畫後執行
```

實際例子：

```
你：「我想加一個完整的使用者權限系統，先規劃，
     並且用 sub-agents 同時探索現有的 UserService 和 SecurityConfig」

Claude：
[Sub-agent 1 探索 UserService] + [Sub-agent 2 探索 SecurityConfig]
（同時進行）

整合結果後輸出計畫：
1. 修改 UserService，加入 Role 欄位（不影響現有介面）
2. 修改 SecurityConfig，加入 role-based access control
3. 新增 PermissionService（全新檔案）
4. 加入 @PreAuthorize 標注到需要權限檢查的 endpoint

要繼續嗎？
```

---

## 練習

### 練習 A：Plan Mode

選一個你目前有在做的功能（或用下面的範例），要求 Claude 先規劃：

```
我想在這個專案加入一個功能：[你的功能描述]

請先列出實作計畫，不要動程式碼：
- 需要改哪些檔案
- 每個檔案做什麼改動
- 有什麼風險或注意事項
```

讀完計畫後，至少給出一個回饋（即使是「OK 開始吧」），然後讓 Claude 執行。

### 練習 B：Sub-agents 平行搜尋

```
用 sub-agents 同時查以下兩件事，然後整合結果：
1. 這個專案有哪些地方處理了錯誤（try-catch 或 error handler）
2. 這個專案有哪些地方有 hardcode 的字串（不是常數或設定檔）

列出清單，然後告訴我哪個問題比較需要優先處理。
```

---

## 常見問題

**Q：Plan Mode 下 Claude 完全不會改程式碼嗎？**
是的，Plan Mode 下 Claude 只會分析和規劃，不會執行任何檔案操作。

**Q：Sub-agents 會用掉更多 token 嗎？**
會，因為同時有多個 agent 在工作。但換來的是速度和主對話上下文不被塞滿。

**Q：Sub-agents 的結果準確嗎？**
Sub-agent 的搜尋是真實執行的（真的在讀你的程式碼），所以準確度和直接問 Claude 一樣。差別只在執行方式。

---

## 完成條件

- 用 Plan Mode 規劃過一個功能，並根據計畫給出至少一個調整意見後再執行
- 用 Sub-agents 做過一次平行搜尋，看到兩個 agent 的結果被整合在一起
