---
title: Nuxt.js 3.x Nuxt DevTools 提升開發者體驗
date: 2023-10-27
tags: [ nuxt3 ]
category: Nuxt3
description: Nuxt DevTools 是 Nuxt 團隊推出的視覺化開發工具，用來協助我們快速了解 Nuxt 專案的內容，近一步提升開發者體驗（DX，Developer Experience）
image: https://i.imgur.com/pgm8UJg.png
---

<div style="display: flex; justify-content: center; margin: 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/pgm8UJg.png">
</div>

## **為什麼需要 Nuxt DevTools？**

[Nuxt DevTools](https://devtools.nuxt.com/) 是 Nuxt 團隊推出的視覺化開發工具，用來協助我們快速了解 Nuxt 專案的內容，近一步提升開發者體驗（DX，Developer Experience）。

{% colorquote info %}
關於開發者體驗（DX）介紹，這裡推薦一篇[文章](https://medium.com/@chiunhau/工程師心中最軟的一塊-談前端開發者體驗-developer-experience-96e0cfacb316)
{% endcolorquote %}

Nuxt3 提供許多內建功能，例如：

- 預設使用 Vite 打包工具，支援 hot module replacement（HMR）
- 引入伺服器引擎 Nitro，讓我們可以將應用程式部署到 Vercel、Netlify、Cloudflare 等託管服務
- 許多內建功能，像是支援 TypeScript、各種組合式函式、SEO 搜尋引擎優化輔助函式等
- auto-imports 自動引入功能
- 依據檔案結構自動生成路由與 API 路徑
- 提供許多模組，讓開發者可以快速整合所需的功能，不需另外進行配置

這些功能是 DX 最直接具體的實踐，雖然能讓我們能輕鬆建立 Nuxt3 專案，但也會面臨**「資訊透明度不足」**的問題。抽象化設計是一把雙面刃，在化繁為簡、避免重工的同時，也會增加額外的學習負擔與除錯困難。

Nuxt DevTools 便是為了解決此問題而設計的視覺化工具，能夠提升 Nuxt 框架和應用程式的透明度、找到效能瓶頸，以及協助我們管理應用和配置。

<!-- more -->

## **安裝 Nuxt DevTools**

{% colorquote info %}
注意：Nuxt 版本需為 v3.1.0 以上
{% endcolorquote %}

### **自動安裝**

如果是透過 [Nuxt CLI 安裝](https://clairechang.tw/2023/06/17/nuxt3/nuxt-v3-installation/)，預設會一起安裝與啟用 Nuxt DevTools

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  devtools: { enabled: true }
})
```

### **手動安裝**

安裝 nuxt 整合模組 `@nuxt/devtools`

```bash
npm i -D @nuxt/devtools
```

接著在 `nuxt.config` 配置

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  modules: [
    '@nuxt/devtools'
  ]
})
```

要關閉 devtools，設定 `enabled: false` 即可，devtools 模組相關配置 [參考文件](https://github.com/nuxt/devtools/blob/main/packages/devtools-kit/src/_types/options.ts#L6)

---

# **Nuxt Devtools 導覽**

安裝並啟用 devtools 後，可以在畫面底部看到一顆 icon，點擊後繼續點擊左側 icon 開啟工具面板

<video controls width="100%">
  <source src="https://i.imgur.com/0HLX9Qa.mp4" type="video/mp4" />
</video>

## **Overview**

可以看到專案 Nuxt 與 Vue 版號，以及頁面、元件、數量、引入的函式、模組、插件數量，也可以直接透過畫面操作升級 Nuxt 版本

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/fbEE8GV.png">
</div>

## **Pages**

顯示頁面路徑與相關資訊。以下為例，當前頁面為 `/about`，layout 為 `default`，點擊其他路徑可以切換頁面

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/NVWVTJq.png">
</div>

## **Components**

顯示專案內使用的元件以及來源路徑，後面會標示該元件在本頁的使用次數（ex：x1），點擊路徑會自動在編輯器內開啟檔案

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/3RBMZ4f.png">
</div>

{% colorquote info %}
如果使用 VSCode 開發，點擊路徑不會自動開啟檔案，且終端機拋以下錯誤

```
Could not open TheHeader.vue in the editor.                                                                         6:04:57 PM
The editor process exited with an error: spawn code ENOENT.
```

在 VSCode 執行 [install code command in Path](https://code.visualstudio.com/docs/setup/mac#_launching-from-the-command-line) 即可解決
{% endcolorquote %}

點擊右側或是底部的錨點 icon，接著可以直接透過畫面檢查元件，點擊元件會自動在編輯器開啟檔案

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/y4IEAAw.png">
</div>

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/d5C5TGr.png">
</div>

**元件分為四類：**

- **User components：**專案內自訂的元件
- **Runtime components：**運行的時候使用的元件
- **Build-in components：**內建元件，像是 `<ClientOnly />`
- **Components from libraries：**第三方套件或是模組提供的元件，像是 `<NuxtWelcome />`

也可以透過右上角 Toggle View 按鈕來切換為**圖形視圖**，圖形視圖可以更清晰的顯示元件間的關聯與依賴

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/NEk1klA.png">
</div>

## **Imports**

顯示專案內自動引入（auto-imports）的函式名稱與來源路徑。同 Components，後面會標示該函式在本頁的使用次數（ex：x1），點擊路徑會自動在編輯器內開啟檔案

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/S3yVkZR.png">
</div>

**自動引入函式分為三類：**

- **User composables：**專案內自訂的函式，像是 Composibles 組合式函式或是 Utils 輔助函式
- **Build-in composables：**Nuxt 內建函式，像是 `useNuxtApp()`、`navigateTo()`
- **Composables from libraries：**第三方套件或是模組提供的函式，像是 `@pinia/nuxt` 模組的 `defineStore()`

## **Modules**

顯示專案內安裝的模組與相關資訊，像是 github repo、官方文件、版本資訊等

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/v9ELe3t.png">
</div>

也可以直接透過操作面板來安裝模組或是升版

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/SKRV6xk.png">
</div>

## **Assets**

顯示 `public/` 目錄內的靜態檔，可以看到詳細的檔案資訊、下載、更新名稱、刪除，也可以透過面板上傳檔案

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/BNc2wOt.png">
</div>

## **Runtime Configs**

顯示專案 App Config 與 Runtime Config 設定，並可透過面板進行編輯

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/kO6TV8C.png">
</div>

## **Payload**

顯示透過 `useState`、`useAsyncData`、`useFetch` 建立的狀態與資料封包，可以由面板手動操作 `useAsyncData`、`useFetch` 重新發出請求

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/AVQtu3V.png">
</div>

## **Open Graph**

顯示該頁面的 Meta Tags 配置，以及缺少的 Tags 資訊，協助我們進行 SEO 搜尋引擎優化，並提供在 Twitter、Facebook 和 LinkedIn 等社群的預覽畫面

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/diSUvbQ.png">
</div>

複製 Code Snippet 程式碼，即可快速補全缺少的 Meta Tags

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/PP0IW8y.png">
</div>

## **Plugins**

顯示專案內所有插件，因為插件會專案載入前就執行，因此執行時間會影響整體效能，可以透過右側的執行時間來找出效能瓶頸

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/luwjUoy.png">
</div>

## **Timeline**

追蹤網頁操作每個事件執行時間，可以協助我們分析並優化網頁效能

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/W5uxKfj.png">
</div>

## **Server Routes**

顯示專案內在 `server/` 目錄建立的 API 或是 sitemap 等在伺服器端建立的路由

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/cajvK1g.png">
</div>

## **Hooks**

協助我們監控瀏覽器端與伺服器端每個 hook 花費時間、註冊了多少監聽器，以及被調用的次數，有助於找出效能瓶頸

## **Virtual Files**

Nuxt 提供了一套虛擬文件系統（VFS，Virtual File System），根據我們的目錄結構自動生成虛擬檔案。透過 devtools，不需進到 `.nuxt` 資料夾，即可查看相關檔案，對於進階 debug 很有幫助

## **Inspect**

整合 [vite-plugin-inspect](https://github.com/antfu/vite-plugin-inspect) 套件，讓我們可以檢視各插件或檔案透過 Vite 編譯、轉譯的過程，對於找出潛在風險很有幫助

---

參考資源：

https://devtools.nuxt.com/
https://nuxt.com/blog/introducing-nuxt-devtools