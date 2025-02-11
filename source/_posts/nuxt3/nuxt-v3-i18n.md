---
title: Nuxt.js 3.x 導入 I18n 實作多國語系
date: 2023-08-29
tags: [ nuxt3 ]
category: Nuxt3
description: 多國語系讓我們的網站邁向國際市場，本篇將說明如何搭配 Nuxt3 整合模組 @nuxtjs/i18n 進行開發
image: https://imgur.com/l0zKfkP.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/articles/10333224)
>

> **@nuxtjs/i18n 版本：v8.0.0-rc.3**
> 

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/l0zKfkP.png">
</div>

多國語系讓我們的網站邁向國際市場，本篇將說明如何搭配 Nuxt3 整合模組 [@nuxtjs/i18n](https://v8.i18n.nuxtjs.org/) 進行開發

### **@nuxtjs/i18n 簡介**

- 整合 Vue I18n
- 自動產生路由
- SEO 搜尋引擎最佳化
- Lazy Loading 延遲載入
- 自動偵測語言並切換
- 多國語言支援不同域名

<!-- more -->

## **套件安裝**

Nuxt3 需搭配 **v8** 版本，目前為 rc 版，透過 `next` tag 安裝

```bash
npm install -D @nuxtjs/i18n@next
```

## **nuxt.config 配置模組**

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  modules: [
    '@nuxtjs/i18n'
  ]
})
```

---

## **nuxt.config 設定模組選項**

i18n module 完整選項參考 [官方文件](https://v8.i18n.nuxtjs.org/options/routing)，以下範例配置：

- **strategy：**定義路由是否加上語系前綴，範例使用 `prefix`，所有語系都會加上路徑前綴（ex: `http://localhost:3000/zh/`）
- **langDir：**翻譯檔目錄路徑，範例為根目錄下 `locales/` 資料夾
- **locales：**語系選項，範例設定**繁體中文**跟**英文**，存放於 `langDir` 目錄下，以下範例檔結構：
  
  ```bash
  locales/
    |—— en.js
    |—— zh.js
  nuxt.config.js
  ```
  
- **defaultLocale：**預設語系
- **detectBrowserLanguage：**是否自動偵測使用者瀏覽器語系
  - **useCookie：**設定為 `true` 將語系儲存於 cookie，避免使用者每次進入網站都重新導向
  - **cookieKey：**預設 `i18n_redirected`
  - **redirectOn：**設定為 `root` 根目錄，只在根目錄偵測語系，避免使用者進入網站時被重新導向，以利 SEO

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  i18n: {
    strategy: 'prefix',
    langDir: 'locales',
    locales: [
      {
        code: 'en',
        iso: 'en-US',
        file: 'en.js'
      },
      {
        code: 'zh',
        iso: 'zh-TW',
        file: 'zh.js'
      }
    ],
    defaultLocale: 'zh',
    detectBrowserLanguage: {
      useCookie: true,
      cookieKey: 'i18n_redirected',
      redirectOn: 'root'
    }
  }
})
```

---

## **新增語系翻譯檔**

路徑結構：

```bash
locales/
  |—— en.js
  |—— zh.js
```

```jsx
// locales/en.js
export default {
  welcome: 'Welcome',
  backHome: 'Back Home',
  about: {
    title: 'About Us',
    description: 'This is About Page'
  }
};
```

```jsx
// locales/zh.js
export default {
  welcome: '您好',
  backHome: '回首頁',
  about: {
    title: '關於我們',
    description: '這是關於我們頁面'
  }
};
```

---

## **多國語系基本應用**

- `useLocalePath`：依據當前語系解析路徑，EX：`/about` → `/zh/about`
- `useSwitchLocalePath`：用來切換語系
- `$t`：Vue 實體方法，用來翻譯訊息

```jsx
// pages/about.vue
<template>
  <div class="m-4">
    <NuxtLink :to="localePath('/')">{{ $t('backHome') }}</NuxtLink>

    <div>
      <NuxtLink :to="switchLocalePath('en')">English</NuxtLink>
      <NuxtLink :to="switchLocalePath('zh')">繁體中文</NuxtLink>
    </div>

    <h2>{{ $t('welcome') }}</h2>
    <h3>{{ $t('about.title') }}</h3>
    <p>{{ $t('about.description') }}</p>
  </div>
