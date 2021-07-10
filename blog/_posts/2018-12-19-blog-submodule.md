---
layout: post
title: Git - 如何操作父repository中的子repository(模塊) submodule 使用教學 - GitHub
image: https://miro.medium.com/max/2000/1*TnGnN4mSbfcvVczYYh2m7g.png
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

> 會使用到 Git Submodule 是因為部落格是使用 GitHub Pages，是GitHub提供的一個網頁代管服務，其中 Jekyll 軟體可以用於將文檔轉換成靜態網頁，該軟體提供了將網頁上傳到GitHub Pages的功能可說是超級方便！

## 稍微來介紹一下 Jekyll
Jekyll是基於Ruby Gem的解析引擎，能夠將樣板、liquid 語言、markdown轉換為 靜態網頁 的產生器。
### 使用的 Jekyll 特色呢?
- No more databases 不需要資料庫
- Liquid Template 動態模板
- Free hosting with GitHub Pages 只要學會git push 就可以丟到github page上
- Markdown 好編寫

## 何謂 Git Submodule
間單的來講，就是主專案中又有一個子專案需要拆開來使用git，另一方面也是一種 Library 套件管理的概念，這時候 Git Submodule 就是一個不錯的解決方案，但首先 Git Submodule 不太好管理就是了~ ~

而 Git Submodule 衍生出巢狀的 Git 結構，意思是你有一個 Repository，裡面還有一個子 Repository。而如果用一般的 Git Push 的話，你會發現子 Repository 的資料並不會同步到 remote repository，因此這邊就要加入 Git Submodule 的指令操作。


## 從子模塊開始
我們將逐步開發一個簡單的項目，該項目已拆分為一個主項目和幾個子項目。

讓我們首先添加一個現有的 Git 存儲庫作為我們正在處理的存儲庫的子模塊。要添加新的子模塊，請使用 git submodule add帶有要開始跟踪的項目的絕對或相對 URL的命令。在本例中，我們將添加一個名為“subCase”的庫。

```
$ git submodule add https://github.com/test/subCase
Cloning into 'subCase'...
remote: Counting objects: 11, done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 11 (delta 0), reused 11 (delta 0)
Unpacking objects: 100% (11/11), done.
Checking connectivity... done.
```

默認情況下，子模塊會將子項目添加到與存儲庫同名的目錄中，在本例中為“subCase”。如果你希望它轉到其他地方，你可以在命令的末尾添加不同的路徑。

如果你此時運行git status，你會注意到一些事情。

```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   .gitmodules
	new file:   subCase
```

首先，你應該注意到新.gitmodules文件。這是一個配置文件，用於存儲項目 URL 和你將其拉入的本地子目錄之間的映射：

```
[submodule "subCase"]
	path = subCase
	url = https://github.com/test/subCase
```

如果你有多個子模塊，則此文件中將有多個條目。請務必注意，此文件與你的其他文件（例如你的.gitignore文件）一起受版本控制。它與項目的其餘部分一起推動和拉動。這就是其他克隆這個項目的人知道從哪裡獲取子模塊項目的方式。

## Reference
https://git-scm.com/docs/git-submodule