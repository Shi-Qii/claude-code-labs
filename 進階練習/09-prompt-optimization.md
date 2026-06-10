# 09 · Prompt 進階優化

**Caching、Few-shot、Chain-of-thought — 花更少錢得到更好的結果**

---

## 你會做到什麼

把一個原本慢又貴的 prompt 優化成快又準的版本：用 prompt caching 省費用、用 few-shot 穩定輸出格式、用 chain-of-thought 提升推理準確度。

---

## 需要先完成

- 03-claude-api（了解 API 呼叫和 token 計算）

---

## 你會學到什麼

- Prompt Cache：哪些內容可以 cache、cache 命中如何確認、最多省 90% 費用
- Few-shot：幾個例子足夠、例子品質 vs 數量的取捨
- Chain-of-thought：extended thinking 模式、何時值得用
- System prompt 結構優化（長度 vs 精準度）
- Token 計算工具和費用試算

---

## 難度

★★☆☆☆ 低（純技巧，無複雜設定）

---

## 時間

約 30 分鐘

---

## 核心概念

三個技術各解決不同問題：

| 技術 | 解決什麼問題 | 效果 |
|------|------------|------|
| Prompt Caching | 重複傳送長 system prompt 很貴 | 省最多 90% token 費用 |
| Few-shot | 輸出格式不穩定 | 穩定格式，減少錯誤 |
| Chain-of-thought | 複雜推理容易出錯 | 提升準確度，可追蹤推理過程 |

**Prompt Caching 原理：**  
每次 API 呼叫都要傳 system prompt（可能幾千 token）。Cache 讓 Anthropic 伺服器記住你的 system prompt，相同內容不重複計算。**前提**：同一段內容連續呼叫，且間隔在 5 分鐘內。

```
沒有 cache：每次呼叫都付 system prompt 的 input token 費用
有 cache：第 1 次付 +25% 寫入費，之後只付 10% 的快取 token 費用
```

**Few-shot 原理：**  
在 prompt 裡給幾個「問題 → 答案」的範例，讓 Claude 學習模式並套用到新問題上。

**Chain-of-thought 原理：**  
要求 Claude 先把推理過程寫出來，最後再給答案。「想出聲音」讓錯誤更容易被發現，也讓複雜邏輯更準確。

---

## 實作步驟

**步驟 1：安裝依賴**

```bash
pip install anthropic
```

**步驟 2：測試 Prompt Caching**

```python
python test_caching.py
```

觀察第一次呼叫和第二次呼叫的 `cache_creation_input_tokens` vs `cache_read_input_tokens`。

**步驟 3：測試 Few-shot**

對比零次（zero-shot）和三次（few-shot）示例的輸出格式穩定性。

**步驟 4：測試 Chain-of-thought**

對同一道數學或推理題，比較直接要答案 vs 要求「step by step」的準確率。

**步驟 5：計算費用差異**

根據 token 使用量計算實際省了多少錢。

---

## 程式碼範本

