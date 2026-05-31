---
layout: post
title: Markdown KB 與 BM25 - 先不用向量資料庫也能做知識庫問答
image: 'https://images.unsplash.com/photo-1491841573634-28140fc7ced7?auto=format&fit=crop&w=1600&q=80'
accent_image: 
  background: url('https://images.unsplash.com/photo-1491841573634-28140fc7ced7?auto=format&fit=crop&w=1600&q=80') center/cover
  overlay: false
accent_color: '#2563EB'
theme_color: '#2563EB'
invert_sidebar: false
related_posts:
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 前言

做 Knowledge Base Q&A Bot 時，很容易一開始就想到 vector database、embedding 和 RAG。

但在 `build-moat-live-sessions/knowledge_base_qa_bot` 裡，有一個很好的對照組：Markdown KB。

它不先用向量資料庫，而是把 Markdown 文件切成 section，建立可檢查的 index，再用 BM25 做 keyword retrieval。這種做法很適合小型知識庫、政策文件、FAQ 和內部文件。

## 為什麼用 Markdown 當 Canonical Format

專案裡的 sample docs 放在 `docs/`：

~~~text
account_help.md
refund_policy.md
shipping_faq.md
~~~

Markdown 的好處是：

1. 人可以直接讀。
2. Git diff 清楚。
3. heading 可以當 section boundary。
4. 容易轉成 HTML、wiki 或其他格式。
5. AI agent 也很容易理解。

對知識庫來說，Markdown 不只是輸入格式，也可以是 canonical knowledge format。

## 解析 Markdown Section

Indexer 使用 heading 來切 section：

~~~python
HEADING_RE = re.compile(r"^(#{1,6})\s+(.+?)\s*$")
~~~

每個 section 保存：

~~~python
@dataclass
class Section:
    id: str
    file: str
    heading: str
    heading_path: list[str]
    content: str
    tokens: list[str]
~~~

其中 `id` 會像這樣：

~~~text
refund_policy.md#refund-timeline
~~~

這個設計非常重要，因為回答問題時可以引用來源，而不是只回答一段沒有根據的文字。

## Index JSON 可檢查

Markdown KB 會把索引寫到：

~~~text
.kb/index.json
~~~

payload 裡包含 sections 和 stats：

~~~python
payload = {
    "sections": [section.to_dict() for section in sections],
    "stats": {
        "files_indexed": files_indexed,
        "sections_indexed": len(sections),
        "avg_doc_len": avg_doc_len,
    },
}
~~~

這和很多黑盒向量檢索不太一樣。你可以直接打開 JSON，看系統到底切了哪些 section、存了哪些 tokens。

對 live session 和教學來說，inspectable index 是很大的優點。

## Tokenize 與 Stop Words

Indexer 使用簡單 regex 做 tokenization：

~~~python
TOKEN_RE = re.compile(r"[a-z0-9]+")

def tokenize(text: str) -> list[str]:
    return [
        t for t in TOKEN_RE.findall(text.lower())
        if t not in STOP_WORDS
    ]
~~~

這不是最強的 NLP pipeline，但足夠解釋 BM25 retrieval 的核心：

1. 把問題切成 tokens。
2. 把文件 section 切成 tokens。
3. 計算 term frequency 和 inverse document frequency。
4. 回傳分數最高的 section。

## BM25 Search

BM25 會考慮三件事：

1. query term 是否出現在 section。
2. 這個 term 在整個 corpus 裡是否稀有。
3. section 長度是否需要 normalize。

專案裡的分數計算：

~~~python
idf = math.log(
    1 + (n_docs - doc_freq[term] + 0.5) / (doc_freq[term] + 0.5)
)
tf = counts[term]
length_norm = 1 - b + b * (len(section.tokens) / avg_doc_len)
score += idf * ((tf * (k1 + 1)) / (tf + k1 * length_norm))
~~~

BM25 特別適合 FAQ 類問題，因為問題和文件常常共享關鍵字，例如 refund、shipping、password、email。

## Chat API

Markdown KB expose 兩個主要 endpoint：

~~~text
POST /index
POST /chat
~~~

`/index` 重建索引，`/chat` 先 retrieval，再把 top sections 放進 prompt：

~~~python
def build_prompt(query: str, ranked_sections: list) -> str:
    context = "\n\n---\n\n".join(context_blocks)
    return f"CONTEXT:\n{context}\n\nQUESTION:\n{query}"
~~~

LLM 的 system prompt 強制要求只能根據 context 回答，並引用 `filename#heading`。

## Fallback 很重要

如果沒有 index：

~~~text
The knowledge base has not been indexed yet. Call POST /index first.
~~~

如果找不到相關 section：

~~~text
I cannot confirm from the knowledge base.
~~~

這種 fallback 對知識庫問答很重要。比起硬回答，承認不知道更可靠。

## 總結

Markdown KB + BM25 是一個很適合入門的知識庫問答架構。

它的優點：

1. 不需要 embedding 成本。
2. Index 可檢查、可 debug。
3. 對 FAQ 和政策文件效果通常不錯。
4. Source citation 很自然。

缺點是語意相近但關鍵字不同的問題可能找不到。下一篇會看 Vector RAG，用 embedding 和 FAISS 改善這個問題。
