---
layout: post
title: Monorepo 與 Docker 的應用技巧
image: https://miro.medium.com/v2/resize:fit:2000/1*vQK4s0lOiK1ZkcXxFNIMDQ.png
accent_image: 
  background: url('https://www.pbs.org/wgbh/nova/media/images/bfcppfu.width-800.png') center/cover
  overlay: false
accent_color: '#2496ed'
theme_color: '#2496ed'
invert_sidebar: false
related_posts:
sitemap: false
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 前言

在現代前端開發中，Monorepo 和 Docker 是兩個強大的工具。Monorepo 可以幫助我們更好地管理大型代碼庫，而 Docker 則提供了輕量級的容器化解決方案。本文將介紹如何將這兩者結合起來，提升開發效率和應用性能。

## Monorepo 簡介

Monorepo 是指將多個項目存放在同一個代碼庫中，這樣可以更好地管理依賴關係、共享代碼和統一配置。常見的 Monorepo 工具有 Lerna 和 Nx。

## Docker 簡介

Docker 是一個開源的平台，通過容器化技術來實現應用的快速部署和運行。它可以將應用及其依賴打包成一個輕量級、可移植的容器，從而實現跨環境的一致性。

## 設置 Monorepo 和 Docker

### 1. 初始化 Nx 工作區

首先，我們需要設置一個 Monorepo。這裡我們使用 Nx 作為示例。

~~~bash
npx create-nx-workspace@latest my-workspace
cd my-workspace
~~~

### 2. 添加 Vue 應用

~~~bash
nx generate @nx-plus/vue:application my-app
~~~

### 3. 添加 Docker 支持

在應用中添加 Docker 支持，首先需要創建一個 Dockerfile。

~~~dockerfile
<!-- apps/my-app/Dockerfile -->
# 使用官方的 Node.js 圖像作為基礎圖像
FROM node:16-alpine

# 設置工作目錄
WORKDIR /app

# 複製 package.json 和 package-lock.json
COPY package*.json ./

# 安裝依賴
RUN npm install

# 複製應用代碼
COPY . .

# 構建應用
RUN npm run build

# 暴露應用運行的端口
EXPOSE 3000

# 啟動應用
CMD ["npm", "start"]
~~~

### 4. 創建 Docker Compose 文件

為了更方便地管理多個容器，我們可以使用 Docker Compose。

~~~yaml
<!-- docker-compose.yml -->
version: '3.8'
services:
  my-app:
    build:
      context: ./apps/my-app
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    volumes:
      - ./apps/my-app:/app
    environment:
      - NODE_ENV=development
~~~

## 在 Monorepo 中使用 Docker

### 1. 本地開發

在本地開發中，我們可以使用 Docker Compose 來啟動應用。

~~~bash
docker-compose up --build
~~~

### 2. 部署到生產環境

在生產環境中，我們可以將應用打包成 Docker 映像，並推送到 Docker Hub 或其他容器託管服務。

~~~bash
# 登錄到 Docker Hub
docker login

# 構建 Docker 映像
docker build -t my-app:latest ./apps/my-app

# 推送 Docker 映像
docker push my-app:latest
~~~

## 總結

通過將 Monorepo 和 Docker 結合，我們可以顯著提升開發效率和應用性能：

1. Monorepo 提供了更好的代碼管理和依賴管理。
2. Docker 提供了輕量級的容器化解決方案，實現跨環境的一致性。
3. Docker Compose 使得多容器應用的管理更加方便。

這些應用技巧讓我們能夠更好地管理大型代碼庫，實現高效的開發和部署流程。

## 參考資料
- [Nx 官方文檔](https://nx.dev/)
- [Docker 官方文檔](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)