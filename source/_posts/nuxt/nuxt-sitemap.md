---
title: Nuxt.js 替網站增加 sitemap 網站地圖
date: 2022-12-15
tags: [ nuxt2, seo, sitemap ]
category: Nuxt
description: 當我們的新網站上線，外部連結少，或是網站規模很大時，透過 sitemap 網站地圖，可以讓 google 搜尋引擎爬蟲可以更快速的了解我們的網站上有哪些網頁並加以收錄，並建立索引，提升網頁出現的機率
image: https://i.imgur.com/Ajnso8p.png
---
> **版本：nuxt 2.15.8**
>

當我們的新網站上線，外部連結少，或是網站規模很大時，透過 sitemap 網站地圖，可以讓 google 搜尋引擎爬蟲可以更快速的了解我們的網站上有哪些網頁並加以收錄，並建立索引，提升網頁出現的機率。

我們可以透過 [@nuxtjs/sitemap](https://www.npmjs.com/package/@nuxtjs/sitemap) 來快速建立 sitemap，執行：`npm i @nuxtjs/sitemap`

<!-- more -->

在 config 資料夾新增 sitemap.js 檔案，接著在 nuxt.config.js 配置：

```jsx
// nuxt.config.js
import sitemap from './config/sitemap';

export default {
    modules: [
        '@nuxtjs/sitemap',
    ],
    sitemap: sitemap
}
```

接著移動到到 config/sitemap.js 進行設定。

#### **sitemap 選項說明：**

- **path：**設定 sitemap 名稱，預設為 ****`/sitemap.xml`
- **hostname**：網站 domain
- **cacheTime：**sitemap 更新的頻率（預設為 `1000 * 60 * 15` 15 分鐘）
- **gzip：**是否生成壓縮檔，預設 `false`
- **exclude：**排除被加進 sitemap 的頁面（陣列）
    
    ```jsx
    sitemap: {
        exclude: [
          '/guideline',
          '/sample/**'
        ]
    }
    ```
    
- **defaults：**預設設定（自動帶入 routes）
    
    ```jsx
    sitemap: {
        defaults: {
            changefreq: 'daily', // 更新頻率
            priority: 0.8, // 優先度
            lastmod: new Date() // 最後更新時間
        }
    }
    ```
    
- **filter：**篩選路徑（function）
    
    ```jsx
    sitemap: {
        filter({ routes }) {
            return routes.map(route => {
                // 設定首頁優先度為 1
                if (route.url === '/') {
                    route.priority = 1;
                }
                return route;
            });
        }
    }
    ```
    
- **routes：**加入路徑**（本篇重點，以下詳細說明）**

### **Routes 設定**

[@nuxtjs/sitemap](https://www.npmjs.com/package/@nuxtjs/sitemap) 會自動加入 **靜態路由** 頁面，但 **動態路由** 必須手動加入

```
-| pages/
---| index.vue  --> static route
---| about.vue  --> static route
---| users/
-----| _id.vue  --> dynamic route
```

#### **加入動態路由方法：**

1. 手動加入（使用陣列）
    
    ```jsx
    sitemap: {
        routes: [
            {
                url: '/users/1',
                changefreq: 'daily',
                priority: 0.6,
                lastmod: new Date()
            },
            {
                url: '/users/2',
                changefreq: 'daily',
                priority: 0.6,
                lastmod: new Date()
            }
        ]
    }
    ```
    
2. 透過 API 串接加入（使用 function 回傳 promise）
    
    範例會使用 `@nuxtjs/axios` 進行 API 串接，執行：`npm i @nuxtjs/axios`
    
    ```jsx
    sitemap: {
        routes: async () => {
            const { data } = await axios.get('https://test/api/sitemap');
            return data.map(user => {
                return {
                    url: `/users/${user.id}`,
                    changefreq: 'daily',
                    priority: 0.6,
                    lastmod: new Date()
                }
            });
        }
    }
    ```
    

將上述設定合併起來：

```jsx
// config/sitemap.js
const axios = require('axios');

const sitemap = {
    path: '/sitemap.xml', // sitemap 名稱
    hostname: 'https://test.com', // domain
    cacheTime: 1000 * 60 * 60 * 24, // update frequency of the day
    defaults: {
        changefreq: 'daily',
        priority: 0.8,
        lastmod: new Date()
    },
    filter({ routes }) {
        return routes.map(route => {
            if (route.url === '/') {
                route.priority = 1;
            }
            return route;
        });
    }
    routes: async () => {
        const { data } = await axios.get('https://test.com/api/sitemap');
        return data.map(user => {
            return {
                url: `/users/${user.id}`,
                changefreq: 'daily',
                priority: 0.6,
                lastmod: new Date()
            }
        });
    }
};

export default sitemap;
```

接著在頁面上輸入 hostname/sitemap.xml（EX：https://test.com/sitemap.xml），就可以看到我們的 sitemap 檔案囉 🙌

![](https://i.imgur.com/Ajnso8p.png)

---

參考文章：

[https://www.hellosanta.com.tw/knowledge/category-13/post-236](https://www.hellosanta.com.tw/knowledge/category-13/post-236)

[https://sitemap.nuxtjs.org/usage/sitemap-options](https://sitemap.nuxtjs.org/usage/sitemap-options#routes-declaration)