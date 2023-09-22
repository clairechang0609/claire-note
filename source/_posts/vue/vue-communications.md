---
title: Vue.js Props, $emit, Provide, Inject 父子元件資料傳遞
date: 2023-01-13
tags: [ vue3 ]
category: Vue
description: 本篇說明 Vue.js 專案內父子元件資料傳遞的方法 Props 搭配 $emit，以及跨層級資料傳遞 Provide 搭配 Inject
image: https://i.imgur.com/VEoagNF.png
---
> **版本：Vue 3.2.x**
>

<div style="display: flex; justify-content: center;">
    <img style="width: 100%; max-width: 600px;" src="https://i.imgur.com/VEoagNF.png">
</div>

# **Props & $emit**

父子元件間溝通的重要管道，因每個元件的作用範圍皆應獨立，因此不應在子元件內直接修改父元件的資料，否則會難以除錯，造成維護上困難。

假設頁面層級如下

```
Root.vue
    |—— Child.vue
```

<!-- more -->

## **Props**

**父層傳遞到子層**

透過 `v-bind:父元件參數名稱="子元件參數名稱"`，將資料傳遞到子元件

```jsx
// views/Root.vue
<template>
    <Child :message="messageToChild" />
</template>

<script>
export default {
    name: 'Root',
    data() {
        return {
            messageToChild: '早安您好'
        }
    }
}
</script>
```

**子層接收父層資料**

透過 `props` 接收由父層傳遞來的資料

- type：指定型別，如果接受多個型別，可以使用陣列（EX：`type: [ String, Number ]`）
- required：是否必填，預設 false
- default：預設值，如果指定型別為**物件或陣列**，需使用函式定義（EX：`default: () => {}`）

```jsx
// components/Child.vue
<template>
    <input type="text" :value="message" />
</template>

<script>
export default {
    name: 'Child',
    props: {
        message: {
            type: String,
            required: true,
            default: ''
        }
    }
}
</script>
```

## **$emit**

前面提到，子元件無法直接修改父層的資料，會報錯誤如下

{% colorquote danger %}
[Vue warn]: Avoid mutating a prop directly since the value will be overwritten whenever the parent component re-renders. Instead, use a data or computed property based on the prop's value.
{% endcolorquote %} 

如果要修改父層傳回的值，需透過 `$emit` 觸發父層事件，簡單來說是通知父層元件更新值，不能直接由子元件更新。

**子層觸發父層事件**

```jsx
this.$emit(/* event name */ 'update-message', /* value */ value)
```

**第一個參數**：父層事件名稱，命名必須用**連字符**（EX: 'update-message'），不能使用**駝峰**（EX: 'updateMessage'），否則會失效

**第二個參數**：傳遞給父層的值

```jsx
// components/Child.vue
<template>
    <input type="text" :value="message" @input="updateMessage($event.target.value)" />
</template>

<script>
export default {
    name: 'Child',
    props: {
        message: {
            type: String,
            required: true,
            default: ''
        }
    },
    methods: {
        updateMessage(value) {
            this.$emit('update-message', value);
        }
    }
}
</script>
```

**父層事件觸發資料更新**

透過 `v-on:$emit事件名稱="父層函式"` ，藉由子元件觸發事件，來接受回傳值

```jsx
// views/Root.vue
<template>
    <Child :message="messageToChild" @update-message="updateMessage" />
</template>

<script>
export default {
    name: 'Root',
    data() {
        return {
            messageToChild: '早安您好'
        }
    },
    methods: {
        updateMessage(value) {
            this.messageToChild = value;
        }
    }
}
</script>
```

**以上例說明：**

1. 當子元件觸發 `v-on:update-message` 事件
2. 進而觸發 `updateMessage` 函式
3. 接收子元件回傳的 `value`
4. 更新 `messageToChild` 內容
5. 再透過 `props` 更新子元件接收值

藉由以上循環達成雙向綁定。

### **Vue 3.x：透過 v-model 指令簡化語法**

除了上述的寫法外，也可以透過 `v-model` 來縮寫

**子元件：**`$emit` 事件命名規則為 `update:參數名稱`

```jsx
// components/Child.vue
<template>
    <input type="text" :value="message" @input="updateMessage($event.target.value)" />
</template>

<script>
export default {
    name: 'Child',
    props: {
        message: {
            type: String,
            required: true,
            default: ''
        }
    },
    methods: {
        updateMessage(value) {
            this.$emit('update:message', value);
        }
    }
}
</script>
```

**父元件：**合併 `:message` 跟 `@update:message`，直接使用 `v-model` 語法糖 `v-model:參數名稱`

```jsx
// views/Root.vue
<template>
    <Child v-model:message="updateMessage" />
</template>

<script>
export default {
    name: 'Root',
    data() {
        return {
            messageToChild: '早安您好'
        }
    }
}
</script>
```

### **Vue 2.x：透過 .sync 修飾符簡化語法**

Vue 2.x 元件間雙向綁定不支援 `v-model:參數名稱` 這樣的寫法，是透過 `.sync` 修飾符

**子元件：**寫法同上 Vue 3.x，不贅述

**父元件：**使用 `v-bind:參數名稱.sync`

```jsx
// views/Root.vue
<template>
    <Child :message.sync="updateMessage" />
</template>

<script>
export default {
    name: 'Root',
    data() {
        return {
            messageToChild: '早安您好'
        }
    }
}
</script>
```

# **Provide & Inject**

假設頁面層級如下

```
Root.vue
    |—— Header.vue
        |—— DeepChild.vue
```

要從 <Root> 傳遞資料到 <DeepChild>，我們必須要 `props` 傳遞兩層才能順利將資料傳入，當層級越來越多，要再往下傳遞就更困難了…

```html
<Root> -> <Header> -> <DeepChild>
```

這時候透過 `provide` 跟 `inject`，父元件作為**所有**子元件的**依賴提供者（dependency provider）**，跨過中間組件，快速達成跨階層元件資料傳遞：

```html
<Root> -> <DeepChild>
```

## **Provide**

父元件透過 `provide` 定義要往下傳送的參數

```jsx
// <Root>
<template>
  ...
</template>

<script>
export default {
    name: 'Root',
    provide() {
        return {
            message: this.message
        }
    },
    data() {
        return {
            message: '傳給元件的訊息'
        }
    }
}
</script>
```

## **App-level Provide**

全域傳遞參數

```jsx
// main.js
import { createApp } from 'vue';
const app = createApp({});

app.provide(/* key */ 'message', /* value */ '傳給元件的訊息');
```

## **Inject**

子元件透過 `inject`，可以注入來自**所有上層**傳遞的內容（以 <DeepChild> 為例，可以接收 <App>、<Root>、<Header> 傳出的內容）

```jsx
// <DeepChild>
<template>
    <div>{{ message }}</div>
</template>

<script>
export default {
    name: 'DeepChild',
    inject: [ 'message' ]
}
</script>
```

---

參考文章：

[https://book.vue.tw/CH2/2-2-communications.html](https://book.vue.tw/CH2/2-2-communications.html)

[https://vuejs.org/guide/components/provide-inject.html](https://vuejs.org/guide/components/provide-inject.html)