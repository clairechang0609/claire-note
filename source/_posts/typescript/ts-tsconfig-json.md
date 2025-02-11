---
title: TypeScript 學習筆記：tsconfig.json 編譯設定
date: 2023-03-25
tags: [ typescript ]
category: Typescript
---

`tsconfig.json` 為 TypeScript 編譯器的設定檔，用來配置 TypeScript 編譯器的行為，簡單說明設定檔內容

### **環境安裝**

產生 tsconfig.json 檔案

```jsx
tsc --init
```

<!-- more -->

## **設定說明**

### **compilerOptions**

物件，定義編譯器的行為

| 屬性 | 介紹 | 預設值 | 可帶入值 |
| --- | --- | --- | --- |
| target | 欲編譯的 JavaScript 版本 | es2016 | ES3、ES5、ES6/ES2015、ES2016、ES2017、ES2018、ES2019、ES2020、ES2021 或 ESNext |
| lib | 編譯器可以使用的標準庫 | 空值 | ES5、ES6、ES2015、ES2016、ES2017、ES2018、ES2019、ES2020、ES2021、ESNext、DOM、DOM.Iterable、WebWorker、ScriptHost… |
| jsx | 支援使用 JSX 語法 | preserve | preserve、react、react-jsx |
| module | 生成的模組格式 | commonjs | monJS、AMD、SystemJS、UMD、ES6、ES2015 或 ESNext |
| rootDir | 編譯時的根目錄 | 空值 | 絕對路徑、相對路徑 |
| outDir | 編譯後檔案存放位置 | 空值 | 絕對路徑、相對路徑 |
| baseUrl | 模組解析的根目錄 | . | 絕對路徑、相對路徑 |
| paths | 定義模組的路徑別名 | 空物件 | 物件 |
| allowJs | 是否允許編譯 JavaScript 文件 | false | boolean |
| checkJs | 是否檢查 JavaScript 文件型別 | false | boolean |
| declaration | 是否生成相對應的裝飾（.d.ts）文件 | false | boolean |
| sourceMap | 是否生成相對應的 .map 文件 | false | boolean |
| outFile | 編譯生成的 JavaScript 代碼合併到一個文件中 | 空值 | 絕對路徑、相對路徑 |
| removeComments | 是否移除註解 | false | boolean |
| noEmit | 是否不生成編譯後的 JavaScript 文件 | false | boolean |
| strict | 是否啟用所有嚴格類型檢查模式，等於noImplicitAny、noImplicitThis、alwaysStrict、strictBindCallApply、strictNullChecks、strictFunctionTypes 和 strictPropertyInitialization的設定為true | false | boolean |
| noImplicitAny | 是否禁止 any 型別 | false | boolean |
| noImplicitThis | 是否禁止 this 關鍵字隱式型別為 any | false | boolean |
| alwaysStrict | 是否在輸出文件中包含 ‘use strict’ | false | boolean |
| strictBindCallApply | 是否在函數 bind、call、apply 的調用中檢查參數的類型符合函數的期望類型 | false | boolean |
| strictNullChecks | 是否開啟嚴格的空值檢查（null 和 undefined） | false | boolean |
| strictFunctionTypes | 函數類型參數的參數和返回值是否精確匹配 | false | boolean |
| strictPropertyInitialization | 類成員是否進行初始化 | false | boolean |
| experimentalDecorators | 是否使用實驗性裝飾器語法 | false | boolean |
| emitDecoratorMetadata | 如果有使用裝飾器，是否要生成裝飾器數據 | false | boolean |
| jsxFactory | 指定使用的 JSX 工廠函數（TS 4.1+） | 空值 | 自訂 |
| noLib | 編譯後的代碼是否排除 TypeScript 標準庫的所有宣告文件 | false | boolean |
| esModuleInterop | 是否兼容 CommonJS module | false | boolean |
| preserveSymlinks | 是否保留符號連接 | false | boolean |
| skipLibCheck | 是否跳過標準庫檢查 | false | boolean |
| noEmitOnError | 編譯出錯是否停止編譯檔案 | false | boolean |

### **files**

指定欲編譯 TypeScript 檔案，陣列，沒有定義此屬性預設為根目錄下所有 `.ts` 及 `.tsx` 檔案，可以使用絕對路徑、相對路徑

```json
{
    "files": [
        "./src/index.ts"
    ]
}
```

### **include & exclude**

陣列，`include` 指定欲編譯 TypeScript 檔案，`exclude` 指定不要被編譯 TypeScript 檔案

可以輸入編譯的檔案、資料夾或者通配符模式（wildcard character）

```json
{
    "include": [
        "./src/test"
    ],
    "exclude": [
        "./src/test/about.ts"
    ]
}
```

{% colorquote info %}
**files、include、exclude 優先權：**
files > include > exclude

1. files 優先度最高，files 指定的檔案不會被 exclude 排除
2. include 包含的檔案，可以透過 exclude 進行過濾
3. 同時寫 files 跟 include，相符的檔案與資料夾都會被編譯
{% endcolorquote %}

### **extends**

字串，使用其他 `tsconfig.json` 檔案作為基礎，並覆寫或是擴充設定，可以使用絕對路徑、相對路徑

```json
{
    "extends": "./base.json"
}
```

---

參考資源：

[https://ithelp.ithome.com.tw/articles/10216636](https://ithelp.ithome.com.tw/articles/10216636)