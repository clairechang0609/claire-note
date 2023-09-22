---
title: Nuxt.js 3.x 替網站增加 Sitemap 網站地圖
date: 2023-09-18
tags: [ nuxt3 ]
category: Nuxt3
description: Sitemap 是一種用來提供網站資訊的檔案，記錄網站內的網頁、圖片等，主要功用為協助搜尋引擎快速了解我們的網頁，本篇說明如何在 Nuxt3 專案內加入 Sitemap 網站地圖
image: https://imgur.com/IkD2tfF.png
---

Sitemap 是一種用來提供網站資訊的檔案，記錄網站內的網頁、圖片等，主要功用為**協助搜尋引擎快速了解我們的網頁**。

新網站上線後，雖然搜尋引擎爬蟲會自動讀取網站內容並收錄，不過當網站規模較大、架構較複雜、彼此間連結性較低，可能需要花較長的時間才能完整收錄，透過 Sitemap，直接告訴搜尋引擎網頁資訊，能有效加快網頁收錄的速度。

接下來說明如何在 Nuxt3 專案內加入 Sitemap 網站地圖：

<!-- more -->

## **套件安裝**

安裝 Nuxt3 整合模組 `nuxt-simple-sitemap`，本篇搭配版本 v.3.3.2

```bash
npm i -D nuxt-simple-sitemap
```

## **nuxt.config 配置模組**

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  modules: [
    'nuxt-simple-sitemap'
  ]
});
```

---

## **標準網址定義**

Canonical URL（標準網址）定義對於 SEO 很重要，用來告訴搜尋引擎標準的網址，解決重複內容網頁問題（ex：`www.example.com` 以及 `example.com`）

{% colorquote info %}
標準網址對 SEO 到底有多重要？這裡推薦一篇 [文章](https://welly.tw/serp-rank-optimization/what-is-canonical-url)
{% endcolorquote %}

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  site: {
    url: 'http://example.com'
  }
});
```

---

## **建立 Sitemap**

假設專案 `pages/` 頁面結構如下

```
pages/
|—— index.vue
|—— hello.vue
|—— about.vue
|—— user/
  |—— [id].vue
|—— post/
  |—— [id].vue
```

執行 `nuxt dev` 編譯後，瀏覽器輸入 `http://localhost:3000/sitemap.xml`，可以看到**靜態路徑**自動被加入 Sitemap 

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/IkD2tfF.png">
</div>

點選 `/sitemap.xml?cancical`，可以看到 URL 調整為前面定義的標準網址

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/rgryULa.png">
</div>

自動加入的靜態頁預設沒有帶上 `priority` 與 `changefreq` 參數，需在設定檔 `defaults` 加入

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  sitemap: {
    defaults: {
      changefreq: 'daily',
      priority: 0.8
    }
  }
});
```

{% colorquote info %}
sitemap configure 相關參數與預設值，請參考 [官方文件](https://nuxtseo.com/sitemap/api/config)
{% endcolorquote %}

---

## **Sitemap 加入動態路徑**

靜態路徑執行編譯時會自動產生，動態路徑則需要結合後端 API 取得

{% colorquote info %}
需先具備 Nuxt3 data fetching 相關觀念，可以參考 [這篇文章](https://clairechang.tw/2023/07/19/nuxt3/nuxt-v3-data-fetching/)
{% endcolorquote %}

### **Step1：新增 `_sitemap-urls.js`**

預設路徑結構如下：

```
server/
|—— api/
  |—— _sitemap-urls.js
```

預設路徑為 `server/api/_sitemap-urls.js`，可以透過 `dynamicUrlsApiEndpoint` 參數調整路徑

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  sitemap: {
    dynamicUrlsApiEndpoint: '/_sitemap'
  }
});
```

### **Step2：透過 API 取得動態路由**

**範例：**

假設 API 如下

```jsx
// server/api/users.js
export default defineEventHandler(() => {
  return [ '/user/1', '/user/2' ];
});
```

