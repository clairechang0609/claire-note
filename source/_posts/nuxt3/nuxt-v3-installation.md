---
title: Nuxt.js 3.x 專案架設
date: 2023-06-17
tags: [ nuxt3 ]
category: Nuxt3
description: 本篇文章說明如何快速架設 Nuxt3 專案，以及要注意的細節
image: https://imgur.com/zvpIl61.png
---

<div style="display: flex; justify-content: center; margin: 0 0 20px;">
    <img style="width: 100%; max-width: 600px;" src="https://imgur.com/zvpIl61.png">
</div>

先前有寫了一系列 [Nuxt.js 2.x 相關文章](https://clairechang.tw/categories/Nuxt/)，Vue 3 也出了好一段時間，Nuxt 3 也終於在 2022 年底推出了穩定版本，除了支援 TypeScript，更徹底的進行重構，精簡了核心、編譯速度更快，提升開發體驗。

接下來一起來建立一個 Nuxt 3 專案吧！

<!-- more -->

首先使用 Nuxt 指令列工具（command line interface）Nuxi 進行安裝，`<project-name>` 專案名稱可以自訂：

```bash
npx nuxi@latest init <project-name>
```

這裡命名為 **my-app**

```bash
npx nuxi@latest init my-app
```

{% colorquote info %}
注意：node 版本必須要 `v16.0.0` 以上才能順利執行，可以透過 `node -v` 指令來查看當前使用版號
{% endcolorquote %}

安裝完成後，接著進入到專案目錄

```bash
cd my-app
```

安裝相依套件

```bash
npm install
```

於開發環境啟動專案

```bash
npm run dev
```

接著於瀏覽器輸入網址，就可以看到畫面囉

<div style="display: flex; justify-content: left; margin: 20px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/RxerC1x.png">
</div>

根據提示，`<NuxtWelcome />` 為預設元件，可以取代成自己的程式碼，到這裡專案就建置完成啦！

<div style="display: flex; justify-content: left; margin: 20px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/IWEfyjV.png">
</div>

如果採用 Visual Studio Code 開發，官方文件建議可以安裝 **Vue Language Features (Volar)** ，另外如果搭配 TypeScript 進行開發，也推薦安裝 **TypeScript Vue Plugin (Volar)** ，這兩個套件可以協助我們在開發時提示、補足、highlight 程式碼，功能強大

---

參考文章：

https://nuxt.com/docs/getting-started/installation