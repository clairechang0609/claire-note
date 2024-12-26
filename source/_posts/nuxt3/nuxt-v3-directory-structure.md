---
title: Nuxt.js 3.x 目錄結構 Directory Structure
date: 2023-07-07
tags: [ nuxt3 ]
category: Nuxt3
description: Nuxt3 提供了完整的目錄定義規則，協助配置許多功能，讓我們可以專注在開發上，本篇說明如何建置 Nuxt3 完整的專案目錄
image: https://imgur.com/bvebnUt.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/users/20130500/ironman/6236)
>

Nuxt3 定義了完整的目錄規則，讓我們可以輕鬆配置功能，專注在開發上

透過 `npx nuxi@latest init <project-name>` 安裝專案後，可以看到如下的預設目錄

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 300px;" src="https://imgur.com/bvebnUt.png">
</div>

看起來分類似乎有點不齊全，接下來跟著[官方文件](https://nuxt.com/docs/guide/directory-structure/nuxt)一起建立完整的專案目錄

<!-- more -->

```
my-app/
|—— .nuxt/
|—— .output/
|—— assets/
|—— components/
|—— composables/
|—— content/
|—— layouts/
|—— middleware/
|—— node_modules/
|—— pages/
|—— plugins/
|—— public/
|—— server/
  |—— api/
  |—— routes/
  |—— middleware/
|—— utils/
|—— .env
|—— .gitignore
|—— .nuxtignore
|—— app.config.ts
|—— app.vue
|—— nuxt.config.ts
|—— package.json
|—— tsconfig.json
```

## **.nuxt**

Nuxt 在編譯過程中生成的暫存資料夾。是 Nuxt 執行編譯和 server 端渲染時使用的重要資源，無法手動調整裡面的內容（每次執行編譯時會覆寫）

---

## **.output**

執行生產環境編譯 `nuxt build`（`npm run build`）時自動生成的資料夾，跟 `.nuxt` 資料夾一樣，每次執行編譯時會更新覆寫

---

## **assets**

用來存放像是 CSS、Sass、字體、圖片等需要被 webpack 或是 Vite 編譯的靜態資源（壓縮、最佳化），如不需經過編譯，則存放於 `public/`

---

## **components**

用來定義 Vue 共用元件，Nuxt 會自動引入，**名稱規則為：路徑前綴 + 元件名稱**，例如巢狀目錄結構如下

```
components/
|—— base/
  |—— about/
    |—— Button.vue
```

`Button.vue` 的元件名稱為

```jsx
<BaseAboutButton />
```

---

## **composables**

組合式函式，利用 Composition API 來封裝和複用 **有狀態邏輯（Stateful Logic）**的函式，取代 Options API `mixins` 的功能。定義在 `composables/` 內的檔案 Nuxt 會自動引入

我們可以將不同的邏輯抽象成單獨的 `composable`，並組合在 `setup` 函式中。比起 `mixins` ，`composable` 協助我們更好理解組件的結構和功能，並提高程式碼的可讀性

以 `composables/useCounter.js` 為例

```jsx
// composables/useCounter.js
export default function() {
  const count = ref(0);
  const increment = () => {
    count.value++;
  };

  return {
    count,
    increment
  };
}
```

在元件內使用共用方法

```jsx
// pages/count.vue
<template>
  <div>
    <span>{{ count }}</span>
    <button type="button" @click="increment">add</button>
  </div>
</template>

<script setup>
const { count, increment } = useCounter();
</script>
```

{% colorquote info %}
composables 和 utils 比較可以參考 [這篇文章](https://clairechang.tw/2023/07/11/nuxt3/nuxt-v3-composables-vs-utils/)
{% endcolorquote %}

---

## **content**

搭配 [Nuxt Content](https://content.nuxtjs.org/) 套件，可以讀取 `content/` 目錄，並解析存放於此資料夾內的 `.md`、`.yml`、`.csv` 以及 `.json` 檔案，建立一套內容管理系統（CMS），主要功能：

- 搭配 components 元件渲染内容
- 使用類似 mongodb 的 API 來 query 文章內容
- 在 Markdown 文件中使用 Vue components
- 使用 [Shiki](https://shiki.matsu.io/) 程式碼 highlight
- 自動渲染內容與路由

簡單來說，Nuxt Content 能夠解析 Markdown 語法文章，做到像 Hexo 一樣的功能，協助我們打造技術部落格

{% colorquote info %}
Nuxt Content 相關應用推薦參考 [這篇文章](https://blog.twjoin.com/%E7%94%A8-nuxt-content-%E9%87%8D%E5%AF%AB%E6%88%91%E7%9A%84%E9%83%A8%E8%90%BD%E6%A0%BC-278ce9e8580c)
{% endcolorquote %}

---

## **layouts**

用來存放共用模板，官方文件提到，如果整個專案只有一個模板，建議直接在 `app.vue` 定義即可

#### **預設模板**

首先新增預設模板 `layouts/default.vue`，必須加上 `<slot />`，引用的頁面才能插入內容

```jsx
// layouts/default.vue
<template>
  <div>
    default layout
    <slot />
  </div>
</template>
```

在頁面上使用 layout

```jsx
// app.vue
<template>
  <div>
    <NuxtLayout>
      <NuxtPage />
    </NuxtLayout>
  </div>
</template>
```

#### **使用其他模板**

```
layouts/
|—— default.vue
|—— custom.vue
```

如果我們想在單一元件使用 `custom.vue` layout，可以用 `definePageMeta` 覆蓋預設模板

```jsx
// pages/about.vue
<script>
definePageMeta({
  layout: 'custom'
});
</script>
```

---

## **middleware**

Nuxt 內的 **路由守衛（Navigation Guards）**，相當於 Vue Router 內的 beforeEach callback，協助我們在進到頁面前執行一些事件，像是權限檢查

**middleware 定義方式：**

- **匿名：**直接在單一元件檔內定義
- **具名：**在 `middleware/` 定義，並在需要的頁面引入
- **全域：**同具名的定義方式，不過檔名需加上 `.global` 後綴，在所有頁面切換時自動執行

```
middleware/
|—— auth.ts
|—— setup.global.ts
```

建立一個 middleware

```jsx
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to, from) => {
  const auth = useState('auth');
  if (!auth.value.isAuth) {
    return navigateTo('/login');
  }
});
```

在頁面內使用

```jsx
// pages/about.vue
<script setup>
definePageMeta({
  middleware: [
    function (to, from) { // 匿名方式
      // 客製 middleware
    },
    'auth'
  ]
});
</script>
```

---

## **pages**

用來配置主要頁面的資料夾，定義後 Nuxt 會自動整合 Vue Router，依照資料夾以及檔案結構配置路由，例如：`pages/work.vue` 會被映射到 `/work`

{% colorquote info %}
如果要使用 `pages/`，`app.vue` 需加上 `<NuxtPage />` 用於顯示定義的頁面
{% endcolorquote %}

#### **動態路由**

動態路由使用方括號 `[]` 包覆（Nuxt2 使用下底線 `_`）

範例：

```
pages/
|—— index.vue
|—— products-[category]/
  |—— [id].vue
```

透過 `$route` 可以取得 `category` 跟 `id` 值：

```jsx
<template>
  <p>{{ $route.params.category }} - {{ $route.params.id }}</p>
</template>
```

當我們切到頁面 `/products-bag/112345`，畫面渲染如下：

```jsx
bag - 112345
```

#### **Catch-all Route**

使用 `[...slug].vue` 匹配該路徑下的所有層級路由，可以用來處理找不到路由的預設頁面

範例：

```
pages/
|—— index.vue
|—— [...slug].vue
```

當使用者輸入不存在的頁面，例如：`/hello` 或是 `/hello/claire`，會自動轉到 `[...slug].vue`

---

## **plugins**

用來定義插件，`plugins/` 內的檔案 Nuxt 會自動引入，如果要限制在 server 或是 client 端使用，檔名需加上 `.server` 或 `.client` 後綴

**範例：**引入 [vue3-notification](https://www.npmjs.com/package/@kyvg/vue3-notification) 第三方套件

1. 安裝套件 `npm i @kyvg/vue3-notification`
2. 建立 `plugin` 

```
plugins/
|—— notification.client.ts
```

```jsx
import Notifications from '@kyvg/vue3-notification';

export default defineNuxtPlugin(nuxtApp => {
  nuxtApp.vueApp.use(Notifications);
});
```

3. 頁面上使用

```jsx
<script>
import { useNotification } from '@kyvg/vue3-notification';

export default {
  setup() {
    const { notify } = useNotification();
    onMounted(() => {
      notify({
        title: "Authorization",
        text: "You have been logged in!",
      });
    });
  }
};
</script>
```

---

## **public**

靜態資源資料夾（同 Nuxt2 的 `static/`），用來存放不需要被編譯的檔案，像 CSS、 文字或圖片，透過根目錄 `/` 即可使用 `public/` 檔案，檔案如需被編譯，則存放於 `assets/`

---

## **server**

Nuxt3 搭配新的伺服器引擎 Nitro，讓我們可以在 server 端定義內容，像是建立 API 以及透過 Server Middleware 處理事件

簡單說明 `server/` 內的資料夾功能：

- **api：**建立帶有 `/api` 前綴的 API 路徑
- **routes：**建立不帶 `/api` 前綴的 API 路徑
- **middleware：**在每次發出請求時觸發。跟 router middleware 不同，頁面切換（page navigation）並不會觸發 server middleware

---

## **utils**

定義在 `utils/` 內的檔案 Nuxt3 會自動引入，官方文件提到 `utils/` 資料夾的拆分主要是為了跟 `composables/` 做區隔，前面有提到 `composables/` 利用 Composition API 來封裝和複用 **有狀態邏輯（Stateful Logic）**，而 `utils/` 則用來定義 **無狀態邏輯（Stateless Logic）**

以 `utils/toThousands.js` 為例（將數字帶入千分號）

```jsx
// utils/toThousands.js
export default num => {
  if (!num) {
    return num;
  }
  const parts = num.toString().split('.');
  parts[0] = parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, ',');
  return parts.join('.');
};
```

在頁面使用

```jsx
// pages/count.vue
<template>
  <div>
    $ {{ toThousands(19999) }}
  </div>
</template>
```

編譯結果： $ 19,999

---

## **.nuxtignore**

設定 Nuxt 編譯時忽略根目錄 `layouts`、`pages`、`components`、`composables`、`middleware` 內的檔案

```
# 忽略 layout custom.vue
layouts/custom.vue
# 忽略 layout -ignore.vue 結尾的檔案
layouts/*-ignore.vue

# 忽略 page about.vue
pages/about.vue
# 忽略 ignore 資料夾內的 page 檔案
pages/ignore/*.vue

# 忽略 route middleware custom 資料夾內檔案，排除 custom/bar.js
middleware/custom/*.js
!middleware/custom/bar.js
```

---

## **app.config.ts**

設定全域共用的資料，只能在 client 端取得

{% colorquote info %}
官方文件提到，不建議將任何機密資料存放於此
{% endcolorquote %}

```jsx
// app.config.ts
export default defineAppConfig({
  theme: {
    color: '#0d6efd'
  }
})
```

在頁面使用

```jsx
// pages/about.vue
<script>
export default {
  setup() {
    const appConfig = useAppConfig();

    onMounted(() => {
      console.log(appConfig.theme.color); // output: #0d6efd
    });
  }
};
</script>
```

---

## **app.vue**

專案進入點，Nuxt3 將 `app.vue` 移到目錄頁，讓開發者可以只使用 `app.vue` 來建置網站（例如單頁 Landing Page），而不定義 `pages/` 資料夾。

如果要使用 `pages/`，`app.vue` 需加上 `<NuxtPage />` 用於顯示定義的頁面（功能同 Vue Router  `<router-view />`）

```jsx
// app.vue
<template>
  <div>
    <NuxtLayout>
      <NuxtPage />
    </NuxtLayout>
  </div>
</template>
```

---

## **nuxt.config.ts**

Nuxt 設定檔，相關設定 [參考文件](https://nuxt.com/docs/api/configuration/nuxt-config#nuxt-configuration-reference)

```jsx
// nuxt.config.ts
export default defineNuxtConfig({
  // My Nuxt config
})
```

---

## **tsconfig.json**

配置 extends 或是覆蓋預設的 TypeScript 設定

{% colorquote info %}
如要設定路徑別名，若直接在 `tsconfig.json` 設定，將會覆寫預設的 `alias`，建議在 `nuxt.config` 檔設定，Nuxt 會加到自動生成的 `tsconfig`
{% endcolorquote %}

```jsx
// tsconfig.json
{
  "extends": "./.nuxt/tsconfig.json" // 自動生成的 tsconfig
}
```

---

參考資源：

https://ithelp.ithome.com.tw/articles/10295432
https://nuxt.com/docs/guide/directory-structure/nuxt