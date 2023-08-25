---
title: Javascript ES6 Class 與建構函式
date: 2023-03-06
tags: [ javascript ]
category: Javascript
description: 本篇文章將透過實例說明 ES6 Class 與 建構函式操作原型的方式。ES6 Class 是 JavaScript 中引入的一種語法糖，本質上還是基於原型，因此必須先了解原型鏈的概念及特性
image: https://i.imgur.com/1S4nH3i.png
---

<div style="display: flex; justify-content: center; margin: 0 0 20px;">
    <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/1S4nH3i.png">
</div>

## **繼承與原型鏈**

在 `class` 出現前，通常使用**建構函式**（`Constructor`）建立**原型**（`Prototype`）。原型並非實體（可以將其理解成藍圖），必須透過 `new` 運算子實體化，繼承該原型。

<!-- more -->

{% colorquote info %}
new operator（運算子）：產生一個新的空白物件，連結原先
建構函式（函式本身也是物件），建構函式內的 this 會綁定在新物件上
{% endcolorquote %}

假設今天有一個『花栗鼠』的藍圖，再根據這張藍圖創造出實體，每個實體都符合藍圖的規格：

```jsx
// 建構函式（藍圖）
function Chipmunk(name, color, size) {
    this.name = name;
    this.color = color;
    this.noseSize = size;
};

const Chip = new Chipmunk('Chip', 'brown', 'small');
const Dale = new Chipmunk('Dale', 'yellow', 'large');
```

透過函式物件內 `prototype` 新增的屬性，會掛載在原型上

```jsx
Chipmunk.prototype.climb = function() {
    console.log(`${this.name} is climbing the tree`);
};
```

透過繼承， 實體化後的物件也都可以取得原型方法

```jsx
console.log(Chip);
console.log(Dale);

// output: 結果如下
```

<div style="display: flex; justify-content: left; margin: 20px 0;">
    <img style="width: 100%; max-width: 600px;" src="https://i.imgur.com/puujd6y.png">
</div>

```jsx
Chip.climb(); // output: Chip is climbing the tree
Dale.climb(); // output: Dale is climbing the tree
```

{% colorquote info %}
將共用方法掛在在原型上，方法只會被定義一次，只需要一個記憶體空間，減少效能的消耗與浪費
{% endcolorquote %}

### **多層繼承**

假設我們希望在狗科前面再加上**動物界原型**，結構如下：

**動物界原型 → 花栗鼠建構函式 → 實體化建構式**

**1. 動物界原型**

```jsx
function Animal(family) {
    this.kingdom = 'animals';
    this.family = family;
};
Animal.prototype.eat = function() {
    console.log(`${this.name} is eating something`);
};
```

**2. 花栗鼠建構函式**
- 透過 `Object.create()` 繼承動物界**原型**
- 透過原型方法 `call()` 調用動物界**建構函式（需指定 this）**
- 因繼承動物界原型，導致 Chipmunk 建構函式被覆寫，透過 `Chipmunk.prototype.constructor = Chipmunk` 補回，讓原型鏈更加完整

```jsx
function Chipmunk(name, color, size) {
    Animal.call(this, 'squirrel');
    this.name = name;
    this.color = color;
    this.noseSize = size;
};
Chipmunk.prototype = Object.create(Animal.prototype);
Chipmunk.prototype.constructor = Chipmunk;
Chipmunk.prototype.climb = function() {
    console.log(`${this.name} is climbing the tree`);
};
```

**3. 實體化建構式**

```jsx
const Chip = new Chipmunk('Chip', 'brown', 'small');
const Dale = new Chipmunk('Dale', 'yellow', 'large');
```

```jsx
console.log(Chip);

// output: 結果如下
```

<div style="display: flex; justify-content: left; margin: 20px 0;">
    <img style="width: 100%; max-width: 800px;" src="https://i.imgur.com/DNHEaUr.png">
</div>

**以上寫法有幾個缺點：**

- 建構函式通常使用大駝峰（PascalCase）命名，但還是容易與一般函式混淆
- 如果沒有加上 `new` 運算子，並不會報錯，但結果會是 **undefined**
- 繼承原型方式較為複雜，共用方法也必須另外定義，缺少『封裝』結構，維護性較差

