---
title: Javascript 常用陣列方法
date: 2023-05-25
tags: [ javascript ]
category: Javascript
description: 本篇說明 JavaScript 常用的陣列方法，陣列方法協助開發者更有效率的進行資料處理，前端框架 Vue 跟 React 都採取視覺單位關注點分離，框架協助處理畫面部分，而資料面的處理就要靠開發者的功力
image: https://imgur.com/lxxT0jf.png
---

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/lxxT0jf.png">
</div>

陣列方法可以幫助開發者更有效率的進行資料處理，前端框架 Vue 跟 React 都採取視覺單位關注點分離，框架協助處理畫面部分，而資料面的處理就要靠開發者的功力。

因自己在工作上大量運用到相關技巧，因此將內容筆記起來，方便未來需要時複習使用。

<!-- more -->

### **範例使用資料**

```jsx
const fruits = [
    { name: 'apple', price: 45 },
    { name: 'strawberry', price: 100 },
    { name: 'banana', price: 30 },
    { name: 'papaya', price: 60 },
    { name: 'watermelon', price: 75 }
]
```

# **迴圈處理方法**

---

## **forEach()**

**不會回傳值（不需要 return）**，跟 for 迴圈功能相似，但操作上更簡潔，只會依序執行每個項目

**參數說明：**

- item：陣列元素，在此為一個物件
- index：索引值
- array：整個陣列

```jsx
fruits.forEach((item, index, array) => {
    item.price += 10; // 價格調漲 10
})

console.log(fruits);
// output: [
//     { "name": "apple", "price": 55 },
//     { "name": "strawberry", "price": 110 },
//     { "name": "banana", "price": 40 },
//     { "name": "papaya", "price": 70 },
//     { "name": "watermelon", "price": 85 }
// ]
```

以下為錯誤使用：

```jsx
const priceIncreasedFruits = fruits.forEach((item, index, array) => {
    return item.price += 10;
})

console.log(priceIncreasedFruits); // output: undefined
```

## **map()**

需要回傳值，用於對陣列中的項目進行調整，透過回傳值組合成新的陣列

```jsx
const priceIncreasedFruits = fruits.map((item, index, array) => {
    return {
        ...item,
        price: item.price + 10 // 價格調漲 10
    }
})

console.log(priceIncreasedFruits);
// output: [
//     { "name": "apple", "price": 55 },
//     { "name": "strawberry", "price": 110 },
//     { "name": "banana", "price": 40 },
//     { "name": "papaya", "price": 70 },
//     { "name": "watermelon", "price": 85 }
// ]
```

## **filter()**

篩選符合條件的項目（條件判斷為 true）並回傳新的陣列

```jsx
const filteredFruits = fruits.filter((item, index, array) => {
    return item.price > 50; // 價格大於 50
})

console.log(filteredFruits);
// output: [
//     { "name": "strawberry", "price": 100 },
//     { "name": "papaya", "price": 60 },
//     { "name": "watermelon", "price": 75 }
// ]
```

## **find()**

跟 `filter()` 類似，差別在於找到符合條件的一筆資料後就會停止，並返回該筆資料，如果沒有符合條件的資料，會回傳 undefined

```jsx
const findFruit = fruits.find((item, index, array) => {
    return item.price > 50; // 價格大於 50
})

console.log(findFruit); // output: { "name": "strawberry", "price": 100 }
```

## **findIndex()**

篩選方式同 `find()`，找到符合條件的一筆資料後就會停止，並返回該筆資料**索引值**，如果沒有符合條件的資料，會回傳 -1

```jsx
const findFruitIndex = fruits.findIndex((item, index, array) => {
    return item.price > 50; // 價格大於 50
})

console.log(findFruitIndex); // output: 1
```

## **every()**

用以檢查陣列內**「所有項目」**是否符合條件，並回傳 Boolean 值

```jsx
const checkPriceValidity = fruits.every((item, index, array) => {
    return item.price > 50; // 價格大於 50
})

console.log(checkPriceValidity); // output: false

const checkPriceValidity2 = fruits.every((item, index, array) => {
    return item.price <= 100; // 價格小於等於 100
})

console.log(checkPriceValidity2); // output: true
```

## **some()**

與 `every()` 相似，差別在於條件的判斷，`some()` 用以檢查陣列內是否有**「任一項目」**符合條件，並回傳 Boolean 值

```jsx
const checkPriceValidity = fruits.some((item, index, array) => {
    return item.price > 50; // 價格大於 50
})

console.log(checkPriceValidity); // output: true
```

## **sort()**

**語法：**`array.sort(compareFunction)`

`sort()` 內帶入比較函式 `compareFunction(a, b)`，各項目依照邏輯來排序，並返回調整排序後的陣列，如果沒有定義條件，則按照預設的字串排序規則進行排序

- a 和 b 依序為陣列裡的項目
- 如果 `compareFunction(a, b)` 返回負數，則 a 會被排在 b 前面
- 如果 `compareFunction(a, b)` 返回正數，則 b 會被排在 a 前面
- 如果 `compareFunction(a, b)` 返回 0，則 a 和 b 相對位置不變

依照上述邏輯，如果想達成**「升冪排列」**：

```jsx
const sortedFruits = fruits.sort((a, b) => {
    return a.price - b.price; // 依價格排序
})

console.log(sortedFruits);
// output: [
//     { "name": "banana", "price": 30 },
//     { "name": "apple", "price": 45 },
//     { "name": "papaya", "price": 60 },
//     { "name": "watermelon", "price": 75 },
//     { "name": "strawberry", "price": 100 }
// ]
```

