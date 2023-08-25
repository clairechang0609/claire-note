---
title: CSS 圖片自適應容器（object-fit / background-image）
date: 2023-05-10 18:30
tags: [ css ]
category: CSS
description: 網頁切版中很常遇到一個問題：當圖片大小不一致時，如何讓圖片在不變形的情況下自適應容器大小？本篇介紹透過 object-fit 以及 background-image 兩種方式達成自適應效果
image: https://i.imgur.com/qohMKbO.png
---

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 700px;" src="https://i.imgur.com/qohMKbO.png">
</div>

網頁切版中很常遇到一個問題：當圖片大小不一致時，如何讓圖片在不變形的情況下自適應容器大小？圖片變形不僅會影響網頁的美感和視覺效果，甚至影響使用者閱讀。本篇介紹兩個方法來解決這個問題

<!-- more -->

## **background-image**

背景圖片是很簡單支援度高的寫法，使用 background-image 取代 `<img>` 圖片標籤，並使用 background 相關屬性來調整圖片

```html
<div class="background-image-container"></div>
```

```css
.background-image-container {
    width: 300px;
    height: 300px;
    background-image: url("https://i.imgur.com/W7xqcWS.jpg");
    background-size: cover;
    background-position: center;
}
```

使用 `background-size: cover` 讓圖片覆蓋整個容器，並保留圖片比例，並搭配 `background-position: center` 確保圖片置中，圖片呈現與裁切如下：

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/z08rJLs.png">
</div>

## **object-fit**

某些情境我們會需要使用 `<img>` 圖片標籤，像是圖片需能被搜尋引擎爬蟲讀取（需要有 alt 屬性），這時就可以透過 object-fit 來達成

#### **object-fit 選項**

- **fill：**預設效果，元素會被拉伸填滿外層容器，長寬比會被扭曲
- **contain：**元素的長寬比不變，最大邊會填滿容器
- **cover：**元素的長寬比不變，最小邊會填滿容器，也就是元素可能會被裁切
- **none：**元素維持本身的大小和長寬比，不會被調整
- **scale-down：**元素維持本身的大小和長寬比，但如果大於容器，會被縮小適應容器

```html
<div class="object-fit-container">
    <img src="https://i.imgur.com/W7xqcWS.jpg" alt="landscape" />
</div>
```

```css
.object-fit-container {
    width: 300px;
    height: 300px;
    img {
        object-fit: cover;
        width: 100%;
        height: 100%;
    }
}
```

`width` 跟 `height` 用來設定圖片需要被填滿的範圍，另外也可以透過 `object-position` 來調整圖片的位置（預設是置中）

### **範例程式碼**

<iframe height="470" style="width: 100%;" scrolling="no" title="css image fit container" src="https://codepen.io/claire-chang-the-bashful/embed/KKGoOOb?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/claire-chang-the-bashful/pen/KKGoOOb">
  css image fit container</a> by Claire Chang (<a href="https://codepen.io/claire-chang-the-bashful">@claire-chang-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

---

參考文章：

[https://ithelp.ithome.com.tw/articles/10250499](https://ithelp.ithome.com.tw/articles/10250499)
[https://www.casper.tw/development/2020/10/11/img-cover/](https://www.casper.tw/development/2020/10/11/img-cover/)