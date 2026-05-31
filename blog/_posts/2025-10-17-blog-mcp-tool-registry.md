---
layout: post
title: MCP Tool Registry Pattern - 避免工具路由變成 if else 迷宮
image: 'https://images.unsplash.com/photo-1743090661053-3d1feb2beab7?auto=format&fit=crop&w=1600&q=80'
accent_image: 
  background: url('https://images.unsplash.com/photo-1743090661053-3d1feb2beab7?auto=format&fit=crop&w=1600&q=80') center/cover
  overlay: false
accent_color: '#7C3AED'
theme_color: '#7C3AED'
invert_sidebar: false
related_posts:
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 前言

在 `build-moat-live-sessions/chatgpt_task` 的 MCP server 裡，有一個很小但很關鍵的設計：`TOOL_REGISTRY`。

當工具只有一兩個時，用 `if name == ...` 也能跑。但 MCP server 很容易從四個工具長成十幾個工具，如果每個 tool call 都靠 if else 判斷，程式很快就會變得難測、難擴充，也很容易在新增工具時漏掉某個分支。

這篇整理為什麼 registry pattern 適合 MCP tool routing。

## Tool 命名先決定一半的體驗

專案裡的工具名稱使用 namespace 加 action verb：

~~~text
task.create
task.list
task.status
task.cancel
~~~

這種命名方式對人和 AI 都比較清楚。

如果工具叫做 `create`、`list`、`status`，在大型 MCP server 裡很容易和其他資源混在一起。加上 `task.` namespace 後，模型更容易理解這組工具都在處理同一種資源。

我會把命名原則整理成三點：

1. 名詞當 namespace，例如 `task`。
2. 動詞當 action，例如 `create`。
3. 回傳資料維持一致，例如都使用 `job_id`。

## Tool Definition 是給模型看的 API 文件

每個 MCP tool 都有 `name`、`description` 和 `inputSchema`。

~~~python
Tool(
    name="task.status",
    description="Get the status of a scheduled task by job_id",
    inputSchema={
        "type": "object",
        "properties": {
            "job_id": {
                "type": "integer",
                "description": "The job ID returned by task.create",
            },
        },
        "required": ["job_id"],
    },
)
~~~

這份 schema 不只是讓 UI 可以顯示表單，也是讓 LLM 判斷怎麼填參數的依據。描述太短，模型可能不知道何時用；描述太長，又可能讓工具選擇變得模糊。

比較好的寫法是把 tool 的目的和參數來源寫清楚，例如 `job_id` 是從 `task.create` 回傳來的。

## Registry Pattern

專案最後用一個 dict 把工具名稱對應到 handler：

~~~python
TOOL_REGISTRY = {
    "task.create": handle_create_task,
    "task.list": handle_list_tasks,
    "task.status": handle_get_status,
    "task.cancel": handle_cancel_task,
}
~~~

dispatch 時只需要查表：

~~~python
def route_tool_call(tool_name: str, arguments: dict, db: Session) -> dict:
    handler = TOOL_REGISTRY.get(tool_name)
    if handler is None:
        return {"error": f"Unknown tool: {tool_name}"}
    return handler(db, **arguments)
~~~

這樣新增一個工具時，只要做三件事：

1. 寫 handler。
2. 加 tool definition。
3. 把 name 加進 `TOOL_REGISTRY`。

比起在 `call_tool` 裡堆 if else，這種方式更像是把工具表攤開給維護者看。

## Handler 要保持純粹

一個好的 handler 應該只處理商業邏輯。

以 `handle_cancel_task` 為例：

~~~python
def handle_cancel_task(db: Session, *, job_id: int) -> dict:
    job = db.query(Job).filter(Job.id == job_id).first()
    if job is None:
        return {"error": f"Job {job_id} not found"}
    if job.status in ("completed", "failed"):
        return {"error": f"Cannot cancel job in '{job.status}' state"}
    job.status = "cancelled"
    db.commit()
    return {"job_id": job.id, "status": "cancelled"}
~~~

它不需要知道 MCP stdio、不需要知道 JSON-RPC，也不需要知道 tool definition 怎麼宣告。這讓 handler 可以在單元測試中直接被呼叫。

## Async Wrapper 的意義

MCP server 的 `call_tool` 是 async function，但 SQLAlchemy handler 是同步的。專案使用 `asyncio.to_thread` 避免阻塞 event loop：

~~~python
result = await asyncio.to_thread(route_tool_call, name, arguments or {}, db)
~~~

這個做法適合 prototype：handler 保持簡單，server 仍然能處理 async protocol。如果之後變成高流量服務，再考慮改成 async DB driver。

## 常見錯誤

做 MCP server 時有幾個坑很常見：

1. 在 stdout 印 debug log，導致 stdio protocol 被污染。
2. tool name 不一致，definition 叫 `task.status`，registry 卻寫 `task.get_status`。
3. handler 回傳不可 JSON serialize 的物件，例如 datetime 或 SQLAlchemy model。
4. schema 沒寫 required，模型填參數時不穩定。

這些錯誤不一定會在 Python import 時爆掉，常常是接到 MCP client 後才發現。因此 inspector 是很好的第一道驗證。

## 總結

Registry pattern 看起來只是幾行 dict，但它讓 MCP server 的擴充方式變得很清楚。

這篇的重點可以收斂成三句：

1. Tool name 要像 API endpoint 一樣穩定。
2. Tool definition 要像給 AI 看的文件。
3. Tool registry 要讓 dispatch 保持可讀、可測、可擴充。

下一篇會進入 scheduler 本身，看看 watcher、queue、worker 怎麼把 `pending` 任務推進到 `completed`。
