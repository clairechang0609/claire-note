---
title: TypeScript 學習筆記：進階應用
date: 2023-03-22
tags: [ typescript ]
category: Typescript
---

### **Type Inference 型別推論**

如果沒有明確的指定型別，TypeScript 會依照**『型別推論』**的規則推斷出型別

```jsx
// 自動判定 string 型別
let num = 'hello';
num = 777;
// Type 'number' is not assignable to type 'string'.
```

如果定義的時候沒有賦值，會被推斷為 any 型別

```jsx
// 自動判定 any 型別
let something;
something = 777;
something = '777';
```

<!-- more -->

### **Union Types 聯合型別**

使用 `|` 符號來表示多種可能的型別

```jsx
let price: number | string;
price = 1000;
price = '1000';
price = true; // Type 'boolean' is not assignable to type 'string | number'
```

### **Type 型別別名**

```jsx
type A = number | string;
type B = (param: string) => string;

let a1: A;
a1 = 999;
a1 = '999';
a1 = false; // Type 'boolean' is not assignable to type 'A'
```

### **Interface 介面**

定義**物件或陣列**的型別，跟上面 `type` 功能類似，差別在 `interface` 可以『擴充』

**屬性介紹**

- 可選屬性：範例 `isMale?: boolean`
- 任意屬性：範例 `[ propName: string ]: any`
- 唯讀屬性：使用 `readonly` 定義唯讀屬性，若初始化後再次賦值會報錯

```jsx
interface User {
    readonly id: number;
    name: string;
    isMale?: boolean;
    [ propName: string ]: any;
}

// 擴充
interface User {
    age: number
}

const obj: User = {
    id: 1,
    name: 'Claire',
    age: 18,
    isMale: false
}

obj.id = 2; // Cannot assign to 'id' because it is a read-only property
```

也可以用來表示陣列

```jsx
interface Users {
    [ index: number ]: string
}

const users: Users = [ 'Daniel', 'Claire', 'Andy', 'Avery' ];
```

或是定義函式

```jsx
interface SearchUser {
    (arr: string[], name: string): boolean
}

const searchName: SearchUser = function(arr: string[], name: string) {
    return arr.includes(name);
}

console.log(searchName([ 'Daniel', 'Claire' ], 'Daniel')); // output: true
```

### **Tuple 元組型別**

定義**陣列**元素數量、型別

```jsx
let tuple: [ number, string, boolean ] = [ 1, 'a', true ];
let tuple2: [ string, string ][] = [ [ 'a', 'b' ] ];
```

### **Enum 列舉型別**

用於定義一組命名常數的方法

```jsx
enum Alert { success, warning, danger }
```

列舉項目預設會從 0 開始遞增賦值，列舉 key 跟值會反向對映

```jsx
console.log(Alert.success); // output: 0
console.log(Alert[1]); // output: warning
```

也可以手動賦值

```jsx
enum Alert {
    success = 1,
    warning = 0,
    danger = -1
}

console.log(Alert.success); // output: 1
console.log(Alert[1]); // output: success

console.log(Alert);
// output:
// {
//   1: 'success',
//   0: 'warning',
//   -1: 'danger',
//   success: 1,
//   warning: 0,
//   danger: -1
// }
```

或使用非數字，必須使用斷言來略過型別檢查

```tsx
enum Alert {
    success = 'green' as any,
    warning = 'yellow' as any,
    danger = 'red' as any
}

console.log(Alert.green); // output: success
```

### **Generics 泛型**

在函式或是 class 後面加上 `<T>` 或是 `<Type>` 表示動態型別（名稱自訂），於調用時將型別傳入，讓型別更加靈活

```jsx
function print<T>(data: T) {
    console.log(data);
}
print<number>(999); // output: 999
print<string>('1000'); // output: '1000'

class Print<T> {
    data: T
    constructor(d: T) {
        this.data = d;
    }
}
const p = new Print<number>(1000);
const p2 = new Print<boolean>(true);
console.log('p', p); // output: Print { data: 1000 }
console.log('p2', p2); // output: Print { data: true }
```

### **Type Assertion 型別斷言**

手動指定型別，並不會影響型別轉換

**範例一：**

