---
title: Nuxt.js 2.x Lifecycle 生命週期
date: 2022-12-03 17:54:00
tags: [ nuxt, nuxt.js, vue, vue.js, ssr ]
category: Nuxt
---
> **版本：nuxt 2.15.8**
> 

Nuxt 最大的特點就是 Server Side Render，因此他有獨立的[生命週期](https://nuxtjs.org/docs/concepts/nuxt-lifecycle#lifecycle)，來看一下官方提供的圖片：

<!-- more -->

![](https://i.imgur.com/W4ZLvrJ.png)

# **nuxtServerInit**

只在 Nuxt 環境初始化時觸發，當我們想將 server 端資料提前傳給 client 端，可以使用此方法，要注意只能寫在 **VueX** **store/index.js  actions**

```jsx
// store/index.js
export const state = () => ({
    userInfo: {}
});

export const mutations = {
    setUserInfo(state, value) {
        state.userInfo = value;
    }
};

export const actions = {
    nuxtServerInit({ commit }, { req }) {
        // req.session.user = { name: 'claire' }
        commit('setUserInfo', req.session.user);
    }
};
```

這樣就可以在 Nuxt 初始化時，觸發 nuxtServerInit 方法，將值傳入 state，我們可以從瀏覽器 Vue 開發者工具看到內容：

![](https://i.imgur.com/tm2jMnC.png)

如果想將資料傳給其他 VueX modules，可以這樣做：

首先新增一支檔案 store/greeting.js

```jsx
// store/greeting.js
export const state = () => ({
    message: ''
});

export const mutations = {
    setMessage(state, value) {
        state.message = value;
    }
};
```

接著在 store/index.js 定義 nuxtServerInit

```jsx
// store/index.js
export const actions = {
    nuxtServerInit({ commit }, { req }) {
        commit('greeting/setMessage', 'Hello World!');
    }
};
```

這樣就可以觸發 store/greeting.js setMessage 方法，見下圖開發者工具

![](https://i.imgur.com/qj8ViHd.png)

# Route Middleware

中間組件，在頁面渲染前執行，有三種定義方式，執行順序為：**Global → Layout → Page**

接下來分別說明該如何定義

**Global Middleware**

在 middleware 資料夾內建立檔案，這裡命名為 global.js

```jsx
// middleware/global.js
export default ({ from, route, redirect, store, error }) => {
    console.log('global middleware');

    if (!store.isLogin) {
        redirect('/login');
    }
};
```

接著在 nuxt.config.js 配置

```jsx
// nuxt.config.js
export default {
    router: {
        middleware: [ 'global' ]
    }
}
```

**Layout Middleware**

建立 middleware 檔案，這裡命名為 middleware/layout.js，然後配置到任一 layouts 檔案，範例使用 layouts/default.vue

```jsx
// layouts/default.vue
export default {
    name: 'Default',
    middleware: 'layout'
};
```

或是匿名配置也可以：

```jsx
// layouts/default.vue
export default {
    name: 'Default',
    middleware({ from, route, redirect, store, error }) {
        console.log('layout middleware');
    }
};
```

**Page Middleware**

概念同 layouts middleware，配置於任一 pages 檔案，範例使用 pages/about.vue，這裡使用匿名配置來說明

```jsx
// pages/about.vue
export default {
    name: 'About',
    middleware({ from, route, redirect, store, error }) {
        console.log('page middleware');
    }
};
```

接著我們從開發者工具查看 console 結果依序為下圖，因此我們可以透過 layout 跟 page middleware 來覆寫 global middleware

![](https://i.imgur.com/mT4lCpY.png)

# validate

於 pages 檔案配置此方法，用來驗證動態路由參數有效性，範例使用 pages/about/_userId.vue

```jsx
// pages/about/_userId.vue
export default {
    name: 'User',
    validate({ params, query }) {
        return true; // 驗證通過
        return false; // 驗證無效，會自動轉導 error page
    }
}
```

> 💡 驗證通過必須 return `true`，否則會自動轉跳 404 error page

# asyncData

於 server 端處理非同步的生命週期，在此傳入的內容可以被搜尋引擎爬蟲取得，是提升 SEO 效能的重點生命週期。

只會在頁面載入時調用，由於生命週期在 Vue 之前，因此無法取得 `this` ，且 asyncData 僅限 pages 底下頁面使用，方法內會自動帶入 [context](https://nuxtjs.org/docs/concepts/context-helpers) 參數，我們可以安裝 [@nuxtjs/axios](https://axios.nuxtjs.org/) 套件，axios 會被注入進 context 內，我們可以物件解構方式使用（ { $axios, params } = context )（範例使用 pages/about.vue） 

```jsx
// pages/about.vue
export default {
    name: 'About',
    async asyncData({ $axios, params }) {
        const id = params.id;
        const { data } = await $axios.$get(`/api/user/${id}`);
        return {
            userName: data
        };
    }
}
```

透過 return value，資料被賦予進 Vue 實體，我們透過 `this.userName` 即可成功取值

```jsx
// pages/about.vue
<template>
    <div>
        <h1>{{ userName }}</h1>
    </div>
</template>

<script>
export default {
    name: 'About',
    async asyncData({ $axios, params }) {
        const id = params.id;
        const { data } = await $axios.$get(`/api/user/${id}`);
        return {
            userName: data
        };
    }
}
</script>
```

> 💡 data 如果有相同變數名稱，會在 asyncData 生命週期被複寫，所以除非需要再次修改變數，否則請避免重複命名變數

> 💡 asyncData 是在 server 端、路由更新前即調用，由於是在瀏覽器渲染前的生命週期，因此無法使用 loading placeholder，也不能使用瀏覽器相關 API

# fetch

Nuxt v2.12 新增功能，功能類似 `asyncData` ，在畫面渲染前，同時於 server 端跟 client 端的生命週期，可以使用於任一 .vue 頁面，由於是在 Vue created 之後，因此可以取得 `this`，初次載入頁面時，fetch 會在 server 端執行，如果是透過 `<nuxt-link>` 進行路由切換，fetch 在 client 端執行，因此可以在此生命週期加入 loading 效果

> 💡 `<nuxt-link>` 為 Nuxt 的路由切換元件，相當於 Vue.js 的 `<router-link>` ，因此我們只能使用內部連結，外部連結必須使用 `<a>` 標籤，透過 `<nuxt-link>` 切換路由，會被視為 SPA 頁面跳轉

以下說明使用方式

```jsx
<template>
    <ul>
        <li v-for="(post, key) in posts" :key="key">
            {{ post }}
        </li>
    </ul>
</template>

<script>
export default {
    data() {
        return {
            posts: []
        }
    },
    async fetch() {
        const { data } = await this.$axios.$get('/api/posts');
        this.posts = data;
    }
}
</script>
```

如果要重複觸發 fetch 生命週期，可以使用 `this.$fetch` 來呼叫：

```html
<template>
    <div>
        <ul>
            <li v-for="(post, key) in posts" :key="key">
                {{ post }}
            </li>
        </ul>
        <button @click="$fetch">重新取得貼文</button>
    </div>
</template>
```

如果我們希望 fetch 只在 client 端運行，可以加上 `fetchOnServer: false`（預設 true）

### **取得 fetch 狀態**

我們可以透過 `this.$fetchState` 取得 fetch 當前執行狀態，有以下參數：

**pending**：Boolean / 是否執行完成，可以在此加入 loading 效果（client 端）

**error**：null or Error 物件 / 判斷是否發生錯誤

**timestamp**：整數 / 最後一次執行時間（搭配 `activated` 使用）

範例：

```jsx
<template>
    <div>
        <p v-if="$fetchState.pending">Loading...</p>
        <p v-if="$fetchState.error">有東西出錯了</p>
    </div>
</template>

<script>
export default {
    data() {
        return {
            posts: []
        }
    },
    activated() {
        // 每 30 秒自動呼叫 fetch
        if (this.$fetchState.timestamp <= Date.now() - 30000) {
            this.$fetch()
        }
    },
    async fetch() {
        const { data } = await this.$axios.$get('/api/posts');
        this.posts = data;
    }
}
</script>
```

### **生命週期執行順序**

接著我們從瀏覽器開發者工具觀察生命週期執行順序

```jsx
asyncData() {
    console.log('asyncData');
},
fetch() {
    console.log('fetch');
},
beforeCreate() {
    console.log('beforeCreate');
},
created() {
    console.log('created');
},
beforeMount() {
    console.log('beforeMount');
}
```

![](https://i.imgur.com/yKRo61U.png)

可以發現，`created` 跟 `beforeCreate` 這兩個 Vue.js 生命週期會同時出現在 server 端跟 client 端，如果要避免方法被重複執行，可以這樣做：

1. 加上 `process.client` 判斷
    
    ```jsx
    created(){
        if (process.client){
            // 執行內容
        }
    }
    ```
    
2. 使用 Nuxt `fetch` 生命週期
3. 使用 Vue `beforeMount` 生命週期

---

參考文章：

[https://stackoverflow.com/questions/60411436/nuxtjs-page-is-created-twice](https://stackoverflow.com/questions/60411436/nuxtjs-page-is-created-twice)

[https://happy9990929.github.io/2021/09/10/vue-nuxt-lifecycle-hooks/](https://happy9990929.github.io/2021/09/10/vue-nuxt-lifecycle-hooks/)