---
title: Nuxt.js 套件應用：Bootstrap 5 搭配 SCSS 打造網頁 UI
date: 2022-12-22
tags: [ nuxt2, bootstrap ]
category: Nuxt
description: 本篇介紹如何在 Nuxt 專案使用 Bootstrap 5 協助我們快速設計與自訂 RWD 網站
image: https://imgur.com/Ssjder7.png
---
> **版本：nuxt 2.15.8**
>

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 700px;" src="https://imgur.com/Ssjder7.png">
</div>

Bootstrap 是一款相當熱門的前端開發工具，可以幫助我們快速的設計跟自訂 RWD 網站，過去 Bootstrap 相依於 jQuery 開發，Bootstrap 5 不需使用 jQuery，新增 Utilities API，可以更簡易的管理或擴充樣式，讓程式碼更輕量，另外也新增了 Bootstrap Icons 供免費使用。

透過 create-nuxt-app 建置 Nuxt.js 專案時，雖然可以選擇 CSS 模板（UI framework），但選項內並沒有 `Bootstrap` （Bootstrap Vue 搭配的是 `Bootstrap 4.5`），因此需手動安裝。

<!-- more -->

搭配 SCSS 開發需要安裝 sass loader，執行： `npm i --save-dev sass sass-loader@10`

接著安裝 Bootstrap： `npm i --save bootstrap @popperjs/core`

新增兩個檔案（檔名自訂）：

**plugins/bootstrap.js**：用來設定動態功能

**assets/scss/app.scss**：用來配置樣式

然後至 nuxt.config.js 加入檔案：

```jsx
// nuxt.config.js
export default {
    css: [
        { src: '@/assets/scss/app.scss', lang: 'scss' }
    ],
    plugins: [
        { src: '@/plugins/bootstrap', mode: 'client' }
    ]
}
```

## **CSS 設定**

{% colorquote info %}
不建議直接引入 `~bootstrap/scss/bootstrap` ，選擇需要的樣式引入即可，避免 css 檔案過大，造成系統負擔
{% endcolorquote %}

需注意引入的順序：

```scss
// assets/scss/app.scss
// 1. 引入 functions，才能操控顏色, svg, calc...
@import "~bootstrap/scss/functions";

// 2. 自訂變數，會覆蓋 bootstrap 的變數
@import "./variables";

// 3. 引入 variables, mixins, root
@import "~bootstrap/scss/variables";
@import "~bootstrap/scss/mixins";
@import "~bootstrap/scss/root";

// 4. 引入 utilities
@import "~bootstrap/scss/utilities";

// 5. 自訂 utilities
@import "./utilities";

// 6. 引入需要的 bootstrap components
@import "~bootstrap/scss/reboot";
@import "~bootstrap/scss/type";
@import "~bootstrap/scss/images";
@import "~bootstrap/scss/containers";
@import "~bootstrap/scss/grid";
@import "~bootstrap/scss/tables";
@import "~bootstrap/scss/forms";
@import "~bootstrap/scss/buttons";
@import "~bootstrap/scss/helpers";
// ...

// 7. 使用 utilities 需引入 utilities api（將 sass map 轉換為 utility classes）
@import "~bootstrap/scss/utilities/api";

// 8. 客製樣式置於最後，覆蓋前面的樣式
@import "./style";
```

{% colorquote info %}
想要了解如何使用 utilities 客製樣式，可以參考 [這篇文章](https://easonchang0115.github.io/blogs/other/2021/20210530_1.html)
{% endcolorquote %}

## **Javascript 設定**

```jsx
// plugins/bootstrap.js
import Vue from 'vue';
import * as bootstrap from 'bootstrap';
const { Modal, Collapse } = bootstrap;

// 避免重複註冊，在 Vue 實體內加入參數判斷
if (!Vue.__bootstrap_mixin__) {
    Vue.__bootstrap_mixin__ = true;
    Vue.mixin({
        methods: {
            bootstrapModal(element) {
                return new Modal(element);
            },
            bootstrapCollapse(element) {
                return new Collapse(element);
            }
        }
    });
}
```

將需使用到的功能（EX：Modal, Collapse）透過 mixin 註冊到 Vue 實體，這樣在 .vue 檔內就可以直接使用。

範例說明如何在頁面上（pages/about.vue）操控元件（components/modal.vue），讓 Modal 開啟：

```jsx
// components/modal.vue
<template>
    <div class="modal fade" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5>Modal Title</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    ...
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal"></button>
                    <button type="button" class="btn btn-secondary">確定</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'Modal'
}
</script>
```

```jsx
// pages/about.vue
<template>
    <h2>關於我們</h2>
    <Modal ref="modal" />
    <button type="button" class="btn btn-primary" @click="openModal()">開啟 Modal</button>
</template>

<script>
import Modal from '@/components/Modal';

export default {
    name: 'About',
    components: { Modal },
    data() {
        return {
            modal: ''
        }
    },
    mounted() {
        // 操控內層元件，需指定 this.$refs.modal 內的 $el
        this.modal = this.bootstrapModal(this.$refs.modal.$el);
    },
    methods: {
        openModal() {
            this.modal.show();
        }
    }
};
</script>
```

---

參考文章：

[https://getbootstrap.com/](https://getbootstrap.com/)

[https://easonchang0115.github.io/blogs/other/2021/20210530_1.html](https://easonchang0115.github.io/blogs/other/2021/20210530_1.html)

[https://stackoverflow.com/questions/71432924/vuejs-3-and-bootstrap-5-modal-reusable-component-show-programmatically](https://stackoverflow.com/questions/71432924/vuejs-3-and-bootstrap-5-modal-reusable-component-show-programmatically)