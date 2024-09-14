---
layout: post
title: Pinia 在 Vue 3 中的進階應用技巧
image: https://pinia.vuejs.org/logo.svg
accent_image: 
  background: url('https://www.pbs.org/wgbh/nova/media/images/bfcppfu.width-800.png') center/cover
  overlay: false
accent_color: '#ffd859'
theme_color: '#ffd859'
invert_sidebar: false
related_posts:
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 前言

Pinia 作為 Vue 3 的官方狀態管理庫,提供了簡潔、靈活且強大的 API。本文將深入探討 Pinia 在 Vue 3 中的進階應用技巧,幫助開發者更好地組織和管理應用狀態。

## Pinia 基礎回顧

在深入探討進階技巧之前,讓我們簡單回顧一下 Pinia 的基本用法:

~~~js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => ({ 
    count: 0 
  }),
  getters: {
    doubleCount: (state) => state.count * 2,
  },
  actions: {
    increment() {
      this.count++
    },
  },
})
~~~



## 進階應用技巧

### 1. 組合式函數與 Pinia 結合

Pinia 可以與 Vue 3 的組合式函數(Composition API)完美結合,實現更靈活的狀態管理:

~~~js
import { ref, computed } from 'vue'
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)
  const doubleCount = computed(() => count.value * 2)
  
  function increment() {
    count.value++
  }

  return { count, doubleCount, increment }
})
~~~



這種方式使得狀態邏輯更容易組織和重用。

### 2. 訂閱 Store 變化

Pinia 允許訂閱 store 的變化,這對於日誌記錄、持久化等場景非常有用:


~~~js
const unsubscribe = store.$subscribe((mutation, state) => {
console.log(mutation.type)
console.log(mutation.storeId)
console.log(mutation.payload)
})
// 停止訂閱
unsubscribe()
~~~



### 3. 插件系統

Pinia 提供了強大的插件系統,可以擴展 store 的功能:

~~~js
import { createPinia } from 'pinia'

function myPiniaPlugin(context) {
  return {
    ...context,
    $reset() {
      // 自定義重置邏輯
    }
  }
}

const pinia = createPinia()
pinia.use(myPiniaPlugin)
~~~


### 4. 持久化存儲

結合插件系統,我們可以輕鬆實現狀態的持久化存儲:

~~~js
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({
    name: '',
    token: '',
  }),
  persist: {
    enabled: true,
    strategies: [
      { storage: localStorage, paths: ['token'] },
      { storage: sessionStorage, paths: ['name'] },
    ],
  },
})
~~~


### 5. 模塊化和命名空間

Pinia 原生支持模塊化,無需額外配置:

~~~js
// stores/user.js
export const useUserStore = defineStore('user', { ... })
// stores/product.js
export const useProductStore = defineStore('product', { ... })
// 使用
import { useUserStore, useProductStore } from '@/stores'
const userStore = useUserStore()
const productStore = useProductStore()
~~~



### 6. 與 Vue Router 集成

Pinia 可以與 Vue Router 無縫集成,實現基於路由的狀態管理:

~~~js
import { useRoute } from 'vue-router'
import { defineStore } from 'pinia'
export const usePageStore = defineStore('page', () => {
const route = useRoute()
const pageTitle = computed(() => route.meta.title || 'Default Title')
return { pageTitle }
})
~~~




### 7. 測試友好

Pinia 的設計使得單元測試變得簡單:

~~~js
import { setActivePinia, createPinia } from 'pinia'
import { useCounterStore } from './counter'

describe('Counter Store', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })

  it('increments', () => {
    const store = useCounterStore()
    expect(store.count).toBe(0)
    store.increment()
    expect(store.count).toBe(1)
  })
})
~~~



## 總結

Pinia 在 Vue 3 中提供了強大而靈活的狀態管理解決方案:

1. 與組合式 API 的無縫集成使得狀態邏輯更易組織和重用。
2. 訂閱機制和插件系統提供了強大的擴展能力。
3. 原生支持模塊化,無需額外配置。
4. 與 Vue Router 的集成使得基於路由的狀態管理變得簡單。
5. 良好的測試支持有助於提高代碼質量。

通過掌握這些進階技巧,開發者可以更好地利用 Pinia 來構建可維護、高性能的 Vue 3 應用。

## 參考資料
- [Pinia 官方文檔](https://pinia.vuejs.org/)
- [Vue 3 文檔](https://v3.vuejs.org/)
- [Vue Router 文檔](https://router.vuejs.org/)