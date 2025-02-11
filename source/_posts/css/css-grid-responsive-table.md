---
title: CSS Grid Layout Module 格線佈局 (3) Responsive Table 響應式表格應用
date: 2023-05-24
tags: [ css ]
category: CSS
description: 本篇介紹如何透過 Grid 格線系統製作響應式表格，桌機上用表格呈現內容，而在行動裝置上使用卡片式版面，這樣的設計優化能夠提升使用者在不同裝置上的瀏覽體驗
image: https://i.imgur.com/7zNxpBR.png
---

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 700px; max-width: 100%;" src="https://i.imgur.com/7zNxpBR.png">
</div>

在響應式網頁設計中，常需要處理不同裝置間版面的切換，以表格為例：在桌機上用表格呈現內容，而在行動裝置上使用卡片式版面，這樣的設計優化能夠提升使用者在不同裝置上的瀏覽體驗。

為了達成這種優化，有以下幾種做法：

1. 製作兩個版面，利用斷點來進行版面切換
2. 製作一個共用版面，搭配 CSS Grid Layout 進行設計，在手機版時調整 `<table>` 的屬性，達成卡片的視覺效果

本篇介紹第二個做法，這麼做可以降低程式碼的複雜度，提升程式碼可重用性，專案維護性也較高。

<!-- more -->

{% colorquote info %}
CSS Grid 相關知識，可以參考 [這篇文章](https://clairechang.tw/2023/05/19/css/css-grid-layout/)
{% endcolorquote %}

## **概念解析**

使用 `<table>` 建立表格，在行動裝置斷點，將 `<tr>` 設定為 `display: grid`，利用格線系統達成卡片式版面

## **HTML**

```html
<table>
    <thead>
        <tr>
            <th scope="col">#</th>
            <th scope="col">Product</th>
            <th scope="col">Ingredient</th>
            <th scope="col">Price</th>
            <th scope="col"></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td class="index">1</td>
            <td class="product">Apple Juice</td>
            <td class="ingredient">Apple, Sugar, Water</td>
            <td class="price">$ 100</td>
            <td class="add-btn">
                <button type="button">Add</button>
            </td>
        </tr>
            
            ...

    </tbody>
</table>
```

## **CSS**

- 設定斷點為 W768（Ipad）
- 當螢幕視窗**小於** 768px，`tbody > tr` 設定為 `display: grid`，`td` 欄位利用格線系統排版

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 350px; max-width: 100%;" src="https://i.imgur.com/P1IkxUm.png">
</div>

```scss
table {
    width: 1000px;
    max-width: 100%;
    margin: 0 auto;
    white-space: nowrap;
    border-spacing: 5px;
}

@media screen and (max-width: 768px) {
    thead {
        display: none;
    }
    tbody > tr {
        display: grid;
        grid-gap: 0.5rem;
        grid-template-columns: repeat(2, 1fr);
        grid-template-rows: repeat(3, auto);
    }
    .index {
        display: none;
    }
    .product {
        justify-self: start;
        margin-bottom: 0.5rem;
    }
    .ingredient {
        grid-column: 1;
        grid-row: 2;
        justify-self: start;
    }
    .price {
        grid-column: 1;
        grid-row: 3;
        justify-self: start;
        align-self: center;
    }
    .add-btn {
        grid-column: 2;
        grid-row: 3;
        justify-self: end;
        align-self: center;
    }
}
```

## **範例程式碼**

<iframe height="470" style="width: 100%;" scrolling="no" title="CSS Grid Responsive Table" src="https://codepen.io/claire-chang-the-bashful/embed/zYmeJKV?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/claire-chang-the-bashful/pen/zYmeJKV">
  CSS Grid Responsive Table</a> by Claire Chang (<a href="https://codepen.io/claire-chang-the-bashful">@claire-chang-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

---

參考資源：

https://adamlynch.com/flexible-data-tables-with-css-grid/
https://codepen.io/gilli/pen/QVaWGR
