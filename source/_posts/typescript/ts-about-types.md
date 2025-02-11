---
title: TypeScript 學習筆記：基本介紹與型別
date: 2023-03-21
tags: [ typescript ]
category: Typescript
---

眾所皆知 JavaScript 是弱型別（Weak Typing）的語言，因此常會發生賦值錯誤導致編譯出錯，TypeScript 可視為進階版 JavaScript，提供靜態型別定義和檢查，目標是讓開發者在編寫 JavaScript 時擁有更好的開發體驗和更高的程式碼可讀性，提升專案穩定度。

### **基本說明**

1. TypeScript 是基於 JavaScript 的超集合（SuperSet），提供型別系統（Type System），能夠在開發時宣告型別
2. 必須編譯成 JavaScript 檔案，瀏覽器、後端框架、Node.js 環境…才能閱讀
3. 類型的定義可以減少 bug，能在編譯前預先檢查，避免許多運算錯誤
4. TypeScript 只會進行靜態檢查，編譯時即使報錯，還是會產生編譯結果，仍然可以使用編譯後的 JavaScript 檔

<!-- more -->

### **環境建置與相關指令**

- 安裝 TypeScript：`npm install -g typescript`
- 進入專案資料夾，輸入 `tsc —-init` 建立 tsconfig.json
    - 設定 `rootDir` ＆`outDir` 路徑
    - 開啟 `inlineSourceMap: true`
- `tsc`：將指令路徑 `rootDir` 內所有 .ts 檔編譯為 .js 檔存放於 `outDir`
- `tsc fileName.ts` ：將指定路徑 `rootDir` 內 `fileName.ts` 檔編譯為 `fileName.js`
- `tsc —-watch`：進行動態監聽

### **原始資料型別（Primitive Data Types）**

- `String` **字串型別**：`const str: string = 'hello'`
- `Number` **數字型別**，包括整數和浮點數：`const num: number = 1000`
- `Boolean` **布林型別**：`const boo: boolean = true`
- `Null`：`const n: null = null`
- `Undefined`：`const un: undefined = undefined`
- `Symbol`：ES6 新型別，是一個唯一且不可變的值，使用 Symbol() 函數來建立一個 Symbol 值，可以參考 [這篇文章](https://www.notion.so/Javascript-ES6-Symbol-cdc162d0109f4ee3b98b8ca1ecce186e)

### **物件型別（Object Types）**

- `Object` **物件型別**

```jsx
const obj: Object = { name: 'Claire', age: 18 }
const obj2: { name: string, age: number } = { name: 'Claire', age: 18 }
```

- `Array` **陣列型別**

使用型別搭配中括號 `[]` 表示法

```jsx
const arr: number[] = [ 111, 222 ];
const arr2: string[][] = [ [ 'a' ], [ 'b' ] ];
```

或是陣列泛型（Array Generic）`Array<type>` 表示法

```jsx
const arr3: Array<boolean> = [ true, false ];
```

- `Function` **函式型別**

```jsx
function hello(a: string, b: string) {
    return a + b;
}

function hello2(a: number, b: number): number {
    return a + b;
}

// hover提示：function hello3(a: number, b: boolean, c: string): number
function hello3(a: number, b: boolean, c: string) {
    return 10000000;
}

// hover提示：function hello4(name: string, age?: number): number | undefined
function hello4(name: string, age?: number) {
    return age;
}
```

『選填』的參數如果沒加上判斷會拋錯，因參數有可能為 `undefined` ，見以下範例

```jsx
function hello4(name: string, age?: number) {
    return age + 1; // 'age' is possibly 'undefined'.
}
```

須加上判斷條件，改寫如下

```jsx
function hello4(name: string, age?: number) {
    if (age === undefined) {
        return 1;
    }
    return age + 1;
}
```

### **TypeScript 特有型別**

- `Any` **任意型別**，等同於不定義型別，可以賦值給任何型別

```jsx
let anyVar: any = true;
anyVar = 18;
anyVar = 'hello';

let str: string;
str = anyVar;
```

- `Unknown`：類似 `any` 型別，但 `any` 可以賦值給任何型別，`unknown` 只能賦值給 `any` 和自己

```jsx
let unknownVar: unknown = true;
unknownVar = 18;
unknownVar = 'hello';

let str: string;
str = unknownVar; // Type 'unknown' is not assignable to type 'string'

let anyVar: any;
anyVar = unknownVar;
```

- `Never`：函式發生無窮迴圈，或是出現例外狀況拋錯誤

```jsx
function throwError(): never {
    throw new Error('error');
}
```

- `Void` **空值**：通常用於表示函式沒有回傳值

```jsx
function hello(a: string, b: string): void {
    alert(a + b);
}

// hover提示：function hello2(a: number, b: number): void
function hello2(a: number, b: number) {
    console.log(a + b);
}
```

### **瀏覽器 DOM 與 BOM 型別**

- `HTMLElement`：所有 HTML 元素的基本型別，包含 `<div>`、`<p>`、`<span>` …
- `HTMLInputElement`：代表 `<input>` 元素
- `HTMLTextAreaElement`：代表 `<textarea>` 元素
- `HTMLAnchorElement`：代表 `<a>` 元素
- `HTMLFormElement`：代表 `<form>` 元素
- `HTMLSelectElement`：代表 `<select>` 元素
- `HTMLFormElement`：代表 `<form>` 元素
- `HTMLTableElement`：代表 `<table>` 元素
- `MouseEvent`：代表滑鼠事件
- `KeyboardEvent`：代表鍵盤事件
- `inputEvent`：代表輸入事件
- `FocusEvent`：代表焦點事件

---

參考資源：

[https://willh.gitbook.io/typescript-tutorial/basics/primitive-data-types](https://willh.gitbook.io/typescript-tutorial/basics/primitive-data-types)

[https://www.youtube.com/watch?v=GinkGJZBHIY](https://www.youtube.com/watch?v=GinkGJZBHIY)