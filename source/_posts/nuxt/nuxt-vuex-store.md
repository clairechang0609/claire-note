---
title: Nuxt.js VueX Store 搭配 vuex-persistedstate 狀態保存工具
date: 2022-11-22
tags: [ nuxt2 ]
category: Nuxt
description: VueX 狀態管理工具適合處理專案內複雜度高的溝通，本篇說明如何在 Nuxt 專案安裝與應用
image: https://i.imgur.com/2y1jUjQ.png
---
> **版本：nuxt 2.15.8**
>

前情提要一下，在 Vue 的專案下，常會需要做父子元件或是頁面之間的溝通傳值，如果說只是單層（ex：父元件 → 子元件、子元件 → 父元件、頁面 → 頁面），我們可以很簡單的使用 `props` 、 `emit` 或 `event bus` 即可，但在大型專案，共用資料就不是如此單純，可能會有元件內含元件、多層級的溝通，如果只用上述方法，對於開發及除錯都不便利，如下圖範例，**元件 1-1** 跟**元件 2-1** 的溝通相對複雜。

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img style="width: 100%; max-width: 600px;" src="https://i.imgur.com/2y1jUjQ.png">
</div>

<!-- more -->

為了處理高難度溝通，VueX 狀態管理工具就誕生了，那麼在 Nuxt 專案下又該怎麼使用呢？

首先先安裝 VueX 套件 `npm i vuex@3.6.2`

{% colorquote info %}
Nuxt v2.x 必須搭配 VueX v3.x
{% endcolorquote %}

接著在專案最外層新增 store 資料夾，並在裡面建立 .js 檔，範例使用 userInfo.js

```jsx
// store/userInfo.js
export const state = () => {};
export const getters = {};
export const mutations = {};
export const actions = {};
```

Nuxt 專案會自動創建實例 `new Vuex.Store()`，將 store 檔案包裝進模組內，像這樣：

```jsx
new Vuex.Store({
    modules: {
        userInfo: {
            namespaced: true,
            state: () => {},
            getters: {},
            mutations: {},
            actions: {}
        }
    }
});
```

**VueX 基本架構：**

- state：用以儲存狀態，功能同 .vue 檔內的 data，因此會使用方法來包裝，並 return 內容
- getters：功能同 computed，用以計算 state 內的狀態，不能直接改變 state
- mutations：用來更改 state，不能使用非同步語法
- actions：非同步語法只能寫在 actions，不能直接改變 state，需透過 mutations 改變 state

接著來替 store 加入一些內容吧

```jsx
// store/userInfo.js
export const state = () => ({
    count: 0,
    products: [
        {
            name: 'food',
            onSale: false
        },
        {
    name: 'drink',
            onSale: true
        }
    ]
});
export const getters = {
    onSaleProducts(state) {
        return state.products.filter(item => item.onSale);
    }
};
export const mutations = {
    increment(state, number) {
        state.count += number;
    }
};
export const actions = {
    incrementAsync({ commit }, number) {
        setTimeout(() => {
            commit('increment', number);
        }, 1000);
    }
};
```

如果要在頁面使用 store 的內容，有以下兩種方式：

### **透過 this.$store 操作**

VueX 將 store 注入到 Vue 實例，因此我們透過 $store 就可以取得相關資料及方法，使用方式如下（範例頁面 pages/about.vue）：

```jsx
// pages/about.vue
export default {
    name: 'About',
    computed: {
        count() { // state
            return this.$store.state.userInfo.count;
        }
        onSaleProducts() { // getters
            return this.$store.getters['userInfo/onSaleProducts'];
        }
    },
    methods: {
        increment(number) { // mutations
            this.$store.commit('userInfo/increment', number);
        },
        incrementAsync(number) { // actions
            this.$store.dispatch('userInfo/incrementAsync', number);
        }
    }
}
```

但是每一筆資料或是方法，都透過 this.$store 取得，程式碼冗長，如果今天我們有三個 store 儲存庫，易讀性又更低了，難道沒有簡單的使用方式嗎？

### **透過輔助函式**

VueX 提供一系列輔助函式（mapState, mapGetters, mapMutations, mapActions），可以幫助我們更簡易的取得 store 內容，首先必須從 VueX 引入輔助函式，接著來改寫 about.vue 頁面

```jsx
// pages/about.vue
import { mapState, mapGetters, mapMutations, mapActions } from 'vuex';

export default {
    name: 'About',
    computed: {
        ...mapState('userInfo', [ 'count' ]),
        ...mapGetters('userInfo', [ 'onSaleProducts' ])
    },
    methods: {
        ...mapMutations('userInfo', [ 'increment' ]),
        ...mapActions('userInfo', [ 'incrementAsync' ])
    }
}
```

透過輔助函式，減少許多程式碼，閱讀起來也輕鬆許多。

---

VueX 相當方便，但還是存在一個問題，就是**當頁面重新整理的時候，會回復到初始狀態。**

某些情境下，我們需要保留更新後的資料，例如登入後儲存使用者資訊到 store，重整頁面後資料被清空，使用者必須重新登入。為了解決這個問題，我們可以將資料儲存於 localStorage，待畫面重整後再將資料取回放入 store，或者是使用套件 **[vuex-persistedstate](https://github.com/robinvdvleuten/vuex-persistedstate)**

{% colorquote info %}
2022.02.04 vuex-persistedstate 套件已不繼續維護更新
{% endcolorquote %}

接下來說明如何 Nuxt 專案如何搭配 vuex-persistedstate，存放在 localStorage 的內容，可以輕易地被讀取，因此我們需要將內容加密（搭配套件 [secure-ls](https://www.npmjs.com/package/secure-ls)） 

首先安裝套件：`npm i vuex-persistedstate` 及加密套件：`npm i secure-ls`，接著於 plugins 增加檔案，範例使用 persistedstate.js

```jsx
// plugins/persistedstate.js
import createPersistedState from 'vuex-persistedstate';
import SecureLS from 'secure-ls';

const ls = new SecureLS({
    encodingType: 'aes', // 加密方式，預設 base64
    isCompression: false, // 是否壓縮數據
    encryptionSecret: process.env.APP_KEY // 加密 key
});

export default ({ store, isHMR }) => {
    if (isHMR) { // 避免在熱更新的時候重複觸發(npm run dev)
        return;
    }
    window.onNuxtReady(() => {
        createPersistedState({
            key: 'test', // 自訂 localStorage key
            storage: {
                getItem: key => ls.get(key),
                setItem: (key, value) => ls.set(key, value),
                removeItem: key => ls.remove(key)
            }
        })(store);
    });
};
```

接著配置到 nuxt.config.js 內

```jsx
// nuxt.config.js
export default {
    plugins: [
        { src: '@/plugins/persistedstate', ssr: false }
    ]
}
```

這樣一來即使畫面重整，store 也可以順利保存資料，在 localStorage 查看 test 內容也確實被加密了！

---

參考文章：

[https://medium.com/itsems-frontend/vue-vuex1-state-mutations-364163b3acac](https://medium.com/itsems-frontend/vue-vuex1-state-mutations-364163b3acac)

[https://chiafangsung.medium.com/使用-vuex-persistedstate-維持-vuex-狀態-f0d7c522c73a](https://chiafangsung.medium.com/%E4%BD%BF%E7%94%A8-vuex-persistedstate-%E7%B6%AD%E6%8C%81-vuex-%E7%8B%80%E6%85%8B-f0d7c522c73a)