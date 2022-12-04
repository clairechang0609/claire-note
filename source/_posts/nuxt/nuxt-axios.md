---
title: Nuxt.js 2.x 搭配 Axios 與自訂 Error Page
date: 2022-12-04 12:41:00
tags: [ nuxt, nuxt.js, vue, vue.js, ssr, axios ]
category: Nuxt
---
> **版本：nuxt 2.15.8**
>

Axios 是一套相當便利的 Promise Base Ajax 套件，而 Nuxt 又有 Axios 整合套件 [@nuxtjs/axios](https://axios.nuxtjs.org/)，此篇範例會使用到 Async、Await，如果對於 Promise 尚不熟悉，可以參考 [Promise 介紹文章](https://www.casper.tw/development/2020/02/16/all-new-promise/)

<!-- more -->

那們我們就來替專案加上 axios 吧！首先執行 `npm install @nuxtjs/axios`

接著在 nuxt.config.js 進行配置（[參數選項](https://axios.nuxtjs.org/options)）：

```jsx
// nuxt.config.js
exportdefault {
    modules: [ '@nuxtjs/axios' ],
    axios: {
        baseUrl: process.env.SERVER_URL,
        browserBaseURL: process.env.BASE_URL,
        credentials: true
    }
}
```

如果需要區分 client 跟 server 端 base url，可以另外設定 browserBaseURL（於client 端會覆寫 baseUrl），套件將 axios 方法直接 **注入(inject)** 至 Vue 實例，擷取套件原始碼片段

```jsx
// node_modules/@nuxtjs/axios/lib/plugin.js
// Inject axios to the context as $axios
ctx.$axios = axios
inject('axios', axios)
```

因此在 Vue 生命週期或屬性使用 `this.$axios` 即可呼叫 axios 方法，那麽在 Nuxt 生命週期呢？

Nuxt 生命週期（asyncData, fetch, plugins, middleware）會自動帶入 [context](https://nuxtjs.org/docs/internals-glossary/context/) 參數，context 包含以下參數 / 物件：

```jsx
const {
    $axios
    app,
    store,
    route,
    params,
    query,
    env,
    isDev,
    isHMR,
    redirect,
    error,
    $config
} = context
```

使用方式如下（範例使用 asyncData）

```jsx
// pages/about.vue
export default {
    async asyncData(context) {
        const { seo } = await context.$axios.$get('/api/seo');
        return {
            seo
        };
    }
}
```

我們也可以物件解構方式使用 $axios

```jsx
// pages/about.vue
export default {
    async asyncData({ $axios }) {
        const { seo } = await $axios.$get('/api/seo');
        return {
            seo
        };
    }
}
```

axios 提供以下方法進行事件捕捉：

- onRequest(config)
- onResponse(response)
- onError(err)
- onRequestError(err)
- onResponseError(err)

如果我們想要全域註冊攔截器(Interceptors) 進行 api 請求或是錯誤捕捉，也很簡單，

首先在 plugins 新增檔案，這裡命名為 axios.js，接著在裡面監聽想捕捉的事件方法

```jsx
// plugins/axios.js
export default ({ $axios, redirect, store }) => {
    $axios.onRequest(config => {
        console.log('on request', config.url);
    });
    $axios.onError(error => {
        const code = parseInt(error.response && error.response.status);
        if (code === 401) {
            redirect('/login');
        }
    });
};
```

接著在 nuxt.config.js plugins 進行擴充

```jsx
// nuxt.config.js
export default {
    plugins: [
        { src: '@/plugins/axios' }
    ]
}
```

axios 在發生錯誤時，預設會導到 error page，如果我們想避免畫面跳轉，可以加入 resolve promise，這樣在發生錯誤時，都不會跳轉到 error page

```jsx
// plugins/axios.js
export default ({ $axios, redirect, store }) => {
    $axios.onError(error => {
        const code = parseInt(error.response && error.response.status);
        if (code === 401) {
            redirect('/login');
        }
        return Promise.resolve(false); // 避免畫面轉導錯誤頁
    });
};
```

> 💡 如果全域設定 resolve promise，在 axios 發生錯誤時，都不會進到 catch error 歐！

**自訂 Error Page**

Nuxt 專案有預設的 error page，可以在 .nuxt 資料夾看到 `components/nuxt-error.vue`，錯誤頁面也可以自訂，檔案放置位置比較特別，在 layouts 資料夾內新增一支檔案 error.vue

```jsx
// layouts/error.vue
<template>
    <div class="nuxt-error-wrap">
        <div class="text-center py-4">
            <h2>{{ error.statusCode }}</h2>
            <h3>{{ error.message }}</h3>
        </div>
        <nuxt-link to="/" class="btn btn-primary p-4">回首頁</nuxt-link>
    </div>
</template>

<script>
export default {
    name: 'ErrorPage',
    layout: 'error-layout',
    props: {
        error: {
            type: Object,
            default: () => {}
        }
    }
};
</script>
```

> 💡 雖然 error.vue 檔置於 layouts 資料夾，但視為 pages 頁面，因此如果沒有配置 layout，預設會讀取 layouts/default.vue 樣板

這樣就完成了，在 Nuxt 發生錯誤時，都會進到我們自訂錯誤頁面，如果要手動進到錯誤頁，呼叫 context 內的 error 方法即可，在 Nuxt 生命週期：

```jsx
// pages/about.vue
export default {
    async asyncData({ $axios, error }) {
        try {
            const { seo } = await $axios.$get('/api/seo');
            return {
                seo
            };
        } catch(err) {
            const { statusCode, message } = err;
            error({ statusCode, message });
        } 
    }
}
```

在 Vue 生命週期或屬性，使用 `this.$nuxt.error()` 就可以囉！

```jsx
// pages/about.vue
export default {
    methods: {
        async getSeo() {
            try {
                const { seo } = await this.$axios.$get('/api/seo');
                this.seo = seo;
            } catch(err) {
                const { statusCode, message } = err;
                this.$nuxt.error({ statusCode, message });
            }
        }
    }
}
```

---

參考文章：

[https://ithelp.ithome.com.tw/articles/10208852](https://ithelp.ithome.com.tw/articles/10208852)

[https://mavrickmaster.medium.com/custom-error-pages-with-nuxt-js-3c70e6c51aff](https://mavrickmaster.medium.com/custom-error-pages-with-nuxt-js-3c70e6c51aff)