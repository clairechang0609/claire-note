---
title: Nuxt.js 3.x Assets vs Public 目錄－靜態資源管理
date: 2023-09-12
tags: [ nuxt3 ]
category: Nuxt3
description: Nuxt3 提供兩個資料夾 assets 以及 public，用來管理靜態資源，像是圖片、CSS 樣式、字體、icon 等，本篇說明兩個目錄適合存放的檔案類型與使用方式
image: https://imgur.com/5VRbaV1.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/users/20130500/ironman/6236)
>

Nuxt3 提供兩個資料夾 `assets/` 以及 `public/`，用來管理靜態資源，像是圖片、CSS 樣式、字體、icon 等，接下來說明兩個目錄適合存放的檔案類型與使用方式

<!-- more -->

## **Public**

靜態資源資料夾（同 Nuxt2 的 `static/`），用來存放不需被編譯的靜態檔，可以透過根目錄 `/` 直接使用檔案

### **適合檔案性質**

- **不需經由 Vite / Webpack 等打包工具編譯：**像是 `sitemap.xml` 通常不需另外壓縮處理
- **檔案固定性高 / 檔名不能被更改：**經過編譯後的檔案，檔名後會加上一組 hash，像是 `robots.txt` 必須固定檔名，搜尋引擎爬蟲才能正確解析，適合放在 `public/`

### **常見檔案類型**

- favicon.ico
- robots.txt
- sitemap.xml
- CNAME（DNS 紀錄）

### **使用方式**

透過根目錄 `/` 直接使用 `public/` 檔案

**範例：**

```
public/
|—— favicon.ico
|—— image/
  |—— picture.jpg
```

**使用 `favicon.ico`：**

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  app: {
    head: {
      link: [
        { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }
      ]
    }
  }
});
```

**使用 `image/picture.jpg`：**

```jsx
// pages/index.vue
<template>
  <div>
    <img src="/image/picture.jpg">
  </div>
</template>
```

透過開發者工具檢視

<div style="display: flex; justify-content: center; margin: 30px 0 10px; border: 1px solid rgb(200, 200, 200);">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/ABhFsgZ.png">
</div>

<small style="display: block; margin-bottom: 30px;">圖片來源：https://unsplash.com/</small>

---

## **Assets**

用來存放像是 CSS、Sass、圖片等需要被 webpack 或是 Vite 編譯的靜態資源。一般來說，`assets/` 的使用率較 `public/` 高

### **適合檔案性質**

- **需要透過 Vite / Webpack 打包工具編譯（壓縮、最佳化）：**像是 JS 或是 CSS、圖片等，希望在專案發佈前，預先對檔案進行壓縮和最佳化處理，提升網站載入速度與效能
- **檔案較常更新，避免被快取影響：**經過編譯後的檔案，檔名會加上依據文件內容產生的 hash，每次更新編譯後 hash 會變更，能避免受瀏覽器快取影響，強制瀏覽器讀取更新後的檔案。例如：`img.png` 編譯後變成 `img.2d8efhg.png`

### **常見檔案類型**

- CSS、Sass
- Javascript
- 圖片
- 字體

### **使用方式**

透過 `~/assets/` 路徑使用檔案

**範例：**

```
assets/
|—— scss/
  |—— style.scss
|—— image/
  |—— picture.jpg
```

**全域使用 `scss/style.scss`：**

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  css: [
    '~/assets/scss/app.scss'
  ]
});
```

**使用 `image/picture.jpg`：**

```jsx
// pages/index.vue
<template>
  <div>
    <img src="~/assets/image/picture.jpg">
  </div>
</template>
```

執行生產環境編譯 `nuxt build` (`npm run build`) 後，可以看到檔名加上了 hash

<div style="display: flex; justify-content: center; margin: 30px 0 10px; border: 1px solid rgb(200, 200, 200);">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/i68fewf.png">
</div>

<small style="display: block; margin-bottom: 30px;">圖片來源：https://unsplash.com/</small>

### **路徑別名**

Nuxt3 預設路徑別名如下：

```json
{
  "~": "/<rootDir>",
  "@": "/<rootDir>",
  "~~": "/<rootDir>",
  "@@": "/<rootDir>",
  "assets": "/<rootDir>/assets",
  "public": "/<rootDir>/public"
}
```

因此也可以透過 `@/assets/image/picture.jpg` 取得圖片，或是自訂別名來簡化路徑

### **動態圖片**

在 Nuxt2 搭配 Webpack 打包工具，透過 `require` 即可動態載入圖片

```jsx
// Nuxt2 + Webpack
<img :src="require(`~/assets/image/${imageName}`)" />
```

Nuxt3 搭配 Vite 解決方法如下（[討論串](https://github.com/nuxt/nuxt/issues/14766)），透過 `import.meta.glob` 方法一次引入多張圖片

```jsx
// Nuxt3 + Vite
<template>
  <div>
    <img :src="getImageUrl(imageName)" />
  </div>
</template>

<script setup>
const getImageUrl = name => {
  const assets = import.meta.glob('~/assets/image/*', { eager: true, import: 'default' });
  return assets[`/assets/image/${name}`];
};

const imageName = 'picture.jpg';
</script>
```

---

## **.output 目錄**

執行生產環境編譯 `nuxt build`（`npm run build`）時自動生成的資料夾，`assets/` 與 `public/` 打包後會放在 `.output/public/` 內

**原始檔案：**

```
public/
|—— public-pic.jpg
assets/
|—— assets-pic.jpg
```

**打包後：**

- `public/` 內的檔案不經處理直接放到 `.output/public/`
- `assets/` 內的檔案編譯後加上 hash 放到 `.output/public/_nuxt/`

```
.output/
|—— public/
  |—— _nuxt/
    |—— assets-pic.ef9916a0.jpg
  |—— public-pic.jpg
```

---

參考資源：

https://nuxt.com/docs/getting-started/assets
https://masteringnuxt.com/blog/handling-assets-in-nuxt-3
https://vitejs.dev/guide/assets.html