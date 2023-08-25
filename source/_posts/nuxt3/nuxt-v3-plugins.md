---
title: Nuxt.js 3.x Plugins 目錄－擴充插件＋自訂指令
date: 2023-07-10
tags: [ nuxt3 ]
category: Nuxt3
description: 本篇說明 Nuxt3 Plugins 目錄功能，包含擴充插件、搭配 Composables、Provide 定義全局變數以及 Directive 自訂指令
image: https://imgur.com/5VRbaV1.png
---

前端開發常會搭配 **第三方套件（package）**使用，如表單驗證、圖片輪播、提示訊息等，這些套件通常已經被開發者設計好並經過測試，可以直接安裝使用，節省專案開發時間

## **Plugins（插件）說明**

Plugins 目錄協助我們在 Nuxt 擴充套件

- 在此定義的檔案（資料夾內第一層），Nuxt 會自動引入，不需要在 `nuxt.config` 檔各別註冊

```
plugins
|— myPlugin.ts  // 自動引入
|— supportingFile
|—— index.ts   // 不會自動引入
```

<!-- more -->

- 要限制在 server 或是 client 端使用，檔名需加上 `.server` 或 `.client` 後綴

```
plugins
|— myPlugin.client.ts
|— testPlugin.server.ts
```

- 也可以指令引入順序，如下 02.testPlugin.ts 能夠使用 01.myPlugin.ts 注入的內容，當 plugin 依賴另一個 plugin 時可以使用此功能

```
plugins
|— 01.myPlugin.ts
|— 02.testPlugin.ts
```

{% colorquote info %}
檔案名稱的排序是根據字串排序而不是數字，10.xxx.ts 排序會在 2.xxx.ts 前，因此上述範例需加上 「0」 前綴
{% endcolorquote %}

---

## **建立 Plugins**

Plugin 只提供 `nuxtApp` 唯一參數，`nuxtApp` 為一個物件（相關屬性 [參考文件](https://nuxt.com/docs/guide/going-further/internals#the-nuxtapp-interface)）

```jsx
export default defineNuxtPlugin(nuxtApp => {
    // Doing something with nuxtApp
})
```

---

## **設定同時載入**

Plugins 預設會依序載入，如果希望同時載入，只要在 plugin 內加上 `parallel: true`，下一個 plugin 就會跟這個 plugin 同時載入

```jsx
// plugins/asynchronous.ts
export default defineNuxtPlugin(nuxtApp => {
    name: 'async-plugin',
    parallel: true,
    async setup (nuxtApp) {
        // the next plugin will be executed immediatly
    }
})
```

---

## **在 Plugin 內使用 Composables（組合式函式）**

```jsx
export default defineNuxtPlugin(nuxtApp => {
    const counter = useCounter();
})
```

**需注意，在 Plugin 使用 Composables 有一些限制：**

- 若 composable 依賴於稍後載入的另一個 plugin，可能無法正常運作
- 若 composable 依賴 Vue 生命週期，由於 composable 綁定的是使用他的元件實體，但 plugin 只會綁定在 `nuxtApp` 實例，可能無法正常運作

---

## **Provide 定義全局變數**

注入全局變數到 NuxtApp，提供兩種定義方式：

```jsx
export default defineNuxtPlugin(nuxtApp => {
    // 方法一
    nuxtApp.provide('hello', (msg: string) => `Hello ${msg} !`);

    // 方法二
    return {
        provide: {
            hello: (msg: string) => `Hello ${msg} !`
        }
    };
})
```

{% colorquote info %}
功能同 Nuxt2 inject 寫法： ```inject('hello', msg => `Hello ${msg} !`)```
{% endcolorquote %}

使用 Plugin

```jsx
// pages/hello.vue
<template>
    <div>
        {{ $hello('World') }}
    </div>
</template>

<script setup lang="ts">
const { $hello } = useNuxtApp();
</script>
```

---

## **使用第三方套件**

以 https://github.com/kyvg/vue3-notification （提示訊息彈跳視窗）為例

#### **安裝套件**

```bash
npm i @kyvg/vue3-notification
```

#### **建立 Plugin**

- 透過 `nuxtApp.vueApp.use()` 在 Vue.js 註冊全局插件
- 並搭配前面提到的 `nuxtApp.provide()` 在 Nuxt 註冊全局變數或方法

```jsx
// plugins/notification.ts
import Notifications, { useNotification } from '@kyvg/vue3-notification';
const { notify } = useNotification();

export default defineNuxtPlugin(nuxtApp => {
    nuxtApp.vueApp.use(Notifications);
    return {
        provide: {
            notify
        }
    };
});
```

#### **使用提示訊息**

```jsx
// pages/hello.vue
<template>
    <div>
        <notifications />
    </div>
</template>

export default {
    setup() {
        const { $notify } = useNuxtApp();

        onMounted(() => {
            $notify({
                type: 'success',
                text: 'message'
            });
        });
    }
};
</script>
```

---

## **Custom Directives 自訂指令**

{% colorquote info %}
Vue 自訂指令說明可以參考 [這篇文章](https://clairechang.tw/2023/01/16/vue/vue-custom-directives/#more)
{% endcolorquote %}

除了 Vue 內建的系列指令，像是 `v-model`, `v-for`, `v-show` 等，我們也可以使用 Vue directives 自訂指令，將 DOM 元素和元件進行動態綁定，並對其進行操作

```jsx
// plugins/focus.ts
export default defineNuxtPlugin(nuxtApp => {
    nuxtApp.vueApp.directive('focus', {
        mounted(el) {
            el.focus();
        }
    });
});
```

使用自訂指令

```jsx
// pages/hello.vue
<template>
    <input type="text" value="hello" v-focus />
</template>
```

---

參考資源：

https://nuxt.com/docs/guide/directory-structure/plugins#plugins-directory
https://ithelp.ithome.com.tw/articles/10299002