---
title: Nuxt.js 3.x 套件應用－Swiper 製作輪播動畫
date: 2023-08-16 18:00
tags: [ nuxt3 ]
category: Nuxt3
description: 網站開發常使用到輪播功能，Swiper 是一款基於 Javascript 開發、功能完整、實用性高的輪播套件，接下來說明在 Nuxt3 專案搭配 nuxt-swiper 實作
image: https://imgur.com/n2e3W3Q.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/articles/10335360)
>

> **nuxt-swiper 版本：v1.2.0**
>

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/n2e3W3Q.png">
</div>

網站開發常使用到輪播功能，[Swiper](https://www.npmjs.com/package/swiper) 是一款基於 Javascript 開發、功能完整、實用性高的輪播套件，接下來說明在 Nuxt3 專案搭配 [nuxt-swiper](https://github.com/cpreston321/nuxt-swiper#readme) 實作

<!-- more -->

{% colorquote info %}
Nuxt2 搭配 Swiper 請參考 [這篇文章](https://clairechang.tw/2022/11/29/nuxt/nuxt-swiper/)
{% endcolorquote %}

## **套件安裝**

使用 Nuxt 整合模組 `nuxt-swiper` 進行安裝（基於 Swiper Vue）

```bash
npm install nuxt-swiper
```

---

## **nuxt.config 配置**

**swiper 配置選項：**

- **prefix：**元件名稱前綴，預設為 Swiper
- **styleLang：**style 語言，預設為 css
- **modules：**引入的模組，預設為全部引入，模組選項參考 [官方文件](https://github.com/cpreston321/nuxt-swiper#module-options)

```jsx
// nuxt.config.js
export default defineNuxtConfig({
    modules: [
        'nuxt-swiper'
    ],
    swiper: {
        modules: ['navigation', 'pagination', 'effect-creative' ]
    }
})
```

---

## **建立輪播動畫**

在頁面上使用元件，範例頁面 pages/swiper.vue

- **自動引入元件：**`<Swiper>`、`<SwiperSlide>` （名稱依照前面設定的 prefix）
- **自動引入模組：**依照前面設定的 modules，這裡為 `<SwiperNavigation>` 以及 `<SwiperPagination>`
- **props 傳入屬性：**屬性選項參考 [Swiper 官方文件](https://swiperjs.com/swiper-api#parameters)

```jsx
// pages/swiper.vue
<template>
    <div>
        <Swiper
            :modules="[ SwiperNavigation, SwiperEffectCreative ]"
            :slides-per-view="1"
            :space-between="16"
            :loop="true">
            <SwiperSlide>Slide 1</SwiperSlide>
            <SwiperSlide>Slide 2</SwiperSlide>
            <SwiperSlide>Slide 3</SwiperSlide>
            <SwiperSlide>Slide 4</SwiperSlide>
        </Swiper>
    </div>
</template>
```

或是將屬性包裝成物件，透過 `v-bind` 傳入，配置選項較複雜時，能提升模板可讀性

```jsx
// pages/swiper.vue
<template>
    <div>
        <Swiper v-bind="swiperConfig">
            <SwiperSlide>Slide 1</SwiperSlide>
            <SwiperSlide>Slide 2</SwiperSlide>
            <SwiperSlide>Slide 3</SwiperSlide>
            <SwiperSlide>Slide 4</SwiperSlide>
        </Swiper>
    </div>
</template>

<script setup>
const swiperConfig = {
    modules: [
        SwiperNavigation,
        SwiperEffectCreative
    ],
    slidesPerView: 1,
    spaceBetween: 16,
    loop: true,
    breakpoints: {
        545: {
            slidesPerView: 2
        },
        1080: {
            slidesPerView: 3
        },
        1280: {
            slidesPerView: 4
        }
    }
};
</script>
```

---

## **Swiper 事件**

事件選項參考 [官方文件](https://swiperjs.com/swiper-api#events)

```jsx
// pages/swiper.vue
<template>
    <div>
        <Swiper @slide-change="slideChange()">
            ...
        </Swiper>
    </div>
</template>

<script setup>
const slideChange = () => {
    console.log('slideChange');
};
</script>
```

---

## **Swiper 樣式**

使用 Swiper 預設樣式（[樣式選擇](https://swiperjs.com/vue#styles)）

```jsx
// import Swiper 與模組樣式
import 'swiper/css';
import 'swiper/css/navigation';
import 'swiper/css/pagination';
```

---


## **useSwiper 實作控制按鈕**

使用 Swiper Vue `useSwiper` hook，來取得 Swiper 實體，呼叫相關方法（ex: `slidePrev()`）

```jsx
// components/SwiperController.vue
<template>
    <button @click="direction === 'prev' ? swiper.slidePrev() : swiper.slideNext()">
        {{ direction }}
    </button>
</template>

<script setup>
const props = defineProps({
    direction: {
        type: String,
        default: '',
        validator: value => {
            return [ 'prev', 'next' ].includes(value);
        }
    }
});

const swiper = useSwiper();
</script>
```

```jsx
// pages/swiper.vue
<template>
    <div>
        <Swiper>
            <SwiperSlide>Slide 1</SwiperSlide>
            <SwiperSlide>Slide 2</SwiperSlide>
            <SwiperSlide>Slide 3</SwiperSlide>
            <SwiperSlide>Slide 4</SwiperSlide>

            <SwiperController direction="prev" />
            <SwiperController direction="next" />
        </Swiper>
    </div>
</template>
```

---

參考資源：

https://github.com/cpreston321/nuxt-swiper#readme
https://swiperjs.com/vue