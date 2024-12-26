---
title: Nuxt.js 3.x ESLint 程式碼規範與自動排版－搭配 TypeScript
date: 2023-06-30
tags: [ nuxt3 ]
category: Nuxt3
description: 說明 Nuxt3 專案如何搭配 ESLint 程式碼規範工具，以及安裝 TypeScript 型別檢查工具，並執行自動排版
image: https://imgur.com/DyfmHP4.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/articles/10318476)
>

<div style="display: flex; justify-content: center;">
  <img style="width: 100%; max-width: 550px;" src="https://imgur.com/DyfmHP4.png">
</div>

Nuxt 3 完整支援 TypeScript 開發，TypeScript 相關設定在專案根目錄 `tsconfig.json`，本篇將說明：

- 安裝 ESLint 程式碼檢核工具（JavaScript／TypeScript）
- TypeScript 型別檢查
- VScode 安裝 Volar Extension
- VSCode 配置程式碼格式化

<!-- more -->

## **ESLint（ECMAScript Lint）**

ESLint 是一個開源的 **JavaScript / TypeScript 程式碼規範工具**，用於檢查和尋找程式碼中的問題、錯誤，例如未使用的變數、未定義的變數、重複的程式碼、錯誤的語法等。

透過 ESLint，我們可以根據專案需求和團隊規範，定義適合的程式碼檢查規則，例如縮排、換行、引號使用等，並整合到開發流程中，以提高程式碼品質、減少錯誤，並確保一致的程式碼風格。

開發者常用的規範有 Google、Airbnb、JavaScript Standard Style 等，本篇使用 Nuxt3 官方搭配的 ESLint 進行說明。

{% colorquote info %}
CSS／SCSS 程式碼檢核需搭配 Stylelint，可以參考 [這篇文章](https://clairechang.tw/2023/08/04/nuxt3/nuxt-v3-stylelint/)
{% endcolorquote %}

### **1. 套件安裝**

使用 nuxt 整合模組 `@nuxtjs/eslint-config-typescript` devDependencies

```bash
npm install --save-dev @nuxtjs/eslint-config-typescript eslint
```

### **2. 配置 .eslintrc.js 設定檔**

在專案根目錄新增 `.eslintrc.js`，配置如下：

**rules：**根據專案需求自訂規範，選項及預設值可以參考 [官方文件](https://eslint.org/docs/latest/rules/)

```jsx
// .eslintrc.js
module.exports = {
  root: true,
  extends: [
    '@nuxtjs/eslint-config-typescript'
  ],
  parserOptions: {
    parser: '@typescript-eslint/parser'
  },
  rules: { // 自訂規則
    quotes: ['error', 'single'], // 使用單引號
    'no-console': 'warn' // 使用 console 時警告提示
  }
}
```

{% colorquote info %}
根據 [官方文件](https://typescript.nuxtjs.org/guide/lint/) 說明，`@nuxtjs/eslint-config-typescript` 擴充設定完成後，ESLint 預設會使用 `@typescript-eslint/parser` 作為語法分析器，不需設定，不過需留意 `parserOptions.parser` 選項不能被其他設定覆寫
{% endcolorquote %}

### **3. 執行 lint 檢查**

在 `package.json` `scripts` 配置 lint 指令

```jsx
"lint": "eslint --ext .ts,.js,.vue ."
```

執行 `npm run lint` 查看檢核結果

<div style="display: flex; justify-content: center;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/4V2G0ty.png">
</div>

---

## **TypeScript 型別檢查**

TypeScript 相較於 Javascript 最大優點是**靜態型別定義和檢查**。Nuxt 基於效能考量，在執行 `nuxi dev` 或 `nuxi build` 時不會自動檢查型別，可以透過以下方式進行型別檢核：

首先在 `app.vue` 加上一段程式碼：

```jsx
// app.vue
<template>
  <div>
    <NuxtWelcome />
  </div>
</template>

<script lang="ts" setup>
const num: number = '123';
</script>
```

### **方法一：手動執行型別檢查**

透過 `nuxi` 指令來執行 `vue-tsc` 檢查型別

```bash
npx nuxi typecheck
```

顯示錯誤提示如下

<div style="display: flex; justify-content: center;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/0TNkJIO.png">
</div>

### **方法二：自動執行型別檢查**

直接安裝 `vue-tsc` 以及 `typescript` devDependencies

```bash
npm install --save-dev vue-tsc typescript
```

在 `nuxt.config` 啟用 `typescript.typeCheck`

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  typescript: {
    typeCheck: true
  }
});
```

之後在執行 `nuxi dev` 或 `nuxi build` 都會自動檢核型別

<div style="display: flex; justify-content: center;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/Crf1U87.png">
</div>

---

## **VSCode 安裝 Volar Extension**

Volar 是一個針對 Vue.js 開發的擴充套件，在編輯器中提供了更好的型別支援、錯誤檢測和自動補全功能，讓我們可以在編譯前預先發現錯誤，有助於提升開發效率和程式碼品質，Nuxt 官方也建議安裝。

- **[TypeScript Vue Plugin (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin)：** 針對 TypeScript 在 Vue 項目中的支援，提供了強大的 TypeScript 支援，包括類型推斷、定義跳轉、重構支援等功能
- **[Vue Language Features (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.volar)：** 主要提供了 Vue 項目的語言支援，包括模板的類型檢查、即時錯誤檢測、自動完成等功能

安裝完成後，可以看到編輯器出現提示訊息：

<div style="display: flex; justify-content: center;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/HluTDPC.png">
</div>

---

## **VSCode 配置自動排版**

許多開發者習慣整合 `ESLint` 與 `Prettier` 開發，不過根據 [Nuxt3 官方文件](https://nuxt.com/docs/community/contribution#no-prettier) 說明，因兩者容易產生衝突，且 `ESLint` 可以配置自動格式化，因此不建議安裝 `Prettier`。

透過在 IDE（整合開發環境）進行設定，就可以執行程式碼格式化。

{% colorquote info %}
官方提到，未來或許還是會兼容 `Prettier`，詳請參考 [討論串](https://github.com/nuxt/eslint-config/issues/224)
{% endcolorquote %}

### **1. 新增 settings.json**

在專案根目錄新增 `.vscode/settings.json` 檔

```
|—— .vscode/
  |—— settings.json
|—— my-app/
  |—— .nuxt/
  |—— app.vue
  |—— ...
```

### **2. 啟用自動修正和格式化功能**

```json
// .vscode/settings.json
{
  "editor.codeActionsOnSave": {
    "source.fixAll": false,
    "source.fixAll.eslint": true
  }
}
```

設定完成後，需要重啟專案才能成功運作，接著在儲存檔案時，ESLint 就可以幫我們自動排版囉！

---

參考資源：

https://github.com/nuxt/eslint-config#readme
https://typescript.nuxtjs.org/guide/lint/
https://nuxt.com/docs/guide/concepts/typescript#typescript