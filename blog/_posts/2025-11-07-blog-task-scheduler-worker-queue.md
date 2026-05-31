---
layout: post
title: Watcher Queue Worker - 任務排程系統的最小可行架構
image: 'https://images.unsplash.com/photo-1775519520461-6b6e068d9250?auto=format&fit=crop&w=1600&q=80'
accent_image: 
  background: url('https://images.unsplash.com/photo-1775519520461-6b6e068d9250?auto=format&fit=crop&w=1600&q=80') center/cover
  overlay: false
accent_color: '#0EA5E9'
theme_color: '#0EA5E9'
invert_sidebar: false
related_posts:
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 前言

在 `build-moat-live-sessions/chatgpt_task` 裡，排程器沒有一開始就接 Celery、SQS 或 Redis，而是用 Python 內建的 `queue.Queue` 和 thread 做出最小可行架構。

這樣的設計很適合教學和 prototype，因為它讓我們看清楚一個任務排程系統最基本的三個角色：

1. Watcher：找出到期任務。
2. Queue：暫存等待執行的任務 ID。
3. Worker：取出任務並執行。

等這三個角色穩定後，再把 in-memory queue 換成真正的 message broker，就不會改動太多商業邏輯。

## Watcher 負責掃描

Watcher 的工作是定期查詢資料庫，找出目前應該執行的任務。

~~~python
def find_due_jobs(current_time: datetime, db: Session) -> list[Job]:
    bucket = get_time_bucket(current_time)
    return (
        db.query(Job)
        .filter(
            Job.time_bucket == bucket,
            Job.scheduled_at <= current_time,
            Job.status == "pending",
        )
        .all()
    )
~~~

這裡不是掃全部任務，而是先用 `time_bucket` 限縮範圍，再判斷 `scheduled_at` 和 `status`。

prototype 裡 watcher 每隔幾秒跑一次：

~~~python
def watcher_loop(interval: int = 10):
    while True:
        db = SessionLocal()
        try:
            now = _utcnow()
            due_jobs = find_due_jobs(now, db)
            for job in due_jobs:
                job.status = "queued"
                db.commit()
                job_queue.put(job.id)
        finally:
            db.close()
        time.sleep(interval)
~~~

這段流程的關鍵是先把狀態改成 `queued`，再把 `job.id` 放進 queue。如此一來，下一輪 watcher 不會重複撿到同一個 `pending` 任務。

## Queue 只放 Job ID

專案裡的 queue 是：

~~~python
job_queue: queue.Queue[int] = queue.Queue()
~~~

只放 `job_id`，不放整個 job payload，這是一個很實用的習慣。

原因有三個：

1. queue 內容越小越穩定。
2. worker 取出後可以重新讀取最新資料。
3. 如果任務已經被取消，worker 可以用 DB 狀態判斷是否跳過。

在正式系統裡，這個 queue 可以換成 SQS、RabbitMQ、Redis Stream 或 Kafka，但 message 裡仍然建議放最小識別資訊，而不是把整包資料塞進去。

## Worker 負責執行

Worker 從 queue 拿到 `job_id` 後，再回資料庫查一次：

~~~python
def worker_loop():
    while True:
        job_id = job_queue.get()
        db = SessionLocal()
        try:
            job = db.query(Job).filter(Job.id == job_id).first()
            if job is None or job.status == "cancelled":
                continue

            job.status = "running"
            db.commit()

            job.result = f"Executed: {job.description}"
            job.status = "completed"
            db.commit()
        except Exception as e:
            job.status = "failed"
            job.result = str(e)
            db.commit()
        finally:
            db.close()
            job_queue.task_done()
~~~

這個 worker 做了幾件基本但重要的事：

1. 執行前重新查 DB。
2. 支援 cancelled 任務跳過。
3. 執行中標記 `running`。
4. 成功後標記 `completed`。
5. 失敗後保存錯誤訊息。
6. 最後呼叫 `task_done()`。

這些狀態轉換是後續做 dashboard、retry、告警和 audit log 的基礎。

## Thread 在 Prototype 裡剛剛好

專案使用 daemon thread 啟動 watcher 和 worker：

~~~python
def start_scheduler():
    watcher = threading.Thread(target=watcher_loop, daemon=True)
    worker = threading.Thread(target=worker_loop, daemon=True)
    watcher.start()
    worker.start()
~~~

這不是 production 最終解，但對 live session 來說很適合，因為它讓架構概念不被基礎設施淹沒。

當需求變大時，可以逐步替換：

1. `queue.Queue` 換成 SQS。
2. `threading.Thread` 換成獨立 worker process。
3. SQLite 換成 Postgres。
4. 單一 worker 換成水平擴充。

關鍵是 interface 不變：watcher 找任務、queue 傳 ID、worker 執行。

## 取消任務的時間差

排程系統常見問題是 race condition。

例如使用者取消任務時，任務可能剛好已經在 queue 裡。這時候如果 worker 只相信 queue，就會把已取消任務執行掉。

所以 worker 裡這段判斷很重要：

~~~python
if job is None or job.status == "cancelled":
    continue
~~~

這表示 DB 才是最後狀態來源，queue 只是通知 worker 有事情可做。

## 總結

Watcher、Queue、Worker 是很多非同步系統的基本骨架。

這篇從 Task Scheduler prototype 可以學到：

1. Watcher 負責找出 due jobs，不負責真正執行。
2. Queue 裡只放 job ID，讓 worker 重新讀取最新狀態。
3. Worker 必須處理 cancelled、completed、failed 等狀態。
4. Prototype 可以先用 thread 和 in-memory queue，把概念跑通再升級基礎設施。

下一篇會看 `time_bucket`，也就是為什麼排程任務不應該每次都掃整張 jobs table。
