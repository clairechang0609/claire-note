---
title: Nuxt.js 3.x Composables vs Utils 目錄－自訂共用方法
date: 2023-07-11
tags: [ nuxt3 ]
category: Nuxt3
description: Nuxt3 Composables 與 Utils 目錄，協助我們自訂共用方法，本篇說明兩者的差異與使用時機
image: https://imgur.com/5VRbaV1.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/articles/10323220)
>

定義在 `composables` 以及 `utils` 資料夾的檔案 Nuxt 都會自動引入（auto-imports），兩者都是用來定義複用邏輯的函式，不過使用情境有點不同，接下來分別說明。

## **Composables & Utils 使用時機比較**

- `composables/`：
組合式函式，利用 Composition API 來封裝和複用 **有狀態邏輯（Stateful Logic）**的函式，取代 Options API `mixins/` 的功能。我們可以將不同的邏輯抽象成單獨的 composable，並組合在 setup 函式中
- `utils/`：
與 composables 做語意上的區隔，用來定義 **無狀態邏輯（Stateless Logic）**的函式

{% colorquote info %}
**狀態邏輯說明：**

- **無狀態邏輯函式：**
例如計算兩個數字相加的函式。當輸入值後，經過函式運算返回值，不會受其他狀態影響。無論何時輸入相同的值，都可以得到相同結果，大部分封裝共用方法屬於此類
- **有狀態邏輯函式：**
例如計數器函式，函式會控制變數。觸發函式時，狀態就會改變，每次觸發的結果不會相同
{% endcolorquote %}

<!-- more -->

## **Composables**

#### **Options API Mixins & Composition API Composables**

Nuxt3 的 Composition API `composables/` 與 Nuxt2 的 Options API `mixins/` 比較：

- `mixins/`：
當專案邏輯越來越複雜，使用多個 `mixins` 時，容易發生不易找到參數、方法來源，或是命名衝突的問題，且 Options API 是依照 **「生命週期與性質」** 拆分程式碼，結構上來說可讀性較低
- `composables/`：
Composition API 是依據 **「邏輯功能」** 拆分程式碼，可讀性較高，且不易造成命名衝突，適用於高複雜邏輯和多功能開發

<div class="column-wrap">
  <div style="display: flex; justify-content: left; margin: 20px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/2cFwG4a.png">
  </div>
  <div style="display: flex; justify-content: left; margin: 20px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/5dF4zYf.png">
  </div>
</div>

#### **用法一：具名匯出**

具名匯出會讀取 **函式名稱**

```jsx
// composables/test.js
export const useCount = () => {
  const count = ref(0);
  const add = () => {
    count.value++;
  };

  return {
    count,
    add
  };
}
```

#### **用法二：匿名匯出**

匿名匯出會讀取 **檔名**，以此為例命名為 `useCount.js` 或 `use-count.js`，組合式函式名稱同為 `useCount()`

```jsx
// composables/useCount.js or composables/use-count.js
export default function() {
  const count = ref(0);
  const add = () => {
    count.value++;
  };

  return {
    count,
    add
  };
}
```

頁面使用組合函式

```html
// pages/index.vue
<template>
  <div>
    {{ count }}
    <button type="button" @click="add">add</button>
  </div>
</template>

<script setup>
const { count, add } = useCount();
</script>
```

#### **延伸應用**

- 在 composable 內使用其它 composable
- 使用 plugin injections

```jsx
// composables/useCount.js
export const function() {
  const { x, y } = usePosition();
  const { $hello } = useNuxtApp();
  // ...
}
```

---

## **Utils**

名稱定義方式同 `composables/`

#### **用法一：具名匯出**

具名匯出會讀取 **函式名稱**

```jsx
// utils/test.js
export const toThousands = () => {
  if (!num) {
    return num;
  }
  const parts = num.toString().split('.');
  parts[0] = parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, ',');
  return parts.join('.');
}
```

#### **用法二：匿名匯出**

匿名匯出會讀取 **檔名**，以此為例命名為 `toThousands.js` 或 `to-thousands.js` 編譯後名稱同為 `toThousands()`

```jsx
// utils/toThousands.js or utils/to-thousands.js
export default num => {
  if (!num) {
    return num;
  }
  const parts = num.toString().split('.');
  parts[0] = parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, ',');
  return parts.join('.');
};
```

使用 Utils

```jsx
// pages/index.vue
<template>
  <div>
    $ {{ {{ toThousands(19999) }} }}
  </div>
</template>
```

執行結果： **$ 19,999**

---

參考資源：

https://nuxt.com/docs/guide/directory-structure/composables
https://nuxt.com/docs/guide/directory-structure/utils
https://ithelp.ithome.com.tw/articles/10279321