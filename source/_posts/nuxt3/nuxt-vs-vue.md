---
title: Nuxt 與 Vue 比較，說明 SSR 基本概念
date: 2023-08-22
updated_at: 2024-12-09
tags: [ nuxt3 ]
category: [ Nuxt3 ]
description: Nuxt.js 是基於 Vue.js 的框架，預設使用通用渲染模式（Universal Rendering），結合了伺服器端渲染（SSR，Server-Side Rendering）和客戶端渲染（CSR，Client-Side Rendering）的優點。
image: /images/nuxt3/nuxt-vs-vue/nuxt-vue.svg

---

<div style="width: 100%; max-width: 600px; margin: 50px auto;">
  <img style="width: 100%; max-width: 100%;" src="/images/nuxt3/nuxt-vs-vue/nuxt-vue.svg">
  <small>source: Nuxt.js and Vue.js</small>
</div>

了解 [Nuxt.js](https://nuxt.com/) 之前，需先從 [Vue.js](https://vuejs.org/) 說起。Vue.js 是一個專注於視圖層的 JavaScript 框架，用於建立單頁應用程式（SPA，Single Page Application）。在 SPA 架構中，使用者首次訪問網站時會下載一個 HTML 文件和所有的 JavaScript 資源，之後的頁面切換和更新都是透過 JavaScript 動態完成，這樣的設計可以減少頁面重新載入的次數，進而提升使用者體驗。

不過，SPA 架構也存在一個主要缺點：搜尋引擎爬蟲只會讀取 HTML，不會執行 JavaScript。因此，可能會導致動態產生的內容無法被正確索引，影響網站在搜尋引擎上的表現。

Nuxt.js 是基於 Vue.js 的框架，預設使用通用渲染模式（Universal Rendering），結合了伺服器端渲染（SSR，Server-Side Rendering）和客戶端渲染（CSR，Client-Side Rendering）的優點。透過在伺服器端生成完整的 HTML 內容並回傳給瀏覽器，讓搜尋引擎爬蟲可以取得較完整的 HTML 內容，以改善 SPA 架構下 SEO 成效不佳的問題。


<!-- more -->

---

## **Vue.js：SPA + CSR**

**SPA（Single Page Application）單頁式應用 ＋ CSR（Client-Side Rendering）客戶端渲染**

在 Vue.js 中，整個網站應用只有單一 HTML 頁面。一旦頁面被載入進來後，就不會再進行該頁面請求，而是透過 AJAX 從後端請求資料，並透過 JavaScript 動態更新與渲染網頁內容。

**優點：**

- **使用者體驗佳：**每次切換頁面時，只有部分畫面會更新，不會重新載入整個頁面
- **頁面回應速度快：**只需更新部分內容，頁面切換速度更快

**缺點：**

- **SEO 表現不佳：**搜尋引擎爬蟲未能抓取到完整的 HTML 內容
- **初始載入效能問題：**應用程式的程式碼和資源只在瀏覽器中執行，使用者需等待瀏覽器載入並解析這些文件，初次載入頁面的時間也可能較長

<div style="width: 100%; max-width: 500px; margin: 50px auto;">
  <img style="width: 100%; max-width: 100%;" src="/images/nuxt3/nuxt-vs-vue/CSR.png">
  <small>單頁式應用與客戶端渲染架構下的頁面載入流程</small>
</div>

---

## **傳統網站：MPA + SSR**

**MPA（Multi Page Application）多頁式應用 ＋ SSR（Server-Side Rendering）伺服器端渲染**

傳統的網站，如基於 PHP 或 Ruby 等後端語言的架構，每個頁面都是獨立的 HTML 文件。每次跳轉頁面時，瀏覽器都會向伺服器發送新的頁面請求，由伺服器生成並回傳新的 HTML 內容。

**優點：**

- **SEO 表現佳：**在伺服器上生成完整的 HTML 文件，搜尋引擎爬蟲可以索引完整的內容
- **首次內容繪製（FCP）速度快：**伺服器端回傳完整的 HTML，頁面可以快速顯示

**缺點：**

- **使用者體驗較差：**每次切換頁面都需要重新刷新整個頁面
- **伺服器負擔大：**每次請求都需要重新產生完整的 HTML 文件，對伺服器資源需求較高

<div style="width: 100%; max-width: 500px; margin: 50px auto;">
  <img style="width: 100%; max-width: 100%;" src="/images/nuxt3/nuxt-vs-vue/SSR.png">
  <small>多頁式應用與伺服器端渲染架構下的頁面載入流程</small>
</div>

---

## **Nuxt.js：Universal Rendering（SSR + CSR）**

Nuxt 預設渲染模式，結合了伺服器端渲染（SSR）和客戶端渲染（CSR）的優點。當使用者首次進入頁面時，使用 SSR，在伺服器端生成完整的 HTML 內容並回傳給瀏覽器；後續動態切換頁面時則使用 CSR。這種方式結合了 SSR 和 SPA 的優點，不僅提供了良好的 SEO 表現，還能維持動態互動體驗。

### **Universal Rendering 的網頁載入流程**

**Step1：伺服器端渲染**

當瀏覽器請求 URL 時，伺服器端執行 JavaScript（Vue.js）程式碼，將 Vue 元件轉換成一個完整渲染的 HTML 頁面，包含必要的樣式，類似於傳統的 SSR 應用程式。

**Step2：客戶端（瀏覽器）載入 HTML 頁面**

瀏覽器接收並載入伺服器傳回的 HTML 頁面，因此在初次載入時，使用者即可看到完整的頁面模板。

**Step3：客戶端再次執行 JavaScript**

由 Vue.js 接管，瀏覽器載入並再次執行 JavaScript（Vue.js）程式碼，建立一個與伺服器端相同的應用實體，將伺服器端生成的靜態 HTML DOM 節點加上事件處理器（Event Handlers），使得使用者能夠與網頁進行互動。這個技術稱為**「Hydration」**。Nuxt.js 和 React 框架 Next.js 都使用此技術。

**Step4：網頁具互動性**

Hydration 步驟完成後，網頁即具備完整的互動性。

**Step5：由 Vue.js 接管使用者互動**

後續的網頁互動以及頁面切換由 Vue.js 接管，保留了 SPA 的優點。

---

<div style="display: inline-flex; flex-direction: column; align-items: center; margin-top: 20px;">
<img style="display: inline-block; width: 100%; max-width: 200px;" src="/images/nuxt3/book.jpg">
<a href="https://www.tenlong.com.tw/products/9786267569313?list_name=r-zh_tw" style="color: #1b6655; text-align: center;">
⭒ 新書上架，更多內容歡迎參考 ⭒<br />
<strong>Nuxt3 入門－打造 SSR 專案</strong>
