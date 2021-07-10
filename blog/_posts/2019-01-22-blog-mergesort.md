---
layout: post
title: JS - 合併排序法 O(n logn) Merge Sort - 資料結構演算法
image: https://upload.wikimedia.org/wikipedia/commons/c/c5/Merge_sort_animation2.gif
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

## 初學者入門學習的演算法 - 合併排序法 Merge Sort

排序是指按特定順序（數字或字母順序）排列列表中的項目。排序通常與搜索一起使用。
如果在視覺上和算法上對列表進行了排序，那麼在給定列表中搜索元素（稱為key值）通常會更容易。
有很多方法（算法）可以對給定的元素列表進行排序。歸併排序是一種更流行、更有效的方法。

## 理解歸併排序背後的邏輯
合併排序使用分而治之的概念對給定的元素列表進行排序。它將問題分解為更小的子問題，直到它們變得簡單到可以直接解決。

以下是歸併排序的步驟：

將給定的列表分成兩半（對於具有奇數個元素的列表，大致相等的一半）。
繼續以相同的方式劃分子數組，直到只剩下單個元素數組。
從單元素數組開始，合併子數組，以便對每個合併的子數組進行排序。
重複第 3 步，最終得到一個排序數組。

![](https://upload.wikimedia.org/wikipedia/commons/thumb/c/cc/Merge-sort-example-300px.gif/220px-Merge-sort-example-300px.gif)

```

function merge(left, right) {
    let arr = []
    // Break out of loop if any one of the array gets empty
    while (left.length && right.length) {
        // Pick the smaller among the smallest element of left and right sub arrays
        if (left[0] < right[0]) {
            arr.push(left.shift())  
        } else {
            arr.push(right.shift()) 
        }
    }
    
    // Concatenating the leftover elements
    // (in case we didn't go through the entire left or right array)
    return [ ...arr, ...left, ...right ]
}

function mergeSort(array) {
  const half = array.length / 2
  
  // Base case or terminating case
  if(array.length < 2){
    return array 
  }
  
  const left = array.splice(0, half)
  return merge(mergeSort(left),mergeSort(array))
}

array = [4, 8, 7, 2, 11, 1, 3];
console.log(mergeSort(array));

[1, 2, 3, 4, 7, 8, 11]

```



## Reference
https://stackabuse.com/merge-sort-in-javascript