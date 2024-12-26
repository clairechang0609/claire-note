---
title: Nuxt.js 3.x Universal（SSR）專案部署到 Firebase
date: 2023-10-08
tags: [ nuxt3, firebase ]
category: Nuxt3
description: 本篇將說明 Nuxt3 Universal 專案部署到 Firebase。 Universal 渲染模式需要伺服器才能運作，因此會搭配 Firebase Cloud Functions（Serverless）來協助處理 SSR 的部分。
image: https://i.imgur.com/r80Hush.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/articles/10336541)
>

專案開發完成後，終於到最後一步**「部署」**，本篇將說明 Universal 專案部署到 [Firebase](https://firebase.google.com/)。Universal 渲染模式需要伺服器才能運作，因此會搭配 Firebase Cloud Functions（Serverless）來協助處理 SSR 的部分。

{% colorquote info %}
Universal Rendering 相關說明可以參考 [這篇文章](https://clairechang.tw/2023/08/22/nuxt3/nuxt-vs-vue/#Universal-Rendering)
{% endcolorquote %}

**實作分為以下步驟：**

- 申請 Firebase Project
- 透過 CLI 部署至 Firebase

<!-- more -->

## **申請 Firebase Project**

到 [Firebase Console](https://console.firebase.google.com/) 新增專案

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/AJhGnFX.png">
</div>

開啟新增的專案，點擊建構 → 加入 Hosting 與 Function（需升級為付費方案，為 Pay as you go 制度，可以限制預算上限）

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/mCT0VbR.png">
</div>

到此步 Firebase 專案就建立完成，先記著專案 ID，後面部署會使用到

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/wR8QFZ7.png">
</div>

---

## **透過 CLI 部署至 Firebase**

接著回到我們的 Nuxt App

### **STEP1：安裝 CLI**

首先安裝 Firebase CLI（Command-Line Interface）指令列工具

```bash
npm install firebase-tools
```

### **STEP2：登入 Firebase**

接著透過指令登入 Firebase

```bash
firebase login
```

登入成功後會看到如下畫面

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/34GCwuw.png">
</div>

### **STEP3：初始化 firebase 專案**

```bash
firebase init hosting
```

接著選擇預部署的 Firebase 專案 ID，也就是前面新增的專案

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/WtYWaGr.png">
</div>

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/0Zl4sCI.png">
</div>

託管設定依序選擇如下，接著會自動產生 `firebase.json` 跟 `.firebaserc` 兩支檔案

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/owkwSt4.png">
</div>

**注意：**必須要刪除上一步驟自動產生的 `public/404.html` 跟 `public/index.html` 兩個靜態檔，否則部署後畫面會受影響

<div style="display: flex; justify-content: left; margin: 30px 0;">
  <img style="width: 100%; max-width: 300px;" src="https://i.imgur.com/Dvkn3Q8.png">
</div>

### **STEP4：配置 nuxt.config**

- `preset`：預設為 node.js server，調整為 firebase，當執行 `npm run build` 時，會產生符合 Firebase 託管所需的檔案內容
- `firebase.gen`：預設使用第一代 Cloud Functions，這裡調整為二代（[一、二代比較](https://firebase.google.com/docs/functions/version-comparison)）

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  nitro: {
    preset: 'firebase',
    firebase: {
      gen: 2
    }
  }
});
```

### **STEP5：配置 `firebase.json` 與 `.firebaserc`**

#### **`firebaserc`**

如果前面步驟運作順利，預期 `.firebaserc` 會綁定 Firebase 專案 ID 如下，不需異動

```jsx
// .firebaserc
{
  "projects": {
    "default": "<your_project_id>"
  }
}
```

#### **`firebase.json`**

需手動調整，這個步驟很重要，會影響專案編譯結果

**functions（Cloud Functions）**

- `source`：指定 Cloud Functions 程式碼路徑

**hosting（Firebase Hosting）**

- `site`：Firebase 專案 ID
- `public`：專案靜態目錄路徑
- `rewrites`：改寫 URL 規則，所有的 URL 請求都會由名為「server」的 Cloud Functions 處理，也就是由伺服器端處理所有請求（SSR）
- `predeploy`：部署前預先執行的指令。這一段的設定為：進到編譯後的 `.output/server` 資料夾，刪除 `node_modules` 並重新安裝，否則在執行部署時會發生找不到 `firebase-functions` 套件的錯誤

{% colorquote info %}
注意：`rm -rf node_modules` 是 macOS 跟 Linux 的指令，Windows 須調整為 `rmdir /s /q node_modules`
{% endcolorquote %}

```jsx
// firebase.json
{
  "functions": {
    "source": ".output/server",
    "runtime": "nodejs18"
  },
  "hosting": {
    "site": "<your_project_id>",
    "public": ".output/public",
    "cleanUrls": true,
    "rewrites": [ {
      "source": "**",
      "function": "server"
    }],
    "predeploy": [
      "cd .output/server && rm -rf node_modules && npm install"
    ]
  }
}
```

### **STEP6：編譯與部署**

首先將專案打包

```bash
npm run build
```

打包後的檔案放在根目錄 `.output` 資料夾內

```
.output/
|—— public/
  |—— ...
|—— server/
  |—— ...
|—— nitro.json
```

接著部署到 Firebase

```bash
firebase deploy
```

看到以下訊息表示部署成功囉！

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/9HyX6sU.png">
</div>

在 Firebase 專案可以看到 Cloud Function 名稱預設為「server」

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/A2qIFu6.png">
</div>

{% colorquote info %}
如果發生 funciton 部署失敗問題，可以在 `nuxt.config` 設定 `serverFunctionName` 如下

```jsx
// nuxt.config.js
export default defineNuxtConfig({
  nitro: {
    preset: 'firebase',
    firebase: {
      gen: 2,
      serverFunctionName: 'server'
    }
  }
})
```
{% endcolorquote %}

打開 Hosting URL，可以看到我們的網站，也可以透過 Firebase 儀表板來查看發布紀錄與用量

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/xmHqFJE.png">
</div>

### **範例檔：**

本篇程式碼放在 GitHub 提供參考：[連結](https://github.com/clairechang0609/nuxt3-firebase)

---

參考資源：

https://nitro.unjs.io/deploy/providers/firebase
https://firebase.google.com/docs/hosting/quickstart
https://www.youtube.com/watch?v=AzO-KVMx7lo