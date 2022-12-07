---
title: Nuxt.js 2.x 目錄結構
date: 2022-12-03 17:00:00
tags: [ nuxt, nuxt.js, vue, vue.js, ssr ]
category: Nuxt
---
> **版本：nuxt 2.15.8**
>

使用 create-nuxt-app 安裝完成後，可以看到以下的資料夾結構：

<!-- more -->

![](https://i.imgur.com/jPvQ66l.png)

**assets, layouts, middleware, mixins, plugins** 以上資料夾在當前版本是需要手動建置的，

依照順序介紹各資料夾功能：

# **assets**

跟 Vue 專案相同，用來存放像是 css, scss, images 需要被 webpack 編譯的靜態資源，如不需被編譯，則存放於 static。

# **components**

自訂的元件檔，例如我們常會建立共用的 Navbar.vue, Sidebar.vue …，通常為**大寫命名**，然後在需要的頁面引入該檔案即可使用，使用方式基本上跟 Vue 專案相同。

如果不想要個別引入元件，Nuxt 也有提供很便利的作法，只要在 nuxt.config.js 檔內設定：

```jsx
// nuxt.config.js
export default {
    components: true
}
```

就會全局引入元件，使用方式只要遵循資料夾結構輸入元件名稱就可以了

範例：

元件路徑：components/home/Banner.vue

使用元件：`<HomeBanner></HomeBanner>`

# **layouts**

共用模板。

大家還記得在 Vue 專案下，只要在 App.vue，或是在 router 建立一個巢狀路由，外層設定共用容器，再將嵌套路由放入 children，子路由就可以共享外層模板

```jsx
export default [
    {
        path: '/products',
        component: Product,
        children: [
            {
                path: 'food',
                component: Food
            },
            {
                path: 'drink',
                component: Drink
            }
        ]
    }
]
```

但是在 Nuxt 架構下，並不存在 App.vue 這隻檔案，router 也會自動配置，那該怎麼做到模板共用呢？其實很簡單，只要在 layouts 資料夾內，新增 default.vue 檔，預設所有 pages 內的檔案都會共享該版面

```jsx
// default.vue
<template>
    <div class="default-wrap">
        <Navbar />
        <Nuxt />
        <Footer />
    </div>
</template>
```

`<Nuxt />` 類似 Vue 的 `<router-view />` （嵌套路由）

如果想要新增更多模板，只要在 layouts 內新增檔案，例如 layouts/products.vue，在欲使用的頁面引入，該頁就可以讀到 layouts/products.vue 模板

```jsx
// pages/food.vue
export default {
    layout: 'products'
}
```

# **middleware**

前面有說到 Nuxt 專案會自動依 pages 內的資料夾結構產生對應的靜態/動態路由，

但如果說我們想要使用 **路由守衛(Navigation Guards)** 來進行路由監聽，像是 Vue router 內的 beforeEach callback，該怎麼做呢？

##### **Vue 專案：**

```jsx
// pages/food.vue
const router = new VueRouter({
    mode: 'history',
    routes
});

router.beforeEach(async (to, from, next) => {
    if (!store.permissions.includes(route.path)) {
        next({ statusCode: 403 });
    }
    next();
});

export default router;
```

##### **Nuxt 專案：**

我們可以手動建立一個 middleware 資料夾，在裡面新增要進行路由監聽的檔案 routeAuth.js

```jsx
// middleware/routeAuth.js
export default ({ from, route, redirect, store, error }) => {
    if (!store.isLogin) {
        redirect('/login');
    }
    if (!store.permissions.includes(route.path)) {
        error({ statusCode: 403 });
    }
};
```

然後在需要進行監聽的檔案加入 middleware

```jsx
// layouts/default.vue
export default {
    middleware: 'routeAuth',
    data() {
        return {};
    }
}
```

或是在 nuxt.config.js 設定全域監聽

```jsx
// nuxt.config.js
export default {
    router: {
        middleware: [ 'routeAuth' ]
    }
}
```

# **mixins**

mixins 提供彈性方式讓頁面可以重複使用方法，可以包含任何 Vue 組件項目(data, computed, watch, 生命週期)，將共用方法包裝進去，首先在 mixins 新增檔案 mixins/utils.js

```jsx
// mixins/utils.js
export default {
    data() {
        return {
            number: 0
        };
    }
    methods: {
        count() {
            this.number++;
        }
    }
};
```

##### **全域註冊：**

全域宣告 mixin 務必小心使用，因為會影響到所有 Vue 檔(pages, components)

在 mixins 新增一支檔案 global-mixins.js，將欲全域註冊的檔案加入 Vue 實例

```jsx
// mixins/global-mixins.js
import Vue from 'vue';
import utils from '@/mixins/utils';

// 避免重複註冊
if (!Vue.__utils_mixin__) {
    Vue.__utils_mixin__ = true;
    Vue.mixin(utils);
}
```

接著加入 nuxt.config.js 內

```jsx
// nuxt.config.js
export default {
    plugins:[
        { src: '@/mixins/global-mixins' }
    ]
}
```

##### **局部註冊：**

局部註冊很簡單，只要在欲使用檔案引入即可，這裡假設在 pages/about.vue

```jsx
// pages/about.vue
import utils from '@/mixins/utils';

export default {
    name: 'About',
    mixins:[ utils ]
}
```

這樣在 About 頁面就可以取得參數 number 的值跟呼叫 count() 方法囉！

# **pages**

主要的頁面檔案，Nuxt 專案會自動依照 pages 內的資料夾結構配置路由，

換句話說，就不需要像 Vue 專案一樣需要自行設定 vue-router，為**小寫命名**（命名會直接是路徑名稱），每個 .vue 檔都是已經被註冊的頁面

以首頁為例：pages/index.vue → `http://localhost:3000`

**巢狀路由：**

直接舉例，先建立一支檔案 pages/about.vue

```jsx
// pages/about/index.vue
<template>
    <div>
        about：
        <nuxt-child />
    </div>
</template>
```

接在 about 資料夾新增兩隻檔案，可取得配置的巢狀路徑如下

pages/about/index.vue → `http://localhost:3000/about`

pages/about/claire.vue → `http://localhost:3000/about/claire`

**動態路由：**

如果想要顯示動態路由，只要在檔名前加上下底線就可以了：

pages/about/ _name.vue → `http://localhost:3000/about/claire`

這樣就可以在檔案內取得動態路由參數

```jsx
// pages/about/_name.vue
export default {
    created () {
        console.log(this.$route.params.name);
    }
}
```

# **plugins**

Nuxt 插件，於 Vue.js 執行前引入第三方套件，以 [vue-notification](https://www.npmjs.com/package/vue-notification) 為例

首先使用 npm 安裝 `npm i vue-notification`

然後在 plugins 資料夾新增 notification.js 檔

```jsx
// plugins/notification.js
import Vue from 'vue';
import Notifications from 'vue-notification';

Vue.use(Notifications);
```

在 nuxt.config.js 配置 plugins

```jsx
// nuxt.config.js
export default {
    plugins: [
        { src: '@/plugins/notification.js' }
    ]
}
```

就可以在所有頁面中使用該套件 `<Notifications></Notifications>`

# **static**

靜態資源資料夾，用來存放不需要被編譯的檔案，像是圖片檔，或是可以供使用者下載的範例檔等，如需被編譯，則存放於 assets。

# **store**

放置 Vuex 狀態管理工具，用來存放全域共用的方法、資料，使用方式與 Vue 大致相同，Vuex 在使用時會碰到一個問題，當頁面重新整理，其會被還原為初始狀態，至於要怎麼解決這個問題，後續會單獨介紹。

---

參考文章：

[https://israynotarray.com/vue/20211011/3406447097/](https://israynotarray.com/vue/20211022/3508219327/)

[https://dev.to/husteadrobert/how-to-use-global-navigation-guards-with-nuxt-middleware-and-why-you-absolutely-should-not-7bl](https://dev.to/husteadrobert/how-to-use-global-navigation-guards-with-nuxt-middleware-and-why-you-absolutely-should-not-7bl)

[https://ithelp.ithome.com.tw/articles/10207822](https://ithelp.ithome.com.tw/articles/10207822)

[https://medium.com/@seyijosh44/how-to-use-mixins-in-nuxt-js-826724fa251](https://medium.com/@seyijosh44/how-to-use-mixins-in-nuxt-js-826724fa251)