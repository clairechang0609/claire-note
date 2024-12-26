---
title: TypeScript 學習筆記：Interface V.S. Type
date: 2024-01-23
tags: [ typescript ]
category: Typescript
description: 說明 TypeScript Interface 與 Type 用途，並比較兩者使用時機
image: https://imgur.com/JBaER3w.png
---

<div style="display: flex; justify-content: center; margin: 30px 0;">
  <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/xOL2ava.png">
</div>

`Interface` 和 `Type` 是用來定義型別的兩種語法，那麼兩者又有什麼區別呢？

<!-- more -->

## **Interface 介面**

---

- 主要用於定義 **「物件型別」**
- 不能直接宣告基本型別、元組、列舉與聯合型別
- 可以重複宣告，支援擴展與合併

### **擴展（Extends）**

假設我們定義了一個 `Person` 介面

```tsx
interface Person {
  name: string;
  age: number;
}
```

接著定義一個新介面，並透過 `extends` 擴充 `Person`

```tsx
interface User extends Person {
  id: number;
  createdAt: string;
}
```

使用 `User` 介面的資料，就必須包含 `name`、`age`、`id`、`createdAt` 參數

```tsx
const user: User = {
  name: 'Daniel',
  age: 18,
  id: 123,
  createdAt: '2024-01-22'
}
```

### **合併（Merging）**

介面可以被重複宣告，以下範例最後結果會是兩者合併

```tsx
interface Vehicle {
  id: number;
  brand: string;
}

interface Vehicle {
  color: string;
  isElectric: boolean;
}

const car: Vehicle {
  id: 123,
  brand: 'audi',
  color: 'white';
  isElectric: false
}
```

## **Type Aliases 型別別名**

---

- 用來賦予型別一個新的名稱
- 可以直接宣告基本型別、元組、列舉、聯合型別、物件以及複雜型別
- 不可以重複宣告，不支援擴展，但可以透過 `&` 交集來組合型別

### 宣告**型別別名**

- **基本型別**
  
  ```tsx
  type Num = number;
  
  const num: Num = 123;
  ```
  
- **元組**
  
  ```tsx
  type Arr = [ string, number ];
  
  const arr: Arr = [ 'Daniel', 18 ];
  ```
  
- **聯合型別**
  
  ```tsx
  type UserId = number | string;
  
  const userId: UserId = 123;
  userId = '123';
  ```
  
- **列舉**
  
  ```tsx
  enum ColorChart {
    Red = 'red',
    Blue = 'blue',
    Green = 'green'
  }
  
  type Color = ColorChart;
  
  const color: Color = ColorChart.Red;
  ```
  
- **物件**
  
  ```tsx
  type Obj = {
    id: number,
    name: string
  }
  
  const obj: Obj = {
    id: 111,
    name: 'Daniel'
  }
  ```
  
- **Mapped Types**
  
  ```tsx
  type Person = {
    name: string,
    age: number,
    address: string
  };
  
  // 將 Person 型別中各屬性調整為可選
  type PartialPerson = {
    [K in keyof Person]?: Person[K];
  };
  ```
  

### **交集（Intersection）與聯集（Union）**

透過交集（`&`）或聯集（`|`）來組合型別

```tsx
type DataA = {
  a: number
};
type DataB = {
  b: string
};

type Combined = DataA & DataB; // Intersection
type Either = DataA | DataB; // Union
```

### **不能重複宣告**

重複宣告會拋錯

```tsx
type Vehicle = {
  id: number,
  brand: string
}

type Vehicle = { // Duplicate identifier 'Vehicle'
  color: string,
  isElectric: boolean
}
```

## **Interface & Type 使用時機**

---

- `Type` 可以直接定義原始型別、元組、列舉、聯合型別、函式、物件等
- `Interface` 只能定義物件
- 物件並且可以被擴充，使用 `Interface` 可讀性較高，`extends` 語意上也較直觀
- 開發第三方套件，型別定義使用 `Interface`，使用者能彈性的擴充屬性

---

參考資源：

https://hackmd.io/@hexschool/rJiN4vCup
https://stackoverflow.com/questions/37233735/interfaces-vs-types-in-typescript
https://blog.logrocket.com/types-vs-interfaces-typescript/