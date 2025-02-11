---
title: Nuxt.js Lazy Loading 圖片延遲載入
date: 2023-02-02 
tags: [ nuxt2 ]
category: Nuxt
description: Lazy Loading 是常見的圖片優化技巧，在使用大量圖片的網站尤其適用，可以提升網頁載入速度，節省流量的浪費，本文介紹如何搭配套件 lazysizes 使用
image: https://imgur.com/bbkpkJ6.png
---
> **版本：nuxt 2.15.8**
>

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 600px;" src="https://imgur.com/bbkpkJ6.png">
</div>

Lazy Loading 是常見的圖片優化技巧，在使用大量圖片的網站尤其適用，可以提升網頁載入速度，節省流量的浪費，Google Lighthouse 也顯示此為有效提升評分的方式，言下之意會影響到 Google 搜索排名，本文介紹如何搭配套件 [lazysizes](https://github.com/aFarkas/lazysizes) 使用。

<!-- more -->

## **引入 lazysizes 套件**

1. 執行 `npm i lazysizes`
2. 接著在 `plugins` 資料夾新增檔案 `lazysizes.js`

```jsx
// plugins/lazysizes.js
import lazySizes from 'lazysizes';
export default lazySizes;
```

1. nuxt.config.js 配置 plugin

```jsx
// nuxt.config.js
export default {
    plugins: [
        { src: '@/plugins/lazysizes', mode: 'client' }
    ],
    build: {
        extend(config, { _isDev, isClient, loaders: { vue } }) {
            if (isClient) {
                vue.transformAssetUrls.img = [ 'data-src', 'src' ];
            }
        }
    }
}
```

## **Lazy Loading 運用**

1. 將 `src` 屬性替換為 `data-src`
2. 加上 `lazyload` class name
3. `src` 建議加上預設圖片，避免在頁面載入時產生破圖

```jsx
<template>
    <div>
        <img data-src="https://.../image.png" class="lazyload" src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mNkYAAAAAYAAjCB0C8AAAAASUVORK5CYII=" />
    </div>
</template>
```

**lazy loading 運作原理如下：**

1. `<img>` 使用 `data-src` 屬性代入要加載的圖片路徑，瀏覽器無法辨識該屬性因此無法載入
2. 監聽圖片 dom 元素是否進入瀏覽器可視區
3. 當 dom 元素進入可視範圍，將 `data-src` 的值複製到 `src` 內，進行圖片載入

也可以試著包裝成共用元件

```jsx
// components/TheLazyLoadImage.vue
<template>
    <img :data-src="src" :src="defaultImage" class="lazyload" :alt="alt" />
</template>

<script>
export default {
    name: 'LazyLoadImage',
    props: {
        src: {
            type: String,
            required: true,
            default: ''
        },
        alt: {
            type: String,
            required: true,
            default: ''
        },
        defaultImage: {
            type: String,
            default: 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mNkYAAAAAYAAjCB0C8AAAAASUVORK5CYII='
        }
    }
};
</script>
```

接著在頁面使用

```jsx
<template>
    <TheLazyLoadImage src="https://.../image.png" alt="example image" />
</template>

<script>
import TheLazyLoadImage from '@/components/TheLazyLoadImage';

export default {
    name: 'LazyLoadImage',
    components: {
        TheLazyLoadImage
    }
};
</script>
```

看一下編譯後的結果

```jsx
<img class="lazyloaded" data-src="https://.../image.png" src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mNkYAAAAAYAAjCB0C8AAAAASUVORK5CYII="
    alt="example image" />
```

當 `<img>` 進入可視範圍後，顯示如下

```jsx
<img class="lazyloaded" data-src="https://.../image.png" src="https://.../image.png"
    alt="example image" />
```

範例程式碼

<iframe height="470" style="width: 100%;" scrolling="no" title="Vue.js Lazy Loading Images" src="https://codepen.io/claire-chang-the-bashful/embed/YzjdPwv?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/claire-chang-the-bashful/pen/YzjdPwv">
  Vue.js Lazy Loading Images</a> by Claire Chang (<a href="https://codepen.io/claire-chang-the-bashful">@claire-chang-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

---

參考文章：

[https://medium.com/@dannjb/a-lazy-loading-image-component-for-nuxt-js-c34e0909e6e1](https://medium.com/@dannjb/a-lazy-loading-image-component-for-nuxt-js-c34e0909e6e1)

[https://medium.com/@mingjunlu/lazy-loading-images-via-the-intersection-observer-api-72da50a884b7](https://medium.com/@mingjunlu/lazy-loading-images-via-the-intersection-observer-api-72da50a884b7)