---
title: Nuxt.js 3.x Stylelint SCSS 程式碼檢查與自動排版
date: 2023-08-04
update:
tags: [ nuxt3 ]
category: Nuxt3
description: 說明 Nuxt3 專案如何安裝 Stylelint CSS 程式碼檢查工具，並執行自動排版
image: https://imgur.com/JBaER3w.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/articles/10319292)
>

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 600px;" src="https://imgur.com/JBaER3w.png">
</div>

[Stylelint](https://stylelint.io/) 是一套用於規範 CSS 和 CSS 預處理器（SCSS、Less 等）程式碼的工具，幫助我們維持程式碼的品質和風格。Stylelint 進行檢查時，會根據指定的規則來發現潛在的錯誤、不一致的風格和不推薦的模式。

以 `*.vue` 檔為例，檔案內包含三種主要的程式碼：`<template>`、`<script>` 和 `<style>`：

1. `<template>`：Vue.js 模板語法，用以定義 HTML，例如插槽 `{{ }}`、條件判斷和迴圈等。雖然這不是嚴格的 JavaScript 語法，但 ESLint 也提供了相關規則對模板語法進行檢查
2. `<script>`：JavaScript 區塊，由 ESLint 進行程式碼檢查
3. `<style>`：CSS 區塊，用於定義元件樣式，由 Stylelint 進行程式碼檢查

ESLint 搭配 Stylelint，讓專案保持良好的 Coding Style。本篇將說明如何在 Nuxt3 專案搭配 Stylelint 工具。

{% colorquote info %}
ESLint JavaScript 程式碼檢查工具參考 [這篇文章](https://clairechang.tw/2023/06/30/nuxt3/nuxt-v3-typescript-and-eslint/)
{% endcolorquote %}

<!-- more -->

## **套件安裝**

**整合套件**

- **@nuxtjs/stylelint-module：**Nuxt 搭配 Stylelint 的整合模組

{% colorquote info %}
`@nuxtjs/stylelint-module` 提供了在 Nuxt 使用 Stylelint 的整合模組，但並未包含 Stylelint、相關配置以及 PostCSS 相關套件，需手動安裝相依套件
{% endcolorquote %}

**相依套件**

- **stylelint：**實際執行 CSS 程式碼檢查的工具
- **stylelint-config-standard-vue：**stylelint 規則配置，用於檢查文件中 `<style>` 區塊的 CSS
- **stylelint-config-standard-scss：**stylelint 規則配置，基於 `stylelint-config-standard` 的擴充，用於檢查 SCSS 程式碼
- **stylelint-order：**stylelint 插件，規範 CSS 屬性的排序順序
- **postcss、postcss-scss、postcss-html：**擴展 CSS 和 SCSS 的**後處理器**工具

```bash
npm install -D @nuxtjs/stylelint-module \
  stylelint \
  stylelint-config-standard-vue \
  stylelint-config-standard-scss \
  stylelint-order \
  postcss postcss-scss postcss-html
```

---

## **nuxt.config 配置**

stylelint 配置選項參考 [官方文件](https://github.com/nuxt-modules/stylelint)

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  modules: [
    ...
    '@nuxtjs/stylelint-module'
  ],
  stylelint: {
    lintOnStart: false, // 專案啟動時不自動檢查所有相關檔案
    chokidar: true // 監聽文件異動進行檢核（文件未列出此選項）
  }
})
```

---

## **.stylelintrc.js 設定檔配置**

目錄新增 `.stylelintrc.js`，配置如下：

- **extends：**擴展 stylelint 配置
- **overrides：**定義特定文件或文件類型覆蓋配置，會覆寫 `rules` 定義的全域規則
- **plugins：**配置 stylelint 插件
- **rules：**定義全域 stylelint 規則，選項參考 [官方文件](https://stylelint.io/user-guide/rules/)

```jsx
// .stylelintrc.js
module.exports = {
  extends: [
    'stylelint-config-standard-scss',
    'stylelint-config-standard-vue/scss'
  ],
  plugins: [
    'stylelint-order'
  ],
  overrides: [
    {
      files: [ '*.scss', '**/*.scss' ], // 指定 .scss 檔
      rules: {
        'scss/no-global-function-names': null // 關閉此規則
      }
    }
  ],
  rules: {
    // ...
    'unit-allowed-list': [ 'em', 'rem', 'deg', 'px' ], // 可使用的單位
    'order/properties-order': [ // 設定排序順序（plugins 必須先定義 stylelint-order）
      'position',
      'top',
      'bottom',
      'right',
      'left',
      'display',
      'align-items',
      'justify-content',
      'float',
      'clear',
      'overflow',
      'overflow-x',
      'overflow-y',
      'margin',
      'margin-top',
      'margin-right',
      'margin-bogttom',
      'margin-left',
      'padding',
      'padding-top',
      'padding-right',
      'padding-bottom',
      'padding-left',
      'width',
      'min-width',
      'max-width',
      'height',
      'min-height',
      'max-height',
      'font-size',
      'font-family',
      'font-weight',
      'text-align',
      'text-justify',
      'text-indent',
      'text-overflow',
      'text-decoration',
      'white-space',
      'color',
      'background',
      'background-position',
      'background-repeat',
      'background-size',
      'background-color',
      'background-clip',
      'border',
      'border-style',
      'border-width',
      'border-color',
      'border-top-style',
      'border-top-width',
      'border-top-color',
      'border-right-style',
      'border-right-width',
      'border-right-color',
      'border-bottom-style',
      'border-bottom-width',
      'border-bottom-color',
      'border-left-style',
      'border-left-width',
      'border-left-color',
      'border-radius',
      'opacity',
      'filter',
      'list-style',
      'outline',
      'visibility',
      'z-index',
      'box-shadow',
      'text-shadow',
      'resize',
      'transition'
    ]
  }
}
```

---

## **VSCode 配置程式碼格式化**

#### **1. 安裝擴充**

首先需在 VSCode 安裝擴充：[Stylelint](https://marketplace.visualstudio.com/items?itemName=stylelint.vscode-stylelint)

#### **2. 設定自動格式化**

在專案根目錄新增 `.vscode/settings.json`


```
|——.vscode/
  |—— settings.json
|——my-app/
  |—— .nuxt/
  |—— .stylelintrc.js
  |—— app.vue
  |—— ...
```

接著配置如下：

- **editor.codeActionsOnSave：**儲存文件時執行的程式碼操作（Code Actions）
- **stylelint.validate：**設定需驗證的文件類型

```jsx
{
  "editor.codeActionsOnSave": {
    "source.fixAll": false,
    "source.fixAll.stylelint": true
  },
  "stylelint.validate": [ "css", "scss", "vue" ]
}
```

設定完成後，可能需要重啟專案才能成功運作，接著在儲存檔案時，Stylelint 就會協助我們自動排版囉！

---

參考資源：

https://github.com/ota-meshi/stylelint-config-standard-vue
https://github.com/nuxt-modules/stylelint
https://ithelp.ithome.com.tw/articles/10297223?sc=pt#_=_
[https://paper-hsiao.medium.com/stylelint-幫助你整理-css-的好幫手-b708adab430e](https://paper-hsiao.medium.com/stylelint-%E5%B9%AB%E5%8A%A9%E4%BD%A0%E6%95%B4%E7%90%86-css-%E7%9A%84%E5%A5%BD%E5%B9%AB%E6%89%8B-b708adab430e)