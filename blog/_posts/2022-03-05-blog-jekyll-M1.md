---
layout: post
title:  jekyll – 如何在 Apple M1 Macbook 上安裝 jekyll
image: https://alexmanrique.com/blog/images/jekyll_m1.jpg
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

## 因為剛好最近買了一台 M1 imac，剛好需要重新安裝 Jekyll 寫BLOG，因為無法順利安裝的坑，真心覺得 M1 架構上處理環境真的不是那麼簡單。如果你也剛好遇到，此篇可以讓你解決M1 Mac 安裝 Jekyll的問題...

### 目前 iMac 的作業環境為 Monterey ios 12.4：

#### 首先需要先安裝套件管理工具：
我使用 Homebrew 在我的 Mac 上安裝第三方軟件包。可以通過簡單的方式安裝：

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### 再來安裝 Rbenv
我們需要安裝一個兼容 ARM 處理器的 Ruby 版本。
```
brew install rbenv ruby-build
```

#### 安裝基於 ARM 的 Ruby 3.0.0

```
rbenv install 3.0.0
rbenv global 3.0.0
ruby -v
rbenv rehash
```

#### 將 ruby​​ 和 gems 路徑添加到您的 shell 配置中
現在，將 rbenv 添加到 bash 以便每次打開終端時加載它

如果您使用的是 zsh
```
echo 'eval "$(rbenv init - zsh)"' >> ~/.zshrc
```

如果您使用的是 bash
```
echo 'eval "$(rbenv init - bash)"' >> ~/.bash_profile
```

如果您不確定您使用的是哪個 shell，您可以使用以下命令進行檢查：

```
echo $SHELL
```


## 安裝 Jekyll
最後，我們可以繼續安裝 Jekyll 和 Bundler。我們將進行本地安裝（不需要sudo特權）。

```
gem install --user-install bundler jekyll
```

#### 如果您使用的是 zsh
替換3.0.0為您的 ruby​​ 版本。您可以通過 . 檢查您的 ruby​​ 版本ruby -v。如果您的 ruby​​ 版本是 2.7，則使用2.7.0.

```
echo 'export PATH="/usr/local/opt/ruby/bin:/usr/local/lib/ruby/gems/3.0.0/bin:$PATH"' >> ~/.zshrc
```

#### 如果您使用的是 bash
替換3.0.0為您的 ruby​​ 版本。您可以通過 . 檢查您的 ruby​​ 版本ruby -v。如果您的 ruby​​ 版本是 2.7，則使用2.7.0.

```
echo 'export PATH="/usr/local/opt/ruby/bin:/usr/local/lib/ruby/gems/3.0.0/bin:$PATH"' >> ~/.bash_profile
```

#### 更新套件(非必要)
對於 M1 Mac，我們可能需要做一些額外的步驟——更新 bundler、添加 webrick 並重建所有內容。

```
bundle update --bundler
bundle add webrick
bundle install --redownload
```

#### 檢查安裝
現在，我們可以運行我們的示例blog。導航到您的blog，然後運行以下命令：

#### 如果您還沒有進行本地安裝
```
gem install bundler jekyll 
```

#### 如果您沒有blog，請使用
```
jekyll new my-awesome-site
cd my-awesome-site
```

## 終於可以運行啦！！
接下來應該就大功告成了，開始寫blog吧～

```
bundle exec jekyll serve
```


## Reference
https://talk.jekyllrb.com/t/jekyll-crashing-on-m1-mac/6367/16