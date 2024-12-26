---
title: Nuxt3 - Nuxt Image 圖片最佳化
date: 2024-12-08
tags: [ nuxt3 ]
category: Nuxt3
description: 網站的圖片與網站體驗指標（Web Vitals）密切相關，因為頁面上的主要元素通常包含圖片。本篇將使用 @nuxt/image 模組，說明如何在 Nuxt3 進行圖片優化。
image: /images/nuxt3/nuxt-image/example-1.png
---

> 本篇搭配 @nuxt/image v1.8.0
> 

[網站體驗指標（Web Vitals）](https://web.dev/articles/vitals) 是 Google 定義的一組用來衡量網站使用者體驗的指標，這些指標反映了使用者在瀏覽網站時的實際體驗，會影響網站在搜尋引擎上的排名。

自 2024 年 3 月起，三大核心指標（Core Web Vitals）更新如下：

- **最大內容繪製**（LCP，Largest Contentful Paint）：測量頁面主要內容的載入效能
- **互動至下一次繪製**（INP，Interaction to Next Paint）：測量頁面在使用者互動過程中的回應速度
- **累積版面位移**（CLS，Cumulative Layout Shift）：測量視覺穩定性，避免不預期的版面配置位移

**網站的圖片**與這些核心指標密切相關，因為頁面上的主要元素通常包含圖片。圖片的大小直接影響載入速度，進而影響使用者互動的回應時間。如果圖片未設定固定的寬高比例，甚至會導致版面配置位移，進而影響使用者體驗。

<!-- more -->

## **圖片優化方向**

- **轉換圖片格式：**`webp` 格式的檔案通常比 `png` 和 `jpeg` 等傳統格式小很多，且幾乎不會失真
- **延遲載入圖片：**使用 Lazy Loading，當圖片進入可視區域時才載入，減少頁面的初始載入時間
- **設置寬高屬性：**為圖片設置明確的寬高屬性，避免發生版面位移
- **設定響應式尺寸：**為不同斷點設置對應的圖片尺寸，根據裝置大小載入適合的圖片

---

本篇將使用 [@nuxt/image](https://image.nuxt.com/) 模組，進行圖片優化。

## **安裝 @nuxt/image**

### **Step1：套件安裝**

```bash
npm i @nuxt/image
```

### **Step2：nuxt.config 配置模組**

在 `nuxt.config` 加入模組，並根據需求透過 `image` 自訂配置。詳細選項請參考官方文件。

**常用選項：**

- **`quality`：**設定圖片品質
- **`screens`：**設定螢幕尺寸斷點
- **`domains`：**若要啟用外部圖片的最佳化，需指定網域
- **`densities`：**為高解析度顯示器（`devicePixelRatio > 1`）指定多個像素密度的圖片
- **`dir`：**指定專案內存放靜態圖片的位置，預設為 `public`
- **`providers`：**設定影像最佳化服務提供商，預設搭配 `ipx`，具體調整方法請參考後續範例

```jsx
// nuxt.config.ts
export default defineNuxtConfig({
  modules: [
    '@nuxt/image'
  ],
  image: {
    quality: 80,
    screens: {
      xs: 320,
      sm: 640,
      md: 768,
      lg: 992,
      xl: 1280,
      xxl: 1536,
      xxxl: 1920
    },
    domains: ['https://my-website.com'],
    densities: [1, 2],
    dir: 'public'
  }
});
```

---

## **實作圖片最佳化**

完成基礎建置後，接下來開始進行實作。

### **`<NuxtImg>` 元件**

用來取代原生的 `<img>` 標籤。

**常用屬性說明：**

- **`src`：**指定圖片的路徑，同 `<img>` 用法
- **`sizes`：**根據螢幕斷點設定圖片尺寸，元件會自動產生對應尺寸的圖片，斷點為 `nuxt.config` 中 `image.screens` 的設定
    
    > 如果未設定斷點，也可以直接使用 `width` 或 `height` 設定圖片的寬高
    > 
- **`format`：**設定圖片格式，例如轉換為 `webp` 格式，來減少檔案大小
- **`loading`：**使用原生的 `lazy` 屬性來延遲載入圖片，僅在圖片進入可視範圍才載入
- **`placeholder`：**圖片載入完成前的佔位符。可以使用預設佔位符或自訂
- **`preload`：**是否預先載入圖片。當圖片位於頁面的首屏時，建議使用 `preload`，並避免延遲載入

**範例一：**

- `src`：使用專案內的靜態圖片 `public/sample.jpg`
- `format`：將圖片轉換為 `webp` 格式
- `loading`：延遲載入圖片
- `width` / `height`：圖片設定固定寬高
- `placeholder`：設定佔位符為 `[ 50, 25, 60, 5 ]`，依序為寬、高、品質與模糊程度

```html
// pages/index.vue
<template>
  <NuxtImg
    src="/sample.jpg"
    format="webp"
    loading="lazy"
    width="500"
    height="250"
    :placeholder="[50, 25, 60, 5]"
  />
</template>
```

開啟瀏覽器，一開始先顯示佔位符，直到圖片載入完成後才替換。

![example-1.png](../images/nuxt3/nuxt-image/example-1.png)

比較轉換為 `webp` 格式的圖片與原始圖片，檔案大小顯著減少，同時保持圖片的清晰度：

<div class="column-wrap">
  <div style="display: flex; flex-direction: column; justify-content: left; margin: 20px 0;">
    <img style="width: 100%; max-width: 100%; margin-bottom: 5px; border: 1px solid #dbdbdb;" src="/images/nuxt3/nuxt-image/nuxt-image.png">
    <small>&lt;NuxtImage&gt;</small>
  </div>
  <div style="display: flex; flex-direction: column; justify-content: left; margin: 20px 0;">
    <img style="width: 100%; max-width: 100%; margin-bottom: 5px; border: 1px solid #dbdbdb;" src="/images/nuxt3/nuxt-image/html-image.png">
    <small>&lt;image&gt;</small>
  </div>
</div>

**範例二：**

- `src`：使用遠端圖片（需在 `nuxt.config` 的 `image.domains` 配置網域）
- `format`：將圖片轉換為 `webp` 格式
- `preload`：預先載入圖片
- `sizes`：根據螢幕斷點設定圖片尺寸（斷點對應 `nuxt.config` 的 `image.screens`）

```html
<template>
  <NuxtImg
    src="https://my-website.com/photo-qz4090M"
    format="webp"
    preload
    sizes="75vw md:50vw lg:500px"
  />
</template>
```

在瀏覽器查看結果，`<head>` 自動加入 `<link rel="preload">`，而 `<img>` 標籤則會根據 `sizes` 的設定生成不同尺寸的圖片，瀏覽器會自動選擇合適的圖片載入。

![example-2.png](../images/nuxt3/nuxt-image/example-2.png)

**範例三：使用 [Cloudinary](https://cloudinary.com/) 進行圖片優化**

Nuxt Image 預設搭配 `ipx` 優化圖片，也可以更換為其他服務商。以下搭配 `cloudinary`，詳細的服務商選擇及自訂請參考官方文件。

首先在 `nuxt.config` 設定服務商，`<your-cloud-name>` 需替換為自己的 `cloudinary` 名稱：

```jsx
export default defineNuxtConfig({
  image: {
    // ...
    cloudinary: {
      baseURL: 'https://res.cloudinary.com/<your-cloud-name>/image/upload/'
    }
  }
})
```

接著使用 `<NuxtImg>` 元件插入圖片：

- `provider`：圖片服務提供商設為 `cloudinary`
- `src`：只需提供圖片的相對路徑，會自動加入上一步設定的 `baseURL`。例如：`v1721532815/DAN05488_zpryx1.jpg`
- `format`：將圖片轉換為 `webp` 格式
- `width`：設定圖片寬度為 500px

```html
<template>
  <NuxtImg
    provider="cloudinary"
    src="v1721532815/DAN05488_zpryx1.jpg"
    format="webp"
    width="500"
  />
</template>
```

開啟瀏覽器，查看經由 Cloudinary 優化後的圖片：

![example-3.png](../images/nuxt3/nuxt-image/example-3.png)

---

<div style="margin-top: 20px; margin-bottom: 0;">
  <img style="width: 100%; max-width: 200px;" src="/images/nuxt3/book.jpg">
</div>

● 更多內容歡迎參考我的書：[Nuxt3 入門－打造 SSR 專案](https://www.tenlong.com.tw/products/9786267569313?list_name=r-zh_tw)