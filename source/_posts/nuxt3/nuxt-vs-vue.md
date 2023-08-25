---
title: Nuxt.js vs Vue.js，淺談 SPA 與 SSR
date: 2023-08-22
tags: [ nuxt3, nuxt ]
category: [ Nuxt3 ]
description: 簡單以圖示說明 SPA、MPA、CSR 與 SSR，Nuxt.js 與 Vue.js 的架構比較，以及選擇的時機
image: https://imgur.com/sCQuKXt.png
---

<div style="display: flex; justify-content: center; margin: 0;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/EyMJEAV.png">
</div>

提到 Nuxt.js，必須先從 Vue.js 說起，Vue.js 為專注在視圖層（View）的 Javascript 框架，為 SPA（Single Page Application）單頁應用程式，搜尋引擎爬蟲未能抓取到渲染後的 HTML 內容，因此 SEO（Search Engine Optimization）表現趨近於零。

而 Nuxt.js 是基於 Vue.js 的框架，為通用渲染模式（Universal Rendering），結合了 SSR（Server Side Render）及 CSR（Client Side Render），搜尋引擎爬蟲可以取得 HTML 內容，大幅提升 SEO 表現。

<!-- more -->

## **SPA / CSR vs MPA / SSR**

<div class="column-wrap" style="margin: 30px 0">
    <div>
        <div style="display: flex; justify-content: left;">
            <img style="width: 100%; max-width: 100%;" src="https://imgur.com/sCQuKXt.png">
        </div>
    </div>
    <div>
        <div style="display: flex; justify-content: left;">
            <img style="width: 100%; max-width: 100%;" src="https://imgur.com/BqLpa1p.png">
        </div>
    </div>
</div>

### **SPA（Single Page Application）單頁式應用**

使用 CSR（Client Side Render）用戶端渲染模式。整個網站應用只有單一 HTML 頁面，一但頁面被載入進來後，就不會再進行該頁面請求（Form Request），而是透過 AJAX 從後端請求資料，並透過 Javascript 動態更新與渲染網頁內容。

採 CSR（Client Side Render）用戶端渲染模式。

- **優點：**使用者體驗佳，因每次切換頁面時只有部分畫面更新，不會重新載入整個頁面
- **缺點：**搜尋引擎爬蟲未能抓取到渲染後的 HTML 內容，因此 SEO（Search Engine Optimization）表現趨近於零

### **MPA（Multi Page Application）多頁式應用**

使用 SSR（Server Side Render）伺服器端渲染模式。在頁面載入前即渲染，每次跳轉頁面，瀏覽器都會重新向伺服器發送頁面請求，由伺服器回傳頁面的 HTML 內容。

- **優點：**在伺服器上生成完整的 HTML 頁面，首次內容繪製（FCP，First Contentful Paint）速度快，且搜尋引擎爬蟲可以取得完整 HTML 內容，SEO 表現佳
- **缺點：**使用者體驗較差，因每次切換頁面都要重新刷新頁面，且對伺服器負擔較大

---

## **Universal Rendering**

通用渲染，**Nuxt.js 使用的渲染模式**，同時支援 SSR 與 CSR 技術，初次進到頁面時採用 SSR 模式，在伺服器中產生完整 HTML 內容，回傳給瀏覽器端。後續動態切換頁面時，則採用 CSR 模式，結合了 SSR 與 SPA 的優點，除了使用者體驗佳，同時保有良好的 SEO 表現。

<div style="display: flex; justify-content: center; margin: 30px 0 10px;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/KuL0KPL.png">
</div>

<small>圖片參考：[Nuxt 官網](https://nuxt.com/docs/guide/concepts/rendering#universal-rendering)</small>

### **流程：**

1. 伺服器端產生靜態 HTML，回傳給瀏覽器端
2. 瀏覽器端載入 JavaScript 並執行
3. Hydration 混合渲染完成，使用者可以開始與網頁互動

{% colorquote info %}
**Hydration 混合渲染：**

在用戶端使用 JavaScript，讓伺服器端產生的靜態 HTML 加上 Event Handlers（事件處理器），讓使用者能與網頁進行互動的技術，Nuxt.js 跟 React 框架 Next.js 即使用此技術
{% endcolorquote %}

---

## **Nuxt3 簡介**

- 使用 Vue3 開發
- 完整 SSR 支援，也可以選用 CSR 模式
- 自動定義路由（使用 Vue-Router），不需手動配置
- 支援 webpack 5 與 Vite 開發（預設搭配 Vite）
- 完整支援 TypeScript
- 定義在 composables、components、plugins 的檔案，Nuxt 會自動引入（Auto-imports）
- 內建支援 SSR 的 AJAX 請求方法（$fetch、useFetch、useAsyncData）

## **Nuxt or Vue**

Vue.js SPA 架構使用者體驗佳，如果專案為後台系統，或是不需要 SEO，選擇 Vue.js 開發相對單純。

Nuxt.js 是基於 Vue.js 開發的框架，適合已經有 Vue.js 或其他 JavaScript 框架使用經驗的開發者，如果專案需要良好的 SEO 表現，推坑 Nuxt.js，安裝方式簡單、自動化繁瑣的步驟，提供很不錯的開發體驗。

---

參考資源：

https://nuxt.com/docs/guide/concepts/rendering#universal-rendering
https://shubo.io/rendering-patterns/
https://ithelp.ithome.com.tw/articles/10262891