---
title: Nuxt.js 套件應用：VeeValidate 表單驗證
date: 2022-12-17
tags: [ nuxt2, vee-validate ]
category: Nuxt
description: 本篇說明如何在 Nuxt.js 專案安裝 vee-validate 輕量表單驗證套件，並搭配簡易語法進行驗證
image: https://i.imgur.com/Skb7Wnb.png
---
> **版本：nuxt 2.15.8**
>

<div style="display: flex; justify-content: center; margin: 20px 0;">
  <img style="width: 100%; max-width: 550px;" src="https://i.imgur.com/Skb7Wnb.png">
</div>

[vee-validate](https://vee-validate.logaretm.com/v3/) 是使用於 Vue.js 的輕量表單驗證套件（參考 php 框架 laravel 表單驗證所開發），僅需在表單上加入簡易語法就能進行驗證，執行：`npm i vee-validate@3.4.14`

{% colorquote info %}
Vue.js v2.x 需搭配 vee-validate v3.x
{% endcolorquote %}

新增 plugins/vee-validate.js，在 nuxt.config.js 配置

```jsx
// nuxt.config.js
export default {
  plugins: [
    { src: '@/plugins/vee-validate.js', mode: 'client' }
  ]
}
```

<!-- more -->

移動到 plugins/vee-validate.js 註冊全域表單驗證元件，步驟如下：

### **1. 規則引入**

**引入部分規則：**

```jsx
import { extend } from 'vee-validate';
import { required, email } from "vee-validate/dist/rules";

extend('email', email);
extend('required', {
  ...required,
  message: '欄位必填'
});
```

**引入全部規則：**

```jsx
import { extend } from 'vee-validate';
import * as rules from 'vee-validate/dist/rules';

Object.keys(rules).forEach(rule => {
  extend(rule, rules[rule]);
});
```

### **2. 語系設定（以繁體中文舉例）**

```jsx
import { localize } from 'vee-validate';
import tw from 'vee-validate/dist/locale/zh_TW.json';

localize('zh_TW', tw);
```

全域自訂提示訊息

```jsx
localize('zh_TW', {
  ...tw,
  messages: {
    required: '{_field_} 必填'
  },
  fields: {
    email: {
      required: '郵件 必填'
    }
  }
});
```

### **3. 調整互動模式跟 CSS classes**

**互動模式選項：**

- aggressive：**預設模式**，`input` 跟 `blur` 事件進行驗證
- lazy：`change` 或 `blur` 事件進行驗證
- passive：不會自動驗證，需主動呼叫
- eager：**官方說明為使用者體驗最佳的互動模式**，結合 `aggressive` (驗證失敗後) 跟 `lazy` (驗證前)

```jsx
import { configure } from 'vee-validate';

configure({
    mode: 'eager', // Interaction Modes
  classes: {
    valid: 'is-valid',
    invalid: 'is-invalid'
  }
});
```

### **4. 將元件註冊至 Vue Instance**

**ValidationObserver：**驗證完整表單

**ValidationProvider：**驗證單一表單內容

```jsx
import { ValidationObserver, ValidationProvider } from 'vee-validate';

Vue.component('ValidationObserver', ValidationObserver);
Vue.component('ValidationProvider', ValidationProvider);
```

將以上內容整合起來：

```jsx
// plugins/vee-validate.js
import Vue from 'vue';
import tw from 'vee-validate/dist/locale/zh_TW.json';
import * as VeeValidate from 'vee-validate';
import * as rules from 'vee-validate/dist/rules';

Object.keys(rules).forEach(rule => {
  VeeValidate.extend(rule, rules[rule]);
});
VeeValidate.localize('zh_TW', tw);

VeeValidate.configure({
  mode: 'eager',
  classes: {
    valid: 'is-valid',
    invalid: 'is-invalid'
  }
});

Vue.component('ValidationObserver', VeeValidate.ValidationObserver);
Vue.component('ValidationProvider', VeeValidate.ValidationProvider);
```

接下來就可以在各頁面、元件使用表單驗證功能囉：

### **Validation Provider：驗證單一表單內容（input）**

**1. v-slot：**取得元件回傳的內容
**2. rules：**規則選擇，多個規則可以使用 '|' 分隔
**3. props：**回傳內容給元件，EX: name

```html
<ValidationProvider rules="required|email" name="E-mail" v-slot="{ errors, classes }">
  <input type="email" placeholder="請輸入信箱" v-model="email" :class="classes" />
  <small>{{ errors[0] }}</small>
</ValidationProvider>
```

### **Validation Observer：驗證完整表單**

**方法一：**使用 ****`v-slot="{ invalid }"` ，驗證不通過 disabled 按鈕（`type="submit"`），`submitForm()` 為自訂方法

```html
<validation-observer v-slot="{ invalid }">
  <form @submit.prevent="submitForm">
    <ValidationProvider>...</ValidationProvider>
    <button type="submit" :disabled="invalid">送出表單</button>
  </form>
</validation-observer>
```

**方法二：**使用 `v-slot="{ handleSubmit }"` ，透過 handleSubmit 方法來監聽表單，驗證通過點擊按鈕才會執行 `submitForm()`

```html
<validation-observer v-slot="{ handleSubmit }">
  <form @submit.prevent="handleSubmit(submitForm)">
    <ValidationProvider>...</ValidationProvider>
    <button type="submit">送出表單</button>
  </form>
</validation-observer>
```

將 `<ValidationProvider />`、`<ValidationObserver />` 結合起來，範例頁面 pages/contact-us.vue：

```jsx
// pages/contact-us.vue
<template>
  <client-only>
    <ValidationObserver v-slot="{ handleSubmit }" ref="observer">
      <form @submit.prevent="handleSubmit(submitForm)">
        // 姓名
        <label for="name">姓名</label>
        <ValidationProvider rules="required" name="姓名" v-slot="{ errors, classes }">
          <input type="text" v-model="name" :class="classes" />
          <small>{{ errors[0] }}</small>
        </ValidationProvider>
        // email
        <label for="email">E-mail</label>
        <ValidationProvider rules="required|email" name="E-mail" v-slot="{ errors, classes }">
          <input type="email" v-model="email" :class="classes" />
          <small>{{ errors[0] }}</small>
        </ValidationProvider>
        <button type="submit">送出表單</button>
      </form>
    </ValidationObserver>
  </client-only>
</template>

<script>
  export default {
    data() {
      return {
        name: '',
        email: ''
      }
    },
    methods: {
      submitForm() {
        // ...
      }
    }
  }
</script>
```

{% colorquote info %}
在 `ssr: true` 模式下，外層必須加上 `<client-only />` ，否則在 server 端會因為 dom 元素無法解析而拋錯誤
{% endcolorquote %}

結果如下

<iframe height="470" style="width: 100%;" scrolling="no" title="Vue.js 2.x 套件應用：VeeValidate 表單驗證" src="https://codepen.io/claire-chang-the-bashful/embed/dyjVYXZ?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/claire-chang-the-bashful/pen/dyjVYXZ">
  Vue.js 2.x 套件應用：VeeValidate 表單驗證</a> by Claire Chang (<a href="https://codepen.io/claire-chang-the-bashful">@claire-chang-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

---

參考文章：

[https://medium.com/@yusufznl/how-to-validate-forms-in-nuxt-with-vee-validate-eef45508c3d4](https://medium.com/@yusufznl/how-to-validate-forms-in-nuxt-with-vee-validate-eef45508c3d4)
[https://vee-validate.logaretm.com/v3/](https://vee-validate.logaretm.com/v3/)