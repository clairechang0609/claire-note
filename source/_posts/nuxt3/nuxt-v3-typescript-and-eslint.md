---
title: Nuxt.js 3.x TypeScript 搭配 ESLint 自動排版
date: 2023-06-30
tags: [ nuxt3 ]
category: Nuxt3
description: 說明 Nuxt3 專案如何安裝 TypeScript 型別檢查工具，以及搭配 ESLint 程式碼分析，並執行自動排版
image: https://imgur.com/DyfmHP4.png
---

<div style="display: flex; justify-content: center;">
    <img style="width: 100%; max-width: 600px;" src="https://imgur.com/DyfmHP4.png">
</div>

Nuxt 3 內建支援 TypeScript，TypeScript 相關設定可以在 `tsconfig.json` 檢視，本篇介紹的 ESLint 環境會搭配 TypeScript 使用。

## **TypeScript 型別檢查**

TypeScript 相較於最大優點是**靜態型別定義和檢查**，基於效能考量，Nuxt 預設不進行型別檢查，可以透過以下方式開啟檢核

<!-- more -->

### **1. 套件安裝**

- **vue-tsc**
- **typescript**

```bash
npm install -D vue-tsc
npm install -D typescript
```

### **2. 進行型別檢查**

- 手動檢查：

```bash
npx nuxi typecheck
```

- 自動檢查：在 `nuxt.config` 檔啟用 `typescript.typeCheck` 選項，之後在執行 `npm run dev` 或是 `npm run build` 時都會自動進行型別檢查

```tsx
// nuxt.config.ts
export default defineNuxtConfig ({
    typescript: {
        typeCheck: true
    }
})
```

### **3. VScode 安裝 Volar Extension**

Volar 是一個針對 Vue.js 開發的擴充套件，在編輯器中提供了更好的型別支援、錯誤檢測和自動補全功能，讓我們可以在編譯前預先發現錯誤，有助於提升開發效率和程式碼的可靠性。

安裝以下擴充：

- [TypeScript Vue Plugin (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin)：官方建議安裝，針對 TypeScript 在 Vue 項目中的支援，提供了強大的 TypeScript 支援，包括類型推斷、定義跳轉、重構支援等功能
- [Vue Language Features (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.volar)：主要提供了 Vue 項目的語言支援，包括模板的類型檢查、即時錯誤檢測、自動完成等功能

接下來可以看到我們的 VScode 出現提示訊息：

<div style="display: flex; justify-content: left; margin: 20px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/mwbHVjX.png">
</div>

---

## **ESLint（ECMAScript Lint）**

ESLint 是一個開源的 **JavaScript / TypeScript 靜態程式碼分析工具**，用於檢查和尋找程式碼中的問題、錯誤，例如未使用的變數、未定義的變數、重複的程式碼、錯誤的語法等。透過設定 ESLint，我們可以根據自己的專案需求和團隊規範，定義適合的程式碼檢查規則，例如縮排、換行、引號使用等，並整合到開發流程中，以提高程式碼品質、減少錯誤，並確保一致的程式碼風格。

開發者常用的規範有 Google、Airbnb、JavaScript Standard Style 等，這裡使用 Nuxt 3 官方搭配的 ESLint 進行安裝。

{% colorquote info %}
CSS／SCSS 程式碼檢核需搭配 Stylelint，可以參考 [這篇文章](https://clairechang.tw/2023/08/04/nuxt3/nuxt-v3-stylelint/)
{% endcolorquote %}

### **1. 套件安裝**

```bash
npm install -D @nuxtjs/eslint-config-typescript eslint
```

### **2. 配置 .eslintrc.js 設定檔**

在專案根目錄新增 `.eslintrc.js` 檔，設定如下：

```jsx
// eslintrc.js
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
根據 [官方文件](https://typescript.nuxtjs.org/guide/lint/) 說明，`@nuxtjs/eslint-config-typescript` 擴充設定完成後，ESLint 會使用 `@typescript-eslint/parser` 作為語法分析器，可以不填，不過需留意 `parserOptions.parser` 選項不能被其他設定給覆寫
{% endcolorquote %}

{% colorquote info %}
`rules` 協助我們根據專案需求和團隊規範自訂規則，選項及預設值可以參考 [官方文件](https://eslint.org/docs/latest/rules/)
{% endcolorquote %}

### **3.  在 package.json 配置 lint 指令**

在 `scripts` 內加入以下

```jsx
"lint": "eslint --ext .ts,.js,.vue ."
```

試著執行看看，可以看到檢核結果

```bash
> lint
> eslint --ext .ts,.js,.vue .
```

---

## **VSCode 配置程式碼格式化**

通常 ESLint 會搭配 Prettier 使用，不過根據 [Nuxt3 官方文件](https://nuxt.com/docs/community/contribution#no-prettier) 說明，因兩者容易產生衝突，且 ESLint 已經可以配置格式化程式碼，也不需要 Prettier 重複相同的工作，因此 Nuxt 專案 **不建議 ESLint 搭配 Prettier 使用**，透過在 IDE（整合開發環境，Integrated Development Environment）進行設定，就可以執行程式碼格式化

{% colorquote info %}
官方提到，未來或許還是會加入 Prettier 兼容，可以參考 [討論串](https://github.com/nuxt/eslint-config/issues/224)。
{% endcolorquote %}

### **1. 新增 settings.json**

在專案根目錄新增 `.vscode/settings.json` 檔

```
.vscode
    |—— settings.json
my-app
    |—— .nuxt
    |—— assets
    |—— pages
    |—— ...
```

### **2. 設定儲存程式碼時啟用自動修復和格式化功能**

```json
// settings.json
{
    "editor.codeActionsOnSave": {
        "source.fixAll": false,
        "source.fixAll.eslint": true
    }
}
```

設定完成後，可能需要重新開啟專案才能成功運作，接著在儲存檔案時，ESLint 就可以幫我們自動排版囉！

---

參考資源：

https://github.com/nuxt/eslint-config#readme
https://ithelp.ithome.com.tw/articles/10293758