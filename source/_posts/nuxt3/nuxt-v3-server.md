---
title: Nuxt.js 3.x Server 目錄－建立 API
date: 2023-09-04
tags: [ nuxt3 ]
category: Nuxt3
description: Nuxt3 搭配了新的 Nitro 伺服器引擎，讓我們能輕鬆在 Nuxt 專內建立 Server API。Nitro 的優點包含跨平台支援 Node.js 與瀏覽器、支援 HMR、自動生成 API 路由等，讓 Nuxt 具備全端功能
image: https://imgur.com/9hA9lOg.png
---

Nuxt3 搭配了新的伺服器引擎 [Nitro](https://nitro.unjs.io/)，讓我們能輕鬆在 Nuxt 專內建立 Server API。Nitro 的優點包含跨平台支援 Node.js 與瀏覽器、支援 HMR、自動生成 API 路由等，讓 Nuxt 具備全端功能，接下來一起進行實作吧。

## **建立 API**

- 在 `server/` 目錄建立 API，Nuxt 會依據資料夾結構自動生成 API 路徑
- 使用 `defineEventHandler()` 建立事件處理器

<!-- more -->

範例檔案結構：

```
server/
|—— api/
|———— hello.js
|—— routes/
|———— hello.js
```

放置於 `/server/api` 下的檔案，依據檔案名稱產生 `/api` 前綴路徑（`/api/hello`），如果不想加上 `/api` 前綴，將檔案放置於 `/server/routes` 即可

{% colorquote info %}
不論副檔名為 `.js`、`.ts`，均依據檔案名稱產生 API 路徑
{% endcolorquote %}

**範例：**新增 `/api/hello` API

```jsx
// server/api/hello.js
export default defineEventHandler(() => 'Hello World!');
```

接著在瀏覽器開啟頁面 `http://localhost:3000/api/hello`

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/LtH9Hcq.png">
</div>

---

## **HTTP Methods**

Server API 預設請求方法為 `get`，如果要調整其他方法 `post`、`patch`、`delete`，加在檔名後綴即可

```
server/
|—— api/
|———— user.post.js
|———— user.delete.js
```

**範例：**新增 `/api/user` API，並使用 `post` 方法

{% colorquote info %}
Nitro 使用 [unjs/h3](https://github.com/unjs/h3) 建立 Server API，`readBody()` 為 [unjs/h3](https://github.com/unjs/h3) 提供的 utilites，用來取得 request body，其他 utilites 可以參考 [官方文件](https://github.com/unjs/h3#utilities)
{% endcolorquote %}


```jsx
// server/api/user.post.js
export default defineEventHandler(async event => {
    const body = await readBody(event);
    return { ...body };
});
```

在頁面使用 Nuxt3 `useFetch` 方法發出請求（[useFetch 參考文章](https://clairechang.tw/2023/07/19/nuxt3/nuxt-v3-data-fetching/)）

```jsx
// pages/index.vue
<template>
    <div>
        <div>name: {{ user.name }}</div>
        <div>age: {{ user.age }}</div>
    </div>
</template>

<script setup>
const { data: user } = useFetch('/api/user', {
    method: 'post',
    body: {
        name: 'Daniel',
        age: 18
    }
});
</script>
```

畫面如下：

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/8SHH2TG.png">
</div>

---

## **捕捉所有路由（Catch-all Route）**

透過檔名 `[…]` 來捕捉未被定義的 API 路徑（fallback route）

範例檔案結構：

```
server/
|—— api/
|———— hello.js
|———— [...].js
```

透過 `createError()` 方法來處理錯誤

```jsx
// server/api/[...].js
export default defineEventHandler(() => {
    throw createError({
        statusCode: 404,
        statusMessage: 'API Path Not Found'
    })
});
```

當我們向未定義的路由發出請求，例如 `/api/nothing`

```jsx
// pages/index.vue
<template>
    <div>
        error:
        <pre>{{ error.data }}</pre>
    </div>
</template>

<script setup>
const { error } = useFetch('/api/nothing');
</script>
```

顯示錯誤訊息如下

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/fnF0enq.png">
</div>

---

## **實作 API 請求**

以下範例搭配靜態資料說明，不會實際整合 database

### **1. 建立靜態資料**

首先在 `public/users.json` 建立資料（範例資料來源：[jsonplaceholder](https://jsonplaceholder.typicode.com/)）

```json
// public/users.json
[
    {
        "id": 1,
        "name": "Leanne Graham",
        "username": "Bret",
        "email": "Sincere@april.biz",
        "phone": "1-770-736-8031"
    },
    {
        "id": 2,
        "name": "Ervin Howell",
        "username": "Antonette",
        "email": "Shanna@melissa.tv",
        "phone": "010-692-6593"
    }
]
```

### **2. 建立 API**

新增 `server/api/user/[id].js`

- 使用方括號 `[]` 表示動態參數
- 使用 `getRouterParam()` 方法取得參數

```jsx
// server/api/user/[id].js
import users from '@/public/users.json';

export default defineEventHandler(event => {
    const id = getRouterParam(event, 'id');
    return users.find(user => user.id === parseInt(id)) || {};
});
```

### **3. 發出請求**

新增 `pages/user/[id].vue`

- 方括號 `[]` 表示動態路由
- 使用 `useRoute()` 方法取得路由參數
- 透過 `useFetch()` 方法發出請求

```jsx
// pages/user/[id].vue
<template>
    <div>
        <div>name: {{ user.name }}</div>
        <div>email: {{ user.email }}</div>
        <div>phone: {{ user.phone }}</div>
    </div>
</template>

<script setup>
    const route = useRoute();
    const { data: user } = useFetch(`/api/user/${route.params.id}`);
</script>
```

接下來在瀏覽器輸入 `http://localhost:3000/user/1`

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/9hA9lOg.png">
</div>

---

參考資源：

https://nuxt.com/docs/guide/concepts/server-engine
https://nuxt.com/docs/guide/directory-structure/server
https://masteringnuxt.com/blog/server-routes-in-nuxt-3