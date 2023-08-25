---
title: Nuxt.js 實作 301 轉址（301 Redirect）
date: 2022-12-18
tags: [ nuxt2 ]
category: Nuxt
description: 301 轉址又稱為重新導向，將指定網址導向另一網址，在網站結構更新、網站搬遷時常會使用到，當用戶輸入舊網址時，會自動跳轉到新網址。如果需轉址的網頁已經經營許久，如果沒有設定轉址，過去累積的頁面權重和流量不會跟著轉移，新的網站必須重新累積，且搜尋引擎可能會出現重複內容，將會影響 SEO 排名
image: https://imgur.com/cWsN68u.png
---
> **版本：nuxt 2.15.8**
>

301 轉址又稱為重新導向，將指定網址導向另一網址，在網站結構更新、網站搬遷時常會使用到，當用戶輸入舊網址時，會自動跳轉到新網址。

### **轉址跟 SEO 的關係**

如果需轉址的網頁已經經營許久，如果沒有設定轉址，過去累積的頁面權重和流量不會跟著轉移，新的網站必須重新累積，且搜尋引擎可能會出現重複內容，將會影響 SEO 排名。

<!-- more -->

### **301 轉址跟 302 轉址差別**

如果只是暫時性網站維護，之後會重新導回舊有網站，需使用 302 暫時轉址；如果是永久更換，則使用 301 轉址。

### **實作 301 轉址**

當網站改版，網址的結構異動，我們可以這樣做：

**舊網頁：** `https://example.com/events/summer-sale`

**新網頁：** `https://new-example.com/special/summer-sale`

| 舊網頁 | 新網頁 |
| --- | --- |
| /events/event-title | /special/event-title |
| /events/treatment-title | /treatment/treatment-title |
| /events/product-title | /product/product-title |

新網頁是當前已開發的頁面，就不多做說明，至於要怎麼轉導舊網頁，我們可以透過 `middleware`，首先新增對應路由的頁面：`pages/events/_id.vue` (由於舊路由皆為 events 底下，統一建立動態路由)，接著進行設定：

```jsx
// pages/events/_id.vue
<template>
    <div></div>
</template>

<script>
export default {
    name: 'EventsUrlRedirection',
    middleware({ route, redirect, error }) {
        const redirectArr = {
            '/events/event-title': '/special/event-title',
            '/events/treatment-title': '/treatment/treatment-title',
            '/events/product-title': '/product/product-title'
        };
        if (redirectArr[route.path]) {
            redirect(301, redirectArr[route.path]);
        } else {
            // 當頁面不存在，導向 404 error page
            error({ statusCode: 404 });
        }
    }
};
</script>
```

{% colorquote info %}
redirect 預設 status code `302` ，使用預設值直接寫 `redirect(path)` 就可以了
{% endcolorquote %}

`middleware` 也可獨立拆分，外層新增 middleware 資料夾，新增檔案 urlRedirection.js

```jsx
// middleware/urlRedirection.js
export default ({ route, redirect, error }) => {
    const redirectArr = {
        '/events/event-title': '/special/event-title',
        '/events/treatment-title': '/treatment/treatment-title',
        '/events/product-title': '/product/product-title'
    };
    if (redirectArr[route.path]) {
        redirect(301, redirectArr[route.path]);
    } else {
        error({ statusCode: 404 });
    }
};
```

接著回到 pages/events/_id.vue：

```jsx
// pages/events/_id.vue
<template>
    <div></div>
</template>

<script>
export default {
    name: 'EventsUrlRedirection',
    middleware: 'urlRedirection'
};
</script>
```

這樣就完成了～

---

參考文章：

[https://welly.tw/serp-rank-optimization/301-and-302-redirection-guide-for-seo](https://welly.tw/serp-rank-optimization/301-and-302-redirection-guide-for-seo)

[https://nuxtjs.org/docs/internals-glossary/context/#redirect](https://nuxtjs.org/docs/internals-glossary/context/#redirect)