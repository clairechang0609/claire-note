---
title: 一次搞懂 JavaScript 的 slice、splice、split
date: 2025-02-04
tags: [ javascript ]
category: Javascript
description: 這篇文章說明並比較 JavaScript 中常見的三個方法—slice、splice、split，並解析主要用法及實際應用範例
image: /images/js/slice-splice-split/all.png
---

<div style="display: flex; justify-content: center; margin-top: 30px; margin-bottom: 30px;">
  <img style="width: 100%; max-width: 100%;" src="/images/js/slice-splice-split/all.png">
</div>

即便使用 JS 開發好幾年，偶爾在使用常用函式時還是要 google 一下（還好意思說），因此筆記起來協助我的石頭腦袋！

<!-- more -->

## **slice**

<div style="margin-top: 20px; margin-bottom: 20px;">
  <img style="width: 100%; max-width: 100%;" src="/images/js/slice-splice-split/slice.png">
</div>

- **陣列方法**
- 取得原陣列的部分區間
- 回傳一個新的陣列，為原陣列 `start`（包含）到 `end`（不含）的淺拷貝
- 原陣列不會被修改

##### **語法**

```jsx
arr.slice([start[, end]])
```

##### **參數**

- `start`：起始索引（選填），未填則取得整個陣列淺拷貝
- `end`：結束索引（選填），未填則取得從 `start` 開始的陣列淺拷貝

##### **範例**

```jsx
const snacks = ['cookie', 'chocolate', 'egg roll', 'apple pie', 'haribo'];

const selectedSnacks1 = snacks.slice(2, 4);
const selectedSnacks2 = snacks.slice(2);
const selectedSnacks3 = snacks.slice();
const selectedSnacks4 = snacks.slice(-1);

console.log(selectedSnacks1); // ['egg roll', 'apple pie']
console.log(selectedSnacks2); // ['egg roll', 'apple pie', 'haribo']
console.log(selectedSnacks3); // ['cookie', 'chocolate', 'egg roll', 'apple pie', 'haribo']
console.log(selectedSnacks4); // ['haribo']
```

---

## **splice**

<div style="margin-top: 20px; margin-bottom: 20px;">
  <img style="width: 100%; max-width: 100%;" src="/images/js/slice-splice-split/splice.png">
</div>

- **陣列方法**
- 對陣列刪除或加入新的元素
- 回傳一個被刪除的元素組成的陣列
- 原陣列會被修改

##### **語法**

```jsx
arr.splice(start[, deleteCount[, item1[, item2[, ...]]]])
```

##### **參數**

- `start`：起始索引（必填）
- `deleteCount`：刪除的數量（選填），未填則移除從 `start` 開始的所有元素
- `item1、item2、…`：要加入陣列的元素（從 `start` 開始加入）（選填）

##### **範例**

```jsx
const snacks1 = ['cookie', 'chocolate', 'egg roll', 'apple pie', 'haribo'];
const deletedSnacks1 = snacks1.splice(2, 3, 'cream cake');

console.log(snacks1); // ['cookie', 'chocolate', 'cream cake']
console.log(deletedSnacks1); // ['egg roll', 'apple pie', 'haribo']

const snacks2 = ['cookie', 'chocolate', 'egg roll', 'apple pie', 'haribo'];
const deletedSnacks2 = snacks2.splice(1);

console.log(snacks2); // ['cookie']
console.log(deletedSnacks2); // ['chocolate', 'egg roll', 'apple pie', 'haribo']

const snacks3 = ['cookie', 'chocolate', 'egg roll', 'apple pie', 'haribo'];
const deletedSnacks3 = snacks3.splice(4, 1);

console.log(snacks3); // ['cookie', 'chocolate', 'egg roll', 'apple pie']
console.log(deletedSnacks3); // ['haribo']
```

---

## **split**

<div style="margin-top: 20px; margin-bottom: 20px;">
  <img style="width: 100%; max-width: 100%;" src="/images/js/slice-splice-split/split.png">
</div>

- **字串方法**
- 將字串切割為一組陣列，包含數個字串片段
- 回傳一個字串切割後的陣列
- 原字串不會被修改

##### **語法**

```jsx
str.split(separator[, limit])
```

##### **參數**

- `separator`：分隔符號（必填），可以為字串或是正則
- `limit`：限制擷取的最大數量（選填），未填則取得所有切割後的字串

##### **範例**

```jsx
const snacks = 'cookie, chocolate, egg roll, apple pie, haribo';

const snacksArr1 = snacks.split(', ');
const snacksArr2 = snacks.split(', ', 3);

console.log(snacksArr1); // ['cookie', 'chocolate', 'egg roll', 'apple pie', 'haribo']
console.log(snacksArr2); // ['cookie', 'chocolate', 'egg roll']
```

---

### **總結**

最後用一張表格彙整，比較三者差異：

| 方法 | 對象 | 用途 | 回傳值 | 是否影響原資料 |
| --- | --- | --- | --- | --- |
| slice | Array | 取得原陣列的部分區間 | 新陣列 | <span style="color:rgb(224, 58, 46);">⨉ 不會</span> |
| splice | Array | 對陣列刪除或加入新的元素 | 刪除的項目 | <span style="color:rgb(23, 161, 85);">○ 會</span> |
| split | String | 將字串切割為一組陣列 | 新陣列 | <span style="color: rgb(224, 58, 46);">⨉ 不會</span> |
