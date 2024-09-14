---
layout: post
title: 善用Google Sheet作為你的簡易表單資料庫！
image: https://lh3.googleusercontent.com/fAQAzXOc_vdu9y9UjXqvGqWbaL0bdJq1CCxljUModhJUA0trlXlHs8gMiOZwcDM_g94h1w9rjIz6YKJqTqpQKmWydv5nNohvkd-x_EkkPbJCzbgtDSQ=s0
accent_color: '#0F9D58'
theme_color: '#0F9D58'
invert_sidebar: false
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

今天我想分享一個既有趣又實用的應用，就是利用Google Sheet來充當表單內容的資料庫。我想許多讀者會猜測這將會是一個麻煩的過程，可能要經過許多繁雜的設置，不過事實是…
其實非常簡單又方便
如果想一探究竟的讀者就繼續往下看吧！

## 客製化表單上傳至Google Sheet
為什麼要這麼做？<br>
基本上大部分的事情都是先有需求，再去因應需求找出對應的解法。因此今天在說明該怎麼做到把Google Sheet當作客製化表單資料庫以前，我們得先談談這樣的方法適合應用在哪些使用情境上。<br>
大部分企業幾乎都會有自己的形象官網，而通常官網也會與企業提供的服務分開，為單純以展示資訊為主要目的的靜態形象頁面。同樣性質的網頁還有特定活動的宣傳網站，也是以展示活動資訊為主要目的，與網頁應用程式（web app）有很大的區別。不過雖然是以呈現資訊為主，這類型的網站通常會有一個例如「聯絡我們」的區塊，讓使用者填寫自己的聯絡資訊，搜集顧客資訊以挖掘更多潛在的商機，例如知名電動車品牌Tesla的官網：<br>

我們都知道通常表單的運作是在提交時會發一個POST請求到後端API，再將資料儲存到後端資料庫裡。但是如果因為團隊人力配置因素或是開發者本身技術限制，沒辦法生出這樣一個API的話該怎麼辦呢？又或者即使有這樣的API，但要運用這些資料的人反而是非工程團隊的夥伴時，請他們下SQL查詢去撈資料好像又不是那麼實際或方便（當然還是有很多能協助達成這件事的工具啦），這時候如果採用文章標題的方法似乎蠻可行的：將客製化的表單內容上傳至Google Sheet中。像這樣資料提交數量不會很大的情況，就很適合這樣的做法，Google Sheet的操作介面也十分平易近人，適合所有人來使用。<br>
了解這個方法適用的情況後，讓我們開始動手做吧！<br>


