---
title: Nuxt.js 套件應用：Swiper 製作輪播動畫
date: 2022-11-29
tags: [ nuxt2, swiper ]
category: Nuxt
description: 網站開發常使用到輪播功能，Swiper 是一款基於 js 開發、功能完整實用性高的輪播套件，本篇將介紹如何在 Nuxt.js 專案內實作輪播動畫
image: https://i.imgur.com/fCx3gAS.png
---
> **版本：nuxt 2.15.8**
>

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="https://i.imgur.com/fCx3gAS.png">
</div>

網站開發常使用到輪播功能，[Swiper](https://www.npmjs.com/package/swiper) 是一款基於 js 開發、功能完整實用性高的輪播套件，本篇將介紹如何在 Nuxt.js 專案內搭配 Vue 整合套件 [vue-awesome-swiper](https://www.npmjs.com/package/vue-awesome-swiper) 實作輪播動畫

<!-- more -->

**套件安裝：**

`npm i swiper@6.8.4`

`npm i vue-awesome-swiper@4.1.1`

{% colorquote info %}
官方文件版本說明：Vue2 搭配 Swiper 5-6 ＆ vue-awesome-swiper@4.1.1
如果是 Vue3 此套件已不支援，需改用 [Swiper Vue.js Components](https://swiperjs.com/vue)
{% endcolorquote %}

接著全域註冊 Swiper 元件，在 plugins 資料夾新增檔案 swiper.js（檔名自訂）

```jsx
// plugins/swiper.js
import Vue from 'vue';
import Swiper, { /* swiper modules... */ } from 'swiper';
import VueAwesomeSwiper from 'vue-awesome-swiper';

Swiper.use([ /* swiper modules... */ ]);

export default () => {
    Vue.use(VueAwesomeSwiper);
};
```

上述 **swiper modules** 區塊可以替換需要的模組（[模組選項](https://swiperjs.com/swiper-api#using-js-modules)），範例使用 **Navigation, Pagination**

在 nuxt.config.js 進行配置

```jsx
// nuxt.config.js
export default {
    css: [
        { src: 'swiper/swiper-bundle.min.css' }
    ],
    plugins: [
        { src: '@/plugins/swiper', mode: 'client' }
    ]
}
```

接下來可以透過兩種方式使用：

### **swiper components 元件**

```jsx
<template>
    <client-only>
        <div>
            <swiper class="swiper" :options="swiperOption" ref="bannerSwiper">
                <swiper-slide>banner 1</swiper-slide>
                <swiper-slide>banner 2</swiper-slide>
                <swiper-slide>banner 3</swiper-slide>
                <swiper-slide>banner 4</swiper-slide>
                <swiper-slide>banner 5</swiper-slide>
            </swiper>
            <div class="swiper-button-prev" slot="button-prev"></div>
            <div class="swiper-button-next" slot="button-next"></div>
        </div>
    </client-only>
</template>

<script>
export default {
    data() {
        return {
            swiperOption: {
                slidesPerView: 3,
                spaceBetween: 5,
                grabCursor: true
            }
        }
    }
};
</script>
```

swiper 設定項目請見[文件](https://swiperjs.com/swiper-api)，如果專案為 server-side render，使用此方式外層需加上 `<client-only />` ，否則在 server 端會因為 dom 元素無法被解析而報錯

{% colorquote danger %}
[Vue warn]: The client-side rendered virtual DOM tree is not matching server-rendered content. This is likely caused by incorrect HTML markup, for example nesting block-level elements inside `<p>`, or missing `<tbody>`. Bailing hydration and performing full client-side render.
{% endcolorquote %}

我們可以使用 `this.$refs.bannerSwiper.$swiper` 來取得 Swiper 實體並使用[相關方法](https://swiperjs.com/swiper-api#methods-and-properties)

根據[文件說明](https://nuxtjs.org/docs/features/nuxt-components/#the-client-only-component)，`<client-only />` 的元素在 mounted 生命週期才渲染完畢，因此在 mounted 呼叫 `$refs` 可能會取不到內容（即便使用 `$nextTick`），需將方法包裝成迴圈：

```jsx
export default {
    mounted(){
        this.getSwiperInstance()
    },
    methods: {
        getSwiperInstance() {
            this.$nextTick(() => {
                if (this.$refs.bannerSwiper) {
                    this.$refs.bannerSwiper.$swiper.slideTo(2);
                } else {
                    this.getSwiperInstance();
                }
            });
        }
    }
}
```

### **directive 指令**

若 Swiper 內容需被搜尋引擎爬蟲讀取，可以利用 [Vue directive](https://vuejs.org/guide/reusability/custom-directives.html#introduction) 方法，使用能被瀏覽器解析的 Dom 元素包裝 Swiper，見以下範例：

```jsx
<template>
    <div class="banner-wrap mx-auto py-4 px-3">
        <div v-swiper:bannerSwiper="swiperOption" class="swiper">
            <div class="swiper-wrapper">
                <div class="swiper-slide">banner 1</div>
                <div class="swiper-slide">banner 2</div>
                <div class="swiper-slide">banner 3</div>
                <div class="swiper-slide">banner 4</div>
                <div class="swiper-slide">banner 5</div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            swiperOption: {
                slidesPerView: 3,
                spaceBetween: 5,
                grabCursor: true
            }
        }
    },
    mounted() {
        this.bannerSwiper.slideTo(2, 500, false);
    }
};
</script>
```

如果同一個頁面有多個 Swipers，必須要透過命名（ ex: `v-swiper:bannerSwiper` ）來進行綁定，我們也可以透過名稱 `this.bannerSwiper` 的取得 Swiper 實體並使用相關方法，是不是簡單許多呢！

---

參考文章：

[https://github.com/surmon-china/surmon-china.github.io/blob/vue2/projects/vue-awesome-swiper/examples/00-typescript-composition-api.vue](https://github.com/surmon-china/surmon-china.github.io/blob/vue2/projects/vue-awesome-swiper/examples/00-typescript-composition-api.vue)

[https://nuxtjs.org/docs/features/nuxt-components/#the-client-only-component](https://nuxtjs.org/docs/features/nuxt-components/#the-client-only-component)

[https://ken551113.github.io/2019/11/26/Using-Vue-Awesome-Swiper-On-Nuxt/](https://ken551113.github.io/2019/11/26/Using-Vue-Awesome-Swiper-On-Nuxt/)