---
layout: post
title: QR Code Lifecycle - 更新 刪除 過期與 Cache Invalidation
image: 'https://images.unsplash.com/photo-1752336459369-d7ff0ab9da43?auto=format&fit=crop&w=1600&q=80'
accent_image: 
  background: url('https://images.unsplash.com/photo-1752336459369-d7ff0ab9da43?auto=format&fit=crop&w=1600&q=80') center/cover
  overlay: false
accent_color: '#0F766E'
theme_color: '#0F766E'
invert_sidebar: false
related_posts:
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 前言

Dynamic QR Code 真正有價值的地方，是 QR Code 發出去之後還能被管理。

在 `build-moat-live-sessions/qr_code_generator` 裡，管理能力主要來自三個功能：

1. 更新目標 URL。
2. Soft delete。
3. Expiration。

這篇整理 QR Code lifecycle 和 cache invalidation 的設計。

## 更新目標 URL

更新 endpoint 使用 `PATCH /api/qr/{token}`：

~~~python
@router.patch("/api/qr/{token}", response_model=QRInfoResponse)
def update_qr(token: str, req: UpdateRequest, db: Session = Depends(get_db)):
    mapping = _get_mapping_or_404(token, db)

    if req.url is not None:
        mapping.original_url = validate_url(req.url)
        redirect_cache.pop(token, None)

    if req.expires_at is not None:
        mapping.expires_at = req.expires_at

    db.commit()
    db.refresh(mapping)
    return mapping
~~~

這裡最重要的是：

~~~python
redirect_cache.pop(token, None)
~~~

因為 redirect endpoint 可能先從 cache 回傳目標 URL，如果更新 DB 但沒有清 cache，使用者掃描時仍然會被導到舊 URL。

## Cache Invalidation 的原則

prototype 使用 in-memory dict 模擬 Redis：

~~~python
redirect_cache: dict[str, str] = {}
~~~

正式環境可能會換成 Redis，但原則不變：

1. 建立 QR Code 時可以 warm cache。
2. 更新 URL 時要刪除 cache。
3. 刪除 QR Code 時要刪除 cache。
4. 過期狀態不能只依賴 cache。

如果 cache 裡只存 original URL，redirect 時命中 cache 就會跳過 DB 檢查。這代表如果有 expiration，正式系統最好把 `expires_at` 和 `is_deleted` 也納入 cache value，或設定短 TTL。

## Soft Delete

刪除 QR Code 時，專案沒有真的 delete row，而是做 soft delete：

~~~python
@router.delete("/api/qr/{token}")
def delete_qr(token: str, db: Session = Depends(get_db)):
    mapping = _get_mapping_or_404(token, db)
    mapping.is_deleted = True
    db.commit()
    redirect_cache.pop(token, None)
    return {"detail": "Deleted"}
~~~

Soft delete 的好處是保留歷史資料：

1. analytics 還可以查。
2. 管理端可以顯示曾經建立過的 QR Code。
3. 誤刪時有機會復原。
4. audit log 比較完整。

刪除後再掃描，系統回傳 410 Gone，比 404 更能表達「這個資源曾經存在，但現在不可用」。

## Expiration

建立 QR Code 時可以帶 `expires_at`：

~~~python
class CreateRequest(BaseModel):
    url: str
    expires_at: datetime | None = None
~~~

redirect 時檢查：

~~~python
if mapping.expires_at and mapping.expires_at < datetime.utcnow():
    raise HTTPException(status_code=410, detail="Gone")
~~~

過期功能很適合活動頁、限時優惠或臨時下載連結。它讓 QR Code 的風險邊界更清楚，不會因為圖片被外流就永久可用。

## URL Validation

更新和建立都應該通過同一個 URL validation：

~~~python
normalized_url = validate_url(req.url)
~~~

這裡的目標不是做複雜安全掃描，而是至少確保：

1. URL 有合法 scheme。
2. 格式可被瀏覽器理解。
3. 儲存前先 normalize，避免同一個 URL 有太多不同寫法。

正式環境還可以加入 denylist、allowlist、malware scanning 或 phishing detection。

## API Response 要一致

查詢和更新都回傳 `QRInfoResponse`：

~~~python
class QRInfoResponse(BaseModel):
    token: str
    original_url: str
    created_at: datetime
    updated_at: datetime
    expires_at: datetime | None
    is_deleted: bool
~~~

一致的 response schema 對前端很重要。管理 UI 可以用同一個 model 更新狀態，不需要為不同 endpoint 寫太多例外處理。

## Lifecycle 狀態整理

可以把 QR Code lifecycle 想成：

~~~text
created -> active
active  -> updated
active  -> expired
active  -> deleted
~~~

掃描結果：

1. Active：302 redirect。
2. Not found：404。
3. Deleted：410。
4. Expired：410。

這樣使用者端、前端管理端和 API client 都能理解資源目前狀態。

## 總結

Dynamic QR Code 不只是產圖工具，它其實是一個短網址管理系統。

這篇的重點：

1. 更新 URL 時一定要處理 cache invalidation。
2. Soft delete 比 hard delete 更適合保留 analytics。
3. Expiration 可以讓 QR Code 有明確生命週期。
4. 404 和 410 要分清楚，回應語意會影響除錯和產品體驗。

接下來的系列會切到 Knowledge Base QA Bot，看如何用 Markdown、BM25、embedding 和 FAISS 做可追溯的問答系統。
