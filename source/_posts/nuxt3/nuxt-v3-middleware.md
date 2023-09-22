---
title: Nuxt.js 3.x Middleware 目錄－監聽路由變化
date: 2023-09-05
tags: [ nuxt3 ]
category: Nuxt3
description: Middleware 為 Nuxt 內的路由守衛（Navigation Guards），相當於 Vue Router 內的 beforeEach callback，協助我們在進到頁面前進行事件處理（例如權限檢查）
image: https://imgur.com/5VRbaV1.png
---

Middleware 為 Nuxt 內的 **路由守衛（Navigation Guards）**，相當於 Vue Router 內的 beforeEach callback，協助我們在進到頁面前進行事件處理（例如權限檢查）

### **Middleware 觸發時機：**

- **頁面初始化：**Server Side 跟 Client Side 同時觸發（觸發兩次）
- **頁面切換：**Client Side 觸發

### **Middleware 定義方式：**

- **具名：**在 `middleware/` 定義，並在需要的頁面引入
- **全域：**同具名的定義方式，不過檔名需加上 `.global` 後綴，在所有頁面切換時自動執行
- **匿名：**直接在單一元件檔內定義

<!-- more -->

## **具名 Middleware**

**範例：**

```
middleware/
|—— auth.js
```

- `defineNuxtRouteMiddleware()` 輔助函示：定義 route middleware
- `navigateTo()` 輔助函示：導航到其他頁面
- `abortNavigation()` 輔助函示：中斷導航，顯示錯誤資訊以及跳到錯誤頁面

```jsx
// middleware/auth.js
export default defineNuxtRouteMiddleware((to, from) => {
    const isLoggedIn = true; // 判斷是否登入
    const hasPermission = false; // 判斷有無頁面權限

    if (!isLoggedIn) {
        return navigateTo('/login');
    }

    if (!hasPermission) {
        return abortNavigation({
            statusCode: 403
            statusMessage: '無頁面權限'
        });
    }
});
```

在頁面透過 `definePageMeta()` 輔助函式定義 middleware

```jsx
// pages/about.vue
<template>
    <div>...</div>
</template>

<script setup>
definePageMeta({
    middleware: [ auth ] // or middleware: 'auth'
});
</script>
```

接下來當我們切到 `http:localhost:3000/about`，驗證無頁面權限跳轉至錯誤畫面

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/XZLxB4e.png">
</div>

---

## **全域 Middleware**

全域引入的 Middleware。同具名定義方式，不過檔名需加上 `.global` 後綴，在所有頁面切換時自動執行

**範例：**

```
middleware/
|—— auth.global.js
```

---

## **匿名 Middleware**

在頁面內各別定義

```jsx
// pages/about.vue
<template>
    <div>...</div>
</template>

<script setup>
definePageMeta({
    middleware: [
        (to, from) => {
            // 這裡是匿名 middleware 內容
        }
    ]
});
</script>
```

---

## **Middleware 執行順序**

執行順序首先為全域 Middleware，接下來才是頁面定義的 Middleware（依照順序）

**範例：**

```jsx
middleware/
|—— auth.global.js
|—— redirect.js
```

```jsx
// pages/about.vue
<template>
    <div>...</div>
</template>

<script setup>
definePageMeta({
	middleware: [
        (to, from) => {
            // ...
        },
        'redirect'
    ]
});
</script> 
```

執行順序分別為：

1. `auth.global.js`
2. 自訂匿名 middleware
3. `redirect.js`

---

## **動態加入 Middleware**

使用 `addRouteMiddleware(name, middleware, options)` 輔助函式動態加入 Middleware

**範例：**在 `plugins/` 加入 middleware

### **具名**

會覆蓋掉定義在 `middleware/` 資料夾內同名檔案（auth.js）

```jsx
// plugins/add-middleware.js
export default defineNuxtPlugin(() => {
    addRouteMiddleware('auth', (to, from) => {
        // ...
    });
});
```

### **匿名**

```jsx
// plugins/add-middleware.js
export default defineNuxtPlugin(() => {
    addRouteMiddleware((to, from) => {
        // ...
    });
});
```

### **全域**

options 加上 `{ global: true }`

```jsx
// plugins/add-middleware.js
export default defineNuxtPlugin(() => {
    addRouteMiddleware('auth', (to, from) => {
            // ...
        },
        { global: true }
    );
});
```

---

參考資源：

https://nuxt.com/docs/guide/directory-structure/middleware
https://nuxt.com/docs/api/utils/define-nuxt-route-middleware