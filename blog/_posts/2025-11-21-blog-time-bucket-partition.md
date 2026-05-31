---
layout: post
title: Time Bucket Partitioning - 排程任務不要每次掃全表
image: 'https://images.unsplash.com/photo-1506784242126-2a0b0b89c56a?auto=format&fit=crop&w=1600&q=80'
accent_image: 
  background: url('https://images.unsplash.com/photo-1506784242126-2a0b0b89c56a?auto=format&fit=crop&w=1600&q=80') center/cover
  overlay: false
accent_color: '#16A34A'
theme_color: '#16A34A'
invert_sidebar: false
related_posts:
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 前言

排程系統最容易被忽略的問題是查詢成本。

如果資料庫只有幾十筆任務，直接查 `scheduled_at <= now` 沒什麼問題。但任務數量變大後，每隔幾秒掃一次全表，就會變成資料庫壓力來源。

`build-moat-live-sessions/chatgpt_task` 在 prototype 裡加了一個簡單但有效的欄位：`time_bucket`。

## Time Bucket 是什麼

`time_bucket` 是把任務時間轉成固定粒度的字串。

專案裡使用小時作為 bucket：

~~~python
def get_time_bucket(scheduled_at: datetime) -> str:
    return scheduled_at.strftime("%Y%m%d%H")
~~~

例如：

~~~text
2026-05-03 09:15:00 -> 2026050309
2026-05-03 09:59:59 -> 2026050309
2026-05-03 10:00:00 -> 2026050310
~~~

Watcher 查詢時先鎖定當前小時：

~~~python
bucket = get_time_bucket(current_time)
~~~

然後再查：

~~~python
db.query(Job).filter(
    Job.time_bucket == bucket,
    Job.scheduled_at <= current_time,
    Job.status == "pending",
)
~~~

這樣 watcher 不需要在每次 tick 都看完整張 jobs table。

## Model 層設計

`Job` model 裡直接保存 `time_bucket`：

~~~python
class Job(Base):
    __tablename__ = "jobs"

    id = mapped_column(Integer, primary_key=True, autoincrement=True)
    time_bucket = mapped_column(String(10), nullable=False, index=True)
    description = mapped_column(Text, nullable=False)
    scheduled_at = mapped_column(DateTime, nullable=False)
    status = mapped_column(String(20), default="pending")
    result = mapped_column(Text, nullable=True)
~~~

另外加上複合 index：

~~~python
__table_args__ = (
    Index("idx_bucket_status", "time_bucket", "status"),
)
~~~

這個 index 對 watcher 很有幫助，因為查詢條件裡最常出現的就是 `time_bucket` 和 `status`。

## 為什麼不只靠 scheduled_at index

只對 `scheduled_at` 建 index 也可以，但它會查出所有過去到現在的 pending 任務。

如果系統裡有很多歷史資料、失敗任務或長期 pending 任務，查詢範圍會越來越大。

Time bucket 的思路是先切時間範圍，再做精準條件：

~~~text
先找目前小時的任務
再找 scheduled_at 已經到期
最後只取 pending
~~~

對大多數「未來任務」系統來說，watcher 最關心的是現在附近的任務，而不是整個歷史。

## Bucket 粒度怎麼選

專案用小時當 bucket，這是很合理的初始設定。

不同情境可以選不同粒度：

1. 秒級排程：bucket 可以用分鐘。
2. 一般提醒：bucket 用小時就夠。
3. 大型批次任務：bucket 可以用天。

粒度越細，單一 bucket 裡的資料越少，但 watcher 要處理跨 bucket 的邏輯可能更多。粒度越粗，實作簡單，但單次查詢範圍較大。

prototype 選小時，是在簡單和實用之間很好的折衷。

## 注意跨 Bucket 的邊界

Time bucket 有一個要注意的情境：如果 watcher 停機很久，重新啟動時只掃當前 bucket，可能會漏掉上一個 bucket 裡還沒執行的 pending 任務。

正式系統可以用幾種方式處理：

1. 啟動時補掃最近幾個 bucket。
2. 額外查詢 `scheduled_at <= now` 的 pending 任務，但限制時間範圍。
3. 對 missed jobs 做獨立補償流程。

prototype 先聚焦在概念，所以查當前 bucket 就好。進 production 前要補上 missed job recovery。

## 建立任務時就算好 Bucket

`task.create` 建立 job 時會直接寫入 bucket：

~~~python
dt = datetime.fromisoformat(scheduled_at)
job = Job(
    description=description,
    scheduled_at=dt,
    time_bucket=get_time_bucket(dt),
)
~~~

這樣 watcher 不用每次查詢時對每筆資料做時間轉換，資料庫也可以用 index 加速。

這是一個常見 tradeoff：把一點點計算提前到寫入階段，換取讀取階段更穩定。

## 總結

Time bucket partitioning 是排程系統裡很實用的技巧。

這篇可以記住幾個重點：

1. Watcher 不應該每次掃全表。
2. `time_bucket` 可以把查詢範圍限制在當前時間區間。
3. `time_bucket + status` 的 index 能讓 pending job 查詢更穩定。
4. 正式系統要補 missed job recovery，避免停機後漏執行。

這個設計雖然簡單，但已經把 prototype 往 production thinking 推進一步。
