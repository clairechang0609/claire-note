---
title: Nuxt.js 自訂頁面 Loading 效果
date: 2022-12-29
tags: [ nuxt2 ]
category: Nuxt
description: 本篇說明如何在 Nuxt.js 專案客製 Loading 進度條樣式
image: https://imgur.com/g7Aezdc.png
---
> **版本：nuxt 2.15.8**
>

<div style="display: flex; justify-content: center; margin: 0;">
    <img style="width: 100%; max-width: 600px;" src="https://imgur.com/g7Aezdc.png">
</div>

Nuxt.js 提供了預設的進度條效果，當切換路徑時會自動顯示。我們可以自由地調整進度條的樣式、關閉進度條，或者進行客製化介面設計。

## **設定 Loading 樣式**

### **1. 關閉 loading 效果**

全域關閉效果

```jsx
// nuxt.config.js
export default {
    loading: false
};
```

<!-- more -->

或是針對單頁關閉效果

```jsx
// pages/about.vue
<template>
    <h1>About</h1>
</template>

<script>
    export default {
        loading: false
    };
</script>
```

### **2. 調整預設元件樣式**

```jsx
// nuxt.config.js
export default {
    loading: {
        color: 'blue',
        height: '5px'
    }
};
```

**參數如下：**

| 參數 | Type | 預設值 | 說明 |
| --- | --- | --- | --- |
| color | String | 'black' | 進度條色彩 |
| failedColor | String | 'red' | 錯誤時進度條色彩 |
| height | String | '2px' | 進度條高度 |
| throttle | Number | 200 | 顯示進度條前的緩衝時間 (ms) |
| duration | Number | 5000 | 進度條顯示最長時間 (ms) |
| continuous | Boolean | false | loading 時間大於 duration 時，是否持續顯示進度條 |
| css | Boolean | true | 設定 false 會移除預設進度條樣式（並可自行設定樣式） |
| rtl | Boolean | false | 進度條方向是否由右至左（預設由左至右） |

### **3. 客製化 Loading 樣式**

如果不想使用預設 loading 樣式，也可以自己建立，新增 `components/TheLoading.vue`（命名自訂）

```jsx
// nuxt.config.js
<template>
    <div v-if="loading" class="loading-page">
        <span>Loading...</span>
    </div>
</template>
<script>
export default {
    data() {
        return {
            loading: false
        }
    },
    methods: {
        start() {
            this.loading = true
    },
    finish() {
        this.loading = false
    }
}
};
</script>
```

接著註冊到 nuxt.config.js

```jsx
// nuxt.config.js
export default {
    loading: '@/components/TheLoading.vue'
};
```

## **觸發 Loading 事件**

我們可以透過 `start()`、`finish()` 方法來觸發跟結束 loading 效果

```jsx
// pages/about.vue
<template>
    <h1>About：</h1>
    <h3>name：{{ info.name }}</h3>
    <h3>age：{{ info.age }}<h3>
</template>

<script>
export default {
    data() {
        return {
            info: {
                name: '',
                age: ''
            }
        };
    },
    mounted() {
        this.getInfo();
    },
    methods: {
        async getInfo() {
            // loading 開始
            this.$nuxt.$loading.start();

            const response = await $axios.$get('/api/info');
            this.info = response;

            // loading 結束
            this.$nuxt.$loading.finish();
        }
    }
};
</script>
```

---

參考文章：

[https://nuxtjs.org/docs/features/loading/](https://nuxtjs.org/docs/features/loading/)

[https://ithelp.ithome.com.tw/articles/10205496](https://ithelp.ithome.com.tw/articles/10205496)