---
title: Nuxt.js 3.x Layouts 目錄－自訂模板
date: 2023-09-01
tags: [ nuxt3 ]
category: Nuxt3
description: Nuxt Layouts 資料夾協助定義共用模板，將重複使用的版面提取到模板內全域共用，像是 Header、Sidebar、Footer 等版面
image: https://imgur.com/JgSJYA4.png
---

Layouts 資料夾協助我們定義共用模板，將重複使用的版面提取到模板內全域共用，看起來跟 Components 有點像，那麼 Layouts 跟 Components 怎麼區分？**可以將 Layouts 視為包覆在頁面外層的包裹元件**，用來定義 Header、Sidebar、Footer 等共用版面或是元件

{% colorquote info %}
官方文件提到，如果整個專案只有一個模板，建議直接在 `app.vue` 定義即可
{% endcolorquote %}

<!-- more -->

## **預設模板**

預設模板名稱為 **default**，新增 `layouts/default.vue`，並透過 `<slot />` 將元件的內容載入

```jsx
// layouts/default.vue
<template>
  <div>
    <nav>Header</nav>
    <slot />
    <div>Footer</div>
  </div>
</template>
```

或是將 Header、Footer 等共用元件（Components）加入模板

```jsx
// layouts/default.vue
<template>
  <div>
    <TheHeader />
    <slot />
    <TheFooter />
  </div>
</template>
```

接著在 `app.vue` 透過 `<NuxtLayout>` 元件使用模板，預設使用 `layouts/default.vue` 模板

```jsx
// app.vue
<template>
  <NuxtLayout>
    <NuxtPage />
  </NuxtLayout>
</template>
```

新增首頁 `pages/index.vue`

```jsx
// pages/index.vue
<template>
  <div>
    <h3>Home Page</h3>
  </div>
</template>
```

畫面呈現：

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/JgSJYA4.png">
</div>

---

## **使用其他具名模板**

```
layouts
|—— default.vue
|—— main.vue
```

```jsx
// layouts/main.vue
<template>
  <div>
    <h2>Main Layout</h2>
    <slot />
  </div>
</template>
```

### **1. app.vue 指定全域模板，覆蓋預設模板**
使用 `name` props 指定模板

{% colorquote info %}
名稱必須使用 **kebab-case**，例如 Layout 檔名為 customLayout，需使用 `name='custom-layout'`
{% endcolorquote %}

```jsx
// app.vue
<template>
  <NuxtLayout name="main">
    <NuxtPage />
  </NuxtLayout>
</template>
```

### **2. 在單一元件指定模板**

使用 `definePageMeta()` 輔助函式指定模板名稱

```jsx
// pages/index.vue
<template>
  <div>...</div>
</template>

<script setup>
definePageMeta({
  layout: 'main'
});
</script>
```

### **3. 在單一元件直接定義 NuxtLayout**

先將預設模板設為 `false`，並透過 `<NuxtLayout name="xxx">` 定義模板

```jsx
// pages/index.vue
<template>
  <div>
    <NuxtLayout name="main">
      <h3>Home Page</h3>
    </NuxtLayout>
  </div>
</template>

<script setup>
definePageMeta({
  layout: false
});
</script>
```

{% colorquote info %}
在單一元件直接使用 `<NuxtLayout>`，要避免放在 **根元素（root element）**，或是關閉 page 跟 layout 的 [transitions](https://nuxt.com/docs/getting-started/transitions#disable-transitions) 效果
{% endcolorquote %}

畫面呈現：

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/CMOeR2c.png">
</div>

---

## **動態切換模板**

透過 `setPageLayout()` 輔助函式動態切換模板

```jsx
// pages/index.vue
<template>
  <div>
    <h3>Home Page</h3>

    <button @click="setPageLayout('default')">default</button>
    <button @click="setPageLayout('main')">main</button>
  </div>
</template>
```

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/CkJhzap.gif">
</div>

---

## **具名插槽**

**範例：**
`layouts/main.vue` 搭配具名插槽

```jsx
// layouts/main.vue
<template>
  <div>
    <nav>Header</nav>
    <slot name="content" />
    <div>Sidebar</div>
    <slot name="footer" />
  </div>
</template>
```

在頁面上使用，需將預設模板設為 `false`

```jsx
// pages/index.vue
<template>
  <div>
    <NuxtLayout name="main">
      <template #content>
        <h3>Home Page</h3>
      </template>

      <template #footer>
        <h3>Footer</h3>
      </template>
    </NuxtLayout>
  </div>
</template>

<script setup>
definePageMeta({
  layout: false
});
</script>
```

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://imgur.com/htWggtg.png">
</div>

---

參考資源：

https://nuxt.com/docs/guide/directory-structure/layouts
https://nuxt.com/docs/examples/features/layouts
https://medium.com/codex/nuxt3-layouts-276ed64a4a1c