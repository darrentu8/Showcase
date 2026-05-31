---
layout: post
title: Knowledge Base QA Bot - 比較 Markdown KB 與 Vector RAG
image: 'https://images.unsplash.com/photo-1655721530791-59f5bbd64a48?auto=format&fit=crop&w=1600&q=80'
accent_image: 
  background: url('https://images.unsplash.com/photo-1655721530791-59f5bbd64a48?auto=format&fit=crop&w=1600&q=80') center/cover
  overlay: false
accent_color: '#111827'
theme_color: '#111827'
invert_sidebar: false
related_posts:
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 前言

`build-moat-live-sessions/knowledge_base_qa_bot` 最有意思的地方，是它同時提供兩種完整答案：

1. Markdown KB：section-level Markdown index + BM25 keyword search。
2. Vector RAG：Markdown chunks + OpenAI embeddings + FAISS vector search。

這篇不再只看實作，而是比較兩種策略的取捨，並整理一個可持續改善的 QA Bot 評估方式。

## 共同 API

兩個版本都提供相同 API：

~~~text
GET  /health
POST /index
POST /chat
~~~

這是很好的設計，因為前端或測試程式不需要知道底層使用 BM25 還是 FAISS。

只要 response shape 一樣，就可以用同一組問題比較兩種 retrieval 策略。

~~~python
class ChatResponse(BaseModel):
    answer: str
    sources: list[SourceInfo]
~~~

對產品來說，這就是把實驗變成可替換的 implementation detail。

## 兩種 Retrieval 的差異

可以先用表格比較：

| 面向 | Markdown KB | Vector RAG |
|---|---|---|
| 檢索方式 | BM25 keyword search | Embedding similarity search |
| 成本 | 低，不需要 embedding | 較高，需要 embedding |
| 可解釋性 | 高，可看 `.kb/index.json` | 中，需要看 chunk 和 distance |
| 語意召回 | 普通 | 較好 |
| 適合資料 | FAQ、政策、短文件 | 問法多變、語意相近文件 |
| 部署複雜度 | 低 | 中 |

我會把 Markdown KB 當 baseline，把 Vector RAG 當進階版本，而不是一開始就假設 vector 一定比較好。

## Source Citation 是核心

兩個版本的 system prompt 都要求：

~~~text
Only answer using the provided CONTEXT.
Cite sources using filename#heading.
If the CONTEXT does not contain the answer, say:
"I cannot confirm from the knowledge base."
~~~

這三條規則對知識庫問答非常重要。

沒有引用來源的 QA Bot，使用者很難信任。沒有 fallback 的 QA Bot，模型很容易在資料不足時硬猜。沒有 context boundary 的 QA Bot，回答就會混入外部知識。

## 評估問題集

要比較 retrieval，先準備固定問題集。

例如 sample docs 裡有 account、refund、shipping，可以設計：

~~~text
How long do refunds take?
Can I change my email address?
How do I reset my password?
Do expedited orders include tracking?
Which items are non-refundable?
Can I delete my account?
~~~

每個問題都要標記期待來源，例如：

~~~text
refund_policy.md#refund-timeline
account_help.md#change-email-address
shipping_faq.md#tracking-number
~~~

評估時不要只看答案順不順，也要看 source 是否抓對。

## 評估指標

簡單版可以看四個指標：

1. Top 1 source 是否正確。
2. Top 3 sources 是否包含正確答案。
3. Answer 是否引用正確文件。
4. 資料不足時是否 fallback。

進階一點可以加：

1. Paraphrase robustness：換一種問法是否還找得到。
2. No-answer precision：未知問題是否拒答。
3. Latency：索引和查詢速度。
4. Cost：embedding 和 LLM token 成本。

這些指標會比單純人工感覺更可靠。

## Paraphrase Comparison

專案 README 的 stretch goal 提到 paraphrase comparison，我覺得這是很值得做的下一步。

同一個問題可以寫成多種問法：

~~~text
How long do refunds take?
When will I get my money back?
What is the refund timeline?
How many days before a refund arrives?
~~~

BM25 可能對第三句效果最好，因為它和文件標題共享 `refund timeline`。Vector RAG 則可能對第二句也有不錯召回。

這種比較能幫我們判斷是否真的需要 embedding。

## Streaming 與 UI

專案也提到可以加：

~~~text
POST /chat/stream
~~~

我會建議 stream event 至少拆成三種：

1. `sources`：先回傳 selected sources。
2. `token`：逐步回傳答案文字。
3. `done`：完成訊號。

這樣前端可以先顯示「本次回答引用了哪些文件」，再開始顯示生成內容。對知識庫產品來說，來源先出現會讓使用者更安心。

## 什麼時候選哪一個

我會這樣選：

1. 文件少、問題直接、成本敏感：先用 Markdown KB。
2. 文件多、問法變化大、需要語意召回：用 Vector RAG。
3. 想要可解釋 baseline：一定先做 Markdown KB。
4. 需要產品級 QA：兩者都保留，用評估集定期比較。

RAG 系統不是只要接上 vector database 就完成。真正的工作是資料整理、檢索評估、fallback、引用來源和持續迭代。

## 總結

這個 Knowledge Base QA Bot 練習把 RAG 的核心問題拆得很清楚：

1. Markdown 是很好的 canonical knowledge format。
2. BM25 是低成本且可解釋的 baseline。
3. Vector RAG 補強語意召回，但帶來成本和複雜度。
4. 共同 API 讓兩種策略可以公平比較。
5. Source citation 和 fallback 是 QA Bot 的信任基礎。

如果要繼續延伸，我會先做一份固定 evaluation set，再把 Markdown KB 和 Vector RAG 的 retrieval 結果寫成報表。這樣每次改 chunk size、prompt 或模型時，都能知道系統是真的變好，還是只是感覺變好。
