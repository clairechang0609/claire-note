---
title: Nuxt.js 3.x 狀態管理 State Management (3)－Pinia Plugin Persistedstate
date: 2023-08-16 14:00
tags: [ nuxt3 ]
category: Nuxt3
description: 前一篇說明了在 Nuxt3 搭配 Pinia 狀態管理工具全域共享狀態，本篇將介紹 pinia-plugin-persistedstate 套件，用來將 Store 狀態儲存於瀏覽器中，避免狀態被還原
image: https://imgur.com/n21QxOI.png
---

[前一篇](https://clairechang.tw/2023/08/15/nuxt3/nuxt-v3-state-management-pinia/) 說明了在 Nuxt3 搭配 [Pinia](https://pinia.vuejs.org/) 狀態管理工具全域共享狀態，本篇將介紹 `pinia-plugin-persistedstate` 套件，用來將 Store 狀態儲存於瀏覽器中，避免狀態被還原

<!-- more -->

# **Pinia Plugin Persistedstate**

`pinia-plugin-persistedstate` 用來將 Store 狀態保存於使用者的瀏覽器中，以下兩個情境推薦使用：

- 使用者下次再瀏覽畫面時，需保存先前操作的狀態
- 避免頁面重新整理時，狀態被還原

## **套件安裝**

搭配 Nuxt 整合模組 `@pinia-plugin-persistedstate/nuxt` 進行安裝：

```bash
npm install -D @pinia-plugin-persistedstate/nuxt
```

{% colorquote info %}
`pinia-plugin-persistedstate` 必須搭配 [Pinia](https://pinia.vuejs.org/) 使用
{% endcolorquote %}

---

## **nuxt.config 配置**

```jsx
// nuxt.config.js
export default defineNuxtConfig({
    modules: [
        '@pinia/nuxt',
        '@pinia-plugin-persistedstate/nuxt'
    ]
})
```

---

## **Store 開啟狀態保存**

### **Option Store**

將 persist 參數設定為 true

```jsx
// store/index.js
export const useMainStore = defineStore('main', {
    state: () => ({
        counter: 0
    }),
    getters: {
        doubleCounter() {
            return this.counter * 2;
        }
    },
    actions: {
        increment() {
            this.counter++;
        }
    },
    persist: true
});
```

### **Setup Store**

defineStore 傳入參數 `{ persist: true }`

```jsx
// store/index.js
export const useMainStore = defineStore(
    'main',
    () => {
        const counter = ref(0);
        const doubleCounter = computed(() => counter.value * 2);
        const increment = () => {
            counter.value++;
        };
        
        return {
            counter,
            doubleCounter,
            increment
        };
    },
    {
        persist: true
    }
);
```

---

## **選項配置**

**預設配置：**

- 儲存於 `cookie`
- 預設 key 為 `store.$id`
- 使用 `JSON.stringify()` 與 `JSON.parse()` 進行序列化 / 反序列化
- 預設整個 state 都會被保存

### **Key**

> 型別：`string`
預設值：`store.$id`
> 

以下為例，預設 key 為 `main`，調整為 `my-custom-key`

```jsx
// store/index.js
export const useMainStore = defineStore('main',
    () => {
        // ...
    }, 
    {
        persist: {
            key: 'my-custom-key'
        }
    }
);
```

在儲存庫可以看到調整後的 key 值

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/UJZhUuP.png">
</div>

### **storage**

> 預設值：`cookie`
選項：`localStorage`、`sessionStorage`、`cookie`
> 

使用自動引入的 `persistedState` 變數進行配置

```jsx
// store/index.js
export const useMainStore = defineStore('main',
    () => {
        // ...
    }, 
    {
        persist: {
            storage: persistedState.localStorage
        }
    }
);
```

{% colorquote info %}
因 storage 只存在瀏覽器端，如果未搭配 `persistedState` 定義，在 `ssr` 會出現錯誤，也可以透過判斷式定義：

```jsx
{
    persist: {
        storage: process.client ? localStorage : null
    }
}
```
{% endcolorquote %}

#### **cookiesWithOptions()**

用來設定 cookie，選項同 Nuxt Composable [useCookie](https://nuxt.com/docs/api/composables/use-cookie#options)，storage 需為 cookie 才能使用

```jsx
// store/index.js
export const useMainStore = defineStore('main',
    () => {
        // ...
    }, 
    {
        persist: {
            storage: persistedState.cookiesWithOptions({
                sameSite: 'strict'
            })
        }
    }
);
```

### **paths**

> 型別：`string[]`
預設值：`undefined`
> 

預設整個 state 都會被保存在 storage。如果想指定參數，使用 paths 進行調整

```jsx
// store/index.js
export const useMainStore = defineStore('main',
    () => {
        const counter = ref(0);
        const user = ref({
            name: 'Daniel',
            age: 18
        });
    }, 
    {
        persist: {
            paths: [
                'user.name'
            ]
        }
    }
);
```

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/DeOsgt8.png">
</div>

### **serializer**

> 預設值：`JSON.stringify`、`JSON.parse`
> 

指定序列化方法（必須包含 `serialize`、`deserialize`），以下範例調整為壓縮檔

```jsx
// store/index.js
import { parse, stringify } from 'zipson';

export const useMainStore = defineStore('main',
    () => {
        // ...
    }, 
    {
        persist: {
            serializer: {
                serialize: stringify,
                deserialize: parse
            }
        }
    }
);
```

### **debug**

> 型別：`boolean`
預設值：`false`
> 

設定為 true，當發生任何錯誤，都會使用 `conosle.error` 印出

---

## **全域選項配置**

在 `nuxt.config` 使用 `piniaPersistedstate` 全域調整選項

```jsx
// nuxt.config.js
export default defineNuxtConfig({
    modules: [
        '@pinia/nuxt',
        '@pinia-plugin-persistedstate/nuxt'
    ],
    piniaPersistedstate: {
        storage: 'cookie',
        cookieOptions: {
            sameSite: 'strict'
        }
    }
})
```

---

參考資源：

[https://prazdevs.github.io/pinia-plugin-persistedstate/](https://prazdevs.github.io/pinia-plugin-persistedstate/frameworks/nuxt-3.html)