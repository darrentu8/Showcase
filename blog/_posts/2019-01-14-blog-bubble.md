---
layout: post
title: JS - 氣泡排序法 O(n²) Bubble Sort - 資料結構演算法
image: https://upload.wikimedia.org/wikipedia/commons/3/37/Bubble_sort_animation.gif
accent_image: 
  background: url('https://www.pbs.org/wgbh/nova/media/images/bfcppfu.width-800.png') center/cover
  overlay: false
accent_color: '#ccc'
theme_color: '#ccc'
invert_sidebar: false
related_posts:
  - blog/_posts/2019-01-22-blog-mergesort.md
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 初學者入門學習的演算法 - 泡沫排序 Bubble Sort

泡沫排序演算法的運作如下：

比較相鄰的元素。如果第一個比第二個大，就交換他們兩個。
對每一對相鄰元素作同樣的工作，從開始第一對到結尾的最後一對。這步做完後，最後的元素會是最大的數。
針對所有的元素重複以上的步驟，除了最後一個。
持續每次對越來越少的元素重複上面的步驟，直到沒有任何一對數字需要比較。
由於它的簡潔，泡沫排序通常被用來對於程式設計入門的學生介紹演算法的概念。

## 如果相鄰元素的順序錯誤，則冒泡排序算法會重複交換相鄰元素

冒泡排序常用於實現排序算法。氣泡中的每個元素都與其周圍的元素形成對比。以泡泡形式。該列表將通過算法進行處理。對具有 n 個元素的列表進行排序需要 N-1 遍。取一個包含 n 個元素的表 A，這些元素必須用某種氣泡排序。算法如下
• 在第 1 輪中，數字 A[0] 由 A[1] 組成，數字 A[1] 與 A[2] 進行比較，數字 A[2] 與 A[3] 進行比較，等等. 列表中最大的項放在第 1 輪的末尾，位於列表的最高索引處。
• A[0] 與第 2 輪中的 A[1] 進行比較，A[1] 與 A[2] 進行比較，依此類推。第二大項放在列表末尾的第二高索引上通過 2。
• 第 n-1 次移動將 A[0] 與 A[1]、A[1] 與 A[2] 進行比較，依此類推。傳球在底部結束。列表的第一個索引是列表中最小的項目。

```
var n = [282, 6, 88, 44, 33, 12, 15, 121, 351, 234, 99];
for (m = 0; m < 10; m++) {
    for (j = 0; j < 10; j++) {
        if (n[m] < n[j]) {
            temp = n[m];
            n[m] = n[j];
            n[j] = temp;
        }
    }
}
hint = "<br>";
document.writeln("數列經泡沫排序後 .." + hint);
for (m = 0; m < 10; m++) {
    document.writeln(n[m]);
    document.writeln(hint);
}

```

![](/assets/img/blog/bubble.png)


## Reference
https://laptrinhx.com/