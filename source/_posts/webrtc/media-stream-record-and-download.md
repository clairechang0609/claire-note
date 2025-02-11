---
title: WebRTC 學習筆記 (3) 錄製／下載影音串流
date: 2023-05-03
tags: [ webrtc ]
category: WebRTC
description: 本篇說明如何在瀏覽器捕捉媒體串流，將其錄製為影音檔，並於錄製停止後執行檔案下載
image: https://i.imgur.com/OY2uttm.png
---

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/OY2uttm.png">
</div>

> 本篇雖未涉及 WebRTC API 的應用，但 WebRTC 實作上常會搭配影音錄製與下載功能，因此將此篇文章歸類為系列文之一
> 

## **錄製影音串流**

[MediaStream Recording API](https://developer.mozilla.org/en-US/docs/Web/API/MediaStream_Recording_API) `MediaRecorder(stream)` 可用於在瀏覽器中捕捉媒體串流並將其錄製為影音檔，通常會搭配 MediaDevices API `getUserMedia()` Promise 函式取得媒體串流

<!-- more -->

```jsx
const createStream = async () => {
    const constraints = { audio: true, video: true };
    // 取得裝置的影音串流
    const stream = await navigator.mediaDevices.getUserMedia(constraints);
    return stream;
};

const startRecord = async () => {
    const stream = await createStream();
    const options = {
        audioBitsPerSecond: 128000,
        videoBitsPerSecond: 2500000,
        mimeType: 'video/webm'
    };
    // 建立 MediaRecorder 物件
    const recorder = new MediaRecorder(stream, options);
    // 開始錄製
    recorder.start();
};

startRecord(); 
```

呼叫 `MediaRecorder.start()` 時，如果沒有傳入參數，會在停止錄製時才回傳一個 Blob 檔，如果希望能拆分 chuck 檔案，可以在 `start(timeslice)` 代入時間參數：

```jsx
// 每秒回傳一次錄製的串流
MediaRecorder.start(1000);
```

**MediaRecorder API 相關方法和事件：**

- `MediaRecorder.start()` 開始錄製媒體
- `MediaRecorder.stop()` 停止錄製媒體
- `MediaRecorder.pause()` 暫停錄製媒體
- `MediaRecorder.resume()` 重新開始錄製媒體
- `MediaRecorder.ondataavailable` 錄製期間產生新的媒體資料（Blob 檔）時觸發
- `MediaRecorder.onstop` 媒體錄製完成並停止時觸發

## **下載影音串流**

搭配 `ondataavailable` 以及 `onstop` 進行監聽


{% colorquote info %}
前面提到的 `MediaRecorder.start(timeslice)` 方法傳入的時間參數，會影響 `ondataavailable` 觸發時機，假設 timeslice = 1000，每秒鐘會觸發一次 `ondataavailable`，並回傳這段時間（每秒）錄製的內容，若沒有傳入參數，會在停止錄製時才觸發，並回傳期間錄製的所有內容
{% endcolorquote %}

**步驟拆解：**

1. 透過 `ondataavailable` 取得錄製的 Blob chuck，並儲存變數內
2. 透過 `onstop` 監聽錄製停止
3. 執行檔案下載

```jsx
const recordedChunks = [];

recorder.start(1000);

setTimeout(() => {
    recorder.stop();
}, 5000);

// 1. 取得錄製的 Blob chuck，並儲存變數內
recorder.ondataavailable = (event) => {
    if (event.data.size > 0) {
        recordedChunks.push(event.data);
    }
};

// 2. 監聽錄製停止
recorder.onstop = () => {
    download();
};

// 3. 執行檔案下載
const download = () => {
    const blob = new Blob(recordedChunks, { type: 'video/webm' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    document.body.appendChild(a);
    a.style = 'display: none';
    a.href = url;
    a.download = 'test.webm';
    a.click();
    window.URL.revokeObjectURL(url);
};
```

`URL.createObjectURL()` 用來將 File 或是 Blob 物件轉成一個 url，利用這個 url 來處理檔案下載，下載完畢後再使用 `URL.revokeObjectURL()` 釋放 url

## **錄製與下載完整範例**

<iframe height="470" style="width: 100%;" scrolling="no" title="Download and Record Videos" src="https://codepen.io/claire-chang-the-bashful/embed/PoyOPwW?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/claire-chang-the-bashful/pen/PoyOPwW">
  Download and Record Videos</a> by Claire Chang (<a href="https://codepen.io/claire-chang-the-bashful">@claire-chang-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

---

參考資源：

[https://developer.mozilla.org/en-US/docs/Web/API/MediaStream_Recording_API](https://developer.mozilla.org/en-US/docs/Web/API/MediaStream_Recording_API)

[https://medium.com/@kf99916/html5-神奇的-object-url-不用後端-前端便能產生獲取指定物件的網址-6df283d58505](https://medium.com/@kf99916/html5-%E7%A5%9E%E5%A5%87%E7%9A%84-object-url-%E4%B8%8D%E7%94%A8%E5%BE%8C%E7%AB%AF-%E5%89%8D%E7%AB%AF%E4%BE%BF%E8%83%BD%E7%94%A2%E7%94%9F%E7%8D%B2%E5%8F%96%E6%8C%87%E5%AE%9A%E7%89%A9%E4%BB%B6%E7%9A%84%E7%B6%B2%E5%9D%80-6df283d58505)