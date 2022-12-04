---
title: Nuxt.js 2.x nuxt.config.js 設定檔
date: 2022-12-03 17:10:00
tags: [ nuxt, nuxt.js, vue, vue.js, ssr ]
category: Nuxt
---
> **版本：nuxt 2.15.8**
> 

全名為 ****Nuxt configuration file****，功能同 Vue 專案內 vue.config.js 檔，如果我們使用 create-nuxt-app 來建置專案，會自動產生這支檔案，在此配置的內容，會全域讀取設定。

接下來介紹一下一些常用的設定：

<!-- more -->

# alias

路徑別名，大家還記得在 Vue 專案內，如果沒有設定路徑別名，則需要寫相對路徑（例如：../components/Navbar.vue），對於維護和開發都很不方便，一些程式碼檢核工具，甚至會視為 bad smell，因此我們會在 vue.config.js 檔內統一設定路徑別名。

Nuxt 專案很貼心的自動配置了路徑別名，預設值如下

```jsx
{
    '~~': `<rootDir>`,
    '@@': `<rootDir>`,
    '~': `<srcDir>`,
    '@': `<srcDir>`,
    'assets': `<srcDir>/assets`, // (unless you have set a custom `dir.assets`)
    'static': `<srcDir>/static`, // (unless you have set a custom `dir.static`)
}
```

因此在引入元件 Navbar 時可以寫成 `@/components/Navbar.vue`

如果需要更改配置也可以

```jsx
import { resolve } from 'path';
export default {
  alias: {
    'images': resolve(__dirname, './assets/images'),
        '@static': resolve(__dirname, './static')
  }
}
```

# ssr

server side render，預設為 true，如果不需 SEO，只要 spa 操作，設定 `ssr: false` 即可

