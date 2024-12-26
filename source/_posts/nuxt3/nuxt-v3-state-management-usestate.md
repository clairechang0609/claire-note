---
title: Nuxt.js 3.x 狀態管理 State Management (1)－useState
date: 2023-08-14
tags: [ nuxt3 ]
category: Nuxt3
description: 本篇說明如何在 Nuxt3 專案搭配 useState Composable，實作全域狀態共享
image: https://imgur.com/pdTkxrx.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/articles/10331341)
>

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 600px;" src="https://imgur.com/pdTkxrx.png">
</div>

專案開發過程中常會有狀態共享的需求。父子元件間資料傳遞可以使用 Props 和 $emit，或是 Provide 和 Inject（[參考文章](https://clairechang.tw/2023/01/13/vue/vue-communications/)），先前在 Nuxt2 介紹了 VueX 管理工具搭配 vuex-persistedstate 保存狀態（[參考文章](https://clairechang.tw/2022/11/22/nuxt/nuxt-vuex-store/)），接下來說明如何在 Nuxt3 利用更便利高效的方式管理共享狀態。

狀態管理預計分為以下三篇說明，**本篇將介紹** `useState`：

**1. useState：**Nuxt Composable
**2. Pinia：**Vue.js 狀態管理工具 [連結](https://clairechang.tw/2023/08/15/nuxt3/nuxt-v3-state-management-pinia/)
**3. pinia-plugin-persistedstate：**維持 Store 狀態 [連結](https://clairechang.tw/2023/08/16/nuxt3/nuxt-v3-state-management-persistedstate/)

<!-- more -->

# **useState**

useState 是 Nuxt3 提供的 Composable，適合用來建立響應式、伺服器端友善（SSR-friendly）的共享狀態

{% colorquote info %}
會特別提到 SSR-friendly，要先理解 Nuxt [Universal 渲染模式](https://nuxt.com/docs/guide/concepts/rendering#universal-rendering)（server-side ＋ client-side），由伺服器端預先載入 HTML，並傳給瀏覽器，接著由瀏覽器載入完整的 Javascript 並執行，至此網頁才具有互動性。
因此，若在頁面上定義響應式動態資料如下：

```jsx
<template>
  <div>
    {{ count }}
  </div>
</template>

<script setup>
const count = ref(Math.round(Math.random() * 1000));
</script>
```

瀏覽器會拋出警告：`Hydration text content mismatch`
原因是在伺服器端預先渲染出來的 count，跟瀏覽器後來執行 JS 運算出來的結果不同
{% endcolorquote %}

**useState 參數：**

- **key：**唯一值，避免重複取得資料（前面提到的 `Hydration` 問題），如果沒帶入，useState 會自動生成
- **init：**回傳初始值的函式，會回傳一個 `ref` 物件
- **T：**使用 TypeScript 型別

```jsx
useState<T>(key: string, init?: () => T | Ref<T>): Ref<T>
```

## **useState 共享狀態**

```jsx
// pages/index.vue
<template>
  <div>
    {{ counter }}
  </div>
</template>

<script setup>
const counter = useState('counter', () => Math.round(Math.random() * 1000));
</script>
```

其他頁面使用 `useState('counter')` 可以同步取得／更新狀態

```jsx
// pages/count.vue
<template>
  <div>
    {{ counter }}
    <button @click="counter++">+</button>
    <button @click="counter--">-</button>
  </div>
</template>

<script setup>
const counter = useState('counter', () => Math.round(Math.random() * 1000));
</script>
```

{% colorquote info %}
注意：若直接使用 `const counter = useState('counter')` 取值，而 `useState('counter')` 尚未被賦值，可能會發生錯誤，可以使用以下方法定義預設值
{% endcolorquote %}

## **useState 搭配 Composables 全域共享狀態**

將狀態定義在 Composables，可以**定義預設值**，並在頁面間共享狀態

名稱自訂，範例使用 `composables/state.js`

```jsx
// composables/state.js
export const useCounter = () => useState('counter', () => Math.round(Math.random() * 1000));
```

頁面使用 `useCounter()` 取值（等同於 `useState('counter')`），並可更新狀態

```jsx
// pages/count.vue
<template>
  <div>
    {{ counter }}
    <button @click="counter++">+</button>
    <button @click="counter--">-</button>
  </div>
</template>

<script setup>
const counter = useCounter();
</script>
```

## **clearNuxtState 清除狀態**

使用 `clearNuxtState(key)` 清除當前儲存的狀態

```jsx
// pages/index.vue
<template>
  <div>
    {{ counter }}
    <button @click="clearCounter()">clear</button>
  </div>
</template>

<script setup>
const counter = useCounter();
const clearCounter = () => {
  clearNuxtState('counter');
};
</script>
```

---

參考資源：

https://nuxt.com/docs/getting-started/state-management
https://ithelp.ithome.com.tw/articles/10302323