# 08 · Evaluation Pipeline

**用 Claude 評估 Claude 的輸出品質（LLM-as-judge）**

---

## 你會做到什麼

建立一套自動評估系統：對同一個問題跑不同 prompt 版本，讓 judge Claude 打分數並說明理由，找出最好的版本。

---

## 需要先完成

- 03-claude-api（API 呼叫基礎）

---

## 你會學到什麼

- LLM-as-judge 的設計原則（避免 bias）
- 評估維度設計：正確性、完整性、格式、安全性
- A/B 比較 prompt 的系統化方法
- 批次跑測試、存結果、生成報告
- 何時用人工評估 vs 自動評估

---

## 難度

★★★☆☆ 中等（技術不難，評估設計有深度）

---

## 時間

約 50 分鐘

---

## 核心概念

LLM-as-judge 是用一個 Claude（judge）來評估另一個 Claude（被評估者）的輸出。這解決了「誰來評 AI 輸出好不好」的問題。

**人工評估 vs 自動評估：**

| | 人工評估 | LLM-as-judge |
|--|----------|-------------|
| 速度 | 慢（幾天）| 快（幾分鐘）|
| 可擴展性 | 差（人力有限）| 好（平行跑）|
| 一致性 | 因人而異 | 穩定（同一 prompt）|
| 準確度 | 高 | 中高（有偏見風險）|

**使用場景：**  
你改了一個 prompt，想知道新版是否比舊版好。手動測 100 個例子太慢，用 LLM-as-judge 幾分鐘內就能有系統性的比較結果。

**避免偏見的關鍵：**  
- 讓 judge 評估時不知道哪個是 A、哪個是 B（blind evaluation）
- 評分標準要明確，不能說「好的回答」，要說「包含 X、Y、Z 要素」
- 比較時隨機交換 A/B 順序（position bias）

---

## 實作步驟

**步驟 1：安裝依賴**

```bash
pip install anthropic
```

**步驟 2：定義測試資料集**

準備一組問題（`test_cases`），每個問題有明確的「好答案標準」。

**步驟 3：準備兩個要比較的 prompt 版本**

```python
PROMPT_A = "Answer the question concisely."
PROMPT_B = "Answer the question with step-by-step reasoning and a clear conclusion."
```

**步驟 4：對所有測試問題同時跑兩個版本**

**步驟 5：讓 judge Claude 評分**

把問題、A 的回答、B 的回答一起傳給 judge，要求它打分數並給理由。

**步驟 6：統計結果並輸出報告**

---

## 程式碼範本

