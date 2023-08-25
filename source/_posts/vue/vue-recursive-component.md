---
title: Vue.js Recursive Component 遞迴元件實作巢狀選單
date: 2023-01-10
tags: [ vue3 ]
category: Vue
description: Vue.js 的元件內除了可以包覆其他元件，也可以將「自己」作為「子元件」使用，稱為遞迴元件（Recursive Component），巢狀的資料渲染，像是多層導覽列或是組織圖，都可以使用遞迴元件進行實作
image: https://i.imgur.com/2ErNzhr.png
---
> **版本：Vue 3.2.x**
> 

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img style="width: 100%; max-width: 700px;" src="https://i.imgur.com/2ErNzhr.png">
</div>

Vue.js 的元件內除了可以包覆其他元件，也可以將**「自己」**作為「子元件」使用，稱為遞迴元件（Recursive Component），巢狀的資料渲染，像是多層導覽列或是組織圖，都可以使用遞迴元件實作。

說到遞迴元件，必須先了解遞迴函式（Recursive Function）。

<!-- more -->

## **遞迴函式（Recursive Function）**

簡單來說是在函式中呼叫**「自己」**的行為，遞回函式必須包含：

1. 結束遞迴的基本情況（base case）
2. 延續遞迴的遞迴情況（recursive case）

假設有一筆多層選單內容如下

```jsx
const menuList = {
    id: 0,
    name: '首頁',
    sub: [
        {
            id: 1,
            name: '關於我們',
            sub: [
                {
                    id: 11,
                    name: '公司介紹',
                    sub: []
                }
            ]
        },
        {
            id: 2,
            name: '產品介紹',
            sub: [
                {
                    id: 21,
                    name: '保濕系列',
                    sub: []
                },
                {
                    id: 22,
                    name: '抗老系列',
                    sub: []
                }
            ]
        },
        {
            id: 3,
            name: '線上購物',
            sub: []
        }
    ]
}
```

當我們想從中找出一筆資料（EX：id: 22）

**一般函式**

如果是一般函式，選單有幾層就要寫幾次判斷式，程式碼冗長難以維護

```jsx
function findItem(list, id) {
    if (list.id === id) {
        console.log(list.name);
    } else {
        list.sub.filter(item => {
            if (item.id === id) {
                console.log(item.name);
            } else {
                ...
            }
        });
    }
}
findItem(menuList, 22); // '抗老系列'
```

**遞迴函式**

透過遞迴函式，程式碼簡潔許多

```jsx
function findItem(list, id) {
    if (list.id === id) { // base case
        console.log(list.name);
        return;
    }
    list.sub.filter(item => this.findItem(item, id)); // recursive case
}
findItem(menuList, 22); // '抗老系列'
```

{% colorquote info %}
遞迴函式必須要寫終止條件，否則會有無窮迴圈（infinite loop）的情況發生，程式碼無法結束，導致 'max stack size exceeded' 錯誤
{% endcolorquote %}

## **遞迴元件（Recursive Component）**

使用前面的 `menuList` 作為範例，元件結構如下：

```
-| components/
---| TheMenu/
-----| index.vue
-----| MenuItem.vue
```

**建立父元件**

元件內包覆子元件，並透過 `props` 進行資料傳遞

```jsx
// components/TheMenu/index.vue
<template>
    <ul>
        <MenuItem :menu="menuList" />
    </ul>
</template>

<script>
export default {
    name: 'Menu',
    components: {
        MenuItem: () => import('./components/TheMenu/MenuItem.vue')
    },
    data() {
        return {
            menuList: { ... } // 同上面範例
        };
    }
}
</script>
```

**建立子元件**

在子元件裡面呼叫本身，將 `MenuItem` 當作子元件使用，就可以實現遞迴元件，遞迴停止的條件是內容渲染完畢的時候

{% colorquote info %}
元件必須加上 `name` 屬性，才可以在元件內使用**「自己」**
{% endcolorquote %}

```jsx
// components/TheMenu/MenuItem.vue
<template>
    <li>
        <button type="button" @click="toggleMenu">{{ menu.name }}</button>
        <ul v-if="menu.sub.length" v-show="isOpen">
            <MenuItem v-for="item in menu.sub" :key="item.id" :menu="item" />
        </ul>
    </li>
</template>

<script>
export default {
    name: 'MenuItem',
    props: {
        menu: {
            type: Object,
            default: () => {}
        }
    },
    data() {
        return {
            isOpen: false
        };
    },
    methods: {
        toggleMenu() {
            this.isOpen = !this.isOpen;
        }
    }
}
</script>
```

**元件應用**

```jsx
// views/Home.vue
<template>
  <TheMenu></TheMenu>
</template>

<script>
export default {
    name: 'Home',
    components: {
        TheMenu: () => import('./components/TheMenu/index.vue')
    }
}
</script>
```

結果呈現如下

<iframe height="470" style="width: 100%;" scrolling="no" title="Vue.js Recursive Component 遞迴元件實作巢狀選單" src="https://codepen.io/claire-chang-the-bashful/embed/OJwjZjX?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/claire-chang-the-bashful/pen/OJwjZjX">
  Vue.js Recursive Component 遞迴元件實作巢狀選單</a> by Claire Chang (<a href="https://codepen.io/claire-chang-the-bashful">@claire-chang-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

---

參考文章：

[https://medium.com/@paulyang1234/使用-vue-recursive-component-實現樹狀菜單-f1128e566cba](https://medium.com/@paulyang1234/%E4%BD%BF%E7%94%A8-vue-recursive-component-%E5%AF%A6%E7%8F%BE%E6%A8%B9%E7%8B%80%E8%8F%9C%E5%96%AE-f1128e566cba)

[https://medium.com/js-dojo/7-vue-patterns-that-you-should-be-using-more-often-b13cde4d2ae6](https://medium.com/js-dojo/7-vue-patterns-that-you-should-be-using-more-often-b13cde4d2ae6)