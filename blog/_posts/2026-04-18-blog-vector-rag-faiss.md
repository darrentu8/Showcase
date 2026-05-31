---
layout: post
title: Vector RAG 與 FAISS - 用 Embedding 做語意檢索
image: 'https://images.unsplash.com/photo-1692607987924-bfa64745358c?auto=format&fit=crop&w=1600&q=80'
accent_image: 
  background: url('https://images.unsplash.com/photo-1692607987924-bfa64745358c?auto=format&fit=crop&w=1600&q=80') center/cover
  overlay: false
accent_color: '#9333EA'
theme_color: '#9333EA'
invert_sidebar: false
related_posts:
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 前言

上一篇用 Markdown KB 和 BM25 做 keyword retrieval。這篇看 `build-moat-live-sessions/knowledge_base_qa_bot` 裡的另一個版本：Vector RAG。

Vector RAG 的目標是讓系統不只靠關鍵字，而是能理解語意相近的問題。

例如使用者問：

~~~text
When will I get my money back?
~~~

文件可能寫的是：

~~~text
Refund Timeline
~~~

兩者不一定共享很多關鍵字，但 embedding 可以讓它們在語意空間中靠近。

## 技術組合

專案使用：

1. FastAPI：提供 `/index` 和 `/chat` API。
2. LangChain：包裝 Document、splitter 和 LLM。
3. OpenAI Embeddings：產生向量。
4. FAISS：本地向量索引。
5. Markdown docs：知識來源。

requirements 裡可以看到：

~~~text
fastapi==0.115.6
langchain==0.3.14
langchain-community==0.3.14
langchain-openai==0.3.0
faiss-cpu==1.9.0.post1
~~~

這是一個很適合 local prototype 的 RAG stack。

## Embedding Model

Indexer 裡指定：

~~~python
EMBEDDING_MODEL = "text-embedding-3-small"
~~~

取得 embeddings 時會檢查 API key：

~~~python
def get_embeddings():
    if not os.getenv("OPENAI_API_KEY"):
        raise RuntimeError("OPENAI_API_KEY is not set")
    return OpenAIEmbeddings(
        model=EMBEDDING_MODEL,
        request_timeout=20,
        max_retries=1,
    )
~~~

Markdown KB 只在生成答案時需要 OpenAI API key，但 Vector RAG 在 `/index` 和 `/chat` query embedding 都需要。

## Markdown Section 到 Document

Vector RAG 仍然先用 Markdown heading 切 section。

~~~python
Document(
    page_content="\n".join([*heading_path, content]),
    metadata={
        "source": section_id,
        "file": path.name,
        "heading": " > ".join(heading_path),
    },
)
~~~

這點很重要。即使用向量檢索，也不要丟掉來源 metadata。否則最後 answer 會很難追溯。

## Chunking

專案使用 `RecursiveCharacterTextSplitter`：

~~~python
splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=100,
    separators=["\n\n", "\n", ". ", " "],
)
~~~

Chunk size 和 overlap 是 RAG 很常調的參數。

1. Chunk 太大：檢索結果可能包含太多無關內容。
2. Chunk 太小：答案需要的上下文可能被切散。
3. Overlap 太小：跨段落資訊可能斷掉。
4. Overlap 太大：成本和重複內容增加。

這裡用 500/100 作為 prototype 起點，很適合 FAQ 型文件。

## 建立 FAISS Index

建立流程：

~~~python
section_docs = []
for path in markdown_files:
    section_docs.extend(load_markdown_sections(path))

chunks = splitter.split_documents(section_docs)
vectorstore = FAISS.from_documents(chunks, get_embeddings())
~~~

接著保存到本地：

~~~python
vectorstore.save_local(str(index_dir))
~~~

並寫入 metadata：

~~~python
metadata = {
    "embedding_model": EMBEDDING_MODEL,
    "files_indexed": files_indexed,
    "sections_indexed": sections_indexed,
}
~~~

保存 embedding model 很重要，因為不同 model 產生的向量空間不能混用。

## 查詢流程

`/chat` 時先做 similarity search：

~~~python
ranked_chunks = indexer.search(question, k=3)
~~~

再把 context 放進 prompt：

~~~python
context_blocks.append(
    f"[Source: {doc.metadata.get('source', 'unknown')}]\n"
    f"[Vector distance: {score:.4f}]\n"
    f"{doc.page_content}"
)
~~~

最後用 LLM 生成答案，並回傳 sources：

~~~python
{
    "answer": response.content,
    "sources": sources,
}
~~~

這樣前端可以顯示答案，也能顯示引用來源。

## 安全載入本地 Index

FAISS local index 會包含 pickle，所以專案註解中特別提醒：

~~~python
FAISS.load_local(
    str(index_dir),
    get_embeddings(),
    allow_dangerous_deserialization=True,
)
~~~

這個 flag 代表只能載入自己應用程式產生且可信任的 index。不要載入外部下載來的 pickle 檔案。

## 總結

Vector RAG 的優點是語意檢索能力比 BM25 更好，尤其是使用者問題和文件用字不同時。

但它也增加了幾個成本：

1. 需要 embedding API key。
2. Index 建立需要時間和費用。
3. FAISS index 需要保存和版本管理。
4. Chunking 參數需要調整。

我的建議是：小型 FAQ 先用 Markdown KB + BM25 做 baseline，再用 Vector RAG 比較召回品質。下一篇會把兩種做法放在一起看，整理什麼情境該選哪一種。
