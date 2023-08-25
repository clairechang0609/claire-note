---
title: Nuxt.js 3.x Composables vs Utils 目錄－自訂共用方法
date: 2023-07-11
tags: [ nuxt3 ]
category: Nuxt3
description: 本篇說明 Nuxt3 Plugins 目錄功能，包含擴充插件、搭配 Composables、Provide 定義全局變數以及 Directive 自訂指令
image: https://imgur.com/5VRbaV1.png
---

## **Composables & Utils 使用時機**

定義在 `composables` 以及 `utils` 資料夾的檔案 Nuxt 都會自動引入

- **Composibles：**組合式函式，利用 Composition API 來封裝和複用 **有狀態邏輯（Stateful Logic）**的函式，取代 Options API `mixins` 的功能。我們可以將不同的邏輯抽象成單獨的 composable，並組合在 setup 函式中
- **Utils：**跟 composibles 做語意上的區隔，用來定義 **無狀態邏輯（Stateless Logic）**的函式

{% colorquote info %}
**無狀態邏輯（Stateless Logic）函式：**例如計算兩個數字相加的函式。當輸入值後，經過函式運算返回值，不會受其他狀態影響。無論何時輸入相同的值，都可以得到相同結果，大部分封裝共用方法屬於此類
**有狀態邏輯（Stateful Logic）函式：**例如計數器函式，函式會控制變數。觸發函式時，狀態就會改變，每次觸發的結果不會相同
{% endcolorquote %}

<!-- more -->

## **Composables**

#### **Options API Mixins & Composition API Composables**

當專案邏輯越來越複雜，使用多個 `mixin` 時，會發生不易找到參數、方法來源，或是命名衝突的問題，且 Options API 是依照**「生命週期與性質」**拆分程式碼，程式碼可讀性較低；Composition API 則是根據**「邏輯功能」**拆分程式碼，尤其適用於高複雜和多功能的組件

以下範例來看，Composition API `Composables` 相同邏輯功能放置同一區塊，可讀性較高

<div class="column-wrap">
    <div>
        <strong style="margin-bottom: 0.5rem; display: block;">Mixins</strong>
        <div style="display: flex; justify-content: left;">
            <img style="width: 100%; max-width: 330px;" src="https://imgur.com/ypkIwCW.png">
        </div>
    </div>
    <div>
        <strong style="margin-bottom: 0.5rem; display: block;">Composables</strong>
        <div style="display: flex; justify-content: left;">
            <img style="width: 100%; max-width: 330px;" src="https://imgur.com/bu9syKB.png">
        </div>
    </div>
</div>

#### **用法一：具名匯出**

具名匯出會讀取 **函式名稱**

```jsx
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

使用組合函式（自動匯入）

```html
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
export const useCount = () => {
    const { x, y } = usePosition();
    const { $hello } = useNuxtApp();
    ...
}
```

---

## **Utils**

名稱定義方式同 Composables

#### **用法一：具名匯出**

具名匯出會讀取 **函式名稱**

```jsx
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
export default num => {
    if (!num) {
        return num;
    }
    const parts = num.toString().split('.');
    parts[0] = parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, ',');
    return parts.join('.');
};
```

使用 utils（自動匯入）

```jsx
<template>
    <div>
        $ {{ {{ toThousands(19999) }} }}
    </div>
</template>
```

執行結果： $ 19,999

---

參考資源：

[https://nuxt.com/docs/guide/directory-structure](https://nuxt.com/docs/guide/directory-structure/utils)
https://ithelp.ithome.com.tw/articles/10298186