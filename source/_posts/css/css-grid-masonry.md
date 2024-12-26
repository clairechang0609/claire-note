---
title: CSS Grid Layout Module 格線佈局 (2) Masonry 瀑布流應用
date: 2023-05-23
tags: [ css, masonry ]
category: CSS
description: 本篇介紹如何使用 Grid 格線系統製作瀑布流版面，瀑布流是一種網頁佈局的設計方式，特點是將內容項目垂直排列，並根據可用的空間進行自適應調整，形成像瀑布一樣的流動效果
image: https://i.imgur.com/XdKc6zc.png
---

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 600px; max-width: 100%;" src="https://i.imgur.com/XdKc6zc.png">
</div>

[上一篇文章](https://clairechang.tw/2023/05/19/css/css-grid-layout/) 介紹了 CSS Grid，接下來運用網格系統來實作瀑布流版面。

瀑布流（Waterfall Flow）是一種網頁佈局的設計方式。瀑布流的特點是將內容項目垂直排列，並根據可用的空間進行自適應調整，形成像瀑布一樣的流動效果 。

瀑布流佈局中，每個元素的寬度通常是固定的，但高度會根據內容的大小變化。這樣可以使元素在垂直方向上緊密堆疊，填滿可用空間。當一列的空間被填滿後，下一個元素會自動排列到空間較少的列中，達到均衡的排版。

透過 CSS Grid 或 Flexbox 可以達成瀑布流佈局的效果，本篇會使用 CSS Grid 搭配 JavaScript 來完成實作。

<!-- more -->

{% colorquote info %}
本篇作法無法達成轉場效果，如果想讓畫面轉換更加流暢，可以參考 [這篇文章](https://clairechang.tw/2022/12/23/nuxt/nuxt-masonry/)（masonry-layout 套件運用）
{% endcolorquote %}

## **概念解析**

**利用 CSS Grid 格線系統來達成瀑布流的效果：**

- `grid-auto-rows` 設定列高
- `grid-gap` 建立網格間距
- 利用 Javascript 取得元素的高度，並計算 `grid-row-end` 屬性
- 利用網格系統，每個元素會去找到空間使用最少的列，並排列在其後

格線的結構如下圖：

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 600px; max-width: 100%;" src="https://i.imgur.com/yA4pjX4.png">
</div>

## **HTML**

- 外層容器 `.container`
- 內層元素 `.item`

```html
<ul class="container">
    <li class="item">
        <div class="content">
            <h3>1.</h3>
            Lorem ipsum dolor sit amet consectetur adipisicing elit. Ipsam voluptatibus hic laborum ea maiores, harum nobis. Placeat numquam ad laboriosam esse est porro, cum deserunt velit libero beatae quam necessitatibus. Iste sapiente commodi est labore amet quae rerum magni! Alias eum molestias sint nostrum optio ipsam quia delectus sunt. Rerum
        </div>
    </li>
    <li class="item">
        <div class="content">
            <h3>2.</h3>
            Lorem ipsum dolor sit amet consectetur, adipisicing elit. Voluptatibus dolore natus ea
        </div>
    </li>

    ...
        
    <li class="item">
        <div class="content">
            <h3>16.</h3>
            Lorem ipsum dolor sit amet consectetur, adipisicing elit. Sequi itaque veniam at labore consequuntur dicta adipisci ut laudantium deserunt
        </div>
  </li>
</ul>
```

## **CSS**

### **外層容器**

- `auto-fill`：搭配 `repeat(count, value)` 函式使用，根據可用空間填滿欄或列
- `minmax(min-value, max-value)`：指定範圍最小與最大值，自動調整欄或列的尺寸

```scss
.container {
    display: grid;
    grid-gap: 1rem;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    grid-auto-rows: 1rem;
}
```

### **內層元素**

- `grid-row-end`：透過 Javascript 動態計算

## **Javascript**

- rowHeight：取得 `grid-auto-rows` 值
- rowGap：取得 `grid-row-gap` 值
- 透過 `getBoundingClientRect().height` 取得元素的高度
- 利用以上值計算元素 `grid-row-end: span <number>`：<number> 表示所佔列數，根據起始位置往後推算
- 監聽螢幕 resize 事件，重新計算元素高度

{% colorquote info %}
**getBoundingClientRect().height vs offsetHeight：**
getBoundingClientRect().height 取得的是相對於螢幕視窗的可視高（渲染後的結果），而 offsetHeight 取得的是元素完整高，getBoundingClientRect().height 會受到 `transform` 影響，例如元素設定高度 100px 以及 `transform: scale(0.5);`，getBoundingClientRect().height 得到 50，而 offsetHeight 得到的是 100
{% endcolorquote %}

```jsx
const grid = document.querySelector(".container");
const style = window.getComputedStyle(grid);
const allItems = document.querySelectorAll(".item");

const resizeGridItem = (item) => {
    rowHeight = parseInt(style.getPropertyValue("grid-auto-rows"));
    rowGap = parseInt(style.getPropertyValue("grid-row-gap"));
    rowSpan = Math.ceil((item.querySelector(".content").getBoundingClientRect().height + rowGap) / (rowHeight + rowGap));
    item.style.gridRowEnd = `span ${rowSpan}`;
}

const resizeAllGridItems = () => {
    allItems.forEach(ele => {
        resizeGridItem(ele);
    });
}

window.onload = resizeAllGridItems();
window.addEventListener("resize", resizeAllGridItems);
```

## **範例程式碼**

<iframe height="470" style="width: 100%;" scrolling="no" title="CSS Grid Layout Masonry" src="https://codepen.io/claire-chang-the-bashful/embed/zYmyZjB?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/claire-chang-the-bashful/pen/zYmyZjB">
  CSS Grid Layout Masonry</a> by Claire Chang (<a href="https://codepen.io/claire-chang-the-bashful">@claire-chang-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

---

參考文章：

[https://css-tricks.com/piecing-together-approaches-for-a-css-masonry-layout/](https://css-tricks.com/piecing-together-approaches-for-a-css-masonry-layout/)
[https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout/Masonry_Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout/Masonry_Layout)
[https://medium.com/@andybarefoot/a-masonry-style-layout-using-css-grid-8c663d355ebb](https://medium.com/@andybarefoot/a-masonry-style-layout-using-css-grid-8c663d355ebb)