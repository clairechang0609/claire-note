---
title: TypeScript 學習筆記：常用 Utility Types
date: 2023-03-23
tags: [ typescript ]
category: Typescript
---

TypeScript `Utility Types` 是一組內建的型別工具，用於操作和組合現有型別，以產生新的型別。這些工具可以讓開發者更容易地處理和操作 TypeScript 型別，提高程式碼的可讀性和可維護性。

以下介紹幾個常用的 Utility Types：

### **Record<Keys, Type>**

```tsx
interface CatInfo {
    age: number;
    breed: string;
}

type CatName = 'miffy' | 'boris' | 'mordred';

const cats: Record<CatName, CatInfo> = {
    miffy: { age: 10, breed: 'Persian' },
    boris: { age: 5, breed: 'Maine Coon' },
    mordred: { age: 16, breed: 'British Shorthair' }
}
```

<!-- more -->

### **Pick<Type, Keys>**

參數選擇

```tsx
interface Todo {
    title: string;
    description: string;
    isCompleted: boolean;
}

type TodoPreview = Pick<Todo, 'title' | 'isCompleted'>

const todo: TodoPreview = {
    title: 'title',
    isCompleted: true
}
```

### **Omit<Type, Keys>**

參數排除

```tsx
interface Todo {
    title: string;
    description: string;
    isCompleted: boolean;
    createdAt: number;
}

type TodoPreview = Omit<Todo, 'description'>

const todo: TodoPreview = {
    title: 'title',
    isCompleted: true,
    createdAt: 20230301
}
```

---

參考資源：

[https://www.typescriptlang.org/docs/handbook/utility-types.html](https://www.typescriptlang.org/docs/handbook/utility-types.html)

[https://www.youtube.com/watch?v=GinkGJZBHIY](https://www.youtube.com/watch?v=GinkGJZBHIY)