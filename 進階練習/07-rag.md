# 07 · RAG + 知識庫

**讓 Claude 擁有你的文件庫作為長期記憶**

---

## 你會做到什麼

建立一個本地知識庫，把你的文件、筆記、程式碼庫向量化，讓 Claude 在對話中直接查詢，解決「context 塞不下」的問題。

---

## 需要先完成

- 03-claude-api（API 呼叫基礎）
- Python 基礎（推薦）或 Node.js

---

## 你會學到什麼

- Embedding 是什麼、如何生成
- 向量資料庫選擇：Chroma（本地）vs Pinecone（雲端）
- 查詢流程：問題 → 向量搜尋 → 取回相關片段 → 餵給 Claude
- 分割文件的策略（chunk size、overlap）
- 用 Claude 的 Embeddings API

---

## 難度

★★★★☆ 中高

---

## 時間

約 75 分鐘

---

## 核心概念

RAG（Retrieval-Augmented Generation）解決的問題：**Claude 的 context 有上限，放不下整個文件庫**。

**沒有 RAG vs 有 RAG：**

| 沒有 RAG | 有 RAG |
|----------|--------|
| 把所有文件塞進 prompt | 只取出最相關的幾段 |
| context 很快爆炸 | context 精準，費用低 |
| Claude 容易「忘記」中間內容 | 檢索到的都是高相關片段 |

**運作流程：**
```
建立階段：文件 → 切段 → 向量化 → 存進向量 DB
查詢階段：問題 → 向量化 → 相似度搜尋 → 取出相關段落 → 餵給 Claude
```

**Embedding 是什麼？**  
把文字轉成一個數字陣列（向量），語意相近的文字在向量空間中距離也近。搜尋時找「距離最近的向量」就是找「語意最相近的段落」。

---

## 實作步驟

**步驟 1：安裝依賴**

```bash
pip install anthropic chromadb
```

**步驟 2：準備文件**

```bash
# 建立測試用文件
mkdir docs
echo "Claude Code 是 Anthropic 開發的 AI 程式設計助手..." > docs/claude-intro.txt
echo "GitHub Actions 是 CI/CD 平台，可以自動化工作流程..." > docs/github-actions.txt
```

**步驟 3：建立向量資料庫並匯入文件**

```python
python build_knowledge_base.py
```

**步驟 4：實作查詢函式**

把使用者問題向量化，從 Chroma 搜出最相關的 3 段，組成 context 傳給 Claude。

**步驟 5：測試查詢**

```python
answer = ask("Claude Code 可以做什麼？")
print(answer)
```

**步驟 6：調整 chunk size 觀察效果**

把 `CHUNK_SIZE` 從 500 改成 200 或 1000，觀察查詢結果品質的差異。

---

## 程式碼範本

