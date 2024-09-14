---
layout: post
title: Vue 3 與 React 16.8 基本技術比較
image: https://media.licdn.com/dms/image/D5612AQG28-Q25BRGDA/article-cover_image-shrink_720_1280/0/1715091435390?e=2147483647&v=beta&t=a-ooQtZ5J4DThRJvSJpQjPG9x5czjYkUtGnwNfzeKRE
accent_image: 
  background: url('https://media.licdn.com/dms/image/D5612AQG28-Q25BRGDA/article-cover_image-shrink_720_1280/0/1715091435390?e=2147483647&v=beta&t=a-ooQtZ5J4DThRJvSJpQjPG9x5czjYkUtGnwNfzeKRE') center/cover
  overlay: false
accent_color: '#61dafb'
theme_color: '#61dafb'
invert_sidebar: false
related_posts:
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 前言

Vue 3和React 16.8都是在2020年前後推出的重要版本,它們都引入了新的特性來改善開發體驗。本文將從幾個關鍵方面比較Vue 3和React 16.8的基本技術,幫助開發者更好地理解這兩個框架的異同。

## 組件定義

### Vue 3

Vue 3引入了組合式API,同時保留了選項式API:

~~~js
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)
    const increment = () => {
      count.value++
    }
    return { count, increment }
  }
}
~~~

### React 16.8

React 16.8引入了Hooks,使函數組件能夠使用狀態和其他React特性:

~~~js
import React, { useState } from 'react'

function Counter() {
  const [count, setCount] = useState(0)
  
  const increment = () => {
    setCount(count + 1)
  }
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={increment}>增加</button>
    </div>
  )
}
~~~

## 響應式系統

### Vue 3

Vue 3使用Proxy實現響應式系統,提供了ref和reactive兩個主要API:

~~~js
import { ref, reactive } from 'vue'

const count = ref(0)
const state = reactive({ name: 'Vue' })
~~~

### React 16.8

React 16.8通過useState和useReducer管理狀態:

~~~jsx
import React, { useState } from 'react'

function Example() {
  const [count, setCount] = useState(0)
  const [state, setState] = useState({ name: 'React' })
}
~~~

## 生命週期

### Vue 3

Vue 3提供了一系列組合式API的生命週期鉤子:

~~~js
import { onMounted, onUpdated, onUnmounted } from 'vue'

export default {
  setup() {
    onMounted(() => {
      console.log('組件已掛載')
    })
    onUpdated(() => {
      console.log('組件已更新')
    })
    onUnmounted(() => {
      console.log('組件已卸載')
    })
  }
}
~~~

### React 16.8

React 16.8使用useEffect Hook模擬生命週期:

~~~js
import React, { useEffect } from 'react'

function Example() {
  useEffect(() => {
    console.log('組件已掛載')
    return () => {
      console.log('組件將卸載')
    }
  }, [])
  
  useEffect(() => {
    console.log('組件已更新')
  })
}
~~~

## 條件渲染

### Vue 3

~~~js
<template>
  <div v-if="show">顯示內容</div>
  <div v-else>隱藏內容</div>
</template>
~~~

### React 16.8

~~~js
function Example({ show }) {
  return (
    <>
      {show ? <div>顯示內容</div> : <div>隱藏內容</div>}
    </>
  )
}
~~~

## 列表渲染

### Vue 3

~~~js
<template>
  <ul>
    <li v-for="item in items" :key="item.id">
      {{ item.name }}
    </li>
  </ul>
</template>
~~~

### React 16.8

~~~jsx
function List({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  )
}
~~~

## 總結

Vue 3和React 16.8都引入了重要的新特性,使得組件開發更加靈活和高效:

1. Vue 3引入組合式API,React 16.8引入Hooks,兩者都旨在提供更好的代碼組織和重用。
2. Vue 3的響應式系統更加直觀,而React 16.8的狀態管理需要顯式調用。
3. Vue 3保留了模板語法,React繼續使用JSX。
4. 兩者都提供了類似的生命週期管理方式,但實現方式不同。

選擇使用哪個框架取決於項目需求、團隊熟悉度和個人偏好。兩個框架都在不斷evolve,深入學習和實踐都能讓你成為出色的前端開發者。

## 參考資料
- [Vue 3 官方文檔](https://v3.vuejs.org/)
- [React 16.8 官方文檔](https://reactjs.org/blog/2019/02/06/react-v16.8.0.html)