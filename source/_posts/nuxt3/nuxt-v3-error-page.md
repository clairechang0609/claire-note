---
title: Nuxt.js 3.x 自訂 Error Page 與錯誤處理
date: 2023-09-07
tags: [ nuxt3 ]
category: Nuxt3
description: 本篇說明如何在 Nuxt3 專案自訂錯誤頁面，以及如何進行錯誤處理
image: https://imgur.com/He7uasf.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/users/20130500/ironman/6236)
>

Nuxt 3 提供大量內建功能，包括預設錯誤頁面，可以在隱藏資料夾 `.nuxt/dev/index.mjs` 看到，我們也可以自訂錯誤頁面，Nuxt 預設錯誤畫面如下：

<div style="display: flex; justify-content: center; margin: 30px 0; border: 1px solid rgb(200, 200, 200);">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/He7uasf.png">
</div>

<!-- more -->

## **自訂錯誤頁**

在專案根目錄新增 `error.vue`，error page 不具有路由，因此不能使用 `definePageMeta` 方法

檔案結構如下：

```
|—— app.vue
|—— error.vue
```

錯誤頁會收到一個名為 `error` 的 `props` 物件

`error` 物件內容包含以下參數 `url`、`statusCode`、`statusMessage`、`message`、`description`、`data`，自訂參數可以放在 `data` 內

```jsx
// error.vue
<template>
  <div>
    <h2>{{ error.statusCode }}</h2>
    <p>{{ error.message }}</p>
    <NuxtLink to="/">回首頁</NuxtLink>
  </div>
</template>

<script setup>
const props = defineProps({
  error: {
    type: Object,
    required: true
  }
});
</script>
```

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/bTPCoaK.png">
</div>

---

## **錯誤頁面渲染時機**

當發生**致命錯誤（fatal error）**時，自動觸發錯誤頁面，如果是非致命錯誤（non-fatal error）只會拋出錯誤訊息，可能觸發錯誤頁的時機如下：

### **Server Side**

- 執行 Nuxt plugins 發生錯誤
- 編譯 Vue app 到 HTML 發生錯誤
- Server API 發生錯誤

### **Client Side**

- 執行 Nuxt plugins 發生錯誤
- `app:beforeMount` 生命週期發生錯誤
- 不會被 `onErrorCaptured` 方法或 `vue:error` 生命週期捕捉的錯誤
- Vue app 初始化與 `app:mounted` 生命週期發生錯誤

---

## **輔助函式**

透過以下輔助函式來手動觸發錯誤頁面，以及進行錯誤處理

### **createError**

同時支援 server-side 與 client-side，用來建立帶有錯誤訊息的物件

- **server-side：**觸發錯誤頁面，視為致命錯誤（fatal error）
- **client-side：**拋出非致命錯誤，如要顯示錯誤頁面，需加上 `fatal: true`

```jsx
// pages/user/[id].vue
<script setup>
const route = useRoute();
const { data } = await useFetch('/api/user/${route.params.id}');
if (!data.value) {
  throw createError({ statusCode: 404, statusMessage: 'Page Not Found', fatal: true });
}
</script>
```

### **showError**

功能同 `createError`

- **server-side：**需定義在 middleware、plugins 或是 `setup()` 方法內，觸發錯誤頁面
- **client-side：**觸發錯誤頁面

```jsx
// pages/user/[id].vue
<script setup>
const route = useRoute();
const { data } = await useFetch('/api/user/${route.params.id}');
if (!data.value) {
  throw showError({ statusCode: 404, statusMessage: 'Page Not Found' });
}
</script>
```

### **clearError**

用來清除當前處理的錯誤訊息，可以透過 `redirect` 重新導向到其他頁面

```jsx
// error.vue
<template>
  <div>
    <h2>{{ error.statusCode }}</h2>
    <p>{{ error.message }}</p>
    <button @click="handleError">回首頁</button>
  </div>
</template>

<script setup>
const props = defineProps({
  error: {
    type: Object,
    required: true
  }
});

const handleError = () => clearError({ redirect: '/' });
</script>
```

---

參考資源：

https://nuxt.com/docs/getting-started/error-handling
https://nuxt.com/docs/api/advanced/hooks