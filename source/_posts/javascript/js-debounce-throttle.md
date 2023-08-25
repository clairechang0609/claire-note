---
title: Javascript Debounce & Throttle 提升網頁效能
date: 2023-02-04 12:00
tags: [ javascript ]
category: Javascript
description: debounce 與 throttle 為 javascript 中常用的技術，用來防止 scroll, resize 等事件處理器在短時間內被頻繁觸發，綁定的函示重複執行，造成網頁不斷重新運算進而影響效能，本篇將分別說明兩者的操作方式
image: https://imgur.com/scEMEZM.png
---

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
    <img style="width: 100%; max-width: 700px;" src="https://imgur.com/scEMEZM.png">
</div>

`debounce` 與 `throttle` 用來防止 scroll, resize 等事件處理器在短時間內被頻繁觸發，綁定的函示重複執行，造成網頁不斷重新運算進而影響效能，作法就是控制函式觸發的次數或頻率，以下分別說明兩者的操作方式。

## **Debounce**

概念是加入一個倒數計時器，連續觸發時會一直重新倒數，直到計時器歸零，才執行函式。

{% colorquote %}
**簡單舉例**：
就像是便利商店的自動門，當一段時間內頻繁有客人進來，自動門會持續開著，直到大家都進入後，等待幾秒才關上。
{% endcolorquote %}

<!-- more -->

```jsx
function debounce(func, delay = 1000) {
    let timer = null;
    return function(...args) {
        clearTimeout(timer);
        timer = setTimeout(() => {
            func.apply(this, args);
        }, delay)
    }
}
```

利用閉包（closure）將 timer 變數與內部函式分享，`func.apply(this, args)` 可以替換成 `func.call(this, ...args)` 或是 `func.bind(this)(...args)`

{% colorquote info %}
如果不太清楚函數原型方法 **call**、**apply**、**bind**，以及裡面代入的 **this** 和 **argument**，可以參考[這一篇](https://clairechang.tw/2023/02/04/javascript/js-call-apply-bind/)
{% endcolorquote %}

**範例：input 輸入搜尋事件**

當輸入停止超過一秒後，才去讀取輸入內容

```html
<input type="text" placeholder="請輸入搜尋內容" id="search" />
<p id="content"></p>
```

```jsx
const input = document.getElementById('search');
const content = document.getElementById('content');

input.addEventListener('input', debounce(updateContent));

function updateContent(e) {
    content.textContent = e.target.value;
}
```

範例畫面：

<iframe height="470" style="width: 100%;" scrolling="no" title="Javascript Debounce" src="https://codepen.io/claire-chang-the-bashful/embed/bGjOXoP?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/claire-chang-the-bashful/pen/bGjOXoP">
  Javascript Debounce</a> by Claire Chang (<a href="https://codepen.io/claire-chang-the-bashful">@claire-chang-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

## **Throttle**

節流器，目的是減少事件執行的次數，事件連續觸發時製造間隔來控制運行次數。

{% colorquote %}
**簡單舉例**：
像是等紅綠燈，在固定的時間才會轉成綠燈，如果錯過了，必須等待一段時間，直到下次綠燈才能通過。
{% endcolorquote %}

#### **首次執行的時機：**

有以下兩種，只差在**首次**執行，後續的結果都是一樣的。

**1. 首次「立刻」執行**

```jsx
function throttle(func, timeout = 1000) {
    let isClose = false;
    let timer = null;
    return function(...args) {
        if (isClose) {
            return;
        }
        func.apply(this, args); // 第一次執行不進入 settimeout() 計算
        isClose = true;
        clearTimeout(timer);
        timer = setTimeout(() => {
            func.apply(this, args);
            isClose = false;
        }, timeout)
    }
}
```

**2. 首次「延遲」執行**

```jsx
function throttle(func, timeout = 1000) {
    let isClose = false;
    let timer = null;
    return function(...args) {
        if (isClose) {
            return;
        }
        isClose = true;
        clearTimeout(timer);
        timer = setTimeout(() => {
        func.apply(this, args); // 第一次執行進入 settimeout() 計算
            isClose = false;
        }, timeout);
    }
}
```

**範例：scroll 視窗滾動事件**

滾動視窗時，每隔一秒才執行函式，判斷是否要顯示下一區塊內容

```html
<ul id="content"></ul>
```

```jsx
const content = document.getElementById('content');

window.addEventListener('scroll', throttle(handleScroll));

function handleScroll() {
    if ((window.innerHeight + window.scrollY) >= document.body.offsetHeight) {
        content.innerHTML += '<li> ... </li>';
    }
}
```

範例畫面：

<iframe height="470" style="width: 100%;" scrolling="no" title="Javascript Throttle" src="https://codepen.io/claire-chang-the-bashful/embed/abjXYBg?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/claire-chang-the-bashful/pen/abjXYBg">
  Javascript Throttle</a> by Claire Chang (<a href="https://codepen.io/claire-chang-the-bashful">@claire-chang-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

---

`debounce` 與 `throttle` 適合的情境略有不同，可以藉由上述範例來觀察兩者間的差異

1. `debounce` 適用在停止觸發後才執行事件（input 停止輸入後，才去讀取更新的值）
2. `throttle` 適用在固定時間區間執行事件（滑鼠滾動時，每隔一秒檢查當前位置，判斷是否要撈取下一頁的資料）

---

參考文章：

[https://medium.com/@alexian853/debounce-throttle-那些前端開發應該要知道的小事-一-76a73a8cbc39](https://medium.com/@alexian853/debounce-throttle-%E9%82%A3%E4%BA%9B%E5%89%8D%E7%AB%AF%E9%96%8B%E7%99%BC%E6%87%89%E8%A9%B2%E8%A6%81%E7%9F%A5%E9%81%93%E7%9A%84%E5%B0%8F%E4%BA%8B-%E4%B8%80-76a73a8cbc39)

[https://ithelp.ithome.com.tw/articles/10222749](https://ithelp.ithome.com.tw/articles/10222749)

[https://medium.com/@steven234/throttle跟-debounce有什麼區別-e0b1979b1b4f](https://medium.com/@steven234/throttle%E8%B7%9F-debounce%E6%9C%89%E4%BB%80%E9%BA%BC%E5%8D%80%E5%88%A5-e0b1979b1b4f)