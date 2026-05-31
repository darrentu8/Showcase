---
layout: post
title: QR Token Redirect Analytics - 短網址服務的三個核心細節
image: 'https://images.unsplash.com/photo-1551288049-bebda4e38f71?auto=format&fit=crop&w=1600&q=80'
accent_image: 
  background: url('https://images.unsplash.com/photo-1551288049-bebda4e38f71?auto=format&fit=crop&w=1600&q=80') center/cover
  overlay: false
accent_color: '#DC2626'
theme_color: '#DC2626'
invert_sidebar: false
related_posts:
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 前言

上一篇整理了 Dynamic QR Code 的基本架構。這篇繼續看 `build-moat-live-sessions/qr_code_generator` 裡三個比較重要的細節：

1. Token generation。
2. 302 redirect。
3. Scan analytics。

這三個設計決定了短網址服務是否能穩定運作。

## Token 產生策略

專案使用 SHA-256、nonce 和 Base62 產生 7 字元 token：

~~~python
BASE62_CHARS = string.ascii_letters + string.digits
TOKEN_LENGTH = 7
MAX_RETRIES = 10
~~~

Base62 的字元包含：

~~~text
a-z A-Z 0-9
~~~

如果 token 長度是 7，可能組合數大約是：

~~~text
62 ^ 7 = 3,521,614,606,208
~~~

對 prototype 來說非常夠用。

## 為什麼要加 Nonce

如果只對 URL 做 hash，同一個 URL 永遠會產生同一個 token。這在某些場景是好事，但在 QR Code 管理系統裡不一定合理。

例如同一個活動頁可能會被不同海報、不同通路或不同客戶使用，我們會希望每個 QR Code 都有獨立 token 和 analytics。

專案用 timestamp 加 attempt 當 nonce：

~~~python
nonce = f"{int(time.time())}_{attempt}"
hash_input = url + nonce
raw_hash = hashlib.sha256(hash_input.encode()).digest()
token = base62_encode(raw_hash)[:TOKEN_LENGTH]
~~~

正式系統可以再加上 `user_id`、`campaign_id` 或 random bytes，讓 token 更可控。

## Collision Retry

短 token 一定存在碰撞機率，所以產生 token 後要查 DB：

~~~python
def token_exists_in_db(db: Session, token: str) -> bool:
    return db.query(UrlMapping).filter(UrlMapping.token == token).first() is not None
~~~

如果已存在，就重新產生：

~~~python
for attempt in range(MAX_RETRIES):
    token = ...
    if not token_exists_in_db(db, token):
        return token

raise RuntimeError("Failed to generate unique token")
~~~

這個設計比單純相信 hash 前 7 碼安全。資料庫上也有 unique constraint，讓應用層和 DB 層都有保護。

## 為什麼用 302 Redirect

掃描 QR Code 時，server 回傳：

~~~python
return RedirectResponse(url=mapping.original_url, status_code=302)
~~~

302 表示 temporary redirect。這對 Dynamic QR Code 很重要，因為我們希望瀏覽器或中間層不要把 redirect 結果永久快取。

如果使用 301 permanent redirect，某些 client 可能記住舊目標，之後即使後台更新 URL，掃描結果也不一定會立即反映。

Dynamic QR Code 的精神是每次掃描都回到 server 查最新狀態，所以 302 比較適合。

## Redirect Flow

redirect endpoint 有一個清楚的 fallback flow：

~~~python
@router.get("/r/{token}")
def redirect(token: str, request: Request, db: Session = Depends(get_db)):
    if token in redirect_cache:
        _record_scan(token, request, db)
        return RedirectResponse(url=redirect_cache[token], status_code=302)

    mapping = db.query(UrlMapping).filter(UrlMapping.token == token).first()

    if mapping is None:
        raise HTTPException(status_code=404, detail="Not Found")

    if mapping.is_deleted:
        raise HTTPException(status_code=410, detail="Gone")

    if mapping.expires_at and mapping.expires_at < datetime.utcnow():
        raise HTTPException(status_code=410, detail="Gone")

    redirect_cache[token] = mapping.original_url
    _record_scan(token, request, db)
    return RedirectResponse(url=mapping.original_url, status_code=302)
~~~

流程可以寫成：

~~~text
Cache hit -> record scan -> 302
Cache miss -> DB lookup
Not found -> 404
Deleted or expired -> 410
Valid -> warm cache -> record scan -> 302
~~~

## Scan Analytics

每次 redirect 前，系統會記錄掃描事件：

~~~python
event = ScanEvent(
    token=token,
    user_agent=request.headers.get("user-agent"),
    ip_address=request.client.host if request.client else None,
)
db.add(event)
db.commit()
~~~

資料模型保存：

1. `token`
2. `scanned_at`
3. `user_agent`
4. `ip_address`

analytics endpoint 再用 SQL 聚合：

~~~python
daily = (
    db.query(
        func.date(ScanEvent.scanned_at).label("date"),
        func.count(ScanEvent.id).label("count"),
    )
    .filter(ScanEvent.token == token)
    .group_by(func.date(ScanEvent.scanned_at))
    .all()
)
~~~

這樣就可以回傳總掃描數和每日掃描數。

## 隱私與正式環境注意事項

prototype 直接保存 IP 和 user agent，方便理解 analytics。

正式環境要多想幾件事：

1. 是否需要匿名化 IP。
2. 是否需要資料保存期限。
3. 是否要排除 bot 或 preview crawler。
4. 是否要加 rate limit 防止 token 被刷。
5. 是否要提供 opt-out 或隱私聲明。

行銷分析很有用，但資料蒐集要有邊界。

## 總結

短網址服務看起來簡單，但細節不少。

這篇整理三個核心：

1. Token 要短、可讀、可碰撞重試。
2. Dynamic QR Code 應使用 302，避免永久快取。
3. 每次 redirect 都是 analytics 的資料來源。

下一篇會看更新、刪除、過期和 cache invalidation，這些功能決定 QR Code 是否真的能被管理。
