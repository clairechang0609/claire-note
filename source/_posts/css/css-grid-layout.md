---
title: CSS Grid Layout Module 格線佈局 (1) 完整介紹與範例
date: 2023-05-19
tags: [ css ]
category: CSS
description: CSS Grid 佈局系統為二維網格系統，能夠幫助我們輕鬆控制網格中元素位置和大小，提供更豐富靈活的佈局控制，同時操控「欄（column）」跟「列（row）」，適合用於複雜度高的網頁版型
image: https://i.imgur.com/rvkEbdJ.png
---

CSS Grid 佈局系統能夠幫助我們輕鬆控制網格中元素位置和大小，跟 Flex 比較：

- **Flex：**一維佈局系統，主要用於單行或是單列的對齊跟排列
- **Grid：**二維網格系統，提供更豐富靈活的佈局控制，同時操控「欄（column）」跟「列（row）」，適合用於複雜度高的網頁版型

## **常用屬性**

### **外層容器**

- `display`：定義容器類型（`grid` / `inline-grid` / `subgrid`）
- `grid-template-columns`：定義網格欄的大小和數量
- `grid-template-rows`：定義網格列的大小和數量
- `grid-gap` / `grid-column-gap` / `grid-row-gap`：設定網格間的間距
- `grid-template-areas`：定義網格空間位置，由單個或數個字串組成，搭配 `grid-area` 使用
- `justify-items` / `align-items` / `place-items`：定義所有元素的對齊方式，同 flex 用法

### **內層元素**

- `grid-column` / `grid-column-start` / `grid-column-end`：控制元素在網格中所佔欄空間位置
- `grid-row` / `grid-row-start` / `grid-row-end`：控制元素在網格中所佔列空間位置
- `grid-area`：定義元素空間名稱，搭配 `grid-template-areas` 使用
- `justify-self` / `align-self` / `place-self`：定義單一元素的對齊方式，同 flex 用法

### **元素自動佈局**

- `grid-auto-columns`：定義預設欄的大小
- `grid-auto-rows`：定義預設列的大小
- `grid-auto-flow`：控制自動佈局的方向和順序

<!-- more -->

## **grid-template-columns / grid-template-rows / grid-column / grid-row**

### **外層容器**

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 600px; max-width: 100%;" src="https://i.imgur.com/LbW8KBl.png">
</div>

```html
<div class="container"> ... </div>
```

`grid-template-columns` 與 `grid-template-rows` 可以指定線段名稱（用 [] 表示），如果不指定，預設會從 1 開始編號，如果輸入 -1，則會從最後一條線開始算起

```css
.container {
    display: grid;
    grid-template-columns: 100px 150px 150px auto;
    grid-template-rows: [start] 75px [line-2] 150px [line-3] 25% [end];
    grid-gap: 5px;
}
```

一條線段可以給定多個名稱

```css
.container {
    grid-template-rows: [start line-1] 75px [line-2] 150px [line-3] 25% [end line-4];
}
```

如果有連續重複寬 / 高，可以使用 `repeat(count, value)` 簡化

```css
.container {
    grid-template-columns: 100px repeat(2, 150px) auto;
}
```

搭配 `minmax(min-value, max-value)` 函式，設定範圍最大與最小值，達成自適應空間效果

```css
.container {
    grid-template-columns: repeat(2, minnax(300px, 50%));
}
```

除了上述範例的單位，也能運用 **fr（fraction）**單位來分配空間，fr 單位用於將可用空間分為相對比例，能夠自適應容器變化。

以下為例：假設目前容器寬 500px，扣掉 100px，容器內可用空間為 400px，共配置了 4fr，因此 1fr 可以分到 100px

```css
.container {
    grid-template-columns: 1fr 2fr 100px 1fr;
}
```

### **內層元素**

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 600px; max-width: 100%;" src="https://i.imgur.com/rvkEbdJ.png">
</div>

```html
<div class="container">
    <div class="item-a"></div>
    <div class="item-b"></div>
    <div class="item-c"></div>
    <div class="item-d"></div>
</div>
```

`grid-column` / `grid-row` 可以帶入以下值：

- line：線段編號
- span <number>：元素佔欄位數
- span <nama>：元素所在格線名稱
- auto：自動