![](https://formfacade.com/itemembed/1FAIpQLSdBK_VbRtL2tnjtOSL1FhLg68XNlOfPbCLa590ZKPXGcMGvXA/item/1608895903/image/1XDRT684McCFkAb4NXCoblkpmI9yLwrgePBPuXTbI1mTKBw)
## 步驟1. 建立一個Google表單
是的，你沒有看錯，不是Google試算表，是Google表單(Google Form)。在我們從客製化表單點擊提交後，我們會對一個URL發一個POST請求，這個URL就是Google表單的連結，URL中我們會帶查詢字串，我們都知道查詢字串會是鍵值對的形式，這邊的鍵會是Google表單各欄位的ID，值則是我們從客製化表單收集到的資訊，當我們完成這個請求後，就會自動將資料回覆到Google表單中，接著就是Google服務強大的地方了，它可以將Google表單收到的回覆同步到一個Google Sheet中，如此一來Google Sheet就成為一個簡單的儲存庫了。<br>
讓我們總結一下這個方式<br>
從客製化表單按下送出按鈕後，會對Google表單發一個POST請求，以查詢字串的方式將資料帶到Google表單回覆中，再利用Google服務相通的功能，將資料同步儲存到Google Sheet中。<br>
不過其實除了一開始設定以外，之後的流程都可以忽略Google表單那層，我們可以看成從客製化表單送出後，資料就直接存到Google Sheet裡面了。<br>


（在開始之前記得將表單權限設為所有人都可以存取喔！）
在建立Google表單之後，就可以來建立問題欄位，注意囉，這邊的欄位要跟你網頁上客製化表單的欄位一樣喔！這邊因為示範方便所以隨便新增兩個欄位。<br>
接著點選回覆的分頁，有個Google Sheet的圖示，點擊創建一個與這個表單連結的試算表（其實也可以連結現有表單不需重新建立）<br>


![](https://allthings.how/content/images/wordpress/2021/01/allthings.how-how-to-link-a-google-form-to-a-google-sheet-lf3.png)
## Google表單連結Google Sheet
接下來我們需要取得兩個東西：<br>
Google表單的ID<br>
表單中各欄位的ID<br>
Google表單的ID<br>

先點選右上角的「傳送」按鈕，再點選連結的傳送方式，會出現一段URL，以示範裡的例子來說，URL為<br>
<a>https://docs.google.com/forms/d/e/1FAIpQLSfniOOcWIeG4FsL14tqLoEfqz9oRYuGru8rwMvOhH2vs7UPPg/viewform?usp=sf_link</a>
我們要的form ID即為/e後的那段字串，即是<br>
1FAIpQLSfniOOcWIeG4FsL14tqLoEfqz9oRYuGru8rwMvOhH2vs7UPPg <br>
表單中各欄位的ID<br>
我們得取得Google表單中各欄位的ID，才能在POST請求時正確傳資料到該欄位，首先先進入上面拿到的用來分享表單的連結<br>

<a>https://docs.google.com/forms/d/e/1FAIpQLSfniOOcWIeG4FsL14tqLoEfqz9oRYuGru8rwMvOhH2vs7UPPg/viewform?usp=sf_link</a>

### 進入後開啟瀏覽器的開發者工具

### 在form元素內層可以找到幾個隱藏的input

entry.979040326與entry.589442071就是我們要找的欄位ID，注意這邊隱藏input的順序是跟表單欄位的順序一樣的喔！<br>
## POST請求
必要的資料都拿到後就可以試著在客製化表單送出時發出POST請求了<br>

~~~js
import { stringify } from 'qs';

// 上傳數據到Google表單的函數
export const uploadToGoogleSheet = (formId: string, query: Record<string, unknown>): Promise<void> => {
  return new Promise((resolve, reject) => {
    fetch(`https://docs.google.com/forms/d/e/${formId}/formResponse?&${stringify(query)}&submit=SUBMIT`, {
      method: 'POST',
      mode: 'no-cors', // Google只接受'no-cors'模式的表單提交
      redirect: 'follow',
      referrer: 'no-referrer',
    })
    .then(() => resolve())
    .catch(() => reject());
  });
};

// 處理表單提交的函數
const handleSubmit = (values: any) => {
  const FORM_ID = '1FAIpQLSfniOOcWIeG4FsL14tqLoEfqz9oRYuGru8rwMvOhH2vs7UPPg';
  const query = {
    'entry.979040326': values.data1, // 傳送使用者在表單填寫的資訊
    'entry.589442071': values.data2,
  };

  uploadToGoogleSheet(FORM_ID, query)
    .then(() => {
      // 提交成功後的處理
      console.log('數據成功提交到Google表單');
    })
    .catch(() => {
      // 提交失敗後的處理
      console.error('提交數據到Google表單時出錯');
    });
};
~~~


程式碼其實相當簡單，我們只需要將剛剛拿到的form ID帶進URL中，再將各欄位的entry ID帶入查詢字串中當作鍵，值則給予使用者在客製化表單上輸入的值，POST請求成功後回到Google Sheet就會看到剛剛提交的值已經被記錄下來囉！<br>

## 小結
雖然說這是一個沒什麼技術含量的分享，但我認為這樣的方式十分有趣，以某些使用情境來說也是非常適合的一種記錄表單資料的方式，Google表單雖然方便，但有時候總會因為缺乏「美感」或是獨立於我們網頁之外而使人卻步，如果你有類似的問題，不妨試試看本篇文章教的方法吧，希望這篇文章能夠幫助到螢幕前的你！