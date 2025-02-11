---
title: Nuxt.js 3.x 自訂 Loading 效果
date: 2023-08-24
tags: [ nuxt3 ]
category: Nuxt3
description: Nuxt.js 提供了預設的進度條效果，在路徑切換時顯示。如果想要調整觸發時機或樣式，也可以透過自訂共用元件來達成
image: https://imgur.com/9QmKCKF.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/articles/10329897)
>

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 600px;" src="https://imgur.com/9QmKCKF.png">
</div>

Nuxt3 提供了預設進度條元件 `<NuxtLoadingIndicator>`，在路徑切換時顯示。也可以自訂共用元件來調整觸發時機或樣式。

<!-- more -->

## **路由切換（Page Navigation）Loading**

### **方法一：<NuxtLoadingIndicator>**

Nuxt3 內建元件，在**頁面切換時觸發**

**使用方式：**

在 `app.vue` 或是 layouts 加上 `<NuxtLoadingIndicator>`

**Props 選項：**

- **color：**loading bar 顏色，預設為漸層色
- **height：**loading bar 高度，預設 3（px）
- **duration：**loading bar 持續時間，預設 2000（毫秒）
- **throttle：**節流時間，會隱藏 loading bar，預設 200（毫秒）

```jsx
// app.vue
<template>
  <div>
    <NuxtLoadingIndicator :throttle="0" />
    <NuxtPage />
  </div>
</template>
```

{% colorquote info %}
注意：Nuxt ≥ 3.1.0 需加上 `:throttle="0"`，否則受節流時間影響看不到 loading 效果（[參考討論串](https://github.com/vrtpro/chocolattech/issues/32)）
{% endcolorquote %}

### **方法二：自訂 Loading Indicator**

建立 Loading 元件，使用參數 `isLoading` 判斷是否顯示 Loading Indicator，接著透過 Nuxt [app runtime hooks](https://nuxt.com/docs/api/advanced/hooks#app-hooks-runtime) 建立攔截器，這裡使用 `page:start` 以及 `page:finish`

```jsx
// components/CustomLoadingIndicator.vue
<template>
  <div class="loading-indicator" :class="{ 'show': isLoading }">
    Loading...
  </div>
</template>

<script setup>
  const nuxtApp = useNuxtApp();
  const isLoading = ref(false);

  nuxtApp.hook('page:start', () => {
    isLoading.value = true;
  });

  nuxtApp.hook('page:finish', () => {
    setTimeout(() => {
      isLoading.value = false;
    }, 200);
  });
</script>

<style lang="scss" scoped>
.loading-indicator {
  opacity: 0;
  transition: opacity 0.5s ease-in-out;
  &.show {
    opacity: 1;
    transition: opacity 0.2s ease-in-out;
  }
}
</style>
```

在 `app.vue` 加入自訂元件

```jsx
// app.vue
<template>
  <div>
    <CustomLoadingIndicator />
    <NuxtPage />
  </div>
</template>
```

效果如下：

<video controls width="100%">
  <source src="https://imgur.com/mvcCSLb.mp4" type="video/mp4" />
</video>

---

## **資料請求（Data Fetching）Loading**

使用 `useFetch` 或 `useAsyncData` Composables，參數 `pending` 為布林值，顯示資料是否還在請求狀態，用來判斷是否顯示 Loading Indicator

{% colorquote info %}
Nuxt3 `useFetch` 相關知識，可以參考 [這篇文章](https://clairechang.tw/2023/07/19/nuxt3/nuxt-v3-data-fetching/)
{% endcolorquote %}

```jsx
// pages/about.vue
<template>
  <div>
    <div v-if="pending">
      Loading ...
    </div>
    <div v-else>
      user: <pre>{{ data }}<pre>
      <button type="button" @click="refresh()">refresh</button>
    </div>
  </div>
</template>

<script setup>
const { data, pending, refresh } = useFetch('/api/about');
</script>
```

效果如下：

<video controls width="100%">
  <source src="https://imgur.com/HyOhm0t.mp4" type="video/mp4" />
</video>

---

參考資源：

https://medium.com/@flanker72/nuxt3-complex-solutions-page-loading-indicator-e34b5a86be52
https://nuxt.com/docs/api/components/nuxt-loading-indicator
https://nuxt.com/docs/getting-started/data-fetching