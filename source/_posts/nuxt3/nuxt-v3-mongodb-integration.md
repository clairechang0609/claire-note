---
title: Nuxt.js 3.x Server API 整合 MongoDB 實作，邁向全端第一步
date: 2023-09-22
tags: [ nuxt3, node.js, mongodb ]
category: Nuxt3
description: 本篇說明 Nuxt3 Plugins 目錄功能，包含擴充插件、搭配 Composables、Provide 定義全局變數以及 Directive 自訂指令
image: https://imgur.com/qhirT5r.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/articles/10328551)
>

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 500px;" src="https://imgur.com/qhirT5r.png">
</div>

Nuxt3 搭配了新的伺服器引擎 [Nitro](https://nitro.unjs.io/)，讓我們能輕鬆建立 Server API，並在 Nuxt3 專案內使用。Nitro 的優點包含跨平台支援 Node.js 與瀏覽器、支援 HMR、自動生成 API 路由等。

實務上 API 開發常會搭配資料庫連接，本篇將說明如何結合 MongoDB，實作後端 API 開發與前端串接，讓我們的 Nuxt 專案具備全端功能。

{% colorquote info %}
本文重點在 Nuxt 與 MongoDB 整合，需先具備以下知識：

- Nuxt Server API，可以參考 [這篇文章](https://clairechang.tw/2023/09/04/nuxt3/nuxt-v3-server/)
- Nuxt `$fetch` 方法，可以參考 [這篇文章](https://clairechang.tw/2023/07/19/nuxt3/nuxt-v3-data-fetching/)
- MongoDB 應用（本文僅簡單說明）
{% endcolorquote %}

<!-- more -->

---

開始前先簡單介紹一下 **MongoDB** 跟 **Mongoose**

### **MongoDB**

MongoDB 是一個 NoSQL 資料庫，以 BSON（Binary JSON）格式儲存，適合彈性和變化性高的資料儲存，一個 Database 是由一個或多個 Collection 組成，一個 Collection 則是由一個或多個 Document 組成，每一筆資料為一個 Document，裡面可以有多個欄位 Field

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 500px;" src="https://imgur.com/gITcY9e.png">
</div>

### **Mongoose**

Mongoose 是 MongoDB 的 ODM（Object Data Modeling）套件，在 Node.js 透過 MongoDB driver 跟資料庫溝通，執行 CRUD（新增、刪除、更新、查詢）操作

---

### **實作分為以下步驟**

1. MongoDB 前置環境準備
2. 安裝 Mongoose
3. 建立 Database
4. 透過 Mongoose 連接 Database
5. 建立 Model 與 Collection
6. 建立 Server API
7. 前端串接 API：實作新增

## **Step1：MongoDB 前置環境準備**

- [MongoDB 安裝](https://www.mongodb.com/try/download/community)
- [Compass 安裝](https://www.mongodb.com/try/download/compass)：mongoDB 的 GUI

## **Step2：安裝 Mongoose**

在專案內執行指令安裝

```bash
npm i -D mongoose
```

---

## **Step3：透過 Mongoose 連接 Database**

新增 `server/index.js`

- 透過 `mongoose.connect(URI)` 連接 database
- URI 格式：`mongodb://[username:password@][host][:port]/dbName`，`dbName` 資料庫名稱自訂

```jsx
// server/index.js
import mongoose from 'mongoose';

export default async () => {
  try {
    await mongoose.connect('mongodb://localhost:27017/nuxt_app');
    console.log('DB connection established');
  } catch (err) {
    console.error('DB connection failed', err);
  }
};
```

URI 通常會定義為 env 變數，在根目錄新增 `.env`

```
// .env
MONGODB_URI="mongodb://localhost:27017/nuxt_app"
```

透過 `process.env.MONGODB_URI` 取得變數

```jsx
mongoose.connect(process.env.MONGODB_URI);
```

將 `server/index.js` 註冊為 [Server Plugins](https://nitro.unjs.io/guide/plugins)，在伺服器初始化時執行資料庫連接

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  nitro: {
    plugins: [
      '@/server/index'
    ]
  }
});
```

---

## **Step4：建立 Model 與 Collection**

**範例：**

新增 `server/models/userModel.js` 定義模型

- 建立 Schema，透過 Schema 規範 Document 資料內容、型別、預設值等
- 建立 Model 模型，連接資料庫 Collection 與上一步的 Schema。當建立的模型資料庫沒有相對應 Collection，Mongoose 會自動建立

```jsx
// server/models/userModel.js
import mongoose from 'mongoose';

// 建立 Schema
const userSchema = new mongoose.Schema({
  name: {
    type: 'string',
    required: true
  },
  email: {
    type: 'string',
    required: true,
    unique: true
  }
}, {
  versionKey: false
});

// 建立 Model
const User = mongoose.model('User', userSchema);

export default User;
```

{% colorquote info %}
`mongoose.model()` 第一個參數為 Collection 名稱、第二個參數為 Schema。其中第一個餐褲 Collection 名稱為大寫開頭單數，會對應到資料庫內小寫開頭複數 Collection

```jsx
mongoose.model('User', userSchema);
```

對應到 Collection 名稱為 `users`
{% endcolorquote %}

執行編譯後，這時候開啟 Compass，應該可以看到 `nuxt_app` Database 跟 `users` Collection，接下來就可以對資料庫進行操作囉

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/mJrFHLK.png">
</div>

---

## **Step5：建立 Server API**

建立一支 POST API `server/api/user.post.js`

- 使用 [unjs/h3](https://github.com/unjs/h3) 提供 `readBody()` 輔助函式，取得 request body 內容
- 透過 Model 對 Collection 進行溝通與操作，使用 Ｍongoose `create()` 方法新增一筆資料

```jsx
// server/api/user.post.js
import User from '@/server/models/userModel';

export default defineEventHandler(async event => {
  try {
    const { name, email } = await readBody(event);
    await User.create({ name, email });
    const user = await User.findOne({ email });
    return user;
  } catch (error) {
    return createError(error);
  }
});
```

---

## **Step6：前端串接 API**

終於來到最後一步啦！

在頁面上使用 Nuxt3 `$fetch` 方法發送請求，新增一筆資料

```jsx
// pages/index.vue
<template>
  <div>
    <label>name</label>
    <input type="text" v-model="user.name" />
    <label>email</label>
    <input type="email" v-model="user.email" />

    <button @click="addUser()">Add User</button>
  </div>
</template>

<script setup>
const user = ref({
  name: '',
  email: ''
});

const addUser = async () => {
  const response = await $fetch('/api/user', {
    method: 'POST',
    body: {
      ...user.value
    }
  });
  console.log(response);
};
</script>
```

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/tlgGdS3.png">
</div>

回到 Compass 確認一下，如果有看到一筆資料，就代表串接成功囉！

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/LTxvKPv.png">
</div>

---

參考資源：

https://medium.com/@flanker72/nuxt3-complex-solutions-database-integration-8df941f0fb82
https://www.mongodb.com/