# 01 · 團隊共用設定

**目標：** 新同事 `git clone` 後打開 `claude`，設定就已經在了，不需要重新設定一遍。

---

## 該 commit 進 repo 的

| 檔案/資料夾 | 說明 |
|-------------|------|
| `CLAUDE.md`（根目錄） | 專案說明、架構、開發規範 —— Claude 每次對話自動載入 |
| `.claude/commands/` | 團隊共用的自訂 slash commands（如 `/pr`、`/check`） |
| `.mcp.json` | 專案用的 MCP 設定（哪些 server、什麼 transport） |
| `.claude/settings.json` | 專案層級：允許/禁止的工具、預設行為 |

**原則：** 只要 commit 進去，所有人 clone 後就自動生效。

---

## 不該 commit 的（加進 .gitignore）

| 檔案/資料夾 | 原因 |
|-------------|------|
| `~/.claude/` | 個人全域設定，不應該影響其他人 |
| `.claude/settings.local.json` | 個人偏好覆蓋，每個人不同 |
| `.env`、任何含 API key 的檔案 | secrets 絕對不進 repo |

**建議在 `.gitignore` 加：**

```
.claude/settings.local.json
```

---

## 設定工作流

### Step 1：建立 CLAUDE.md

```bash
# 讓 Claude 幫你產生初稿
/init
```

`/init` 會掃描專案結構，自動產生 `CLAUDE.md` 草稿，再依團隊規範補充即可。

### Step 2：建立共用 slash commands

在 `.claude/commands/` 建立 `.md` 檔，例如：

```
.claude/commands/pr.md
.claude/commands/check.md
```

命名即指令名稱：`pr.md` → `/pr`。  
詳細格式參考[基礎篇 02 — Slash Commands & Skills](../02-skills/guide.md)。

### Step 3：設定 MCP（如有需要）

```bash
# 加入 MCP server，自動寫入 .mcp.json
claude mcp add --scope project <name> <config>
```

commit `.mcp.json` 後，其他人 clone 就能直接用。  
詳細設定參考[基礎篇 06 — MCP 工具](../06-mcp/guide.md)。

### Step 4：設定 settings.json（保守起步）

新團隊建議的初始設定：

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force)"
    ]
  }
}
```

禁止危險指令，等團隊熟悉後再鬆綁。

---

## 驗收：新同事的體驗

1. `git clone <repo>`
2. `cd <repo>` → `claude`
3. 輸入 `/help`，確認自訂 slash commands 出現在清單裡
4. 輸入 `/mcp`，確認 MCP server 狀態正常

**完成條件：** 另一台電腦執行以上步驟，不需要任何額外設定即可正常使用。

---

[← 回到團隊導入首頁](./README.md) ｜ [下一個：02 工作坊腳本 →](./02-workshop.md)
