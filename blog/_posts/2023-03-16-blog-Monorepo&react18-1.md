---
layout: post
title: React 18 與 Monorepo 的實際應用
image: https://miro.medium.com/max/1400/1*HSisLuifMO6KbLfPOKtLow.jpeg
accent_image: 
  background: url('https://www.pbs.org/wgbh/nova/media/images/bfcppfu.width-800.png') center/cover
  overlay: false
accent_color: '#61DAFB'
theme_color: '#61DAFB'
invert_sidebar: false
related_posts:
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 前言

在現代前端開發中，Monorepo 已經成為管理大型代碼庫的一種流行方式。React 18 帶來了許多新特性，這些特性在 Monorepo 中的應用可以進一步提升開發效率和應用性能。本文將介紹如何在 Monorepo 中實際應用 React 18。

## Monorepo 簡介

Monorepo 是指將多個項目存放在同一個代碼庫中，這樣可以更好地管理依賴關係、共享代碼和統一配置。常見的 Monorepo 工具有 Lerna 和 Nx。

## 設置 Monorepo

首先，我們需要設置一個 Monorepo。這裡我們使用 Nx 作為示例。

### 1. 初始化 Nx 工作區
~~~js
bash
npx create-nx-workspace@latest my-workspace
cd my-workspace
~~~

### 2. 添加 React 應用

~~~js
bash
nx generate @nrwl/react:application my-app
~~~


### 3. 添加 React 庫

~~~js
bash
nx generate @nrwl/react:library my-lib
~~~

## 在 Monorepo 中使用 React 18

### 1. Concurrent Mode

在 Monorepo 中，我們可以在應用和庫中使用 React 18 的 Concurrent Mode 來提升性能。

~~~js
// apps/my-app/src/app/app.tsx
import React, { useState, useTransition } from 'react';

function App() {
  const [isPending, startTransition] = useTransition();
  const [count, setCount] = useState(0);

  const handleClick = () => {
    startTransition(() => {
      setCount(c => c + 1);
    });
  };

  return (
    <div>
      <h1>計數: {count}</h1>
      <button onClick={handleClick} disabled={isPending}>
        {isPending ? '更新中...' : '增加'}
      </button>
    </div>
  );
}

export default App;
~~~

### 2. Automatic Batching

自動批處理功能可以在 Monorepo 中的多個應用和庫中使用，以減少重新渲染次數。

~~~js
// libs/my-lib/src/lib/my-lib.tsx
import React, { useState } from 'react';

export function MyLib() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');

  const handleClick = () => {
    setCount(c => c + 1);
    setText('計數已更新');
  };

  return (
    <div>
      <h1>計數: {count}</h1>
      <p>{text}</p>
      <button onClick={handleClick}>增加</button>
    </div>
  );
}
~~~

### 3. Suspense for Data Fetching

在 Monorepo 中，我們可以使用 Suspense 來處理數據加載，這使得異步操作更加簡單和直觀。

~~~js
// apps/my-app/src/app/app.tsx
import React, { Suspense } from 'react';
import { MyLib } from '@my-workspace/my-lib';

const DataComponent = React.lazy(() => import('./data-component'));

function App() {
  return (
    <div>
      <h1>React 18 與 Monorepo</h1>
      <Suspense fallback={<div>加載中...</div>}>
        <DataComponent />
      </Suspense>
      <MyLib />
    </div>
  );
}

export default App;
~~~

## 總結

通過在 Monorepo 中應用 React 18 的新特性，我們可以顯著提升開發效率和應用性能：

1. Concurrent Mode 提升了應用的響應速度。
2. Automatic Batching 減少了重新渲染次數。
3. Suspense for Data Fetching 使異步操作更加簡單。

這些特性在 Monorepo 中的應用，讓我們能夠更好地管理大型代碼庫，實現高效的開發流程。

## 參考資料
- [React 官方文檔](https://reactjs.org/)
- [Nx 官方文檔](https://nx.dev/)
- [React 18 發布公告](https://reactjs.org/blog/2022/03/29/react-v18.html)