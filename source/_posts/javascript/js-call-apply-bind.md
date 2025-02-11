---
title: Javascript 函數原型方法：call、apply、bind 改變 this 指向
date: 2023-02-04 18:00
tags: [ javascript ]
category: Javascript
description: 本篇說明如何利用 Javascript 函數原型方法 call、apply、bind 來改變 this 指向，以及三種方法間的比較
image: https://imgur.com/yUwijHC.png
---

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
    <img style="width: 100%; max-width: 500px;" src="https://imgur.com/yUwijHC.png">
</div>

函數原型 **call**、**apply**、**bind** 方法可以用來改變 **this** 指向的對象，bind 和其他兩個方法 call、apply 較不同，call、apply 會直接執行函式，bind 只是將 this 綁定到函式內，並不會實際執行，這樣說明還是有點難懂，那就接下去看看吧。

<!-- more -->

首先建立基本參數

```jsx
const customer = {
    name: 'Daniel'
}

function callName() {
    console.log(this.name);
}

callName(); // undefined
```

如果我們直接執行函式，這時候會得到 undefined，原因是 `this` 指向的是 window，裡面並沒有 name 這個參數，如果要將 this 指向 customer 這個物件，該怎呢做呢？

# **bind**

bind 會建立一個被包裹後的新函式，該函式被呼叫時，會將 `this` 設為綁定的參數，另外也可以綁定其他的參數，綁定之後的參數在該函式就**無法再修改**。

#### **語法**

```jsx
fun.bind(this, arg1, arg2, ...)
```

使用 bind 將 this 「綑綁」進去，建立一個新的函式

```jsx
const copyCallName = callName.bind(customer);
copyCallName(); // Daniel
```

這時候執行 `copyCallName()`，就可以取得 customer 物件的內容

也可以直接執行

```jsx
callName.bind(customer)(); // Daniel
```

或是寫在函式表達式後面

```jsx
const callName = function() {
    console.log(this.name);
}.bind(customer)

callName(); // Daniel
```

this 被綁定後將無法變更

```jsx
const student = {
    name: 'Andy'
}

const callName = function() {
    console.log(this.name);
}.bind(customer)

callName.bind(student)(); // Daniel
```

#### **綁定其他參數**

首先在 callName 函式加入兩個參數

```jsx
callName(age, gender) {
    console.log(this.name);
    console.log(age);
    console.log(gender);
}
```

沒有綁定的情況下，可以任意代入值

```jsx
const copyCallName = callName.bind(customer);
copyCallName(30, 'male');

// Daniel
// 30
// male
```

參數被 bind 綁定後將無法變更

```jsx
const copyCallName = callName.bind(customer, 18, 'female');
copyCallName(30, 'male'); // 賦值無效

// Daniel
// 18
// female
```

# **call**

`fun.call()` 會直接執行函式，並且將 this 跟其他參數依序代入

#### **語法**

```jsx
fun.call(this, arg1, arg2, ...)
```

回到前面定義的參數

```jsx
const customer = {
    name: 'Daniel'
}

callName(age, gender) {
    console.log(this.name);
    console.log(age);
    console.log(gender);
}
```

呼叫方式如下

```jsx
function useCallName() {
    callName.call(customer, ...arguments);
}
useCallName(30, 'male');

// Daniel
// 30
// male
```

{% colorquote info %}
`arguments`：javascript 內建參數，不需預先定義，函式內自動建立此參數，在函式呼叫時傳遞進去的引數透過陣列傳入
{% endcolorquote %}

直接呼叫結果相同

```jsx
callName.call(customer, 30, 'male');

// Daniel
// 30
// male
```

# **apply**

有點像 `call()` 的簡化版本，只傳入兩個參數，第二個參數是其餘參數包裝成的陣列

#### **語法**

```jsx
fun.apply(this, [ arg1, arg2, ... ])
```

呼叫方式如下

```jsx
function useCallName() {
    callName.apply(customer, arguments);
}
useCallName(30, 'male');

// Daniel
// 30
// male
```

直接呼叫結果相同

```jsx
callName.apply(customer, [ 30, 'male' ]);

// Daniel
// 30
// male
```

---

### **延伸：改變 this 指向的其他做法**

以下述為例進行說明：

```jsx
const customer = {
    name: 'Daniel',
    LazyCallName() {
        setTimeout(function() {
            console.log(this.name);
        }, 1000);
    }
}

customer.LazyCallName(); // undefined
```

如果要取到 name 的值，可以透過三個方法：

**1. bind**

透過函數原型方法 bind，將 this 指向 customer 這個物件

```jsx
const customer = {
    name: 'Daniel',
    LazyCallName() {
        setTimeout(function() {
            console.log(this.name);
        }.bind(this), 1000);
    }
}

customer.LazyCallName(); // Daniel
```

**2. 在外層函式宣告變數儲存 this**

在外層 LazyCallName function 先定義變數 vm，將 this 儲存起來

```jsx
const customer = {
    name: 'Daniel',
    LazyCallName() {
        const vm = this;
        setTimeout(function() {
            console.log(vm.name);
        }, 1000);
    }
}

customer.LazyCallName(); // Daniel
```

**3. ES6 Arrow Function**

**由於箭頭函式的 `this` 綁定的是定義時所在的物件**，透過箭頭函式，this 以 LazyCallName function 為基準產生作用域（函式也是物件型別），指向 customer 物件

```jsx
const customer = {
    name: 'Daniel',
    LazyCallName() {
        setTimeout(() => {
            console.log(this.name);
        }, 1000);
    }
}

customer.LazyCallName(); // Daniel
```

---

參考文章：

[https://realdennis.medium.com/javascript-聊聊call-apply-bind的差異與相似之處-2f82a4b4dd66](https://realdennis.medium.com/javascript-%E8%81%8A%E8%81%8Acall-apply-bind%E7%9A%84%E5%B7%AE%E7%95%B0%E8%88%87%E7%9B%B8%E4%BC%BC%E4%B9%8B%E8%99%95-2f82a4b4dd66)
[https://medium.com/swlh/call-apply-bind-javascript-methods-7d96a73816e8](https://medium.com/swlh/call-apply-bind-javascript-methods-7d96a73816e8)
[https://b-l-u-e-b-e-r-r-y.github.io/post/BindCallApply/](https://b-l-u-e-b-e-r-r-y.github.io/post/BindCallApply/)