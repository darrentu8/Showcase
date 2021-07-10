---
layout: post
title: Git - 如何在現有的專案中建立子repository模塊， submodule 使用教學 - GitHub
image: https://frank198978104.github.io/images/2017-07-28-welcome-to-jekyll/2017-07-28-welcome-to-jekyll-image0.jpg
accent_image: 
  background: url('https://www.pbs.org/wgbh/nova/media/images/bfcppfu.width-800.png') center/cover
  overlay: false
accent_color: '#ccc'
theme_color: '#ccc'
invert_sidebar: false
# related_posts:
#   - example/_posts/2017-11-23-example-content-ii.md
#   - /example/2012-02-07-example-content/
sitemap: false
---

## 稍微來介紹一下 Jekyll
Jekyll是基於Ruby Gem的解析引擎，能夠將樣板、liquid 語言、markdown轉換為 靜態網頁 的產生器。
#### 打算使用 Jekyll 的朋友們，會建議熟悉 Git & Html & Markdown

## 使用的 Jekyll 特色呢?
- No more databases 不需要資料庫
- Liquid Template 動態模板
- Free hosting with GitHub Pages 只要學會git push 就可以丟到github page上
- Markdown 好編寫

## Github Pages & ekyll 技術介紹
Git, Github, Github Pages
Git版本控制工具，Github是使用git版本控制項目的的虛擬主機服務
![](https://photo.minwt.com/img/Content/server/github-page/github-page_00.jpg) 
<br>
連結在此：http://pages.github.com/
Github Pages 是 Github 提供的服務，可以利用 Repository 建立自己的靜態頁面，其中不需要任何 databases 的 Jekyll 強大功能，在每次的 git push 後會將 markdown 轉換為頁面，所以 Jekyll 是很適合用來建立部落格等內容！

## 強大特色

- 快速 html靜態頁就是快速。
- 穩定 ”github” 這個品牌能不信嗎?
- 完全免費，可惜大部分的模板不怎麼美就是了...
- 容量無限 只要會git commit、git push。
- Markdown是一種輕量級標記式語言，他在編寫完後就像是一般的文字檔，也可轉換成有效的XHTML（或者HTML）文件。
- 其易懂易讀的語法都是用標點符號組成，方便作為網站blog的寫作語言。附上相關範例 http://markdown.tw/

## Jekyll 官方介紹
可於官網下來或在 git hub 搜尋相關主題下載下來使用，其資料夾編寫後會產生(如：_site等資料夾)，再將此資料夾的靜態網頁上傳於 Repository 即可。
https://jekyllrb.com/

## jekyll-theme
Jekyll所使用的樣板語言是使用 Ruby 編寫，透過渲染器運行生成為靜態網站。本地端需要下載 ruby 來運行。以下官方提供模板範例可下載使用。
https://www.ruby-lang.org/zh_tw/
https://github.com/topics/jekyll-theme

## Github Page
在發布Repository後，之後每次都只需要在本地端push文章就可以了！而每次push到github上的文章及Template，都會透過Jekyll server轉換為靜態的網站。
![](/assets/img/blog/Jekyll.jpg)