---
title: Nuxt.js 設定 GA & GTM
date: 2022-12-14
tags: [ nuxt2, seo, gtag ]
category: Nuxt
description: 網站上線後，我們會需要 Google Analytics 和 Google Tag Manager 工具來追蹤分析網站流量，本文將說明如何在 Nuxt.js 專案內加入追蹤碼
image: https://imgur.com/TMVTu86.png
---
> **版本：nuxt 2.15.8**
>

<div style="display: flex; justify-content: center; margin: 0;">
    <img style="width: 100%; max-width: 600px;" src="https://imgur.com/TMVTu86.png">
</div>

網站上線後，我們會需要 Google Analytics 和 Google Tag Manager 工具來追蹤分析網站流量，接下來說明如何在 Nuxt.js 專案內加入追蹤碼。

### **Google Analytics**

##### **申請代碼：**

1. 至 [Google Analytics 分析](https://analytics.google.com/analytics/web/)，使用 google 帳戶登入，點選左下角的**管理**

<!-- more -->

![](https://i.imgur.com/mfP02by.png)

2. 選擇**建立資源**

![](https://i.imgur.com/1OJjgHR.png)

3. 設定**資源名稱**，選項勾選後，點選建立

![](https://i.imgur.com/vdj1oVP.png)

4. 選擇資料串流平台：**網站**

![](https://i.imgur.com/UrwlaOH.png)

5. 設定**網站網址**跟**串流名稱**，接著就新增成功並取得 GA 代碼。

![](https://i.imgur.com/LznGmph.png)

##### **設定代碼：**

搭配套件 [vue-gtag](https://matteo-gabriele.gitbook.io/vue-gtag/v/master/) ，執行：`npm i vue-gtag@1.16.1`

{% colorquote info %}
Vue v2.x 需搭配 vue-gtag v1.x
{% endcolorquote %}

接著我們新增檔案至 plugins，這裡命名為 gtag.js

```jsx
// plugins/gtag.js
import Vue from 'vue';
import VueGtag from 'vue-gtag';

Vue.use(VueGtag, {
    config: { id: 'UA-XXXXXXXXX-X' }
});
```

配置到 nuxt.config.js

```jsx
// nuxt.config.js
export default {
    plugins: [
        { src: '@/plugins/gtag.js' }
    ]
}
```

### **Google Tag Manager**

##### **申請代碼：**

1. 至 [Google Tag Manager 代碼管理工具](https://tagmanager.google.com/)，使用 google 帳戶登入並建立帳戶

![](https://i.imgur.com/s70CuqO.png)

2. 設定帳戶名稱（命名一個容易辨識網站的名稱），接著設定容器（GTM 要放入的網域），按下建立，就可以看到新增的容器跟 GTM 代碼囉

![](https://i.imgur.com/sPonBte.png)

##### **設定代碼：**

搭配套件 [@nuxtjs/gtm](https://github.com/nuxt-community/gtm-module#readme) ，執行：`npm i @nuxtjs/gtm`，然後配置 nuxt.config.js

```jsx
// nuxt.config.js
export default {
    modules: [
        '@nuxtjs/gtm',
    ],
    gtm: {
        id: 'GTM-XXXXXXX'
    }
}
```

[預設內容](https://github.com/nuxt-community/gtm-module#options) 參考，也可以調整設定

```jsx
// nuxt.config.js
export default {
    modules: [
        '@nuxtjs/gtm',
    ],
    gtm: {
        id: 'GTM-XXXXXXX',
        enabled: true, // for dev mode
        pageTracking: true
    }
}
```

如果代碼都是固定的內容，這樣就完成囉～

不過事情總沒這麼簡單，如果今天專案有後台，GA 跟 GTM 都可供使用者設定，代碼都是變動的參數，又該怎麼做呢？

## **整合後端 API 進行代碼設定**

這裡同時說明 GA 跟 GTM 的配置，首先安裝套件 `npm i vue-gtag@1.16.1` `npm i @nuxtjs/gtm`

由於要串接 API，這裡搭配 [@nuxtjs/axios](https://axios.nuxtjs.org/) 使用，執行：`npm i @nuxtjs/axios`

新增 plugins/gtag.js

```jsx
// plugins/gtag.js
import Vue from 'vue';
import VueGtag from 'vue-gtag';

export default async ({ $gtm, $axios }) => {
    const { gaFour, gtm } = await $axios.$get('/api/ga');

    // GA4
    Vue.use(VueGtag, {
        config: { id: gaFour }
    });

    // GTM
    if (gtm) {
        $gtm.init(gtm);
    }
};
```

配置 nuxt.config.js

```jsx
// nuxt.config.js
export default {
    plugins: [
        { src: '@/plugins/gtag.js' }
    ],
    modules: [
        '@nuxtjs/gtm'
    ],
    gtm: {
        enabled: true,
        pageTracking: true
    }
}
```

---

參考文章：

[https://www.turingdigital.com.tw/blog/install-ga4-property](https://www.turingdigital.com.tw/blog/install-ga4-property)

[https://www.cyberbiz.io/support/?p=228](https://www.cyberbiz.io/support/?p=228)

[https://stackoverflow.com/questions/64612031/setup-google-analytics-4-in-nuxt-js](https://stackoverflow.com/questions/64612031/setup-google-analytics-4-in-nuxt-js)