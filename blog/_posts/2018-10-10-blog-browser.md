---
layout: post
title: 現代瀏覽器架構與渲染機制
image: https://img.technews.tw/wp-content/uploads/2017/12/29164436/browsers-1265309_1280.png
accent_image: 
  background: url('https://www.pbs.org/wgbh/nova/media/images/bfcppfu.width-800.png') center/cover
  overlay: false
accent_color: '#ccc'
theme_color: '#ccc'
invert_sidebar: false
sitemap: false
---

0. 這篇文章將介紹現代瀏覽器的架構與渲染機制
{:toc}

> 「瀏覽器的進步使得我們能在網頁上實現越來越多的功能。」

作為 Web 工程師，尤其是前端開發者，了解瀏覽器的架構和渲染機制是非常重要的。這篇文章將帶你從瀏覽器的單一程序架構到多程序架構，再到現代的多層次架構，深入了解瀏覽器的演進過程。

## 單一程序瀏覽器時期
早期在2007年之前瀏覽器基本上都是單一程序架構。所有功能模組都運行在同一個程序中，這導致了不穩定性、不流暢性和安全性問題。

![img by Crank Lee](https://miro.medium.com/v2/resize:fit:1400/0*TSW5YPSGBsBiBaRw)

### 問題
- **不穩定性**：一個模組出問題會導致整個瀏覽器崩潰。
- **不流暢性**：單一執行緒導致資源被獨佔，影響整體性能。
- **安全性問題**：缺乏沙盒機制，容易被惡意程式攻擊。

## 進化：多程序瀏覽器時期
2008 年，Chrome 推出了多程序架構，將渲染程序和插件程序獨立出來，通過 IPC 進行溝通。

![](https://developer.chrome.com/static/blog/inside-browser-part1/image/browser-architecture-9d143004c2a63.png)

### 改進
- **穩定性**：各個程序互相隔離，單一模組崩潰不會影響整體。
- **流暢性**：每個 Tab 獨立運行在不同的渲染程序中，提升了性能。
- **安全性**：引入沙盒機制，提升了安全性。

## 現代瀏覽器架構
現代瀏覽器進一步將網路資源操作和 GPU 操作獨立出來，形成更加豐富的多程序架構。

![](https://webperf.tips/static/0085f9f18930b6e5b069b04bfbf33c07/906b5/ProcessModel02.png)

### 優勢
- **性能提升**：多程序並行運行，提升了整體性能。
- **更高的穩定性和安全性**：進一步隔離不同功能模組，提升了穩定性和安全性。

## SOA 瀏覽器架構
Chrome 團隊提出了以 SOA 為基礎的新架構，將各個功能模組作為服務運行，實現高內聚、低耦合、易擴展與易維護的特性。

![](https://miro.medium.com/max/700/1*r05OaXt93q9ZcWo2mV1dmw.png)

### 彈性架構
- **高性能設備**：拆分成多個程序運行，提升穩定性和效能。
- **低性能設備**：合併成單一程序運行，節省記憶體。

## 瀏覽器渲染機制
現代瀏覽器的渲染機制包括 DOM Tree、CSSOM Tree、Render Tree、Layout Tree 和 Paint 階段。

![](https://developer.chrome.com/static/docs/chromium/layoutng/image/thumbnail.jpg)

### 渲染流程
1. 生成 DOM Tree
2. 生成 CSSOM Tree
3. 生成 Render Tree
4. 生成 Layout Tree
5. Paint 畫面

### Layer 分層與 Compositing
瀏覽器會根據 Layout Tree 產生 Layer Tree，並在 Compositor Thread 進行合成，提升渲染效能。

![](https://miro.medium.com/max/700/1*cwAlMho6lZ6Mycj-iFap8A.png)

## 結語
瀏覽器的進步使得我們能在網頁上實現越來越多的功能。作為 Web 開發者，了解瀏覽器的架構和渲染機制，能夠幫助我們更好地開發高效能的網頁應用。

### References
- Inside look at modern web browser (Part 1, 2, 3, 4)
- 浏览器工作原理与实践
- Chrome Site Isolation
- Rendering Performance
- Reflow 和 Repaint 引發的性能問題
- DOM Performance