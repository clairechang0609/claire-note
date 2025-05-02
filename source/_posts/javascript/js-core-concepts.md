---
title: JavaScript 執行機制與核心概念
date: 2025-04-29
tags: [ javascript ]
category: Javascript
description: 為了更紮實地掌握 JavaScript 的本質，整理了直譯式特性、語法作用域、單執行緒模型，到執行環境（Execution Context）、執行堆疊（Call Stack）、事件迴圈（Event Loop）與程式提升（Hoisting）等核心概念。
image: /images/js/core-concepts/call-stack.gif
---

<div style="display: flex; justify-content: center; margin-top: 30px; margin-bottom: 30px;">
  <img style="width: 100%; max-width: 100%;" src="/images/js/core-concepts/event-loop.png">
</div>

從事前端開發幾年了，卻一直對 JavaScript 和瀏覽器的底層機制理解得不夠深入。這篇整理了執行機制與核心概念，希望補齊這塊知識，讓自己在開發、效能優化與問題分析時，能有更清晰的理解與判斷基礎。

<!-- more -->

---

### **直譯式語言（Interpreted Language）**

JavaScript 為直譯式語言，透過直譯器來產生並執行程式碼。不管是直譯式語言還是編譯式語言，都須將撰寫的高階語言，轉換成電腦能讀取的機器碼（Machine Code）（低階語言）。

- **編譯式語言（Compiled Language）：**C / C++ / Go，開發完成後會先經過編譯程序，把原始碼編譯成機器碼，產生一支可直接執行的檔案
- **直譯式語言（Interpreted Language）：**JavaScript / Python / Ruby / PHP，在執行時由直譯器協助轉譯成機器碼，過程中不會產生獨立的可執行檔案，而是必須依賴執行平台（Runtime Environment），如瀏覽器或 Node.js 才能運作

JavaScript 在瀏覽器或 Node.js 中由 JavaScript 引擎（如 V8）即時執行解析、轉為中間碼（ByteCode）、即時編譯為機器碼並執行。

---

### **語法作用域（Lexical Scope）**

JavaScript 語言為語法作用域，變數的作用域在程式撰寫階段（也就是語法解析時）就已經決定，而不是在程式執行時才動態判斷。

- **靜態作用域：**也稱為**語法作用域（Lexical Scope）**，在語法解析時即確定作用域
- **動態作用域：**在函式調用時才決定作用域

因為 JavaScript 採用靜態作用域，當我們在函式中存取外部變數時，會往語法上的外層尋找，而不是根據函式被呼叫的位置來決定作用範圍。

在這個過程中，JavaScript 引擎會依據**範圍鏈（Scope Chain）**來逐層查找變數。範圍鏈是由當前執行環境與其外部環境組成的結構，確保在當前作用域找不到變數時，可以沿著外層作用域一路向上搜尋，直到全域作用域為止。

以下範例，因作用域在語法解析時即確定，因此無論在什麼時候、什麼位置執行 `findAnimal()`，`showAnimal()` 函式中的 `animal` 變數結果都一樣（`Cat`）。

```jsx
const animal = 'Cat';

function showAnimal() {
  console.log(animal);
}

function findAnimal() {
  const animal = 'Dog';
  showAnimal();
}

findAnimal();
// print: Cat
```

---

### **單執行緒（Single Threaded）**

JavaScript 為單一執行緒程式語言，意即一次只能處理一件事情。在瀏覽器中，只有一條 JS 主執行緒執行 JavaScript 程式碼，但瀏覽器本身是多執行緒，可以同時處理計時器、事件監聽等背景工作。當條件達成後，將 callback 加入事件佇列（Event Queue），等待 JavaScript 主執行緒有空時再執行。

> JavaScript 是單執行緒，只能靠一條主線程跑程式；而瀏覽器或 Node.js 是多執行緒，幫 JavaScript 處理非同步工作。
> 

---

### **執行環境（Execution Context）**

直譯式語言需依賴像瀏覽器或是 Node.js 這樣的執行平台（Runtime Enviroment）才能運作，其中包含了 JavaScript 引擎（如 V8），負責轉譯跟執行程式碼。程式碼在執行過程中，JavaScript 引擎會為每段可執行的程式碼建立執行環境（Execution Context），用來管理當前的作用域、變數與 `this`狀態。

在網頁開啟時，會先產生一個全域執行環境，包含：`window（this）`。函式在宣告時，即確定作用域，不過一直到函式執行時，才會產生執行環境。若反覆執行該函式，會產生多個執行環境。

---

### **執行堆疊（Call Stack）**

延續前面提到的執行環境概念，當網頁載入時，JavaScript 引擎會先建立全域執行環境。

當函式被呼叫時，JavaScript 引擎會建立對應的執行環境，並放到執行環境堆疊（Call Stack）中，佔用相應的記憶體資源。函式執行結束後，該執行環境會從堆疊中移除。需要特別注意的是，執行環境的建立與函式的宣告順序無關，而是根據執行時的順序動態建立。

執行堆疊的運作模式為 LIFO（Last In, First Out，後進先出），最新被呼叫的函式會優先執行，然後從堆疊中移除。

若該執行環境內的變數、參數不再被其他程式引用，便會標記為可回收，等待垃圾回收機制（Garbage Collector）在適當時機釋放記憶體；若仍被其他閉包或外部物件參考，則相關資料會持續留在記憶體中。

