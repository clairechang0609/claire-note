---
title: Vue.js Custom Directives 自訂指令
date: 2023-01-16
tags: [ vue3 ]
category: Vue
description: 除了 Vue 內建的系列指令，像是 v-model, v-for, v-show 等，我們也可以使用 Vue directives 自訂指令，將 DOM 元素和元件進行動態綁定，並對其進行操作，提高程式碼的可複用性
---
> **版本：Vue 3.2.x**
>

除了 Vue 內建的系列指令，像是 `v-model`, `v-for`, `v-show` 等，我們也可以使用 Vue directives 自訂指令，將 DOM 元素和元件進行動態綁定，並對其進行操作，提高程式碼的可複用性

## **結構**

1. 呼叫 `v-自訂名稱` 指令來綁定 DOM 元素
2. 透過類似元件的生命週期鉤子（Lifecycle Hooks）來定義指令（EX：mounted, updated…），並觸發更新

<!-- more -->

## **範例**

### **註冊指令**

EX：指令名稱 `v-focus`

```html
<input type="text" v-model="text" v-focus />
```

**1. 全域註冊**

```jsx
import { createApp } from 'vue';
import App from './App.vue';

const app = createApp(App);

app.directive('focus', {
    mounted(el) {
        el.focus();
    }
});

app.mount('#app');
```

**2. 區域註冊**

```jsx
<template>
    <input type="text" v-model="text" v-focus />
</template>

<script>
const focus = {
    mounted: (el) => el.focus()
}

export default {
    name: 'Home',
    directives: {
        focus
    },
    data() {
        return {
            text: ''
        }
    }
}
</script>
```

### **指令傳值 / 帶入參數 / 修飾符**

```jsx
<template>
    <input type="text" v-model="text" v-focus:[bindingArg].bar="bindingVal" />
</template>

<script>
export default {
    name: 'Home',
    data() {
        return {
            text: '',
            bindingArg: 'foo',
            bindingVal: 'disabled'
        }
    }
}
</script>
```

可以在 directive 函式 `binding` 取得內容（[binding 物件屬性](https://vuejs.org/guide/reusability/custom-directives.html#hook-arguments)）

```jsx
...
app.directive('focus', {
    mounted(el, binding) {
        el.focus();
        console.log(binding);
    }
});
...
```

結果如下

```json
{
    arg: 'foo', // 傳給指令的參數
    instance: Proxy{ ... }, // 使用指令的元件實體
    dir: { mounted: ƒ, updated: ƒ } // 指令定義內容
    modifiers: { bar: true }, // 修飾符
    value: 'disabled', // 當前值
    oldValue: undefined // 前一次的值
}
```

### **實際應用**

```jsx
<template>
    <form @submit.prevent="submitForm()" v-form="tel">
        <input type="tel" v-model="tel" placeholder="請輸入手機（10碼）" />
        <button type="submit" id="submit" disabled>送出表單</button>
    </form>
</template>

<script>
export default {
    name: 'Home',
    data() {
        return {
            tel: ''
        }
    }
}
</script>
```

```jsx
import { createApp } from 'vue';
import App from './App.vue';

const app = createApp(App);

app.directive('form', {
    mounted(el) {
        el.querySelector('input').focus();
    },
    updated(el, binding) {
        if (binding.value.length === 10) {
            el.querySelector('#submit').disabled = false;
        } else {
            el.querySelector('#submit').disabled = true;
        }
    }
});

app.mount('#app');
```

範例程式碼：

<iframe height="470" style="width: 100%;" scrolling="no" title="Vue.js Custom Directives 自訂指令" src="https://codepen.io/claire-chang-the-bashful/embed/NWBvMKN?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/claire-chang-the-bashful/pen/NWBvMKN">
  Vue.js Custom Directives 自訂指令</a> by Claire Chang (<a href="https://codepen.io/claire-chang-the-bashful">@claire-chang-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

---

參考文章：

[https://vuejs.org/guide/reusability/custom-directives.html](https://vuejs.org/guide/reusability/custom-directives.html)