> 💡 舊版會使用 `mode: 'spa'` 來進行設定，但[官方文件](https://nuxtjs.org/docs/configuration-glossary/configuration-mode/)有說明 [v2.14.5](https://github.com/nuxt/nuxt.js/pull/8044) 版本開始已棄用，改用 `ssr: false`

```jsx
export default {
    ssr: false // spa
}
```

# server

用來設定 port（預設 3000） 跟 host（預設 localhost）

```jsx
export default {
    server: {
        port: 8000,
        host: 'my-website'
    }
}
```

# router

配置 Nuxt 路由（vue-router），這裡以設定 base url 舉例，其他請參考[官方文件](https://nuxtjs.org/docs/configuration-glossary/configuration-router)

```jsx
export default {
    router: {
        base: '/backoffice/'
    }
}
```

# head

在 Vue 專案，我們要設定 head 內容，只要在 public/index.html 檔設定就好了，但 Nuxt 專案又該怎麼設定呢？

**全域設定**

在 nuxt.config.js 檔 head 內即可全域設定，通常在建立專案會預先帶入

```jsx
export default {
    head: {
        title: 'test',
        htmlAttrs: {
            lang: 'en'
        },
        meta: [
            { charset: 'utf-8' },
            { name: 'viewport', content: 'width=device-width, initial-scale=1' },
            { hid: 'description', name: 'description', content: '' },
            { name: 'format-detection', content: 'telephone=no' }
        ],
        link: [
            { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }
        ]
    }
}
```

這樣所有頁面都可以讀到 head 設定

**單頁設定**

Nuxt 專案很重要的一點是 SEO 效能優化，基於這點，通常會需要依照各頁設定自己的 SEO，例如pages/about.vue 頁面的 title 和 description 要單獨設定，方法也很簡單

```jsx
export default {
    head: {
        title: '關於我們',
        meta: [
            {
                hid: 'description',
                name: 'description',
                content: '這是關於我們頁面'
            }
        ]
    }
}
```

在各頁面設定的內容**優先度較高**，會覆寫全域設定的 head 內容，因此當我們開啟 about 頁面，會看到 **title** 變成 **關於我們**、**description** 變成 **這是關於我們頁面**，其他配置則依照 nuxt.config.js 檔

# css

引入全域使用的 css 檔，像是外部套件，或是自訂檔，如果要使用 sass，必須安裝 sass-loader

`npm install --save-dev sass sass-loader`

```jsx
export default {
    css: [
        // 外部套件
        { src: 'bootstrap-icons/font/bootstrap-icons.css' },
        // scss 檔
        { src: '@/assets/scss/app.scss', lang: 'scss' }
    ]
}
```

# plugins

全局引入外部套件，以 vue-notification 舉例，在 plugins 新增 notification.js 檔

```jsx
// plugins/notification.js
import Vue from 'vue';
import Notifications from 'vue-notification';

Vue.use(Notifications);
```

在 nuxt.config.js 配置 plugins

```jsx
export default {
    plugins: [
        { src: '@/plugins/notification.js' }
    ]
}
```

就可以在所有頁面中使用該套件 `<Notifications></Notifications>`

# components

如設定為 true，會全局引入元件，使用方式只要遵循資料夾結構輸入元件名稱就可以了

```jsx
export default {
    components: true
}
```

範例：

元件路徑：components/home/Banner.vue

使用元件：`<HomeBanner></HomeBanner>`

# buildModules

用來配置只在開發環境使用的模組，註冊在此可以讓專案在生產環境部署速度提升，並減少 node_modules 容量，詳情可以參考各套件的配置建議

`@nuxtjs/eslint-module` 套件官方建議配置於 buildModules

```jsx
export default {
    buildModules: [
        '@nuxtjs/eslint-module'
    ]
}
```

> 💡 使用條件：Nuxt 版本必須大於 v2.9

# modules

用來配置開發環境與生產環境共用模組，buildModules 及 modules 配置位置詳請參考各套件建議

```jsx
export default {
    modules: [
        '@nuxtjs/style-resources'
    ]
}
```

# styleResources

用來注入全域共用 sass, scss，如果不引入，在各頁面（.vue）內的 style 會無法使用像是 mixin 等的常用變數，或是要在各頁面單獨 @import scss 檔

首先必須另外安裝 `@nuxtjs/style-resources` 套件

執行：`npm i @nuxtjs/style-resources`

在 nuxt.config.js 檔設定

```jsx
export default {
    modules: [
        '@nuxtjs/style-resources'
    ],
    styleResources: {
        scss: [
            '@/assets/scss/components/_color.scss',
            '@/assets/scss/components/_mixin.scss'
        ]
    }
}
```

這樣就可以達到變數共用了

> 💡 [官方文件](https://github.com/nuxt-community/style-resources-module/) 提到，請勿引入實際的 css 樣式，因為每個頁面跟元件都會重複編譯，造成系統極大負擔（筆者踩坑過OQ），建議只注入變數, mixins，因為這些值在編譯後就會消失了。

# build

客製化 webpack 設定，這裡舉例，在開發時，發現在 Vue 樣板無法使用可選串連 Optional Chaining（es2020語法），研究後發現必須需安裝擴充套件

`npm install vue-template-babel-compiler --save-dev`

接著在 nuxt.config.js 檔設定，就可以成功使用囉

```jsx
export default {
    build: {
        loaders: {
            vue: {
                compiler: require('vue-template-babel-compiler')
            }
        }
    }
}
```

# Loading

Loading 效果，基礎設定如下（[參數選項](https://nuxtjs.org/docs/features/loading)）

```jsx
export default {
    loading: {
        color: 'black',
        height: '5px',
        continuous: true
    }
}
```

如果想自訂更多樣式，也可以包裝成元件後引入

```jsx
export default {
    loading: '@/components/TheLoading.vue'
}
```

# env

用來定義全局共用的環境變數，Nuxt 專案預設只有 nuxt.config.js 檔內可以讀取環境變數，

因此如果要讓 .vue 檔或是 .js 檔讀到變數，這裡提供以下兩個做法：

**方法一：直接在 nuxt.config.js 檔內配置屬性**

```jsx
export default {
    env: {
        BASE_URL: process.env.BASE_URL
    }
}
```

但有時變數多，不想各別配置，可以採用方法二

**方法二：安裝套件 `@nuxtjs/dotenv`**

執行：`npm i @nuxtjs/dotenv`，在 modules 內設定：

```jsx
export default {
    modules: [
        '@nuxtjs/dotenv'
    ]
}
```

這樣就可以全域使用 .env 變數囉。

專案如果有區分開發跟生產環境 .env 檔，也可以設定如下：

```jsx
export default {
    modules: [
        [ '@nuxtjs/dotenv', { filename: `.env.${process.env.ENV}` } ]
    ]
}
```

> 💡 使用方法二需注意，變數如果內含變數，會讀取不到第二層的變數（饒口），直接提供範例，假設在 .env 檔內有兩個變數
`BASE_DOMAIN=my-websites`
`BASE_URL="http://${BASE_DOMAIN}"`
在 .vue 檔直接使用 process.env.BASE_URL，取得值為 http://${BASE_DOMAIN}，
無法解析成 http://my-websites，此情況還是需搭配方法一使用

---

參考文章：

[https://ithelp.ithome.com.tw/articles/10207330](https://ithelp.ithome.com.tw/articles/10207330)

[https://nuxtjs.org/docs/configuration-glossary](https://nuxtjs.org/docs/configuration-glossary/configuration-alias)

[https://hackmd.io/@xq/nuxt-config](https://hackmd.io/@xq/nuxt-config)