```css
.item-a {
    grid-column-start: 1;
    grid-column-end: 3;
    grid-row-start: start;
    grid-row-end: line-2;
}
.item-b {
    grid-column: 1 / 2; // 合併 grid-column-start & grid-column-end
    grid-row: line-2 / line-3; // 合併 grid-row-start & grid-row-end
}
.item-c {
    grid-column: 2 / span 3;
    grid-row: line-2 / span 1;
}
.item-d {
    grid-column: 1 / -1;
    grid-row: line-3;
}
```

---

## **grid-template-areas / grid-areas**

### **外層容器＋內層元素**

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 600px; max-width: 100%;" src="https://i.imgur.com/VLhVGXk.png">
</div>

```css
.container {
    display: grid;
    grid-template-columns: 100px 150px 150px auto;
    grid-template-rows: [start] 75px [line-2] 150px [line-3] 25% [end];
    grid-gap: 5px;
    grid-template-area:
        "header header . ."
    "sidebar content content content"
    "footer footer footer footer";
}

.item-a {
    grid-area: header;
}
.item-b {
    grid-area: sidebar;
}
.item-c {
    grid-area: content;
}
.item-d {
    grid-area: footer;
}
```

---

## **justify-items / align-items / justify-self / align-self**

用於控制內層元素在容器中的對齊方式，與 flex 用法相同，沒有定義時預設為 `stretch` （填滿容器）

{% colorquote info %}
元素若未定義位置，預設會是由左至右，由上至下順排，一個元素佔一個網格位置
{% endcolorquote %}

### **外層容器統一定義**

- align-items
- justify-items
- place-items：合併 align-items 與 justify-items

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 500px; max-width: 100%;" src="https://i.imgur.com/rLftI3p.png">
</div>

```css
.container {
    justify-items: start;
    align-items: stretch;
}
```

等同於

```css
.container {
    place-items: stretch start;
}
```

### **內層元素個別定義**

- align-self
- justify-self
- place-self：合併 align-self 與 justify-self

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 500px; max-width: 100%;" src="https://i.imgur.com/DAAcFne.png">
</div>

```css
.item-a {
    justify-self: start;
    align-self: start;
}
.item-b {
    justify-self: center;
    align-self: stretch;
}
.item-c {
    justify-self: stretch;
    align-self: center;
}
.item-d {
    place-self: end stretch; // 合併寫法
}
```

## **grid-auto-columns / grid-auto-rows / grid-auto-flows**

當透過內層元素數量超過預先定義的網格數量，或是元素超出網格外，grid-auto-columns / grid-auto-rows 可以協助我們自動定義欄或列的尺寸，範例說明：

```html
<div class="container">
    <div class="item">1</div>
    <div class="item">2</div>
    ...
    <div class="item">16</div>
</div>
```

```css
.container {
    grid-template-columns: repeat(4, 100px);
    grid-template-rows: repeat(2, 50px);
}
```

未定義前，畫面呈現如下：

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 600px; max-width: 100%;" src="https://i.imgur.com/PMD9rRL.png">
</div>

加入 grid-auto-columns 與 grid-auto-rows

```css
.container {
    grid-template-columns: repeat(4, 100px);
    grid-template-rows: repeat(2, 50px);
    grid-auto-columns: 100px;
    grid-auto-rows: 50px;
}
```

畫面呈現如下：

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 600px; max-width: 100%;" src="https://i.imgur.com/jVQN67D.png">
</div>

grid-auto-flow 用來自動定義網格內元素的排列方式，預設值為 `row`，以上面範例作為延伸

```css
.container {
    grid-auto-flow: column;
}
```

畫面呈現如下：

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/m3mgRNx.png">
</div>

---

### **完整範例程式碼**

<iframe height="470" style="width: 100%;" scrolling="no" title="CSS Grid Layout Guide" src="https://codepen.io/claire-chang-the-bashful/embed/MWPXyrd?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/claire-chang-the-bashful/pen/MWPXyrd">
  CSS Grid Layout Guide</a> by Claire Chang (<a href="https://codepen.io/claire-chang-the-bashful">@claire-chang-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

---

參考文章：

[https://css-tricks.com/snippets/css/complete-guide-grid/](https://css-tricks.com/snippets/css/complete-guide-grid/)
[https://www.casper.tw/css/2017/03/22/css-grid-layout/](https://www.casper.tw/css/2017/03/22/css-grid-layout/)
[https://ithelp.ithome.com.tw/articles/10268087](https://ithelp.ithome.com.tw/articles/10268087)