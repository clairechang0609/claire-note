---
title: Nuxt.js 3.x 將靜態網站部署到 GitHub Pages
date: 2023-10-03
tags: [ nuxt3 ]
category: Nuxt3
description: 本篇將說明 Nuxt3 專案搭配靜態網站生成（SSG），部署到 GitHub Pages 託管靜態網頁內容
image: https://i.imgur.com/r80Hush.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/articles/10335369)
>

專案開發完成後，終於到最後一步**「部署」**，本篇使用靜態網站生成（SSG），搭配 GitHub Pages 託管靜態網頁內容。

### **Static Site Generation（SSG）靜態網站生成**

在編譯時產生靜態 HTML 頁面。靜態部署適合內容固定、不需資料庫、不含伺服器端動態渲染的網站（例如公司形象網站或是個人部落格），雖然不像 Universal Rendering 一樣彈性，不過靜態網站很適合搭配 CDN 緩存。

<!-- more -->

## **實作部署**

### **Step1：建立 GitHub Repository**

在 Github 建立一個儲存庫 `<username>.github.io/<repository>`（[連結](https://github.com/new)）

- `<username>`：自己的使用者名稱
- `<repository>`：自訂專案名，本篇範例命名為 **nuxt3-generate**

### **Step2：上傳專案至 GitHub**

接著看到以下指引畫面，回到專案，依照指令上傳至 GitHub

![](https://i.imgur.com/JZOZS2z.png)

### **Step3：安裝 `gh-pages`**

專案安裝 `gh-pages` 套件至 devDependencies

```bash
npm i -D gh-pages
```

### **Step4：`nuxt.config` 配置**

- `baseURL`：因部署上 GitHub 網址會加上 `/<repository>/`，因此須調整為 `/<repository>/`
  {% colorquote info %}
  `process.env.NODE_ENV` 為 Node.js 環境變數，執行 `npm run dev` 開發模式下啟動專案，`process.env.NODE_ENV` 值為 "development"，在生產模式啟動專案，則為 "production"
  {% endcolorquote %}
- `buildAssetsDir`：`assets/` 編譯後的目錄名稱，預設為 `/_nuxt/`，Github Pages 預設使用 Jekyll 靜態網站生成工具，Jekyll 會自動忽略所有下底線（`_`）前綴的資料夾與檔案，因此將 `_nuxt` 調整為其他名稱

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  app: {
    baseURL: process.env.NODE_ENV === 'production' ? '/nuxt3-generate/' : '/',
    buildAssetsDir: '/static/'
  }
}
```

### **Step5：加上 `public/.nojekyll` 檔案**

上一步有提到，Jekyll 會自動忽略所有下底線前綴的資料夾與檔案，因此專案部署到 GitHub 後，會因 `_payload.json` 讀取不到，拋出 404 錯誤。在根目錄 `public/` 資料夾加上 `.nojekyll` 空白檔案，用來告訴 GitHub 當前網站並不是基於 Jekyll 建置

### **Step6：加入 `gh-pages` 指令**

`package.json` `scripts` 加入以下指令，以下指令會將編譯完成的 `.output/public` 靜態內容部署至 GitHub Pages

```jsx
"deploy": "gh-pages --dotfiles -d .output/public"
```

### **Step7：執行部署**

執行以下指令，產生靜態檔

```bash
npm run generate
```

編譯完成後，專案根目錄會產生一個 `.output` 資料夾

- `public` 為靜態資源檔，定義在 `public/` 與 `assets/` 的檔案都會被打包在這
- `assets/` 編譯後的檔案又會被包在 `static/`，對應前面設定的 `buildAssetsDir` 目錄名稱
- 每個靜態頁會各自編譯成 `_payload.json` 與 `index.html`

```
.output/
|—— public/
  |—— static/
    |—— bg.7e80f349.png
    |—— index.3fc7cdc4.js
    |—— index.9c7e60ac.css
  |—— about/
    |—— _payload.json
    |—— index.html
  |—— _payload.json
  |—— index.html
  |—— favicon.ico
  |—— ...
|—— nitro.json
```

接著執行部署到 GitHub Pages

```bash
npm run deploy
```

部署完成後，回到 GitHub 切到 `gh-pages` 分支，順利的話，在瀏覽器輸入網址

`https://<username>.github.io/<repository>/` 就可以看到首頁畫面囉～

![](https://i.imgur.com/r80Hush.png)

### **範例檔**

- **程式碼：**[連結](https://github.com/clairechang0609/nuxt3-generate/tree/gh-pages)
- **頁面：**[連結](https://clairechang0609.github.io/nuxt3-generate/about)

---

## **加入動態路由頁面**

執行 `npm run generate` 時，Nitro 爬蟲會先抓取靜態頁面，以及有被連結到的動態路由，沒有被連結到的動態路由必須手動配置

**範例：**動態路由結構如下

```
pages/
｜—— user/
  ｜—— [id].vue
```

在 `nuxt.config` 手動配置預渲染路由，也可以透過 `ignore/` 排除路由

```jsx
// nuxt.config.js
defineNuxtConfig({
  nitro: {
    prerender: {
      routes: ['/user/1', '/user/2'],
      ignore: ['/dynamic']
    }
  }
});
```

執行 `npm run generate` 編譯後，在 `.output` 可以看到 `/user/1`、`/user/2` 也預先編譯為靜態頁

```
.output/
|—— public/
  |—— user/
    |—— 1/
      |—— _payload.json
      |—— index.html
    |—— 2/
      |—— _payload.json
      |—— index.html
  |—— ...
```

---

## **延伸：自訂網域**

除了使用 GitHub 提供的免費域名，也可以購買自己的網域，先前寫過一篇文章提供參考：[GitHub Pages 自訂域名與 HTTPS 設定](https://clairechang.tw/2023/06/28/web/github-pages-with-custom-domain/)

---

參考資源：

https://nuxt.com/docs/getting-started/deployment
https://github.com/lucpotage/nuxt-github-pages