---
title: Javascript 物件迴圈方法
date: 2023-06-01
tags: [ javascript ]
category: Javascript
description: 除了 for...in 方法，還有許多迴圈方法可以協助處理陣列資料，但物件並不支援這些方法，必須先將物件轉為陣列，才能使用陣列相關方法
image: https://imgur.com/pInwjQh.png
---

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/pInwjQh.png">
</div>

除了 **for...in** 方法**，**還有許多迴圈方法可以協助處理陣列資料，但物件並不支援這些方法，以下為例

```jsx
const fruits = {
    apple: { price: 45 },
    strawberry: { price: 100 },
    banana: { price: 30 },
    papaya: { price: 60 },
    watermelon: { price: 75 }
}
```

假設我們想取得每個元素的價格，直接使用陣列方法 `forEach()` 會產生錯誤，因為物件並沒有該寫法

```jsx
fruits.forEach(item => {
    console.log(item.price);
});

// output: Uncaught TypeError: fruits.forEach is not a function
```

<!-- more -->

# **將物件轉為陣列**

- **Object.keys()**
- **Object.values()**
- **Object.entries()**

---

## **Object.keys()**

取得一個物件的所有可枚舉屬性名稱（key），並回傳陣列

```jsx
console.log(Object.keys(fruits));
// output: ['apple', 'strawberry', 'banana', 'papaya', 'watermelon']
```

```jsx
Object.keys(fruits).forEach(key => {
    console.log(fruits[key].price);
});
// output:
// 45
// 100
// 30
// 60
// 75
```

## **Object.values()**

取得一個物件的所有可枚舉屬性的值（value），並回傳陣列

```jsx
console.log(Object.values(fruits));
// output: [
//     { "price": 45 },
//     { "price": 100 },
//     { "price": 30 },
//     { "price": 60 },
//     { "price": 75 }
// ]
```

```jsx
Object.values(fruits).forEach(item => {
    console.log(item.price);
});
// output:
// 45
// 100
// 30
// 60
// 75
```

## **Object.entries()**

可以取得最完整的資訊，將一個物件的所有可枚舉屬性（key）與值（value）組合一個陣列，第一個值為 key，第二個值為 value

```jsx
console.log(Object.entries(fruits));
// output: [
//     [ "apple", { "price": 45 } ],
//     [ "strawberry", { "price": 100 } ],
//     [ "banana", { "price": 30 } ],
//     [ "papaya", { "price": 60 } ],
//     [ "watermelon", { "price": 75 } ]
// ]
```

```jsx
Object.entries(fruits).forEach(item => {
    console.log(item[1].price);
});
// output:
// 45
// 100
// 30
// 60
// 75
```

- `Object.keys()` 跟 `Object.entries()` 可以用來取得物件的屬性名稱跟值
- `Object.values()` 適合適合用來取得物件各元素值

以上三個方法在處理物件時提供了不同的優點和功能，常會搭配陣列方法使用（[參考文章](https://clairechang.tw/2023/05/25/javascript/js-array-methods/)），可以根據需求選擇適合的方法

---

參考資源：

https://ithelp.ithome.com.tw/articles/10239942

https://www.casper.tw/development/2022/03/10/object-for-each/