> 可以透過 Chrome 開發者工具的 **Sources 面板**，觀察執行時的 Scope（作用域）跟 Stack（堆疊）變化，幫助理解執行流程。
>

<div style="display: flex; justify-content: center; margin-top: 30px; margin-bottom: 30px;">
  <img style="width: 100%; max-width: 100%;" src="/images/js/core-concepts/call-stack.gif">
</div>

以下範例，在執行 `greet(friend)` 時，執行堆疊由上到下為：

- `greet()` 執行環境
- `startGreeting()` 執行環境
- 全域執行環境

```jsx
function greet(friend) {
  return `Hello, ${friend}`;
} 

function startGreeting() {
  const friend = 'Nana';
  console.log('greeting', greet(friend));
}

startGreeting();
```

以下範例，在執行 `inviteGuests()` 過程中，每次呼叫 `sendInvitation()`，執行堆疊變化如下：

1. `sendInvitation()` 執行環境
2. `inviteGuests()` 執行環境
3. 全域執行環境

`sendInvitation()` 執行完後會彈出堆疊，回到 `inviteGuests()` 繼續下一次迴圈。這個過程重複五次。

```jsx
function sendInvitation(number) {
  return `Invitation sent to guest ${number}`;
}

function inviteGuests() {
  sendInvitation(1);

  for (let i = 2; i <= 5; i++) {
    sendInvitation(i);
  }
}

inviteGuests();
```

---

### **事件迴圈（Event Loop）**

前面提到，JavaScript 為單執行緒程式語言，一次只能處理一件事情，為了在單執行緒中支援非同步行為（如計時器、網路請求、事件監聽），瀏覽器或 Node.js 使用事件迴圈（Event Loop）機制來協調任務的執行順序。

<div style="display: flex; justify-content: center; margin-top: 30px; margin-bottom: 30px;">
  <img style="width: 100%; max-width: 100%;" src="/images/js/core-concepts/event-loop.png">
</div>

#### **事件迴圈流程：**

1. 整個 JavaScript 主檔（如 `<script>` 內的程式碼）為一個 Macro Task
2. 當 Macro Task 執行時，其中所有同步程式碼會依序放入事件堆疊（Call Stack）執行。遇到非同步操作時，則交由瀏覽器背景處理，等到條件達成後將 callback 放到佇列（Macro Task Queue 或 Micro Task Queue）
3. 當前的 Macro Task 所有同步程式碼執行完畢後，JavaScript 引擎會先檢查是否有 Micro Task Queue，並依序放入事件堆疊執行，過程中如果有再產生新的 Micro Task 也會持續處理
4. Micro Tasks 清理完成後，如果有需要會重新渲染頁面
5. 接著才會從 Macro Task Queue 取出下一個 Macro Task 執行，進入下一輪事件迴圈

**佇列 Queue：**

- **Macro Task Queue（Event Queue）**：當執行像 `setTimeout`、`setInterval` 等非同步操作時，callback 會在條件達成後放入 Macro Task Queue
- **Micro Task Queue（Job Queue）**：當執行 `Promise.then()`、`Promise.catch()`、`Promise.finally()`、`MutationObserver` 等操作時，callback 會被放入 Micro Task Queue

> 每次執行完一個 Macro Task 後，會優先處理所有的 Micro Tasks，接著才執行下一個 Macro Task
> 

---

### **提升（Hoisting）**

JavaScript 執行環境（Execution Context）又分為**創造階段（Creation Phase）**跟**執行階段（Execution Phase）**。在創造階段，JavaScript 引擎會將**變數宣告（`var`）**（初始值為 `undefined`）和**函式陳述式**放到記憶體空間，等到執行階段，變數才被賦予指定的值，這就是提升（Hoisting）的特性。

> 函式陳述式會提升，函式表達式只有變數宣告提升
創造階段為函式優先
> 

以下範例，根據提升特性，最後印出的結果為 `undefined`。

```jsx
function callName() {
  console.log(Bosu);
}
callName();
var Bosu = '撥鼠';
```

將順序拆解為執行階段跟創造階段：

```jsx
// 創造
function callName() {
  console.log(Bosu);
}
var Bosu;

// 執行
callName();
Bosu = '撥鼠';
```

**暫時性死區（TDZ，Temporal Dead Zone）**

前面提到了 `var` 宣告的情境，如果是 `let` 或 `const` 宣告，在創造階段雖然也會被提升（Hoisting），但並不會被初始化，因此在執行階段被賦值之前，會處在一個特殊狀態，稱為暫時性死區，這個時候雖然變數存在，但不能存取，會直接拋出 `ReferenceError`。

將前面範例的 `var` 宣告改為 `let`，最後印出的結果為 `Uncaught ReferenceError: Bosu is not defined`。

```jsx
function callName() {
  console.log(Bosu);
}
callName();
let Bosu = '撥鼠';
```

---

參考資源：
https://www.linkedin.com/pulse/javascript-under-hood-microtasks-macrotasks-eliran-elnasi/?trk=read_related_article-card_title
https://medium.com/itsems-frontend/javascript-hoisting-589488622dd7
https://medium.com/itsems-frontend/javascript-event-loop-event-queue-call-stack-74a02fed5625
https://courses.hexschool.com/p/javascript