</template>

<script setup>
const localePath = useLocalePath();
const switchLocalePath = useSwitchLocalePath();
</script>
```

畫面呈現如下：

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/a5exB0g.gif">
</div>

---

## **單一元件翻譯**

前面說明如何建立各語系翻譯檔，也可以單獨在元件內透過 `<i18n>` 定義翻譯資訊

`useScope: 'local'` 將 `t` 方法作用域限制於元件內，這麼做可以避免受到全域翻譯檔影響

```jsx
// pages/about.vue
<template>
  <div class="m-4">
    元件翻譯
    <h2>{{ t('welcome') }}</h2>

    全域翻譯
    <h2>{{ $t('welcome') }}</h2>
  </div>
</template>

<i18n lang="yaml">
  en:
    welcome: 'hello world'
  zh:
    welcome: '哈囉世界'
</i18n>

<script setup>
const { t } = useI18n({
  useScope: 'local'
});
</script>
```

全域翻譯檔

```jsx
// locales/zh.js
export default {
  welcome: '您好'
};
```

畫面呈現如下：

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/4lRaJS0.png">
</div>

---

## **插入動態變數**

### **1. 具名變數**

```jsx
// pages/about.vue
<template>
  <h1>{{ t('hello', { name: 'Daniel' }) }}</h1>
</template>

<i18n lang="yaml">
  en:
    hello: 'hello {name}'
  zh:
    hello: '哈囉 {name}'
</i18n>

<script setup>
const { t } = useI18n({
  useScope: 'local'
});
</script>
```

### **2. 匿名變數（陣列）**

```jsx
// pages/about.vue
<template>
  <h1>{{ t('hello', [ 'Daniel' ]) }}</h1>
</template>

<i18n lang="yaml">
  en:
    hello: 'hello {0}'
  zh:
    hello: '哈囉 {0}'
</i18n>

// ...
```

---

## **`<NuxtLinkLocale>` 元件**

等同於 `<NuxtLink>` 搭配 `useLocalePath()` Composable，以下兩個寫法編譯後結果相同

```jsx
<template>
  <NuxtLinkLocale to="/">{{ $t('home') }}</NuxtLinkLocale>
</template>
```

```jsx
<template>
  <NuxtLink :to="localePath('/')">{{ $t('home') }}</NuxtLink>
</template>

<script setup>
const localePath = useLocalePath();
</script>
```

---

## **SEO 優化**

透過 `useLocaleHead()` 優化多國語系的 `<head>` 設定：

- `useLocaleHead`：依據當前語系回傳 head 屬性
  - `<html>` 標籤 `lang` 屬性
  - `hreflang` 標籤屬性：告訴搜尋引擎頁面使用的語言，讓使用者在搜尋時，搜尋引擎能提供對應語言給用戶
  - `og:locale` 以及 `og:locale:alternate` 標籤
  - Canonical URL（標準網址）標籤：告訴搜尋引擎此頁面的主要連結，避免搜尋引擎處理重複內容網頁
- `addSeoAttributes`：是否加上 SEO 屬性
- `useHead`：Nuxt Composable，用來定義完整 `<head>` 標籤內容

{% colorquote info %}
Nuxt3 meta tags 相關配置可以參考 [這篇文章](https://clairechang.tw/2023/07/26/nuxt3/nuxt-v3-meta-tags/#useHead-amp-useHeadSafe)
{% endcolorquote %}

```jsx
// pages/about.vue
<template>
  <div>...</div>
</template>

<script setup>
const i18nHead = useLocaleHead({
  addSeoAttributes: true
});

useHead({
  htmlAttrs: {
    lang: i18nHead.value.htmlAttrs.lang
  },
  link: [ ...(i18nHead.value.link || []) ],
  meta: [ ...(i18nHead.value.meta || []) ]
});
</script>
```

產生的網頁原始碼： 

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/D7PWGYf.png">
</div>

---

參考資源：

https://v8.i18n.nuxtjs.org/
https://vue-i18n.intlify.dev/