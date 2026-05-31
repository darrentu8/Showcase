---
layout: post
title: MCP Task Scheduler - 讓 AI 助理可以安排任務
image: 'https://images.unsplash.com/photo-1642543492457-39a2ce63bb59?auto=format&fit=crop&w=1600&q=80'
accent_image: 
  background: url('https://images.unsplash.com/photo-1642543492457-39a2ce63bb59?auto=format&fit=crop&w=1600&q=80') center/cover
  overlay: false
accent_color: '#2F80ED'
theme_color: '#2F80ED'
invert_sidebar: false
related_posts:
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 前言

在 `build-moat-live-sessions/chatgpt_task` 這個練習裡，核心目標不是做一個單純的 Todo List，而是做一個可以被 AI 助理透過 MCP 呼叫的任務排程器。

傳統排程系統通常是人打開後台、填表單、選時間，最後由後端處理任務。但是 MCP 的情境剛好反過來：使用者用自然語言描述需求，AI 助理判斷要呼叫哪個工具，然後工具把結構化參數寫進系統。

這篇先整理整體設計，從「AI 工具」角度看一個任務排程器應該怎麼拆。

## 系統目標

這個 prototype 提供四個 MCP tools：

1. `task.create`：建立未來要執行的任務。
2. `task.list`：列出目前所有任務。
3. `task.status`：查詢單一任務狀態。
4. `task.cancel`：取消還沒有完成的任務。

對使用者來說，操作可能只是：

~~~text
明天早上 9 點提醒我 review PR #123
~~~

但對系統來說，這句話最後必須被轉成明確的 tool call：

~~~json
{
  "description": "review PR #123",
  "scheduled_at": "2026-05-03T09:00:00"
}
~~~

這裡的重點是讓 AI 可以把模糊語意變成穩定 API，而不是把商業邏輯塞進 prompt 裡。

## 架構設計

專案的核心流程可以拆成幾層：

~~~text
User
  -> MCP Client
  -> MCP Tool Call
  -> Tool Handler
  -> SQLite
  -> Watcher
  -> Queue
  -> Worker
~~~

`mcp_server.py` 負責暴露 tools 和處理 tool call，`models.py` 負責保存任務資料，`scheduler.py` 則處理 watcher、queue 和 worker。

這樣拆的好處是 MCP 層不需要知道任務怎麼執行，scheduler 也不需要知道任務是 Claude、ChatGPT、CLI 還是其他 client 建立的。

## Tool Handler 的角色

在 `mcp_server.py` 裡，handler 是純商業邏輯：

~~~python
def handle_create_task(db, *, description: str, scheduled_at: str) -> dict:
    dt = datetime.fromisoformat(scheduled_at)
    job = Job(
        description=description,
        scheduled_at=dt,
        time_bucket=get_time_bucket(dt),
    )
    db.add(job)
    db.commit()
    db.refresh(job)
    return {
        "job_id": job.id,
        "status": job.status,
        "scheduled_at": str(job.scheduled_at),
    }
~~~

這段程式碼有幾個值得注意的地方：

1. handler 接收的是已經結構化的參數。
2. handler 不負責解析自然語言。
3. handler 回傳 JSON-friendly 的 dict。
4. handler 不直接碰 MCP protocol 細節。

這樣寫會讓每個 tool 很容易測試，也可以把同一份商業邏輯拿去做 REST API 或 CLI。

## 為什麼不是直接呼叫後端 API

如果只是內部系統，REST API 已經夠用。但 MCP 的價值在於它提供了一個 AI client 可以理解的工具描述方式。

例如 tool definition 會告訴 client：

~~~python
Tool(
    name="task.create",
    description="Schedule a new task for future execution",
    inputSchema={
        "type": "object",
        "properties": {
            "description": {"type": "string"},
            "scheduled_at": {"type": "string", "format": "date-time"},
        },
        "required": ["description", "scheduled_at"],
    },
)
~~~

這份 schema 不只是文件，它也是 AI 助理選 tool、填參數、避免亂猜的重要依據。

## 任務狀態

任務狀態設計很簡單：

~~~text
pending -> queued -> running -> completed
                     -> failed
pending -> cancelled
queued  -> cancelled
~~~

prototype 裡先保留六種狀態：

1. `pending`：任務已建立，尚未到執行時間。
2. `queued`：watcher 已找到任務，放進 queue。
3. `running`：worker 正在執行。
4. `completed`：任務完成。
5. `failed`：任務失敗。
6. `cancelled`：任務被取消。

這個狀態模型不複雜，但對 AI tool 很重要。因為使用者常會問「那個任務現在怎樣了」，系統必須有穩定狀態可以回答。

## 開發時的驗證方式

專案建議先用 MCP inspector 驗證，不需要一開始就接 Claude Desktop：

~~~bash
npx @modelcontextprotocol/inspector python -m app.mcp_server
~~~

驗證流程：

1. 確認 inspector 看得到四個 tools。
2. 呼叫 `task.create` 建立任務。
3. 用 `task.status` 查詢剛剛回傳的 `job_id`。
4. 用 `task.list` 確認資料有寫入。
5. 用 `task.cancel` 測試取消流程。

這個流程可以把 MCP protocol、tool schema、handler 和 DB 寫入一次驗乾淨。

## 總結

MCP Task Scheduler 的重點不是任務排程本身，而是把「AI 助理可以操作的系統」拆成可維護的工程結構。

我覺得這個練習有三個收穫：

1. AI tool 的 schema 設計會直接影響模型是否容易選對工具。
2. MCP handler 應該保持單純，避免混入 protocol 和排程細節。
3. 任務排程要從一開始就保留狀態，後面才容易做查詢、取消和錯誤處理。

下一篇會接著看 tool naming 和 registry pattern，因為工具越多，命名和 dispatch 會開始影響整個系統的可擴充性。
