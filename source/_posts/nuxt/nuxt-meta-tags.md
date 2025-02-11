---
title: Nuxt.js Meta Tags and SEO
date: 2022-12-10
tags: [ nuxt2, seo ]
category: Nuxt
description: 說明如何在 Nuxt.js 各頁面定義不同的 meta tags，讓搜尋引擎爬蟲掌握頁面內容，提升 SEO
image: https://imgur.com/ZQejyJJ.png
---
> **版本：nuxt 2.15.8**
>

<div style="display: flex; justify-content: center; margin: 0;">
    <img style="width: 100%; max-width: 700px;" src="https://imgur.com/ZQejyJJ.png">
</div>

在 Vue.js 專案，我們可以直接在 `index.html` `<head>` 設定全站共用的 meta tags，那在 Nuxt 又該怎麼設定呢？

Nuxt.js 藉由 [vue-meta](https://vue-meta.nuxtjs.org/api/#link) 來更新應用內的 head 設定跟 meta 屬性，使用 `create-nuxt-app` 建構專案會自動安裝，我們可以在頁面定義不同的內容，幫助搜尋引擎爬蟲掌握頁面內容，提升 SEO。

<!-- more -->

### **全域配置**

```jsx
// nuxt.config.js
export default {
    head: {
        title: 'my website title',
        meta: [
            { charset: 'utf-8' },
            { name: 'viewport', content: 'width=device-width, initial-scale=1' },
            {
                hid: 'description',
                name: 'description',
                content: 'my website description'
            },
            {
                hid: 'og:title',
                property: 'og:title',
                content: 'my website title'
            }
        ],
        link: [
            { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }
        ]
    }
}
```

我們可以看到 head 自動渲染：

```html
<head>
    <title>my website title</title>
    <meta data-n-head="ssr" charset="utf-8">
</head>
```

{% colorquote info %}
為了避免元件內 **局部配置** head 造成 meta tags 重複，需加上 `hid` key，並賦予唯一值，這樣當元件定義相同 `hid` 的 meta 標籤，`vue-meta` 會自動覆蓋掉 **全域配置**
{% endcolorquote %}

### **局部配置**

以物件定義：

```jsx
// pages/about.vue
export default {
    head: {
        title: 'about-us',
        meta: [
            {
                hid: 'description',
                name: 'description',
                content: 'about-us'
            }
        ]
    }
}
```

如果需要透過 `this` 來取得 meta 資料，需要調整為 **function return**：

```jsx
// pages/about.vue
export default {
    data() {
        return {
            seo: {
                title: 'about-us',
                description: 'about-us'
            }
        }
    },
    head() {
        return {
            title: this.seo.title,
            meta: [
                {
                    hid: 'description',
                    name: 'description',
                    content: this.seo.description
                }
            ]
        }
    }
}
```

---

參考文章：

[https://nuxtjs.org/docs/features/meta-tags-seo/](https://nuxtjs.org/docs/features/meta-tags-seo/)