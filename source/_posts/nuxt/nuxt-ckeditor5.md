---
title: Nuxt.js 套件應用：CKEditor 5 文字編輯器
date: 2022-12-01
tags: [ nuxt2, ckeditor ]
category: Nuxt
description: CKEditor 是一套歷史悠久且功能完整、輕量的富文本編輯器（rich text editor），提供所見即所得的編輯區域，CKEditor 5 使用 MVC 架構、ES6 編寫、UI 簡潔，並與前端框架 Vue.js、React、Angular 做整合，讓我們可以更便利的開發應用
image: https://i.imgur.com/gjZEQ2A.png
---
> **版本：
nuxt 2.15.8
ckeditor: 38.0.1**
>

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/gjZEQ2A.png">
</div>

CKEditor 是一套歷史悠久且功能完整、輕量的富文本編輯器（rich text editor），為使用者提供所見即所得（WYSIWYG）的編輯區域，CKEditor 5 與舊版不同，使用 MVC 架構、ES6 編寫、UI 簡潔，且因應現在的前後端分離趨勢，與前端框架 React、Angular、and Vue.js 做整合，讓我們可以更便利的開發應用。

<!-- more -->

那麼我們就開始在 Nuxt 專案使用 [CKEditor 5](https://ckeditor.com/ckeditor-5/) 吧，首先選擇[編輯器類型](https://ckeditor.com/docs/ckeditor5/latest/installation/getting-started/predefined-builds.html#available-builds)（這裡使用 [Classic editor](https://ckeditor.com/docs/ckeditor5/latest/installation/getting-started/predefined-builds.html#classic-editor)），可以透過兩種方式安裝：

## **Adding a plugin to a build**

如果我們想要直接使用 Classic editor 配置好的功能，不需額外擴充，此方法可以快速簡易的建置，首先安裝套件：

```shell
npm install --save \
    @ckeditor/ckeditor5-vue2 \
    @ckeditor/ckeditor5-build-classic
```

接著在元件內加入編輯器

```jsx
<template>
    <div>
        <ckeditor :editor="editor" v-model="editorData"></ckeditor>
    </div>
</template>

<script>
import CKEditor from '@ckeditor/ckeditor5-vue2';
import ClassicEditor from '@ckeditor/ckeditor5-build-classic';

export default {
    name: 'Editor',
    components: {
        ckeditor: CKEditor.component
    },
    data() {
        return {
            editor: ClassicEditor,
            editorData: '<div>Hello World!</div>'
        };
    }
};
</script>
```

![](https://i.imgur.com/df3FV1i.png)

這樣就完成了，是不是很簡單呢！

但如果我們想擴充功能，像是新增下底線 `@ckeditor/ckeditor5-basic-styles/src/underline`，專案會直接報錯：

{% colorquote danger %}
Uncaught CKEditorError: ckeditor-duplicated-modules
{% endcolorquote %}

原因是 `@ckeditor/ckeditor5-build-classic` 為包裝好的內容，試著進到 node_modules/@ckeditor/ckeditor5-build-classic/package.json 可以看到相依套件已經安裝進去：

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img style="width: 100%; max-width: 500px;" src="https://i.imgur.com/0JNnw2N.png">
</div>

因此 CKeditor 在初始化時會因為模組重複執行導致錯誤，可以改用以下方法

## **Adding a plugin to an editor**

客製化配置所有功能，步驟相對複雜，首先安裝基本套件：

```bash
npm install --save \
    @ckeditor/ckeditor5-vue2 \
    @ckeditor/ckeditor5-editor-classic \
    @ckeditor/ckeditor5-dev-webpack-plugin \
    @ckeditor/ckeditor5-dev-utils \
    @ckeditor/ckeditor5-theme-lark \
    postcss@8 \
    postcss-loader@4 \
    raw-loader@4
```

**重點步驟：**需手動設定 webpack，這一步漏掉會直接報錯

```jsx
// nuxt.config.js
const path = require('path');
const CKEditorWebpackPlugin = require('@ckeditor/ckeditor5-dev-webpack-plugin');
const { styles } = require('@ckeditor/ckeditor5-dev-utils');

export default {
    build: {
        transpile: [ /ckeditor5-[^/\\]+[/\\]src[/\\].+\.js$/ ],
        plugins: [
            // Only with ssr: false
            new CKEditorWebpackPlugin({
                // See https://ckeditor.com/docs/ckeditor5/latest/features/ui-language.html
                language: 'zh',
                additionalLanguages: 'all',
                addMainLanguageTranslationsToAllAssets: true
            })
        ],
        // If you don't add postcss, the CKEditor css will not work.
        postcss: {
            postcssOptions: styles.getPostCssConfig({
                themeImporter: {
                    themePath: require.resolve('@ckeditor/ckeditor5-theme-lark')
                },
                minify: true
            })
        },
        extend(config) {
            // If you do not exclude and use raw-loader to load svg, the following errors will be caused.
            // Cannot read property 'getAttribute' of null
            const svgRule = config.module.rules.find(item => {
                return /svg/.test(item.test);
            });
            svgRule.exclude = [ path.join(__dirname, 'node_modules', '@ckeditor') ];
            // add svg to load raw-loader
            config.module.rules.push({
                test: /ckeditor5-[^/\\]+[/\\]theme[/\\]icons[/\\][^/\\]+\.svg$/,
                use: [ 'raw-loader' ]
            });
        }
    }
}
```

**webpack 設定說明：**

- [語言設定](https://ckeditor.com/docs/ckeditor5/latest/features/ui-language.html#building-the-editor-using-a-specific-language)：這裡設為繁體中文（'zh'）
- 必須在 `ssr: false` 條件下才能運作，否則會報錯誤 window is not defined，跟 [CKEditorWebpackPlugin] Error: No translation has been found for the XX language
- 排除 `@ckeditor` svg，並使用 raw-loader 讀取 svg 內容，否則會拋 cannot read property 'getAttribute' of null 錯誤
- 使用 PostCSS 轉換 css 代碼，否則會 css 樣式會讀取不到

---

接著來試著安裝編輯器功能，範例安裝 **基本樣式 / 字級 / 字體**：

```bash
npm install --save \
    @ckeditor/ckeditor5-essentials \
    @ckeditor/ckeditor5-font \
    @ckeditor/ckeditor5-paragraph \
```

{% colorquote info %}
所有 ckeditor 套件**版本**必須相同，否則會發生錯誤（@ckeditor/ckeditor5-dev-* 跟 @ckeditor/ckeditor5-vue2 除外）
{% endcolorquote %}

**基礎設定介紹：**

- placeholder：編輯器佔位符
- plugins：編輯器應用功能
- toolbar：工具列配置，也可以加入分隔符號 `|`
- 其他：各功能的相關設定請見官方文件

```jsx
<template>
    <div>
        <ckeditor :editor="editor" :config="editorConfig" v-model="editorData"></ckeditor>
    </div>
</template>

<script>
import CKEditor from '@ckeditor/ckeditor5-vue2';
import ClassicEditor from '@ckeditor/ckeditor5-editor-classic/src/classiceditor';
import Essentials from '@ckeditor/ckeditor5-essentials/src/essentials';
import FontFamily from '@ckeditor/ckeditor5-font/src/fontfamily';
import FontSize from '@ckeditor/ckeditor5-font/src/fontsize';
import Paragraph from '@ckeditor/ckeditor5-paragraph/src/paragraph';

export default {
    name: 'Editor',
    components: {
        ckeditor: CKEditor.component
    },
    data() {
        return {
            editor: ClassicEditor,
            editorData: '',
            editorConfig: {
                placeholder: '請輸入內容',
                plugins: [ Essentials, FontFamily, FontSize, Paragraph ],
                toolbar: {
                    items: [ 'fontSize', 'fontFamily', '|' ],
                    shouldNotGroupWhenFull: true
                },
                fontSize: {
                    options: [ 12, 16, 18, 20, 24, 32, 40, 48 ],
                    supportAllValues: true
                }
            }
        };
    }
};
</script>
```

這樣就可以成功看到畫面囉 🙌

![](https://i.imgur.com/q950cN2.png)

{% colorquote info %}
是否覺得功能安裝跟配置有點麻煩呢？官方貼心的提供 [online-builder](https://ckeditor.com/ckeditor-5/online-builder/) ，可以先挑選功能並配置好 toolbar，確認畫面後再安裝
{% endcolorquote %}

## **全域註冊**

當專案有很多地方會使用到編輯器，我們也可以全域引入

新增檔案置於 plugins，這裡命名 ckeditor.js

```jsx
// plugins/ckeditor.js
import Vue from 'vue';
import CKEditor from '@ckeditor/ckeditor5-vue2';

Vue.use(CKEditor);
```

nuxt.config.js 配置

```jsx
// nuxt.config.js
export default {
    plugins: [
        { src: '@/plugins/ckeditor' }
    ]   
}
```

## **CSS style**

實務應用上會在後台使用編輯器來設計內容，然後將 HTML 內容儲存到資料庫，於前台取回資料並渲染在畫面上，所以會需要配置相關的 CSS，確保前後台樣式一致。

只要複製 [Content styles](https://ckeditor.com/docs/ckeditor5/latest/installation/advanced/content-styles.html) 在專案內，然後在編輯器內容的外層容器加上 **`ck-content`** CSS class 就可以囉，範例如下：

```html
<template>
    <div class="ck-content">
        // 編輯器內容
        <div v-html="content"></div>
    </div>
</template>
```

---

參考文章：

[https://github.com/changemyminds/nuxtjs-integrate-ckeditor5?ref=vuejsexamples.com](https://github.com/changemyminds/nuxtjs-integrate-ckeditor5?ref=vuejsexamples.com)
[https://ithelp.ithome.com.tw/articles/10198816](https://ithelp.ithome.com.tw/articles/10198816)
[https://medium.com/@charming_rust_oyster_221/ckeditor-5-文字編輯器-研究心得-519c97f20a4](https://medium.com/@charming_rust_oyster_221/ckeditor-5-%E6%96%87%E5%AD%97%E7%B7%A8%E8%BC%AF%E5%99%A8-%E7%A0%94%E7%A9%B6%E5%BF%83%E5%BE%97-519c97f20a4)
[https://ckeditor.com/docs/ckeditor5/latest/installation/frameworks/vuejs-v2.html](https://ckeditor.com/docs/ckeditor5/latest/installation/frameworks/vuejs-v2.html)