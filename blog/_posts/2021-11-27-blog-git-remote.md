---
layout: post
title:  Git – git push 同時推送到多個遠端儲存庫
image: https://2.bp.blogspot.com/-EYBE6CHprjg/W4O3ZR-1uhI/AAAAAAAAz6E/iQ-1-NXU_hIbECnTSEnIXT19AX9HHocbACLcBGAs/s1600/Multiple%2BRepo.PNG
accent_image: 
  background: url('https://www.pbs.org/wgbh/nova/media/images/bfcppfu.width-800.png') center/cover
  overlay: false
accent_color: '#ccc'
theme_color: '#ccc'
invert_sidebar: false
related_posts:
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 同一個Repositories如何設定兩組不同的遠端儲存庫
### 如果你在工作目錄想同時保有兩個不同的遠端儲存庫，而且兩個遠端儲存庫你都有權限，都有可能會做 Push 或 Pull/Fetch 動作。

那麼我們本地開發目錄的設定步驟如下：

取得原始碼 (取出 develop 分支)

```
git clone https://github.com/XXX.git -b develop
cd demo1
```
設定第二組遠端儲存庫位址

```
git remote add demo2 https://github.com/XXX-2.git
```
如果第二組遠端儲存庫位址是 origin 的 上游專案(upstream)，我們有時候會直接設定 Remote 名稱為 upstream，如此一來語意就會非常清楚。

檢查 Git Remote 位址

此時你應該會看見兩組 Remote 位址，個別都有 fetch 與 push 位址：

```
# git remote -v
origin  https://github.com/XXX.git (fetch)
origin  https://github.com/XXX.git (push)
demo2  https://github.com/XXX-2.git (fetch)
demo2  https://github.com/XXX-2.git (push)
```

當你想取回 origin 的新版本時，使用 git pull 或 git fetch 就可以取得。

當你想取回 demo2 的新版本時，使用 git pull demo2 或 git fetch demo2 就可以取回新版。

當你想取回所有 Remote 的新版本時，就不能用 git pull 命令了，因為 Git 不會知道你想合併哪一條遠端分支。你只能用 git fetch --all 取回所有新的版本(Commits)，但你就要自行決定如何合併遠端的版本變更了！


## Reference
https://blog.miniasp.com/