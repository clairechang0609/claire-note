---
title: Nuxt.js 套件應用：Masonry 打造瀑布流版面
date: 2022-12-23
tags: [ nuxt2, masonry ]
category: Nuxt
description: 瀑布流版面能夠快速的幫我們達成 RWD 區塊排版，本篇說明如何在 Nuxt.js 專案利用 masonry-layout 套件打造瀑布流版面
image: https://i.imgur.com/AFsr4mX.png
---
> **版本：nuxt 2.15.8**
>

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img style="width: 100%; max-width: 700px;" src="https://i.imgur.com/AFsr4mX.png">
</div>

前端網頁開發想必對瀑布流版面不陌生，能夠快速的幫我們達成 RWD 區塊排版，其中最有名的就是 [masonry-layout](https://masonry.desandro.com/) 這套 javascript 套件。

<!-- more -->

**步驟如下：**

### **1. 安裝套件**

`npm i masonry-layout`

### **2. 製作全域共用 mixins**

在 plugins 新增檔案 masonry.js，這裡使用原生 JS 寫法：`new Masonry(element, optoins)` ，options 選項參考 [官方文件](https://masonry.desandro.com/options.html)

```jsx
// plugins/masonry.js
import Vue from 'vue';
import Masonry from 'masonry-layout';

// 避免重複註冊
if (!Vue.__masonry_mixin__) {
    Vue.__masonry_mixin__ = true;
    Vue.mixin({
        methods: {
            masonry(element, optoins) {
                return new Masonry(element, optoins);
            }
        }
    });
}
```

{% colorquote info %}
全域 mixins 相當方便，但也很耗記憶體，因此必須加上判斷條件（EX：Vue.__masonry_mixin__），避免重複註冊 mixins
{% endcolorquote %}

### **3. nuxt.config.js 配置 plugins**

必須加上 `mode: 'client'` ，因套件有使用到 `window` 物件，在 server 端讀取不到變數會報錯：

> window is not defined
> 

```jsx
// nuxt.config.js
export default {
    plugins: [
        { src: '@/plugins/masonry', mode: 'client' }
    ]
}
```

### **4. 頁面應用**

接下來就可以在頁面上使用了，範例頁面 pages/index.vue

```jsx
// pages/index.vue
<template>
    <div class="masonry-wrap" ref="masonry">
        <div class="masonry-item" v-for="item in 12" :key="item" :class="`masonry-item-${item}`">
            <div class="card">
                {{ item }}
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'Home',
    mounted() {
        this.masonry(this.$refs.masonry, {
            columnWidth: 160,
            gutter: 20,
            itemSelector: '.masonry-item',
            transitionDuration: '0.3s',
            horizontalOrder: true
        });
    }
}
</script>

<style lang="scss" scoped>
    * {
        box-sizing: border-box;
    }
    .masonry-wrap {
        max-width: 1000px;
        padding: 50px;
    }
    .masonry-item {
        width: 160px;
        height: 100px;
        margin-bottom: 20px;
        .card {
            padding: 1rem;
            height: 100%;
            background: rgb(225, 225, 225);
            border-radius: 5px;
            border: 1px solid rgb(200, 200, 200);
        }
    }
    .masonry-item-1, .masonry-item-7, .masonry-item-12 {
        height: 200px;
    }
    .masonry-item-3, .masonry-item-9 {
        height: 150px;
    }
</style>
```

結果如下

<iframe height="470" style="width: 100%;" scrolling="no" title="Vue.js 2.x 套件應用：Masonry 打造瀑布流版面" src="https://codepen.io/claire-chang-the-bashful/embed/gOjGpKX?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/claire-chang-the-bashful/pen/gOjGpKX">
  Vue.js 2.x 套件應用：Masonry 打造瀑布流版面</a> by Claire Chang (<a href="https://codepen.io/claire-chang-the-bashful">@claire-chang-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

---

參考文章：

[https://masonry.desandro.com/](https://masonry.desandro.com/)

[https://medium.com/@jake101/using-masonry-js-in-nuxt-c4dbf8c17647](https://medium.com/@jake101/using-masonry-js-in-nuxt-c4dbf8c17647)