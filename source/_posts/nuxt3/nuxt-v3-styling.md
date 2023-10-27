---
title: Nuxt.js 3.x 搭配 CSS Framework－以 Bootstrap 5 為例
date: 2023-08-25
tags: [ nuxt3 ]
category: Nuxt3
description: 在 Nuxt 專案中，我們可以自由選擇 CSS 預處理器、CSS 框架或是 UI Library 來定義樣式。本篇將以 Bootstrap 5 搭配 SCSS 進行說明。
image: https://imgur.com/FHpABZ8.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/users/20130500/ironman/6236)
>

> **Bootstrap 版本：v5.3.1
Sass 版本：v1.63.6**
>

在 Nuxt 專案中，我們可以自由選擇 CSS 預處理器、CSS 框架或是 UI Library 來定義樣式。在 CSS 框架百家爭鳴的時代，這幾年熱門的 Tailwind CSS、基於 Vue.js 開發的 Quasar、或是搭配 Vue3 開發的 Element Plus 都能快速上手，協助我們打造精美的網站。

由於工作上較常使用 Bootstrap 協作，本篇將以 **Bootstrap 5 搭配 SCSS** 進行說明。

<!-- more -->

## **Bootstrap 5 簡介**

<div style="display: flex; justify-content: center; margin: 20px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/FHpABZ8.png">
</div>

[Bootstrap](https://www.npmjs.com/package/bootstrap) 有豐富的 Sass 變數、mixins、網格系統、元件、JS 插件，Bootstrap 5 與先前版本最大的不同，除了將 jQuery 從相依項目中移除，也新增 Utilities API（基於 Sass Maps 生成 Utilities Class），可以更簡易的管理或擴充樣式，不需手刻太多 CSS 即可完成多元、複雜的畫面。

## **套件安裝**

```bash
npm install bootstrap
npm install sass
```

---

## **使用 Bootstrap 樣式搭配自訂樣式**

在 `assets/` 靜態資源目錄定義 SCSS，範例 `assets/scss/app.scss`，需注意 Bootstrap 樣式與自訂樣式引入順序：

{% colorquote info %}
不建議直接引入整包 bootstrap components `bootstrap/scss/bootstrap` ，選擇需要的樣式引入即可，避免 CSS 檔案過大，造成系統負擔
{% endcolorquote %}

```scss
// assets/scss/app.scss

/** 1. 引入 functions，才能操控 color, svg, calc... */
@import "bootstrap/scss/functions";

/** 2. 自訂變數置於 bootstrap variables 前，覆蓋 bootstrap 變數 */
@import "./color";
@import "./variables";

/** 3. 引入 variables, mixins, root */
@import "bootstrap/scss/variables";
@import "bootstrap/scss/variables-dark";
@import 'bootstrap/scss/maps';
@import "bootstrap/scss/mixins";
@import "bootstrap/scss/root";

/** 4. 引入 utilities */
@import "~bootstrap/scss/utilities";

/** 5. 自訂、擴充、調整 utilities */
@import "./utilities";

/** 6. 引入需要的 bootstrap components */
@import "bootstrap/scss/reboot";
@import "bootstrap/scss/type";
@import "bootstrap/scss/containers";
@import "bootstrap/scss/grid";
@import "bootstrap/scss/tables";
@import "bootstrap/scss/forms";
@import "bootstrap/scss/buttons";
@import "bootstrap/scss/helpers";
/** ... */

/** 7. 使用 utilities 需引入 utilities api（將 sass map 轉換為 utility classes） */
@import "bootstrap/scss/utilities/api";

/** 8. 客製樣式置於最後，覆蓋前面的樣式 */
@import "./style";
```

**自訂樣式檔案結構：**

```
assets/
|—— scss/
  |—— app.scss
  |—— _color.scss
  |—— _variables.scss
  |—— _style.scss
```

**自訂、擴充、調整 utilities：**

透過 `map-merge` 合併 bootstrap utilities，詳細定義方式參考 [官方文件](https://getbootstrap.com/docs/5.0/utilities/api/#add-utilities) 

```scss
// assets/scss/utilities.scss
$utilities: map-merge(
  $utilities,
  (
    "cursor": (
      property: cursor,
      class: cursor,
      responsive: true,
      values: auto pointer grab
    )
  )
);
```

### **配置全域共用 CSS**

接著在 `nuxt.config` 配置全域共用 CSS，接下來整個專案 HTML 都能使用編譯後的樣式

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  css: [
    '@/assets/scss/app.scss'
  ],
  postcss: { // CSS 屬性加上瀏覽器相容性前綴
    plugins: {
        autoprefixer: true
    }
  }
})
```

{% colorquote info %}
Nuxt 專案已內建 postcss，加上 `autoprefixer: true` 會自動為屬性加上瀏覽器相容性前綴
範例定義以下樣式：

```scss
.container {
  display: flex;
}
```

編譯後的 CSS：

```scss
.container {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
}
```
{% endcolorquote %}

---

## **定義全域共用 Sass / SCSS 變數（搭配 Vite）**

如果想在 .vue 檔內的 `<style>` 直接使用 Sass / SCSS 變數，需搭配 `preprocessorOptions` 進行配置

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  vite: {
    css: {
      preprocessorOptions: {
        scss: {
          additionalData: `
            @import "@/assets/scss/_color.scss";
            @import "@/assets/scss/_variables.scss";
          `
        }
      }
    }
  }
});
```

