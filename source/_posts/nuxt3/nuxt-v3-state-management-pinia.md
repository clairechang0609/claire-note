---
title: Nuxt.js 3.x 狀態管理 State Management (2)－Pinia
date: 2023-08-15
tags: [ nuxt3 ]
category: Nuxt3
description: 本篇說明如何在 Nuxt3 專案搭配 Pinia 狀態管理工具，實作全域狀態共享
image: https://imgur.com/n21QxOI.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/users/20130500/ironman/6236)
>

<div style="display: flex; justify-content: center; margin: 0;">
  <img style="width: 100%; max-width: 500px;" src="https://imgur.com/n21QxOI.png">
</div>

專案開發過程中常會有狀態共享的需求。父子元件間資料傳遞可以使用 Props 和 $emit，或是 Provide 和 Inject（[參考文章](https://clairechang.tw/2023/01/13/vue/vue-communications/)），先前在 Nuxt2 介紹了 VueX 管理工具搭配 vuex-persistedstate 保存狀態（[參考文章](https://clairechang.tw/2022/11/22/nuxt/nuxt-vuex-store/)），接下來說明如何在 Nuxt3 利用更便利高效的方式管理共享狀態。

狀態管理預計分為以下三篇說明，**本篇將介紹 Pinia**：

**1. useState：**Nuxt Composable [連結](https://clairechang.tw/2023/08/14/nuxt3/nuxt-v3-state-management-usestate/)
**2. Pinia：**Vue.js 狀態管理工具
**3. pinia-plugin-persistedstate：**維持 Store 狀態 [連結](https://clairechang.tw/2023/08/16/nuxt3/nuxt-v3-state-management-persistedstate/)

<!-- more -->

# **Pinia**

[Pinia](https://pinia.vuejs.org/) 是 Vue 官方推薦使用的狀態管理工具，支援 Vue2 跟 Vue3，可以理解為 VueX 的接班人

**Pinia vs VueX：**

- 移除 `mutations`，統一透過 `action` 操作 `state`
- 支援 Typescript，不需要 types 來包裝
- 不需引入各種 magic strings，直接引入函式即可
- 不需要動態引入模組，預設為自動 import
- 移除巢狀 modules，提供更扁平的結構，store 間也可相互使用
- 移除 namespaced modules，所有模組都已自動定義

**Pinia 其他優點：**

- 支援 Devtools
- Hot module replacement（HMR）
- 支援 Plugins
- 支援 Server Side Render

---

## **套件安裝**

搭配 Nuxt 整合模組 `@pinia/nuxt` 進行安裝：

```bash
npm install pinia @pinia/nuxt
```

{% colorquote info %}
使用 npm 安裝時如果報錯 `ERESOLVE could not resolve`，在 `package.json` 加入以下內容，重新安裝即可

```json
// package.json
"overrides": {
  "vue": "latest"
}
```
{% endcolorquote %}

---

## **nuxt.config 配置**

如果不想全域引入 `defineStore`，也可以在頁面各別引入：`import { defineStore } from 'pinia'`

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  modules: [
    '@pinia/nuxt'
  ],
  pinia: { // 想要各別引入可以移除這段
    autoImports: [
      'defineStore'
    ]
  }
})
```

---

## **定義 Store**

在 store 資料夾建立檔案，範例 `store/index.js`

### **Option Store**

先以 Option Stores 進行說明，比較直觀好理解

- **state：**功能同 data
- **getters：**功能同 computed
- **actions：**功能同 methods

```jsx
// store/index.js
export const useMainStore = defineStore('main', {
  state: () => ({
    counter: 0
  }),
  getters: {
    doubleCounter() {
      return this.counter * 2;
    }
  },
  actions: {
    increment() {
      this.counter++
    }
  }
})
```

### **Setup Store**

接著將以上內容改寫為 Composition API

- `ref()`：用來定義 state 屬性
- `computed()`：定義 getters
- `function()`：定義 actions

```jsx
// store/index.js
export const useMainStore = defineStore('main', () => {
  const counter = ref(0);
  const doubleCounter = computed(() => counter.value * 2)
  const increment = () => {
    counter.value++
  }

  return {
    counter,
    doubleCounter,
    increment
  }
})
```

---

## **使用 Store**

```jsx
// pages/count.vue
<template>
  <div>
    {{ mainStore.counter }}
    {{ mainStore.doubleCounter }}
    <button @click="mainStore.increment()">increment</button>
  </div>
</template>

<script setup>
import { useMainStore } from '@/store/index';
const mainStore = useMainStore();
</script>
```

### **使用 Plugins Provide 全域注入 Store**

```jsx
// plugins/pinia.js
import { useMainStore } from '@/store';

export default defineNuxtPlugin(({ $pinia }) => {
  return {
    provide: {
      store: useMainStore($pinia)
    }
  };
});
```

使用 `$store` 取得狀態

```jsx
// pages/count.vue
<template>
  <div>
    {{ $store.counter }}
    {{ $store.doubleCounter }}
    <button @click="$store.increment()">increment</button>
  </div>
</template>

<script setup>
const { $store } = useNuxtApp();
</script>
```

---

## **還原狀態**

### **Option Store**

使用 `$reset()` 還原狀態 

```jsx
// pages/index.vue
<template>
  <div>
    {{ $store.counter }}
    <button @click="reset()">reset</button>
  </div>
</template>

<script setup>
const { $store } = useNuxtApp();
const reset = () => {
  $store.$reset();
};
</script>
```

### **Setup Store**

必須自訂 reset 方法

```jsx
// store/index.js
export const useMainStore = defineStore('main', () => {
  const counter = ref(0);

  const reset = () => {
    counter.value = 0;
  }

  return {
    counter,
    reset
  }
})
```

---

參考資源：

https://nuxt.com/docs/getting-started/state-management
https://pinia.vuejs.org/ssr/nuxt.html#nuxt-js
https://nuxt.com/docs/migration/configuration#vuex
[https://blog.twjoin.com/vue-state-management-介紹-pinia-9f8695110cd7](https://blog.twjoin.com/vue-state-management-%E4%BB%8B%E7%B4%B9-pinia-9f8695110cd7)