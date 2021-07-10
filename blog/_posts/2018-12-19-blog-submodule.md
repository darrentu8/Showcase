---
layout: post
title: Git - 如何在現有的專案中建立子repository模塊， submodule 使用教學 - GitHub
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

> 主要會使用到 Git Submodule 是因為部落格是使用 GitHub Pages，是GitHub提供的一個網頁代管服務，其中 Jekyll 軟體可以用於將文檔轉換成靜態網頁，該軟體提供了將網頁上傳到GitHub Pages的功能可說是超級方便！

## 何謂 Git Submodule
簡單的來講，就是巢狀的 Git 結構，意思是主專案中又有一個子專案需要拆開來使用git，類似一種 Library 套件管理的概念，像是大家在開發新專案的時候，如果想要用到其他專案的程式碼時，像是主流的 CSS Framwork 如：Vuetify、Bootstrap、Element UI，這些大型套件都放在 Github 方便進行維護更新，從前的作法就是先 git clone 下來，把要的檔案分別複製到自己專案，但是這樣麻煩的問題就來了！<br>
若不是用 npm 或 webpack 這種套件管理，假如官方有釋出新版本，那要如何更新呢？這時可別再下載zip手動更新了，這時候 Git Submodule 就是一個不錯的解決方案，幫助大家方便管理程式碼！


## 先在 GitHub 建立好 子Repository，然後在主專案輸入指令

正常來說當你有一個 Repository，裡面如果包有一個子 Repository的話。是不能用 Git Push 推送到 remote repository裡的，這可能會發生問題，因此這邊就要加入 Git Submodule 的指令操作。

讓我們首先在遠端建立好欲添加的子Repository。並複製http或SSH連結，在主專案裡輸入git submodule add來開起追蹤命令，在此示範建立子專案"subCase"：

### 建議不先預留資料夾否則會無法建立

```
$ git submodule add {子專案的repo.git} {選填：主專案中欲儲存的子資料夾之資料夾名稱}

Cloning into 'subCase'...
remote: Counting objects: 11, done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 11 (delta 0), reused 11 (delta 0)
Unpacking objects: 100% (11/11), done.
Checking connectivity... done.
```


在默認地情況下，子專案會將子項目添加到與Repository同名的目錄中，在本例中為“subCase”。所以如果你希望它建立在其他資料夾，你可以在命令的末尾添加想要的資料夾名稱。

## 子Repository Clone 完成
```
Reactivating local git directory for submodule 'subCase'.
warning: LF will be replaced by CRLF in .gitmodules.
The file will have its original line endings in your working directory
```

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

然後在主專案的.git資料夾裡，可以注意到新.gitmodules文件的配置文件，為用於存儲項目 URL 和你將其拉入的本地子目錄之間的映射：

```
[submodule "subCase"]
	path = subCase
	url = https://github.com/test/subCase
```

#### 如果你有多個子專案，請務必注意，此文件與你的其他文件（例如你的.gitignore文件）一起受版本控制。它與項目的其餘部分一起更新或取回。


## Reference
https://git-scm.com/docs/git-submodule