```python
import os
import anthropic
import chromadb

client = anthropic.Anthropic()
chroma_client = chromadb.Client()

CHUNK_SIZE = 500
CHUNK_OVERLAP = 50
COLLECTION_NAME = "my_knowledge_base"

def chunk_text(text: str, chunk_size: int = CHUNK_SIZE, overlap: int = CHUNK_OVERLAP) -> list[str]:
    """把長文件切成有重疊的小段"""
    chunks = []
    start = 0
    while start < len(text):
        end = start + chunk_size
        chunks.append(text[start:end])
        start += chunk_size - overlap
    return chunks

def get_embedding(text: str) -> list[float]:
    """用 Voyage AI Embeddings（透過 Anthropic）取得向量"""
    # 注意：Anthropic 本身建議搭配 Voyage embeddings
    # 這裡用簡化版，實際可換成 voyage-ai 或 OpenAI embeddings
    response = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=1,
        messages=[{"role": "user", "content": text}],
    )
    # 實際使用時換成真正的 embedding API
    # 這裡回傳假向量僅作示意，請替換為真實實作
    import hashlib
    hash_val = int(hashlib.md5(text.encode()).hexdigest(), 16)
    return [(hash_val >> i & 0xFF) / 255.0 for i in range(384)]

def build_knowledge_base(docs_dir: str = "docs"):
    """把資料夾內所有 .txt 檔建立成向量知識庫"""
    collection = chroma_client.get_or_create_collection(COLLECTION_NAME)
    
    for filename in os.listdir(docs_dir):
        if not filename.endswith(".txt"):
            continue
        
        filepath = os.path.join(docs_dir, filename)
        with open(filepath, "r", encoding="utf-8") as f:
            text = f.read()
        
        chunks = chunk_text(text)
        print(f"  {filename}: {len(chunks)} chunks")
        
        for i, chunk in enumerate(chunks):
            doc_id = f"{filename}_{i}"
            embedding = get_embedding(chunk)
            collection.add(
                ids=[doc_id],
                embeddings=[embedding],
                documents=[chunk],
                metadatas=[{"source": filename, "chunk_index": i}],
            )
    
    print(f"知識庫建立完成，共 {collection.count()} 個片段")
    return collection

def retrieve(query: str, n_results: int = 3) -> list[dict]:
    """從知識庫找出最相關的片段"""
    collection = chroma_client.get_collection(COLLECTION_NAME)
    query_embedding = get_embedding(query)
    
    results = collection.query(
        query_embeddings=[query_embedding],
        n_results=n_results,
        include=["documents", "metadatas", "distances"],
    )
    
    retrieved = []
    for doc, meta, dist in zip(
        results["documents"][0],
        results["metadatas"][0],
        results["distances"][0],
    ):
        retrieved.append({
            "content": doc,
            "source": meta["source"],
            "relevance_score": 1 - dist,  # 轉成相似度（越高越相關）
        })
    
    return retrieved

def ask(question: str) -> str:
    """RAG 完整流程：查詢 + 生成回答"""
    # 1. 檢索相關片段
    relevant_chunks = retrieve(question)
    
    # 2. 組合 context
    context_parts = []
    for chunk in relevant_chunks:
        context_parts.append(
            f"[來源: {chunk['source']} | 相關度: {chunk['relevance_score']:.2f}]\n{chunk['content']}"
        )
    context = "\n\n---\n\n".join(context_parts)
    
    # 3. 呼叫 Claude
    response = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=1024,
        system="你是一個知識庫助手。根據提供的參考資料回答問題，如果資料中沒有相關資訊請明確說明。",
        messages=[
            {
                "role": "user",
                "content": f"參考資料：\n\n{context}\n\n問題：{question}",
            }
        ],
    )
    
    return response.content[0].text

if __name__ == "__main__":
    print("=== 建立知識庫 ===")
    build_knowledge_base("docs")
    
    print("\n=== 測試查詢 ===")
    questions = [
        "Claude Code 主要有什麼功能？",
        "GitHub Actions 怎麼設定觸發條件？",
    ]
    for q in questions:
        print(f"\nQ: {q}")
        print(f"A: {ask(q)}")
```

---

## 常見問題

**Q1：Embedding 維度不一致導致查詢失敗**  
建立知識庫和查詢時必須用**同一個 embedding 模型**，否則向量維度不同無法比較。Chroma 在 `create_collection` 時會記住維度，之後用不同模型會報錯。

**Q2：找到的片段相關但回答品質差**  
通常是 chunk size 問題。太小（< 100 字）缺少上下文，太大（> 1000 字）包含太多無關內容。建議從 300-500 字開始測試。

**Q3：同一個問題每次回答都不同**  
Claude 有一定的隨機性（temperature）。如果需要穩定輸出，加上 `temperature=0`。但 RAG 的不穩定更常來自 embedding 搜尋結果的順序，調整 `n_results` 也有幫助。

**Q4：文件更新後查詢結果沒有更新**  
Chroma 不會自動偵測文件變更。需要重新執行 `build_knowledge_base()`，或先 `collection.delete(ids=[...])` 再重新 add。

**Q5：`collection.count()` 很大但查詢很慢**  
Chroma 本地版在資料量大時效能下降。超過 10 萬筆考慮換 Pinecone 或 Qdrant 等有索引優化的向量 DB。

---

## 完成條件

- [ ] `docs/` 資料夾有至少 3 份不同主題的文件
- [ ] `build_knowledge_base()` 執行成功，`collection.count()` 顯示正確片段數量
- [ ] `ask()` 能正確回答文件內存在的問題
- [ ] 問一個文件裡**沒有**的問題，Claude 回答「資料中沒有相關資訊」而不是亂猜
- [ ] 嘗試調整 `CHUNK_SIZE`，觀察並記錄兩種大小對回答品質的影響差異