```tsx
const user = {
    name: 'Claire',
    age: 20,
    gender: 'female'
}

function userInfo(param: string) {
    return user[param]; // Element implicitly has an 'any' type because expression of type 'string' can't be used to index type '{ name: string; age: number; gender: string; }'.
}
```

因 `param` 有可能不存在 user key 值，因此拋錯，可以加上斷言如下

```jsx
function userInfo(param: string) {
    return user[param as keyof typeof user];
}
```

**範例二：**

```jsx
function toBoolean(something: string | number): boolean {
    return something; // Type 'string | number' is not assignable to type 'boolean'.
}
```

直接調整成 `as boolean` 依舊會拋錯

```jsx
function toBoolean(something: string | number): boolean {
    return something as boolean; // Conversion of type 'string | number' to type 'boolean' may be a mistake because neither type sufficiently overlaps with the other. If this was intentional, convert the expression to 'unknown' first.
}
```

因為 `something` 預先被定義為 `string | number`，跟 `boolean` 沒有關聯，必須先斷言成 `unknown`

```tsx
function toBoolean(something: string | number): boolean {
    return something as unknown as boolean;
}
console.log(toBoolean(123)); // output: 123
```

前面提到斷言不會進行型別轉型，因此結果並不會變成 `boolean`

### **ES6 Class 與建構函式**

{% colorquote %}
如果對 ES6 Class 尚不熟悉，可以參考 [這篇文章](https://clairechang.tw/2023/03/06/javascript/js-class/)
{% endcolorquote %}

**TypeScript Class 修飾符**

- public：公有的，可以在任何地方使用，預設屬性跟方法皆為 public
- private：私有的，不能在類別的外部調用（ex: new 實體或是 extends 繼承）
- protected：受保護的，extends 繼承可以調用，但 new 實體不能調用

public & protected 只存在於 typescript，通常使用於開法階段（避免共同開發時異動到程式碼），編譯成 javascript 後沒什麼作用，因 js 不存在這兩個類型

```tsx
class Live {
    public roomName: string
    private id: string
    protected name: string

    constructor(roomName1: string, id1: string, name1: string) {
        console.log('建立直播中...');

        this.roomName = roomName1;
        this.id = id1;
        this.name = name1;
    }
}

const live = new Live('no.1', '00001', 'Daniel');
console.log(live); // output: Live { roomName: 'no.1', id: '00001', name: 'Daniel' }
```

如果需要使用**私有變數**，可以改用 javascript 的變數前綴 `#`

```tsx
class Live2 {
    // js 私有變數
    #name
    constructor(name: string) {
        this.#name = name;
    }
}

const live2 = new Live2('Daniel');
console.log(live2); // output: Live2 {}
```

**Class & Interface**

因類別可以繼承其他類別，有時類別間有共同的特性，可以提取出來作為介面共用（對類別的一部分行為進行抽象），並用 `implements` 來實現

```tsx
interface Eat {
    eat(): string;
}

class Animal {
    name: string;
    constructor(name: string) {
        this.name = name;
    }
}

class Person extends Animal implements Eat {
    constructor(name: string) {
        super(name);
        this.name = name;
    }
    eat() {
        return `${this.name} is eating food`;
    }
}

const daniel = new Person('Daniel');
console.log(daniel.eat()); // output: Daniel is eating food
```

可以一次實現多個介面

```tsx
class Person implements Eat, Sleep {
    ...
}
```

也可以介面繼承介面

```tsx
interface Animal {
    eat(): string;
}

interface Dog extends Animal {
    bark(): string;
}

const Coco: Dog = {
    eat() {
        return 'eating';
    }
    bark() {
        return 'barking';
    }
}
```

---

參考資源：

[https://medium.com/enjoy-life-enjoy-coding/javascript-es6-中最容易誤會的語法糖-class-基本用法-23e4a4a5e8ed](https://medium.com/enjoy-life-enjoy-coding/javascript-es6-%E4%B8%AD%E6%9C%80%E5%AE%B9%E6%98%93%E8%AA%A4%E6%9C%83%E7%9A%84%E8%AA%9E%E6%B3%95%E7%B3%96-class-%E5%9F%BA%E6%9C%AC%E7%94%A8%E6%B3%95-23e4a4a5e8ed)

[https://www.youtube.com/watch?v=GinkGJZBHIY](https://www.youtube.com/watch?v=GinkGJZBHIY)