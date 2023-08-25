---
title: Javascript Web Share API- Navigator.share 分享網頁內容
date: 2023-02-22
tags: [ javascript ]
category: Javascript
description: Web Share API 可以協助我們在 Android 或 IOS 網頁上做到類似像 App 的分享功能，主要是透過作業系統原生介面分享，可以輕鬆地將內容分享至第三方平台（Facebook、Twitter、Line…）
image: https://i.imgur.com/veVgLUo.png
---

<div style="display: flex; justify-content: center; margin: 20px 0 30px;">
    <img style="width: 100%; max-width: 350px;" src="https://i.imgur.com/veVgLUo.png">
</div>

手機 App 可以透過原生介面實現分享功能，而 **Web Share API** 可以協助我們在 Android 或 IOS 網頁上做到類似像 App 的分享功能，主要是透過作業系統原生介面分享，可以輕鬆地將內容分享至第三方平台（Facebook、Twitter、Line…），分享內容可以為**連結、文字或是檔案**。

<!-- more -->

## **方法介紹**

#### **Navigator.share()**

回傳一個 Promise 函式，可以捕捉成功 / 失敗結果，需透過按鈕 click 等 UI 事件來觸發

#### **Navigator.canShare()**

回傳 Boolean，用以判斷內容是否可以分享

## **使用限制**

- 必須透過 HTTPS（secure contexts）連結網站
- 必須透過 UI 事件觸發，像是按鈕 click
- 瀏覽器支援度較低（[caniuse-Web-Share-API](https://caniuse.com/web-share)）

## **Navigator.share 參數選擇**

- **url：**分享的連結（String）
- **title：**分享的標題（String）
- **text：**分享的內文（String）
- **files：**分享的檔案（Array）

## **功能實作**

#### **連結、文字分享**

```jsx
<button type="button" id="share-btn">分享</button>
```

因瀏覽器支援度較低，建議加上判斷 `Navigator.share` 是否存在，避免直接報錯

```jsx
const btn = document.querySelector('#share-btn');

btn.addEventListener('click', async () => {
    try {
        if (!navigator.share) {
            return;
        }
        await navigator.share({
            title: 'Title',
            text: 'Share Content',
            url: 'https://xxx.com'
        })
    } catch (err) {
        console.log(err.message);
    }
});
```

#### **檔案分享**

```jsx
<input type="file" id="file-input" accept="image/*" />
<button type="button" id="share-btn">分享</button>
```

先判斷檔案是否能共享，使用 `Navigator.canShare`，如果回傳 true，表示 `Navigator.share` 可以成功呼叫

```jsx
const btn = document.querySelector('#share-btn');
const input = document.querySelector('#file-input');

btn.addEventListener('click', async () => {
    try {
        const files = input.files;

        if (!files.length || !navigator.canShare || !navigator.canShare({ files })) {
            return;
        }
        await navigator.share({
            files,
            title: 'Image',
            text: 'Share Content'
        })
    } catch (err) {
        console.log(err.message);
    }
});
```

範例畫面

<iframe height="470" style="width: 100%;" scrolling="no" title="Web Share API：Navigator.share" src="https://codepen.io/claire-chang-the-bashful/embed/YzOwjKM?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/claire-chang-the-bashful/pen/YzOwjKM">
  Web Share API：Navigator.share</a> by Claire Chang (<a href="https://codepen.io/claire-chang-the-bashful">@claire-chang-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

---

參考文章：

[https://developer.mozilla.org/en-US/docs/Web/API/Navigator/share](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/share)
[https://web.dev/web-share/](https://web.dev/web-share/)