在 `server/api/_sitemap-urls.js` 發出請求並回傳 URL 內容

```jsx
// server/api/_sitemap-urls.js
export default defineEventHandler(async () => {
  const [
    users,
    posts
  ] = await Promise.all([
    $fetch('/api/users'),
    $fetch('/api/posts')
  ]);
  return [ ...users, ...posts ].map(url => {
    return {
      loc: url,
      lastmod: new Date()
    };
  });
});
```

執行 `nuxt dev` 編譯後，可以看到動態路由也被加入 Sitemap

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/Vj2HpEb.png">
</div>

---

## **Sitemap 整合 I18n**

搭配 Nuxt3 I18n 整合模組 `@nuxtjs/i18n` 開發，`nuxt-simple-sitemap` 會自動整合，產出各語系 sitemap 檔

{% colorquote info %}
`@nuxtjs/i18n` 套件安裝參考 [這篇文章](https://clairechang.tw/2023/08/29/nuxt3/nuxt-v3-i18n/)
{% endcolorquote %}

**範例：**語系分為**英文**與**繁中**

預設會產出一個 index sitemap（ex：`sitemap_index.xml`）作為各語系 sitemap 索引

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/zSDU2BB.png">
</div>

點擊查看英文版 sitemap `http://localhost:3000/en-sitemap.xml`

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/IqomlpC.png">
</div>

如果想合併語系檔，將 `sitemaps` 設為 `false` 即可

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  sitemap: {
    sitemaps: false
  }
});
```

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/jhErKCq.png">
</div>

{% colorquote info %}
看到這裡，或許有人會納悶為什麼 sitemap 缺少了重要的 `hreflang` 語系標籤，其實
`hreflang` 是存在的，上面看到的只是閱覽用的 XSL 檔，要查看原始的 `sitemap.xml`，請見下一段說明
{% endcolorquote %}

---

## **調整 Sitemap UI**

前面看到的 sitemap 只是方面我們閱覽用的 XSL 檔，搜尋引擎真實讀取到的 `sitemap.xml` 是沒有 CSS 樣式的檔案

### **查看原始 Sitemap**

如果想確認原始 `sitemap.xml`，將 `xsl` 設為 `false`

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  sitemap: {
    xsl: false
  }
});
```

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/YUKnZKU.png">
</div>

sitemap 如果有 `<xhtml:link />` 屬性，像是多國語系搭配 `hreflang` 標籤，瀏覽器畫面渲染會異常，如下圖所示。這只是瀏覽器渲染問題，並不影響 Sitemap 本身功能，搜尋引擎爬蟲一樣可以正常讀取，詳細說明可以參考 [討論串](https://support.google.com/webmasters/thread/161147229/will-google-still-recognize-my-sitemap-if-i-change-the-declaration-protocol?hl=en)

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/LKFeyvy.png">
</div>

### **調整欄位顯示內容**

透過 `xslColumns` 調整欄位

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  sitemap: {
    xslColumns: [
      { label: 'URL', width: '25%' }, // URL 欄位必填
      { label: 'Last Modified', select: 'sitemap:lastmod', width: '25%' },
      { label: 'Change Frequency', select: 'sitemap:changefreq', width: '25%' },
      { label: 'Priority', select: 'sitemap:priority', width: '12.5%' },
      { label: 'Hreflangs', select: 'count(xhtml:link)', width: '12.5%' }
    ]
  }
});
```

{% colorquote info %}
`Hreflangs` 屬性，必須要搭配 `nuxt-simple-sitemap` v3.3.2 以上版本才能使用
{% endcolorquote %}

來看一下調整後的畫面

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/EIQMFkR.png">
</div>

---

參考資源：

https://nuxtseo.com/
https://nuxt.com/docs/getting-started/deployment#selective-pre-rendering
https://stackblitz.com/edit/nuxt-starter-umyso3?file=nuxt.config.ts