```python
import json
import random
import anthropic

client = anthropic.Anthropic()

TEST_CASES = [
    {
        "question": "解釋什麼是 API？",
        "criteria": "應包含定義、實際例子、和非技術人員也能理解的比喻",
    },
    {
        "question": "Python list 和 tuple 的差別是什麼？",
        "criteria": "應涵蓋可變性差異、效能差異、適用場景",
    },
    {
        "question": "如何除錯一個不知道哪裡出錯的程式？",
        "criteria": "應有系統性步驟、提到 print/log 和 debugger 工具、給出實際策略",
    },
]

PROMPT_A = "Answer the following question clearly and directly."
PROMPT_B = "Answer the following question step by step, then provide a concise summary at the end."

def generate_response(system_prompt: str, question: str, model: str = "claude-haiku-4-5-20251001") -> str:
    response = client.messages.create(
        model=model,
        max_tokens=512,
        system=system_prompt,
        messages=[{"role": "user", "content": question}],
    )
    return response.content[0].text

def judge_responses(question: str, criteria: str, response_1: str, response_2: str) -> dict:
    """讓 judge Claude 評比兩個回答，回傳哪個更好及理由"""
    
    # 隨機交換順序，避免 position bias
    swap = random.random() > 0.5
    if swap:
        response_1, response_2 = response_2, response_1
    
    judge_prompt = f"""You are an objective evaluator. Compare two responses to the same question.

Question: {question}

Evaluation criteria: {criteria}

Response 1:
{response_1}

Response 2:
{response_2}

Evaluate both responses based on the criteria. Respond in this exact JSON format:
{{
  "winner": 1 or 2,
  "scores": {{"response_1": <1-10>, "response_2": <1-10>}},
  "reasoning": "Brief explanation of why one is better"
}}"""

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=512,
        messages=[{"role": "user", "content": judge_prompt}],
    )
    
    result = json.loads(response.content[0].text)
    
    # 修正因為交換順序導致的 winner 判斷
    if swap:
        result["winner"] = 3 - result["winner"]  # 1→2, 2→1
        result["scores"] = {
            "response_1": result["scores"]["response_2"],
            "response_2": result["scores"]["response_1"],
        }
    
    return result

def run_evaluation():
    print("=== Evaluation Pipeline ===\n")
    
    results = []
    
    for i, case in enumerate(TEST_CASES):
        print(f"[{i+1}/{len(TEST_CASES)}] Q: {case['question'][:50]}...")
        
        # 生成兩個版本的回答
        response_a = generate_response(PROMPT_A, case["question"])
        response_b = generate_response(PROMPT_B, case["question"])
        
        # 評分
        judgment = judge_responses(
            case["question"],
            case["criteria"],
            response_a,
            response_b,
        )
        
        results.append({
            "question": case["question"],
            "response_a": response_a,
            "response_b": response_b,
            "judgment": judgment,
        })
        
        winner = "A" if judgment["winner"] == 1 else "B"
        print(f"  Winner: Prompt {winner} | A={judgment['scores']['response_1']}/10, B={judgment['scores']['response_2']}/10")
        print(f"  Reason: {judgment['reasoning'][:100]}...\n")
    
    # 統計總結
    a_wins = sum(1 for r in results if r["judgment"]["winner"] == 1)
    b_wins = len(results) - a_wins
    avg_a = sum(r["judgment"]["scores"]["response_1"] for r in results) / len(results)
    avg_b = sum(r["judgment"]["scores"]["response_2"] for r in results) / len(results)
    
    print("=== Summary ===")
    print(f"Prompt A wins: {a_wins}/{len(results)} | Average score: {avg_a:.1f}/10")
    print(f"Prompt B wins: {b_wins}/{len(results)} | Average score: {avg_b:.1f}/10")
    
    if avg_b > avg_a:
        print("\n✓ Recommendation: Use Prompt B (step-by-step reasoning)")
    else:
        print("\n✓ Recommendation: Use Prompt A (concise direct answer)")
    
    # 儲存完整結果
    with open("eval_results.json", "w", encoding="utf-8") as f:
        json.dump(results, f, ensure_ascii=False, indent=2)
    print("\n完整結果已儲存至 eval_results.json")

if __name__ == "__main__":
    run_evaluation()
```

---

## 常見問題

**Q1：judge Claude 總是偏向較長的回答**  
這是 LLM 的常見 length bias。在 judge prompt 中明確說明「長度不是評分標準，只評估內容品質」，並在評分標準中強調「精確簡潔同樣是優點」。

**Q2：每次跑出來的結果不一樣，無法信任**  
judge 用 `temperature=0` 讓結果更穩定。另外每個問題跑 3 次取多數決（majority vote），比只跑一次更可靠。

**Q3：judge 的評分格式不對，JSON 解析失敗**  
用 `response_format={"type": "json_object"}` 強制要求 JSON 輸出，或在 prompt 中加「只輸出 JSON，不要有任何其他文字」。

**Q4：測試問題太少，結果沒有代表性**  
至少要 20-30 個涵蓋不同難度和類型的問題才有統計意義。問題應該覆蓋你的實際使用場景，而不是隨便想的。

**Q5：A 和 B 的差距很小，不知道要選哪個**  
分數差距小於 0.5 分時，差異可能不顯著。可以增加測試題目數量，或改用更嚴格的評分標準（5 個明確子項目各 20 分）讓差異更明顯。

---

## 完成條件

- [ ] `TEST_CASES` 至少有 5 個不同主題的問題，每個都有明確的評估標準
- [ ] 程式成功執行，對每個問題都生成 A/B 兩個版本的回答並完成評分
- [ ] judge 輸出包含 winner、分數、理由，且 JSON 格式正確解析
- [ ] 最終輸出總結報告，顯示哪個 prompt 版本整體勝出
- [ ] 結果儲存到 `eval_results.json`，能事後查看每題的詳細比較