```scss
// assets/scss/_color.scss
$primary: #49240F;
$secondary: #E4A79D;
```

接著就可以在 SCF（單一元件檔）使用 SCSS 變數

```jsx
// pages/hello.vue
<template>
  <div>
    <h1>Hello World</h1>
  </div>
</template>

<style lang="scss" scoped>
h1 {
  color: $primary;
}
</style>
```

{% colorquote info %}
加上 `scoped`，將樣式作用域限制在元件內，避免造成全域污染，如果希望樣式可以渲染到子元件，透過 `:deep()` 定義如下

```scss
:deep(h3) {
  color: $primary;
}
```
{% endcolorquote %}

---

## **使用 Bootstrap Plugins**

在 `plugins/` 目錄註冊 Bootstrap JavaScript Plugins

1. 新增 `plugins/bootstrap.client.js`

{% colorquote info %}
注意：Nuxt 會自動引入（auto imports）plugins，bootstrap plugins 要限制在 client 端使用，否則會拋錯 document is not defined ，檔名需加上 `.client` 後綴
{% endcolorquote %}

2. 透過 `Provide` 定義全局變數，將需要的 Bootstrap Plugins 注入到 `NuxtApp`

```jsx
// plugins/bootstrap.client.js
import * as bootstrap from 'bootstrap';
const { Modal, Collapse } = bootstrap;

export default defineNuxtPlugin(_nuxtApp => {
  return {
    provide: {
      bootstrap: {
        modal: element => new Modal(element),
        collapse: element => new Collapse(element)
      }
    }
  };
});
```

3. 使用 Modal Plugins

```jsx
// pages/about.vue
<template>
  <div>
    <div class="modal fade" tabindex="-1" ref="modalRef">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            ...
          </div>
          <div class="modal-body">
            ...
          </div>
          <div class="modal-footer">
            <button type="button" data-bs-dismiss="modal">close</button>
          </div>
        </div>
      </div>
    </div>
    <button type="button" class="btn btn-success" @click="showModal">
      點我看 Modal
    </button>
  </div>
</template>

<script setup>
const { $bootstrap } = useNuxtApp();
const modalRef = ref(null);
let modal;
const showModal = () => {
  modal.show();
};

onMounted(() => {
  modal = $bootstrap.modal(modalRef.value);
});

onBeforeUnmount(() => {
  // 加上 dispose，避免切換頁面時或是 HMR 看到殘留畫面
  modal.dispose();
});
</script>
```

---

## **動態樣式**

動態樣式定義方式同 [Vue3](https://vuejs.org/api/sfc-css-features.html#v-bind-in-css)，在 CSS 使用 v-bind function

```jsx
// pages/hello.vue
<template>
  <div>
    <h1>hello</h1>
    <h2>world</h2>
    <button @click="theme.color = 'red'">change color</button>
  </div>
</template>

<script setup>
const theme = ref({
  color: 'green'
});
</script>

<style lang="scss" scoped>
h1, h2 {
  color: v-bind('theme.color');
}
</style>
```

---

參考資源：

https://nuxt.com/docs/getting-started/styling
https://vite.nuxtjs.org/misc/common-issues#styleresources
https://getbootstrap.com/