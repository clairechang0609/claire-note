---
title: Nuxt.js 3.x Pages 目錄－自動生成路由
date: 2023-08-18
tags: [ nuxt3 ]
category: Nuxt3
description: Nuxt 提供基於 Pages 的路由設定，當我們在 Pages 資料夾建立檔案，Nuxt 會根據資料夾以及檔案結構自動生成基於 Vue Router 的路由，讓我們能更有效率進行頁面的設定和管理
image: https://imgur.com/5VRbaV1.png
---

Pages 資料夾讓我們可以自由新增頁面。Nuxt 提供基於 Pages 的路由設定，當我們在 Pages 資料夾建立檔案，Nuxt 會根據資料夾以及檔案結構自動生成基於 [Vue Router](https://router.vuejs.org/) 的路由，讓我們能更有效率進行頁面的設定和管理

{% colorquote info %}
Pages 目錄是**可選擇的**，使用 nuxi 建立專案預設並無此資料夾，只有 `app.vue` 單一頁面，在此情況下 Vue Router 不會被載入
{% endcolorquote %}

<!-- more -->

## **app.vue**

專案進入點，Nuxt3 將 `app.vue` 移到目錄頁，我們可以只使用 `app.vue` 來建置網站（例如單頁 Landing Page），而不定義 `pages` 資料夾。

如果要使用 `pages`，`app.vue` 需加上 `<NuxtPage>` 用於顯示定義的頁面（功能同 Vue.js 專案 的 `<router-view>`）

```jsx
// app.vue
<template>
    <div>
        <NuxtPage />
    </div>
</template>
```

---

## **新增 Pages**

首先建立首頁 `pages/index.vue`

```jsx
// pages/index.vue
<template>
    <div>
        <h1>Home Page</h1>
    </div>
</template>
```

專案編譯後切到根目錄，看到畫面示意如下：

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/OHH4Tqy.png">
</div>

自動產生的 Routes 結構：

```jsx
{
    "routes": [
        {
            "path": "/",
            "component": "pages/index.vue"
        }
    ]
}
```

{% colorquote info %}
一個頁面只能存在一個 **根元素（root element）**，路由才能正常在頁面間切換（html 註解也視為一個元素）
以下為錯誤示範：

```jsx
<template>
    <!-- 這也視為元素，頁面無法正常渲染 -->
    <h1>Home page</h1>
</template>
```

```jsx
<template>
    <h1>Home page</h1>
    <p>兩個元素，頁面無法正常渲染</p>
</template>
```
{% endcolorquote %}

---

## **動態路由**

Nuxt2 使用下底線 `_` 產生動態路由，Nuxt3 調整為使用方括號 `[]`，範例：

```
pages/
|—— index.vue
|—— product-[category]/
|———— [id].vue
```

透過 `$route` 或是 `useRoute` Composable 取得路由參數（[route 參數選項](https://nuxt.com/docs/api/composables/use-route#useroute)）

```jsx
// pages/product-[category]/[id].vue
<template>
    <div>
        <h3>{{ $route.params.category }}</h3>
        <p>{{ $route.params.id }}</p>
    </div>
</template>

<script setup>
const route = useRoute();
console.log(route.params.category);
console.log(route.params.id);
</script>
```

當我們切到頁面 `/product-apple/112345`，畫面渲染如下

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/2XYhCnC.png">
</div>

---

## **匹配路徑下所有路由**

透過 `[…slug].vue` 將動態路由解構，來捕捉在此路徑下未被定義的頁面，範例：

```
pages/
|—— index.vue
|—— [...slug].vue
|—— about.vue
```

```jsx
// pages/[…slug].vue
<template>
    <div>
        <h3>Page Not Found</h3>
        <p>{{ $route.params.slug }}</p>
    </div>
</template>
```

切到未定義的頁面 `/hello/world`，畫面渲染如下

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/QvLn5ZK.png">
</div>

透過 `[…slug].vue`，我們可以簡單地捕捉特定路徑下不存在的頁面，全域的錯誤頁面（不只捕捉 404 錯誤），則由 `app.vue` 同層的 `error.vue` 處理

---

## **嵌套路由**

顧名思義就是在一個路由中嵌套另一個路由。在介面中建立複雜的層次結構，假設頁面結構如下，子頁面共享父層畫面：

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/Z15wuI1.png">
</div>

檔案結構（parent 資料夾與 parent.vue 命名必須相同）：

```
pages/
|—— parent/
|———— index.vue
|———— child-a.vue
|———— child-b.vue
|—— parent.vue
```

自動產生的 Routes 結構：

```jsx
{
    "routes": [
        {
            path: '/parent',
            component: '~/pages/parent.vue',
            name: 'parent',
            children: [
                {
                    path: 'child',
                    component: '~/pages/parent/child-a.vue',
                    name: 'parent-child-a'
                },
                        {
                    path: 'child',
                    component: '~/pages/parent/child-b.vue',
                    name: 'parent-child-b'
                }
            ]
        }
    ]
}
```

`parent.vue` 必須加上 `<NuxtPage>`，用來顯示切換 `child` 內容

```jsx
// pages/parent.vue
<template>
    <div>
        // 共用 sidebar
        <div class="sidebar">
            <NuxtLink to="/parent/child-a">child-a</NuxtLink>
            <NuxtLink to="/parent/child-b">child-b</NuxtLink>
        </div>
        // 用來切換子頁面內容
        <NuxtPage />
    </div>
</template>
```

在 `child-a.vue` 和 `child-b.vue` 都可以共享 sidebar 介面，`parent/index.vue` 用來定義 `/parent` 路徑畫面，如果不需要 `/parent`，可以透過 Nuxt 內建 `navigateTo()` 方法，轉導到子頁面

```jsx
// pages/parent/index.vue
<template>
    <div></div>
</template>

<script setup>
navigateTo({
    path: '/parent/child-a'
});
</script>
```

---

## **頁面切換**

透過 `<NuxtLink>` 元件進行切換，Nuxt3 `<NuxtLink>` 整合了 Vue Router `<RouterLink>` 跟 HTML `<a>` tag，能夠智能判斷內部或外部連結並加以優化（像是 prefetching）

```jsx
<template>
    <div>
        // 內部連結
        <NuxtLink to="/hello">Internal</NuxtLink>
        // 外部連結
        <NuxtLink to="https://mywebsite.com">External</NuxtLink>
    </div>
</template>
```

編譯後結果

```html
// 內部連結
<a href="/hello">Internal</a>
// 外部連結
<a href="https://mywebsite.com" rel="noopener noreferrer">External</a>
```

也可以透過 `props` 自定義屬性（[屬性選項](https://nuxt.com/docs/api/components/nuxt-link#props)）

```jsx
<template>
    <div>
        <NuxtLink to="https://mywebsite.com" target="_blank" external="false" no-rel>
            External
        </NuxtLink>
    </div>
</template>
```

---

參考資源：

https://nuxt.com/docs/guide/directory-structure/pages
https://nuxt.com/docs/getting-started/routing
[https://medium.com/unalai/認識-vue-router-嵌套路由-nested-routes-8168f5395941](https://medium.com/unalai/%E8%AA%8D%E8%AD%98-vue-router-%E5%B5%8C%E5%A5%97%E8%B7%AF%E7%94%B1-nested-routes-8168f5395941)