---
title: Javascript ES6 資料型別 Symbol
date: 2023-03-28
tags: [ javascript ]
category: Javascript
description: ES6 Symbols 是 JavaScript 中的一種新資料型別，用於表示唯一的識別符。每個 Symbol 值都是唯一且不可變的，並且可以作為物件屬性的 key 值
image: https://imgur.com/Bcm7EYS.png
---

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
    <img style="width: 100%; max-width: 700px;" src="https://imgur.com/Bcm7EYS.png">
</div>

JavaScript 的型別分為兩種：原始資料型別（Primitive Data Types）以及物件型別（Object Types）

原始資料型別包含以下：

- `String` **字串型別**
- `Number` **數字型別**
- `Boolean` **布林型別**
- `Null`
- `Undefined`
- `Symbol`

ES6 Symbols 是 JavaScript 中的一種新資料型別，用於表示唯一的識別符。每個 Symbol 值都是唯一且不可變的，並且可以作為物件屬性的 key 值。也就是說，即使兩個 Symbol 值具有相同的描述（description），兩者也不相等。

<!-- more -->

我們可以使用 Symbol() 函數來建立一個 Symbol 值，可以帶一個參數，用以描述該 Symbol。

### **屬性說明**

- 非建構函式，不能搭配 new 使用（`new Symbol()`）
- 不能使用 `for...in`、`Object.keys()` 等迴圈迭代，必須搭配 `Object.getOwnPropertySymbols()` 方法
- 不能使用點運算子 `.` 取得屬性值，必須使用中括號 `[]`
- 可以作為唯一 key 使用，避免相同 key 值覆寫的情況

### **使用方式**

**Symbol(description) 建立唯一識別**

```jsx
const Daniel = Symbol('student');
const Claire = Symbol('student');

console.log(Daniel === Claire); // output: false
```

**Symbol.for(key) 全域存取共用**

`Symbol.for()` 方法建立的 Symbol 會被保存在全域中，接收一個字符串作為參數，這個字符串稱為 Symbol 的 key，如果在全域 Symbol 表中已經存在該 key 對應的 Symbol，則直接回傳此 Symbol；如果不存在則新增一個 Symbol

```jsx
const Daniel = Symbol.for('student'); // 新建
const Claire = Symbol.for('student'); // 複用

console.log(Daniel === Claire); // output: true
```

**Symbol.keyFor(symbol)** 

取得 Symbol.for() 的 key 值

```jsx
const Daniel = Symbol.for('student');

console.log(Symbol.keyFor(Daniel)); // output: student
```

**Object.getOwnPropertySymbols() 迴圈**

一般來說，如果物件內有重複 key 值，後者會覆寫前者，見以下範例

```jsx
const students = {
    daniel: { age: 18, gender: 'male' },
    claire: { age: 17, gender: 'female' },
    andy: { age: 16, gender: 'male' },
    daniel: { age: 20, gender: 'male' }
}

console.log(students);
// output:
// {
//     andy: {age: 16, gender: 'male'},
//     claire: {age: 17, gender: 'female'},
//     daniel: {age: 20, gender: 'male'}
// }
```

但如果真的存在相同 key 值，像是相同名稱的學生，可以搭配 symbol 使用

```jsx
const students = {
    [Symbol('daniel')]: { age: 18, gender: 'male' },
    [Symbol('claire')]: { age: 17, gender: 'female' },
    [Symbol('andy')]: { age: 16, gender: 'male' },
    [Symbol('daniel')]: { age: 20, gender: 'male' }
}

console.log(students);
// output:
// {
//     Symbol(andy): {age: 16, gender: 'male'},
//     Symbol(claire): {age: 17, gender: 'female'},
//     Symbol(daniel): {age: 18, gender: 'male'},
//     Symbol(daniel): {age: 20, gender: 'male'}
// }
```

搭配 `Object.getOwnPropertySymbols` 運行迴圈判斷

```jsx
const adults = Object.getOwnPropertySymbols(students).filter(symbol => {
    return students[symbol].age >= 18;
});

console.log(adults);
// output: [ Symbol(daniel), Symbol(daniel) ]
```

---

參考文章：

[https://ithelp.ithome.com.tw/articles/10220499](https://ithelp.ithome.com.tw/articles/10220499)

[https://ithelp.ithome.com.tw/articles/10242050](https://ithelp.ithome.com.tw/articles/10242050)