# 05 · CI/CD 整合

**把 Claude Code 接進 GitHub Actions，PR 自動 review**

---

## 你會做到什麼

設定 GitHub Actions workflow，讓每次 PR 自動觸發 Claude 做程式碼審查、安全掃描、自動生成 changelog。

---

## 需要先完成

- 02-skills（了解 slash 指令）
- GitHub Actions 基本概念（workflow YAML）

---

## 你會學到什麼

- 在 CI 環境使用 Claude Code CLI（非互動模式）
- `claude -p "..."` headless 執行方式
- 設定 `ANTHROPIC_API_KEY` secret
- 把 Claude 輸出寫回 PR comment
- 控制觸發條件（只在特定檔案變動時執行）

---

## 難度

★★★☆☆ 中等

---

## 時間

約 45 分鐘

---

## 核心概念

CI/CD 整合的核心是讓 Claude 在「無人值守」的環境中自動執行。一般使用 Claude Code 是互動式的（你問、它答），但在 GitHub Actions 裡沒有人可以互動，所以需要用 **headless 模式**。

| 模式 | 指令 | 情境 |
|------|------|------|
| 互動模式 | `claude` | 本機開發，你來回對話 |
| Headless 模式 | `claude -p "..."` | CI/CD、腳本自動化 |

**為什麼要這樣做？**  
每次 PR 都等人工 review 是瓶頸。把「找明顯問題、生 changelog、檢查安全漏洞」這些重複工作交給 Claude，人只需要確認最後判斷。

流程：
```
PR 開啟 → GitHub Actions 觸發 → claude -p "review this diff" → 輸出寫回 PR comment
```

---

## 實作步驟

**步驟 1：在 GitHub repo 設定 Secret**

1. 進入 repo → Settings → Secrets and variables → Actions
2. 新增 `ANTHROPIC_API_KEY`，值貼上你的 API key

**步驟 2：建立 workflow 檔案**

```bash
mkdir -p .github/workflows
touch .github/workflows/claude-review.yml
```

**步驟 3：寫 workflow YAML**（內容見下方程式碼範本）

**步驟 4：取得 PR diff 並傳給 Claude**

```bash
# 在 workflow 中取得 diff
git diff origin/main...HEAD > diff.txt
# 傳給 Claude
claude -p "Review this diff for bugs and issues: $(cat diff.txt)"
```

**步驟 5：把輸出寫回 PR comment**

使用 `gh` CLI 或 GitHub API 把 Claude 的輸出貼到 PR。

**步驟 6：設定觸發條件**

只在 `.py`、`.ts` 等程式碼檔案變動時觸發，避免 markdown 改動也跑完整 review。

---

## 程式碼範本

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize]
    paths:
      - '**.py'
      - '**.ts'
      - '**.js'

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      contents: read

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Get PR diff
        run: |
          git diff origin/${{ github.base_ref }}...HEAD > diff.txt
          echo "Diff size: $(wc -l < diff.txt) lines"

      - name: Run Claude Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          REVIEW=$(claude -p "
          You are a code reviewer. Review this git diff and provide:
          1. Summary of changes (2-3 sentences)
          2. Potential bugs or issues (if any)
          3. Security concerns (if any)
          4. Overall verdict: APPROVE / REQUEST_CHANGES

          Keep the response concise and actionable.

          Diff:
          $(cat diff.txt)
          " --output-format text)

          echo "REVIEW_RESULT<<EOF" >> $GITHUB_ENV
          echo "$REVIEW" >> $GITHUB_ENV
          echo "EOF" >> $GITHUB_ENV

      - name: Post Review Comment
        uses: actions/github-script@v7
        with:
          script: |
            const review = process.env.REVIEW_RESULT;
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: `## 🤖 Claude Code Review\n\n${review}`
            });
```

---

## 常見問題

**Q1：workflow 跑完但沒有 PR comment**  
確認 `permissions` 區塊有設 `pull-requests: write`。沒這個權限，`gh` 和 `github-script` 都無法寫入。

**Q2：`claude: command not found`**  
`npm install -g @anthropic-ai/claude-code` 步驟需要在 `Run Claude Review` 之前完成。檢查 steps 順序是否正確。

**Q3：diff 太長導致 token 超限**  
加入 diff 長度限制：
```bash
head -c 50000 diff.txt > diff_truncated.txt
```
或只取變動的特定目錄：`git diff origin/main...HEAD -- src/`

**Q4：ANTHROPIC_API_KEY 沒帶進去**  
Secret 名稱區分大小寫，確認 GitHub Secrets 裡的名稱和 `${{ secrets.ANTHROPIC_API_KEY }}` 完全一致。

**Q5：每次 push 都觸發，費用太高**  
用 `paths` 過濾只在特定檔案變動時觸發，或加 `draft: false` 條件讓草稿 PR 不觸發。

---

## 完成條件

- [ ] GitHub repo 已設定 `ANTHROPIC_API_KEY` Secret
- [ ] `.github/workflows/claude-review.yml` 存在且語法正確（用 `act` 本機測試或直接推一個測試 PR）
- [ ] 開一個 PR 修改任意 `.py` 或 `.ts` 檔案後，Actions 自動執行
- [ ] PR 的 Comments 區出現 Claude 的 review 內容（含 Summary 和 Verdict）
- [ ] Workflow 執行時間在 2 分鐘內完成
