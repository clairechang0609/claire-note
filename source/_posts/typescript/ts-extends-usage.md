---
title: TypeScript extends 的三種用法
date: 2024-11-07
tags: [ typescript ]
category: Typescript
---

TypeScript `extends` 有三種用法，語義和應用場景上有所不同，因此筆記起來讓自己參考使用。

### **Extends 用法**

- **介面繼承**
- **泛型約束**
- **條件判斷**

<!-- more -->

## **介面繼承**

**用法：**

```tsx
interface B extends A
```

在介面繼承中，`extends` 用於建立**子介面繼承父介面**的關係，讓子介面自動擁有父介面的屬性和方法。這是用來組織結構的機制，使得子介面能夠在基礎之上擴展屬性。

**範例：**

`Admin` 繼承了 `Role`，並且另外增加 `permissions` 屬性。這種 `extends` 用法建立了一個結構性的繼承關係，僅適用於介面和類別之間的繼承。

```tsx
interface Role {
  id: number;
  name: string;
}

interface Admin extends Role {
  permissions: string[];
}
```

---

## **泛型約束**

**用法：**

```tsx
T extends SomeType
```

在泛型的型別參數中，`extends` 用來限制泛型參數的型別範圍，確保泛型符合特定型別或結構。

**範例：**

在這裡，`T extends Role` 限制 `T` 必須符合 `Role` 的結構，這樣 `T` 才能被 `printRole` 函式使用。這種 `extends` 用法用於**型別約束**，並非建立繼承或子介面，只是限制了泛型的使用範圍。

```tsx
function printRole<T extends Role>(role: T): void {
  console.log(role.name);
}
```

---

## **條件判斷（Conditional Types）**

**用法：**

```tsx
T extends SomeType ? TrueType : FalseType
```

這個 `extends` 表達式用於條件型別判斷，而不是用來建立繼承或約束型別。`extends` 用來進行條件判斷時，類似於三元運算符 (`? :`) 用法。這樣可以根據型別 `T` 是否符合 `SomeType` 的結構來決定回傳 `TrueType` 還是 `FalseType`。

**範例：**

以下範例，`T extends U ? never : T` 用來排除聯合型別 `T` 中的指定型別 `U`。TypeScript 的條件型別會對聯合型別的每個成員單獨應用條件，並將結果**合併**為一個新的聯合型別。

```tsx
type Exclude<T, U> = T extends U ? never : T;

type Result1 = Exclude<'a' | 'b' | 'c', 'a'>; // "b" | "c"
type Result2 = Exclude<string | number | boolean, boolean>; // string | number
```

要注意的是，TypeScript 型別系統中並沒有 `if` / `else` 以及 `switch` 判斷，因此只能透過多層的條件型別來進行判斷。例如，判斷 HTTP 方法類型：

```tsx
type HttpMethod<T> = T extends "GET"
  ? "GET"
  : T extends "POST"
  ? "POST"
  : T extends "PUT"
  ? "PUT"
  : T extends "DELETE"
  ? "DELETE"
  : "unknown";
```

### **延伸：infer 在條件判斷中的型別推斷**

在符合條件的情況下，使用 `infer` 來進行型別推斷，提取型別使用。

**範例：**

以下條件型別判斷 `T` 是否是 `Promise<U>`。如果是，則回傳推斷出來的型別 `U`；否則，回傳 `T` 本身。

```tsx
type ExtractPromiseType<T> = T extends Promise<infer U> ? U : T;
```

---

### **比較表**

| 用法 | 說明 | 作用對象 |
| --- | --- | --- |
| **介面繼承** | 建立子介面繼承父介面的屬性和方法 | 介面繼承介面 |
| **泛型約束** | 限制泛型的型別範圍，使泛型符合某結構 | 泛型參數的型別限制 |
| **條件判斷** | 判斷型別結構，並根據條件決定回傳的型別 | 泛型參數的型別判斷 |

---

參考資源：

[https://chentsulin.medium.com/typescript-infer-的強大功用-9b43c4eac6fb](https://chentsulin.medium.com/typescript-infer-%E7%9A%84%E5%BC%B7%E5%A4%A7%E5%8A%9F%E7%94%A8-9b43c4eac6fb)