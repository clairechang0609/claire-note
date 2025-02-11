---
title: Javascript Currying 柯里化
date: 2023-02-16
tags: [ javascript ]
category: Javascript
description: 本篇介紹 Javascript Currying 是什麼，以及如何運用 Curry Function 來協助我們將複雜的函式拆分成更小的單元，提高程式碼的可複用性以及維護性
image: https://i.imgur.com/HD2eteb.pngs
---

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img style="width: 100%; max-width: 600px;" src="https://i.imgur.com/HD2eteb.pngs">
</div>

> Currying is a technique in functional programming where a function that takes multiple arguments is transformed into a sequence of functions that each take a single argument.
> 

Currying 是一種函式編程（程式設計的一種，可以參考[此篇文章](https://totoroliu.medium.com/javascript-functional-programming-%E5%87%BD%E5%BC%8F%E7%B7%A8%E7%A8%8B%E6%A6%82%E5%BF%B5-e8f4e778fc08)）技術，將**接收多個參數的函式，拆解為多個函式，每個函式只接受一個參數**

<!-- more -->

## **使用目的**

1. 將程式碼拆成片段，讓函式能重複運用（DRY 避免重複代碼原則），有助於創建 higher-order function
2. 簡化參數處理，一次處理一個參數，提高彈性和易讀性

## **範例**

#### **一般函式：**

```jsx
function add(x, y) {
    return x + y;
}

add(5, 3); // output：8
```

#### **Curry Function：**

```jsx
function add(x) {
    return (y) => {
        return x + y;
    }
}

add(5)(3); // output：8
```

**應用方式：**

```jsx
const addFive = add(5);
addFive(3); // output：8
```

{% colorquote info %}
curry function 利用 **closure（閉包）**的特性，當內部函式被回傳後，可以取得內部函式「所在環境」的變數值。
{% endcolorquote %}

**用以下拆解進行說明：**

內部函式（inner）被回傳後，可以取得所在環境（add）的變數值（x）

```jsx
function add(x) {
    function inner(y) {
        return x + y;
    }
    return inner;
}

const addFive = add(5);
addFive(3); // output：8
```

首先將 `add(5)` 存放在 `addFive` 參數，當呼叫 `addFive(3)`，`add` 內部函式（inner）被調用，完成運算後回傳結果 8。

## **Currying vs. Partial application**

Currying 有時會跟 Partial application（偏函數應用）混淆，以下舉例說明差異

**Curry Function：**一個函式只能包含一個 Arity（參數個數）

```jsx
function add(x) {
    return (y) => {
        return (z) => {
            return x + y + z;
        }
    }
}

add(1)(2)(3); // output：6
```

**Partial application：**一個函式可以包含多個參數個數

```jsx
function add(x) {
    return (y, z) => {
        return x + y + z;
    }
}

add(1)(2, 3); // output：6
```

## **進階應用**

需搭配 Javascript 函數原型方法 call、apply（[參考文章](https://clairechang.tw/2023/02/04/javascript/js-call-apply-bind/)），以及遞迴函式概念（[參考文章](https://clairechang.tw/2023/01/10/vue/vue-recursive-component/#%E9%81%9E%E8%BF%B4%E5%87%BD%E5%BC%8F%EF%BC%88Recursive-Function%EF%BC%89)）

建立共用 Curry function，將函式作為參數

```jsx
function curry(func) {
    return curried = (...args) => {
        // 引數個數大於或等於參數個數
        if (args.length >= func.length) {
            return func.apply(this, args);
        }
        return (...moreArgs) => {
            return curried.apply(this, args.concat(moreArgs));
        }
    }
}
```

將 `add` 作為引數代入 `func` 參數

```jsx
function add(x, y, z) {
    return x + y + z;
}

const curriedAdd = curry(add);
```

以下三個使用方式執行結果相同

```jsx
curriedAdd(1)(2)(3); // output：6
curriedAdd(1, 2)(3); // output：6
curriedAdd(1)(2, 3); // output：6
```

利用 Curry Function，函式可以更加彈性的運用，提高程式碼的維護性

```jsx
const curriedAdd1 = curriedAdd(1);
curriedAdd1(2, 3); // output：6
curriedAdd1(3, 4); // output：8

const curriedAdd2 = curriedAdd(1, 2);
curriedAdd2(3); // output：6
curriedAdd2(5); // output：8
```

---

參考文章：

[https://medium.com/javascript-scene/curry-and-function-composition-2c208d774983](https://medium.com/javascript-scene/curry-and-function-composition-2c208d774983)

[https://ithelp.ithome.com.tw/articles/10236374](https://ithelp.ithome.com.tw/articles/10236374)

[https://medium.com/swlh/currying-in-javascript-bbe4ddedf86b](https://medium.com/swlh/currying-in-javascript-bbe4ddedf86b)