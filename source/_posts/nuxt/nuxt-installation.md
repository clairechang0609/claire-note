---
title: Nuxt.js 專案架設
date: 2022-11-16
tags: [ nuxt2 ]
category: Nuxt
description: Nuxt.js 是基於 Vue.js、Node.js、Webpack 以及 Badel.js 的輕量框架，可以同時創建 SSR（Server Side Render）及 SPA，在頁面載入前即渲染（伺服器回傳完整 HTML 檔，每次跳轉頁面，瀏覽器都需要刷新），搜尋引擎爬蟲可以取得資料，大幅解決 SEO 的問題
image: https://i.imgur.com/s38N3oo.png
---
> **版本：nuxt 2.15.8**
>

說到 Nuxt，必須先從 Vue.js 說起，Vue.js 為專注在視圖層(View) 的 JS 框架，為 SPA（Single Page Application）應用程式，簡而言之整個網站應用只有單一頁面，一但頁面被加載進來後，就不會再進行該頁面請求，由於 Vue 是利用 JS 載入資料，並動態產生元件，SEO 只能抓取到 HTML 內容，因此 SEO 表現趨近於零。

而 Nuxt 是基於 Vue.js、Node.js、Webpack 以及 Badel.js 的輕量框架，可以同時創建 SSR（Server Side Render）及 SPA，在頁面載入前即渲染（伺服器回傳完整 HTML 檔，每次跳轉頁面，瀏覽器都需要刷新），搜尋引擎爬蟲可以取得資料，大幅解決 SEO 的問題。

以下圖片說明：

![](https://i.imgur.com/Qyp5Sat.png)

<!-- more -->

接下來一起來嘗試創建一個 Nuxt 專案吧！

如同 Vue CLI，Nuxt 也有類似的指令列(command-line)工具 create-nuxt-app，

依據官方文件提供的專案包建置方式：

`npx create-nuxt-app <project-name>`

{% colorquote info %}
使用 npx 安裝，安裝的套件在執行完後就會被移除 [npx介紹](https://hoyis-note.coderbridge.io/2021/07/20/npm-npx-%E5%B7%AE%E5%88%A5/)
{% endcolorquote %}

**接著會跑出一些選項：**

- **Project name：**設定專案名稱
- **Programming language：**選擇程式語言
- **Package manager：**軟體套件管理系統 npm / yarn
- **UI framework：**css 模板
- **Template engine：**樣版引擎
- **Nuxt.js modules：**相依套件
- **Linting tools：**程式碼檢查工具
- **Testing framework：**測試工具
- **Rendering mode：**渲染模式
- **Deployment target：**運行模式 （在此示範 Server Side Render)
- **Development tools：**開發工具
- **Version control system：**版控工具

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img style="width: 100%; max-width: 600px;" src="https://i.imgur.com/MSYs5f0.png">
</div>

以上選擇完畢就開始安裝專案包，運行完成就可以開始編譯專案，跟著提示訊息執行：
`cd test`
`npm run dev`

在 package.json 內可以看到相關指令

```bash
"scripts": {
    "dev": "nuxt",
    "build": "nuxt build",
    "start": "nuxt start",
    "generate": "nuxt generate"
}
```

接下來就可以看到畫面囉 👏

![](https://i.imgur.com/s38N3oo.png)

---

參考文章：

[https://medium.com/web-design-zone/建立nuxt-js專案初體驗-21920735e38b](https://medium.com/web-design-zone/%E5%BB%BA%E7%AB%8Bnuxt-js%E5%B0%88%E6%A1%88%E5%88%9D%E9%AB%94%E9%A9%97-21920735e38b)

[https://levelup.gitconnected.com/spa-ssg-ssr-and-jamstack-a-front-end-acronyms-guide-6add9543f24d](https://levelup.gitconnected.com/spa-ssg-ssr-and-jamstack-a-front-end-acronyms-guide-6add9543f24d)