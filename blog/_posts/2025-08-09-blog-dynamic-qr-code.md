---
layout: post
title: Dynamic QR Code - 用短網址讓 QR Code 可以更新
image: 'https://images.unsplash.com/photo-1661169398346-aecdc4f5068b?auto=format&fit=crop&w=1600&q=80'
accent_image: 
  background: url('https://images.unsplash.com/photo-1661169398346-aecdc4f5068b?auto=format&fit=crop&w=1600&q=80') center/cover
  overlay: false
accent_color: '#F97316'
theme_color: '#F97316'
invert_sidebar: false
related_posts:
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 前言

`build-moat-live-sessions/qr_code_generator` 是一個 Dynamic QR Code prototype。

它和一般 QR Code 最大的不同是：QR Code 圖片裡不直接放原始 URL，而是放一個短網址。

也就是說，使用者掃到的是：

~~~text
http://localhost:8000/r/Os8TXDb
~~~

伺服器再把這個短網址 redirect 到真正目標：

~~~text
https://example.com
~~~

這樣設計後，QR Code 圖片不用重新產生，後台就可以修改目標網址，也能記錄掃描次數。

## Static QR Code 的限制

傳統 static QR Code 是把資料直接寫死在圖片裡。

例如把 `https://example.com/product-a` 編碼進 QR Code 後，使用者掃描就直接得到這個網址。

問題是：

1. 目標網址改了，QR Code 圖片就失效。
2. 印刷品已經發出去，無法回收。
3. 無法知道有多少人掃描。
4. 無法做過期、停用或 A/B test。

Dynamic QR Code 的核心就是多加一層 server redirect。

## API 設計

專案提供的主要 endpoint：

~~~text
POST   /api/qr/create
GET    /r/{token}
GET    /api/qr/{token}
PATCH  /api/qr/{token}
DELETE /api/qr/{token}
GET    /api/qr/{token}/image
GET    /api/qr/{token}/analytics
~~~

這組 API 把「使用者掃描」和「管理 QR Code」分開：

1. `/r/{token}` 給掃描者使用，只負責 redirect。
2. `/api/qr/*` 給管理端使用，負責建立、查詢、更新、刪除和分析。

這樣路由語意很清楚，後續要加權限也比較容易。

## 建立 QR Code

建立時會先驗證 URL，再產生 token：

~~~python
@router.post("/api/qr/create", response_model=CreateResponse)
def create_qr(req: CreateRequest, db: Session = Depends(get_db)):
    normalized_url = validate_url(req.url)
    token = generate_token(normalized_url, db)

    mapping = UrlMapping(
        token=token,
        original_url=normalized_url,
        expires_at=req.expires_at,
    )
    db.add(mapping)
    db.commit()

    short_url = f"{BASE_URL}/r/{token}"
    redirect_cache[token] = normalized_url

    return CreateResponse(
        token=token,
        short_url=short_url,
        qr_code_url=f"{BASE_URL}/api/qr/{token}/image",
        original_url=normalized_url,
    )
~~~

這裡回傳三個很實用的欄位：

1. `token`：資料庫和管理 API 使用。
2. `short_url`：實際被 QR Code 編碼的網址。
3. `qr_code_url`：取得 PNG 圖片的 endpoint。

## QR 圖片只放短網址

產生圖片的 endpoint 很單純：

~~~python
@router.get("/api/qr/{token}/image")
def get_qr_image(token: str, db: Session = Depends(get_db)):
    _get_mapping_or_404(token, db)
    short_url = f"{BASE_URL}/r/{token}"

    img = qrcode.make(short_url)
    buf = io.BytesIO()
    img.save(buf, format="PNG")
    buf.seek(0)
    return StreamingResponse(buf, media_type="image/png")
~~~

重點是圖片裡永遠只放 short URL。

當管理者之後更新 `original_url` 時，QR 圖片不用變，只要 redirect target 變就好。

## Database Model

URL mapping 保存 token 和目標網址：

~~~python
class UrlMapping(Base):
    __tablename__ = "url_mappings"

    id = mapped_column(Integer, primary_key=True, autoincrement=True)
    token = mapped_column(String(8), unique=True, nullable=False, index=True)
    original_url = mapped_column(Text, nullable=False)
    expires_at = mapped_column(DateTime, nullable=True)
    is_deleted = mapped_column(Boolean, default=False)
~~~

這裡的 `expires_at` 和 `is_deleted` 讓 QR Code 有生命週期管理，而不只是永久 redirect。

## 使用情境

Dynamic QR Code 很適合這些場景：

1. 活動海報：活動頁網址可能調整。
2. 商品包裝：產品頁或促銷頁可能換檔期。
3. 餐廳菜單：菜單內容常更新。
4. 內部文件：掃描後連到最新版本文件。
5. 行銷追蹤：需要知道掃描量和日期分布。

真正的價值不是 QR Code 本身，而是 QR Code 背後多了一個可管理的 redirect layer。

## 總結

Dynamic QR Code 的架構其實很直覺：

~~~text
QR Code -> short URL -> server -> original URL
~~~

但多了這一層，就得到三個重要能力：

1. 目標網址可更新。
2. 掃描行為可追蹤。
3. QR Code 可過期或停用。

下一篇會繼續看 token 怎麼產生，以及為什麼 prototype 使用 SHA-256、nonce 和 Base62。
