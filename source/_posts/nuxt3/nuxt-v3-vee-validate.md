---
title: Nuxt.js 3.x 套件應用－VeeValidate v4.x 表單驗證
date: 2023-07-24
tags: [ nuxt3, vee-validate ]
category: Nuxt3
description: 本篇說明 Nuxt3 專案如何使用 VeeValidate 客製表單驗證
image: https://imgur.com/n3NS3CQ.png
---

> 本篇文章同步發表於 2023 iThome 鐵人賽：[Nuxt.js 3.x 筆記－打造 SSR 專案](https://ithelp.ithome.com.tw/articles/10334584)
>

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 550px;" src="https://imgur.com/n3NS3CQ.png">
</div>

{% colorquote info %}
Nuxt2 vee-validate 請參考 [這篇文章](https://clairechang.tw/2022/12/17/nuxt/nuxt-veevalidate/)
{% endcolorquote %}

[vee-validate](https://vee-validate.logaretm.com/v4/) 是使用於 Vue.js 的輕量表單驗證套件，僅需在表單上加入簡易語法就能進行驗證

{% colorquote info %}
Vue.js v3.x 需搭配 vee-validate v4.x
{% endcolorquote %}

<!-- more -->

## **step1：vee-validate 套件安裝**

使用 Nuxt 整合套件 [@vee-validate/nuxt](https://vee-validate.logaretm.com/v4/integrations/nuxt/)

```jsx
npm i @vee-validate/nuxt
```

---

## **step2：nuxt.config 配置**

```jsx
// nuxt.config.ts
export default defineNuxtConfig({
  // ...
  modules: [
    '@vee-validate/nuxt',
  ],
  veeValidate: {
    // 啟用 auto imports
    autoImports: true,
    // 更換 components 名稱
    componentNames: {
      Form: 'VeeForm',
      Field: 'VeeField',
      FieldArray: 'VeeFieldArray',
      ErrorMessage: 'VeeErrorMessage',
    }
  }
});
```

---

## **step3：@vee-validate/rules 驗證規則擴充**

我們可以在 vee-validate 自訂驗證規則，或是搭配預設規則，若要使用預設規則，步驟如下

安裝驗證規則套件：

```jsx
npm i @vee-validate/rules
```

新增 plugins/vee-validate.client.js，擴充所有規則：

```jsx
// vee-validate.client.js
import { defineRule } from 'vee-validate';
import allRules from '@vee-validate/rules';

// 迴圈依序加入規則
Object.keys(allRules).forEach(rule => {
  defineRule(rule, allRules[rule]);
});

// 必須定義，用來封裝 plugin
export default defineNuxtPlugin(_nuxtApp => {});
```

---

## **step4：@vee-validate/i18n 語系設定**

安裝 i18n 套件：

```jsx
npm i @vee-validate/i18n
```

新增 plugins/vee-validate.client.js，進行語系調整：

```jsx
// vee-validate.client.js
import { configure } from 'vee-validate';
import { localize, setLocale } from '@vee-validate/i18n';
import zhTW from '@vee-validate/i18n/dist/locale/zh_TW.json';

// 配置訊息
configure({
  generateMessage: localize({ zh_TW: zhTW })
});

setLocale('zh_TW');

// 必須定義，用來封裝 plugin
export default defineNuxtPlugin(_nuxtApp => {});
```

---

## **step5：表單驗證**

- 觸發 submit 時，會自動傳出表單內容，因此欄位不需要另外綁定 `v-model`
- `<VeeForm />` 元件 submit listener 觸發時會自動調用 `event.preventDefault()`
- rules 個別設定驗證項目，可以自訂規則或使用預設規則。若使用 `@vee-validate/rules` 搭配多個規則，用 `|` 做區隔

```jsx
<template>
  <VeeForm @submit="submit">
    <VeeField name="email" type="email" rules="required|email" />
    <VeeErrorMessage name="email" />
    <VeeField name="password" type="password" :rules="checkPassword" />
    <VeeErrorMessage name="password" />
    <button type="submit">submit</button>
  </VeeForm>
</template>

<script setup>
const checkPassword = value => {
  if (...) {
    return false;
  }
  return true;
}
const submit = value => {
  console.log('submit', value);
};
</script>
```

#### **1. 驗證成功前禁止送出表單**

利用 `<VeeForm />` 的 `meta.valid` 屬性來檢查是否驗證成功

```jsx
<VeeForm @submit="submit" v-slot="{ meta }">
  ...
  <button type="submit" :disabled="!meta.valid">submit</button>
</VeeForm>
```

#### **2. 表單送出完成／失敗前禁止重複觸發**

利用 `<VeeForm />` 的 `meta.isSubmitting` 屬性判斷 submit handler 是否在執行中

```jsx
<template>
  <VeeForm @submit="submit" v-slot="{ isSubmitting }">
    ...
    <button type="submit" :disabled="isSubmitting">
      <span class="spinner" v-show="isSubmitting"></span>
      submit
    </button>
  </VeeForm>
</template>

<script setup>
const submit = value => {
  return new Promise(resolve => {
    setTimeout(() => {
      console.log('sumbit', value);
      resolve();
    }, 2000);
  });
};
</script>
```

#### **3. schema 統一管理規則**

除了透過 rules 設定驗證規則，也可以直接在表單外層透過 `validation-schema` props 統一定義規則

```jsx
<template>
  <VeeForm @submit="submit" :validation-schema="schema">
    <VeeField name="email" type="email" />
    <VeeErrorMessage name="email" />
    <VeeField name="password" type="password" />
    <VeeErrorMessage name="password" />
    <button type="submit">submit</button>
  </VeeForm>
</template>

<script setup>
const schema = {
  email: 'required|email',
  password: 'required|min:8'
};
</script>
```

#### **4. 帶入預設值**

**setValues(fields) ：帶入整個表單資料，自動觸發驗證**

```jsx
<template>
  <VeeForm @submit="submit" ref="veeForm">
    <VeeField name="email" type="email" />
    <VeeErrorMessage name="email" />
    <VeeField name="password" type="password" />
    <VeeErrorMessage name="password" />
    <button type="submit">submit</button>
  </VeeForm>
</template>

<script setup>
const veeForm = ref(null);

onMounted(() => {
  const form = {
    email: 'test@myweb.com',
    password: '12344321'
  };
  veeForm.value.setValues(form);
});
</script>
```

**setFieldValue(field, value) ：帶入單欄資料，觸發此欄驗證**

```jsx
<template>
  <VeeForm @submit="submit" ref="veeForm">
    <VeeField name="email" type="email" />
    <VeeErrorMessage name="email" />
    <button type="submit">submit</button>
  </VeeForm>
</template>

<script setup>
const veeForm = ref(null);

onMounted(() => {
  veeForm.value.setFieldValue('email', 'test@myweb.com');
});
</script>
```

#### **5. 調整驗證觸發行為**

- `validateOnBlur`：離開焦點時觸發，預設 `true`
- `validateOnChange`：欄位在 change 事件觸發，預設 `true`
- `validateOnInput`：輸入內容時觸發，預設 `false`
- `validateOnModelUpdate`：`update:modelValue` (v-model) 事件觸發，預設 `true`

**全域調整：**

```jsx
// vee-validate.client.js
import { configure } from 'vee-validate';

configure({
  validateOnInput: true
});

export default defineNuxtPlugin(_nuxtApp => {});
```

**局部調整：**

```jsx
<Field name="email" :validateOnBlur="false" :validateOnChange="false" :validateOnInput="false" />
```

#### **6. 清除表單**

利用 `<VeeForm />` 的 `resetForm` 方法來清除表單

```jsx
<VeeForm @submit="submit" v-slot="{ resetForm }">
  ...
  <button type="buttom" @click="resetForm">clear</button>
</VeeForm>
```

#### **7. Checkbox ／Radio**

```jsx
<VeeField v-slot="{ field }" name="gender" type="radio" value="male">
  <label>
    <input type="radio" name="gender" v-bind="field" value="male" />
    male
  </label>
</VeeField>
<VeeField v-slot="{ field }" name="gender" type="radio" value="female">
  <label>
    <input type="radio" name="gender" v-bind="field" value="female" />
    female
  </label>
</VeeField>
```

---

參考資源：

https://vee-validate.logaretm.com/v4/