```python
import time
import anthropic

client = anthropic.Anthropic()

# ===== 1. Prompt Caching =====

LONG_SYSTEM_PROMPT = """
You are an expert technical writer specializing in software documentation.

Guidelines:
- Use clear, concise language
- Include code examples when relevant
- Structure content with headers
- Target audience: intermediate developers
- Always provide practical, actionable advice
- Format output as Markdown

Style rules:
- Active voice preferred
- Maximum 3 levels of heading
- Code blocks must include language identifier
- No passive constructions like "it can be seen that"

You have deep expertise in: Python, JavaScript, TypeScript, Go, Rust,
cloud infrastructure (AWS, GCP, Azure), DevOps practices, API design,
database optimization, and software architecture patterns.

When reviewing code: check for performance, security, readability, and maintainability.
Always suggest specific improvements with code examples.
""" * 5  # 重複讓 prompt 更長，更能展示 cache 效果

def test_prompt_caching():
    print("=== Prompt Caching 測試 ===\n")
    
    for i in range(3):
        response = client.messages.create(
            model="claude-haiku-4-5-20251001",
            max_tokens=100,
            system=[
                {
                    "type": "text",
                    "text": LONG_SYSTEM_PROMPT,
                    "cache_control": {"type": "ephemeral"},  # 標記這段可以被 cache
                }
            ],
            messages=[{"role": "user", "content": f"呼叫 #{i+1}：解釋什麼是 REST API（一句話）"}],
        )
        
        usage = response.usage
        print(f"呼叫 #{i+1}:")
        print(f"  input tokens:              {usage.input_tokens}")
        print(f"  cache_creation_tokens:     {getattr(usage, 'cache_creation_input_tokens', 0)}")
        print(f"  cache_read_tokens:         {getattr(usage, 'cache_read_input_tokens', 0)}")
        print(f"  output tokens:             {usage.output_tokens}")
        print()
        time.sleep(1)

# ===== 2. Few-shot =====

def test_few_shot():
    print("=== Few-shot 測試 ===\n")
    
    # Zero-shot：直接問，輸出格式難以預測
    zero_shot = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=200,
        messages=[{"role": "user", "content": "分析這個字的情緒：今天天氣真好！"}],
    )
    
    # Few-shot：給範例，輸出格式穩定
    few_shot = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=200,
        messages=[
            {
                "role": "user",
                "content": """Classify the sentiment of text. Always respond in this exact format:
SENTIMENT: [positive/negative/neutral]
CONFIDENCE: [high/medium/low]
REASON: [one sentence]

Examples:
Text: "今天真的超棒的！"
SENTIMENT: positive
CONFIDENCE: high
REASON: 使用了強烈正面詞語「超棒」。

Text: "還行吧，沒什麼特別的。"
SENTIMENT: neutral
CONFIDENCE: medium
REASON: 表達中性、缺乏強烈情感。

Now classify:
Text: "今天天氣真好！"
""",
            }
        ],
    )
    
    print("Zero-shot 輸出（格式不固定）：")
    print(zero_shot.content[0].text)
    print("\nFew-shot 輸出（格式穩定）：")
    print(few_shot.content[0].text)

# ===== 3. Chain-of-Thought =====

def test_chain_of_thought():
    print("\n=== Chain-of-Thought 測試 ===\n")
    
    question = "一個商店有 137 個蘋果，每天賣出 23 個。第 6 天結束時還剩幾個？"
    
    # 直接要答案
    direct = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=50,
        messages=[{"role": "user", "content": f"{question}\n只回答數字。"}],
    )
    
    # Chain-of-thought
    cot = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=300,
        messages=[
            {
                "role": "user",
                "content": f"{question}\n請一步一步計算，最後寫出答案。",
            }
        ],
    )
    
    print(f"問題：{question}")
    print(f"\n直接回答：{direct.content[0].text.strip()}")
    print(f"\nChain-of-thought：\n{cot.content[0].text}")

# ===== 4. Extended Thinking（claude-sonnet-4-6 以上）=====

def test_extended_thinking():
    print("\n=== Extended Thinking 測試 ===\n")
    
    hard_question = """
    有三個盒子：A、B、C。其中一個裝金，兩個裝石頭。
    你選了 A。主持人打開 C，裡面是石頭。
    你要換成 B，還是堅持選 A？勝率分別是多少？請嚴格計算。
    """
    
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=8000,
        thinking={
            "type": "enabled",
            "budget_tokens": 5000,  # 給 Claude 思考的 token 預算
        },
        messages=[{"role": "user", "content": hard_question}],
    )
    
    for block in response.content:
        if block.type == "thinking":
            print(f"[思考過程（前 200 字）]：{block.thinking[:200]}...\n")
        elif block.type == "text":
            print(f"[最終答案]：{block.text}")

if __name__ == "__main__":
    test_prompt_caching()
    test_few_shot()
    test_chain_of_thought()
    # test_extended_thinking()  # 需要 claude-sonnet-4-6，取消註解後執行
```

---

## 常見問題

**Q1：加了 `cache_control` 但 `cache_read_input_tokens` 一直是 0**  
Cache 需要連續兩次呼叫使用**完全相同**的 system prompt 內容，且間隔小於 5 分鐘。常見原因：每次呼叫前動態修改了 system prompt 內容（即使只加了時間戳），導致 cache 失效。

**Q2：Few-shot 範例給了 3 個，輸出還是偶爾跑版**  
範例的格式要和你期待的輸出**一模一樣**（包含換行、空格）。如果格式不一致，Claude 不確定哪個才是「對的」格式。另外，越難的任務可能需要 5-7 個範例。

**Q3：Chain-of-thought 讓回答變超長，token 費用增加**  
這是正常的取捨：更準確但更貴。可以要求「簡短的 step by step」或最後加「只輸出最終答案，過程可以省略」來平衡。

**Q4：Extended Thinking 的 `budget_tokens` 要設多少？**  
簡單問題 1000-2000，中等推理 5000，非常複雜的問題 10000+。設太小 Claude 可能來不及推導完整，設太大費用高但效果邊際遞減。

**Q5：Cache 的 5 分鐘 TTL 到期後怎麼辦？**  
Cache 自動失效，下一次呼叫會重新建立 cache（付 +25% 寫入費）。高頻應用中這是正常的，只要請求夠密集，大多數呼叫都會命中 cache。

---

## 完成條件

- [ ] Prompt Caching 測試：第 2、3 次呼叫的 `cache_read_input_tokens` > 0，確認命中 cache
- [ ] 計算並記錄：有 cache vs 沒有 cache，對同樣 10 次呼叫的 token 費用差多少（%）
- [ ] Few-shot 測試：零次和三次示例的輸出格式對比，能看出差異
- [ ] Chain-of-thought 測試：直接回答和 step-by-step 對比，驗證哪個更準確
- [ ] 能說明每個技術的「適用場景」：在什麼情況下應該用哪個技術
