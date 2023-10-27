---
title: Nuxt.js 3.x Meta Tags and SEO
date: 2023-07-26
tags: [ nuxt3 ]
category: Nuxt3
description: 本篇說明如何在 Nuxt3 應用程式中動態控制 <head> 標籤，來優化 SEO、meta 資訊、樣式表、程式碼片段等在網頁 <head> 中需要的內容
image: https://imgur.com/1uJRpte.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/users/20130500/ironman/6236)
>

<div style="display: flex; justify-content: center; margin: 0;">
  <img style="width: 100%; max-width: 600px;" src="https://imgur.com/1uJRpte.png">
</div>

Nuxt3 搭配 [Unhead](https://unhead.harlanzw.com/) 套件，讓我們可以在應用程式中動態控制 `<head>` 標籤，定義 meta 資訊、樣式表、程式碼片段等在網頁中需要的內容，有助於 SEO 搜尋引擎優化

<!-- more -->

### **動態控制函式：**

**Composables 組合式函式**

- `useHead()` & `useHeadSafe()`
- `useSeoMeta()` & `useServerSeoMeta()`

**Components 內建元件**

- `<Title>`、`<Style>`、`<Meta>`、`<Link>`、`<Body>`、`<Html>`、`<Head>`

---

## **Composables：useHead & useHeadSafe**

能夠定義完整 `<head>` 內容。 `useHeadSafe()` 是 `useHead()` 的包裝函式，對輸入內容進行檢核，能夠避免潛在的安全風險，像是 XSS 攻擊等安全漏洞

```jsx
<script setup>
useHead({
  title: 'My Website',
  htmlAttrs: {
    lang: 'zh-TW'
  },
  meta: [
    { name: 'description', content: 'Hello this is my site.' }
  ],
  script: [],
  link: []
})
</script>
```

### **Composables：useSeoMeta & useServerSeoMeta**

專門用來定義 meta tags，物件結構簡潔扁平。

如果我們的資料非響應式，可以使用 `useServerSeoMeta()`，在 server 端預先處理完 meta 相關邏輯，提升網頁效能

```jsx
<script setup>
useSeoMeta({
  title: 'My Website',
  ogTitle: 'My Website',
  description: 'Hello this is my site.',
  ogDescription: 'Hello this is my site.',
  ogImage: 'https://mysite/image.png',
  twitterCard: 'summary_large_image'
})
</script>
```

---

## **Components**

Nuxt3 內建 `<Title>`、`<Style>`、`<Meta>`、`<Link>`、`<Body>`、`<Html>`、`<Head>` 等元件讓我們可以直接在 template 配置 `<head>` 內容

```jsx
<template>
  <div>
    <Head>
      <Title>{{ title }}</Title>
      <Meta name="description" :content="title" />
    </Head>
  </div>
</template>

<script setup>
const title = 'About Page';
</script>
```

---

## **實作：全域配置**

### **方法一：nuxt.config**

{% colorquote info %}
`nuxt.config` 配置不支援響應式資料，因此官方文件建議在 `app.vue` 中使用 `useHead()` 等方法定義
{% endcolorquote %}

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  app: {
    head: {
      title: 'My Website',
      htmlAttrs: {
        lang: 'zh-TW'
      },
      meta: [
        { charset: 'utf-8' },
        { name: 'viewport', content: 'width=device-width, initial-scale=1, viewport-fit=cover' }
      ],
      link: [
        { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }
      ]
    }
  }
})
```

### **方法二：app.vue**

在 `app.vue` 中使用 `useHead()` 等方法定義 `<head>` 預設值，可以在其他 `pages` 頁面中定義相同的 meta 屬性覆寫預設值

```jsx
// app.vue
<template>
  <div>
    <NuxtPage />
  </div>
</template>

<script setup>
useHead({
    title: 'My Website',
    meta: [
      { name: 'description', content: 'Hello this is my site.' }
    ]
});
</script>
```

---

## **實作：局部配置**

在頁面上定義，能夠覆蓋在 `app.vue` 配置的 `<head>` 預設值，以下範例搭配 `useHead()`

```jsx
// pages/about.vue
<template>
  <div>
    ...
  </div>
</template>

<script setup>
useHead({
  title: 'About Page',
  meta: [
    { name: 'description', content: 'Hello this is about page.' }
  ]
});
</script>
```

---

## **延伸：透過後端 API 動態取得 meta 內容**

**範例情境：**每次進入頁面時請求該頁 meta 資料

使用 `useAsyncData()` 搭配 `$fetch` 方法實作，並搭配 `useSeoMeta()` 進行配置

{% colorquote info %}
需先具備 Nuxt3 data fetching 相關知識，可以參考 [這篇文章](https://clairechang.tw/2023/07/19/nuxt3/nuxt-v3-data-fetching/)
{% endcolorquote %}

```jsx
// pages/about.vue
<template>
  <div>
    ...
  </div>
</template>

<script setup>
useAsyncData('seo', async () => {
  const { title, description, image } = await $fetch('/api/seo/about');
  useSeoMeta({
    title,
    ogTitle: title,
    description,
    ogDescription: description,
    ogImage: image
  });
});
</script>
```

---

參考資源：

https://nuxt.com/docs/getting-started/seo-meta
https://unhead.harlanzw.com/