## **ES6 Class 類別**

先前的寫法，容易混淆 Class 跟 Constructor 的關係，**ES6 Class 是簡化操作 Constructor 的語法糖**，結構上更好理解

```jsx
class Chipmunk {
    constructor(name, color, size) {
        this.name = name;
        this.color = color;
        this.noseSize = size;
    }
    climb() {
        console.log(`${this.name} is climbing the tree`);
    }
}

const Chip = new Chipmunk('Chip', 'brown', 'small');
const Dale = new Chipmunk('Dale', 'yellow', 'large');
```

{% colorquote info %}
`Class` 只是語法糖，並非類別基礎（Class-Based）物件導向（Object Oriented Programming），它仍然是原型基礎（Prototype-Based）物件導向
{% endcolorquote %}

### **Class 多層繼承**

結構同前述：**動物界原型 → 花栗鼠建構函式 → 實體化建構式**

**1. 動物界原型**

```jsx
class Animal {
    constructor(family) {
        this.kingdom = 'animals';
        this.family = family;
    }
    eat() {
        console.log(`${this.name} is eating something`);
    }
}
```

**2. 花栗鼠建構函式**
- 透過 `extends` 繼承動物界**原型**
- 透過 `super()` 調用動物界**建構函式**

```jsx
class Chipmunk extends Animal {
    constructor(name, color, size) {
        super('squirrel');
        this.name = name;
        this.color = color;
        this.noseSize = size;
    }
    climb() {
        console.log(`${this.name} is climbing the tree`);
    }
}
```

**3. 實體化建構式**

```jsx
const Chip = new Chipmunk('Chip', 'brown', 'small');
const Dale = new Chipmunk('Dale', 'yellow', 'large');
```

### **Class 常用方法**

**私有變數：**變數前綴加上 `#`，外部無法取用，class 內可以使用

```jsx
class Chipmunk {
    #tree = 'oak'; // 私有變數
    constructor(name, color, size) {
        this.name = name;
        this.color = color;
        this.noseSize = size;
    }
    climb() {
        console.log(`${this.name} is climbing the ${this.#tree}`);
    }
}
const Chip = new Chipmunk('Chip', 'brown', 'small');

console.log(Chip.tree); // output: undefined
Chip.climb(); // output: Chip is climbing the oak
```

**Getter／Setter：**取得與設定方法，通常用於計算

```jsx
class Chipmunk {
    realAge = 0;
    constructor() {
        // ...
    }
    set age(val) {
        this.realAge = val;
    }
    get age() {
        return this.realAge * 30;
    }
}
const Chip = new Chipmunk('Chip', 'brown', 'small');
Chip.age = 1;
console.log(Chip.age); // output: 30
```

**Static 靜態方法／變數：**

- 不經過實體化即可取用 Class 內方法
- 實體化物件不可取用該方法（不能調用 `this`）

```jsx
class Chipmunk {
    constructor() {
        // ...
    }
    static caculateAge(age, minus) {
        return age * minus;
    }
}
const Chip = new Chipmunk('Chip', 'brown', 'small');

console.log(Chipmunk.caculateAge(2, 30)); // output: 60
console.log(Chip.caculateAge(2, 30)); // output: Uncaught TypeError: Chip.caculateAge is not a function
```

---

參考文章：

[https://medium.com/enjoy-life-enjoy-coding/javascript-es6-中最容易誤會的語法糖-class-基本用法-23e4a4a5e8ed](https://medium.com/enjoy-life-enjoy-coding/javascript-es6-%E4%B8%AD%E6%9C%80%E5%AE%B9%E6%98%93%E8%AA%A4%E6%9C%83%E7%9A%84%E8%AA%9E%E6%B3%95%E7%B3%96-class-%E5%9F%BA%E6%9C%AC%E7%94%A8%E6%B3%95-23e4a4a5e8ed)
[https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Inheritance_and_the_prototype_chain](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)
[https://ithelp.ithome.com.tw/articles/10185583](https://ithelp.ithome.com.tw/articles/10185583)