## **reduce()**

**語法：**`arr.reduce(callback[accumulator, currentValue, currentIndex, array], initialValue)`

應用的範圍較廣，常用於累計運算。將陣列中的項目依序傳給 callback 函式，並將回傳值傳入下次運算，最後返回累計結果

**參數說明：**

- accumulator：累計值，上一次運算的回傳值，如果是第一次計算，則為初始值
- currentValue：當前項目
- currentIndex：當前項目索引值
- array：整個陣列
- initialValue：初始值

```jsx
const countPrice = fruits.reduce((accumulator, currentValue, currentIndex, array) => {
    return accumulator + currentValue.price; // 價格計算
}, 0)

console.log(countPrice); // output: 310
```

`reduce()` 也可以做更複雜的運算，例如：

**將所有水果名稱組成陣列**

```jsx
const fruitsName = fruits.reduce((accumulator, currentValue, currentIndex, array) => {
    return [ ...accumulator, currentValue.name ];
}, []);

console.log(fruitsName);
// output: [ "apple", "strawberry", "banana", "papaya", "watermelon" ]
```

**將陣列結構轉成物件（**`key: value`**）**

```jsx
const fruitPrices = fruits.reduce((accumulator, currentValue, currentIndex, array) => {
    accumulator[currentValue.name] = currentValue.price;
    return accumulator;
}, {});

console.log(fruitPrices);
// output: { "apple": 45, "strawberry": 100, "banana": 30, "papaya": 60, "watermelon": 75 }
```

# **其他方法**

---

## **push()**

在陣列最後面加入項目

```jsx
fruits.push({ name: 'guava', price: 35 });

console.log(fruits);
// output: [
//     ...
//     { name: 'guava', price: 35 }
// ]
```

## **pop()**

移除陣列內最後一筆項目

```jsx
fruits.pop();

console.log(fruits);
// output: [
//     ...
//     { "name": "watermelon", "price": 75 }
// ]
```

## **unshift()**

在陣列最前面加入項目

```jsx
fruits.unshift({ name: 'guava', price: 35 });

console.log(fruits);
// output: [
//     { name: 'guava', price: 35 },
//     ...
// ]
```

## **shift()**

移除陣列內第一筆項目

```jsx
fruits.shift();

console.log(fruits);
// output: [
//     { name: 'apple', price: 45 },
//     ...
// ]
```

## **slice()**

**語法：**`arr.slice(begin, end)`

選擇開始與結束索引值間的項目（包含開始，不包含結束），結束索引值可不填，並回傳一個新的陣列，原陣列不受影響

```jsx
const remainFruits = fruits.slice(1, 3);

console.log(remainFruits);
// output: [
//     { "name": "strawberry", "price": 100 },
//     { "name": "banana", "price": 30 }
// ]
```

## **splice()**

**語法：**`array.splice(start, deleteCount, addItem1, addItem2, ...)`

移除索引值起的數筆項目，並加入新的項目，加入項目為選填

```jsx
fruits.splice(1, 3);

console.log(fruits);
// output: [
//     { "name": "apple", "price": 45 },
//     { "name": "watermelon", "price": 75 }
// ]
```

加入新的項目

```jsx
fruits.splice(1, 3, { name: 'guava', price: 35 });

console.log(fruits);
// output: [
//     { "name": "apple", "price": 45 },
//     { "name": "guava", "price": 35 }
//     { "name": "watermelon", "price": 75 }
// ]
```

## **join()**

**語法：**`arr.join(separator)`

將陣列中的值合併成一個字串，並可加入分隔字符，回傳一個字串

```jsx
const fruits = [ 'apple', 'strawberry', 'banana', 'papaya', 'watermelon' ];

console.log(fruits.join(', '));
// output: apple, strawberry, banana, papaya, watermelon
```

## **indexOf()**

跟 `findIndex()` 的目的相同，均為找到項目在陣列中的索引值，如果找不到該項目，會返回 -1，但使用方式與情境有些差異，`indexOf()` 不適合用在物件搜尋

```jsx
const fruits = [ 'apple', 'strawberry', 'banana', 'papaya', 'watermelon' ];

console.log(fruits.indexOf('strawberry')); // output: 1
```

## **includes()**

檢查陣列中是否包含特定值，回傳結果為 Boolean，不適合用在物件搜尋

```jsx
const fruits = [ 'apple', 'strawberry', 'banana', 'papaya', 'watermelon' ];

console.log(fruits.includes('papaya')); // output: true
```

## **reverse()**

反轉陣列內的項目排序

```jsx
const reverseFruits = fruits.reverse();

console.log(reverseFruits);
// output: [
//     { "name": "watermelon", "price": 75 },
//     { "name": "papaya", "price": 60 },
//     { "name": "banana", "price": 30 },
//     { "name": "strawberry", "price": 100 },
//     { "name": "apple", "price": 45 }
// ]
```

## **concat()**

組合兩個陣列，並返回新陣列，原陣列不受影響

```jsx
const newFruits = [ { name: 'guava', price: 35 } ];
const concatFruits = fruits.concat(newFruits);

console.log(concatFruits);
// output: [
//     ...
//     { name: 'guava', price: 35 }
// ]
```

`concat()` 的功能也可以用 ES6 解構取代

```jsx
const concatFruits = [ ...fruits, ...newFruits ];
```

---

參考資源：

https://www.casper.tw/javascript/2017/06/29/es6-native-array/
https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce
https://hackmd.io/@hsuchihting/ryvYW2YvL