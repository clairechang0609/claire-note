---
title: WebRTC 學習筆記 (1) 基本觀念與流程
date: 2023-04-25
tags: [ webrtc ]
category: WebRTC
description: 本篇說明什麼是 WebRTC，以及 WebRTC 如何達成連線。透過流程拆解，更快速的掌握相關知識
image: https://i.imgur.com/XprAibC.png
---

<div style="display: flex; justify-content: center; margin: 20px 0 30px;">
    <img style="width: 100%; max-width: 450px;" src="https://i.imgur.com/XprAibC.png">
</div>

WebRTC（Web Real-Time Communication）是一種用於瀏覽器之間進行即時通訊的技術。讓用戶可以透過瀏覽器建立視訊、音訊和資料傳輸的連線，並實現 P2P 點對點通訊，而無須透過中介服務。

<!-- more -->

## **WebRTC 如何達成連線？**

WebRTC 即時通訊是透過 P2P（Peer-to-Peer）連線來達成，不過 P2P 連線並沒有這麼簡單，因大部分裝置處於區域網路中，通常是透過 NAT（Network Address Translation）將內部網路 IP 轉換成外部網路 IP 來達成連線，也就是說並沒有直接開放的通訊路徑（無法直接連到內網 IP），因此需要：

1. 先透過 ICE 框架來找到共同的連線路徑，並用 Signaling Server 交換資訊<span style="font-weight: bold; color: rgb(91, 155, 213)">（上圖步驟 ①②③④）</span>
2. 透過 P2P 達成即時通訊<span style="font-weight: bold; color: rgb(237, 125, 49)">（上圖步驟 ⑤）</span>

<div style="margin-top: 30px;"></div>

> 為什麼要使用 P2P 連線？
ICE 框架是什麼？
Signaling Server又是什麼？
P2P 連線如何進行視訊通話？
> 

<div style="margin-bottom: 30px;"></div>

看完了 WebRTC 的概念還是充滿問號？？？
接下來依序說明～

---

## **P2P（Peer-to-Peer）Connection 點對點連線**

P2P 指的是兩個瀏覽器直接建立點對點連接，不需透過伺服器中轉，可以達到即時影音串流。WebRTC 內的 P2P 連線由 [WebRTC API](https://developer.mozilla.org/zh-TW/docs/Web/API/WebRTC_API) 達成，先透過 `RTCPeerConnection()` 建立實體，來進行本地跟遠端的溝通。

{% colorquote info %}
**WebRTC API**
**Constructor 建構式：**
• RTCPeerConnection()：建立物件實體，用來處理本地跟遠端點對點連線
**Event Handler 事件處理器：**
• icecandidate：找尋到 ICE 候選位置
• iceconnectionstatechange：ICE 連線狀態改變
• addstream：新增串流
**Methods：**
• addTrack：增加視訊、音訊，以傳輸到另一端
• createOffer：建立 Offer SDP
• createAnswer：建立 Answer SDP
• setLocalDescription：設定本地 SDP 資訊
• setRemoteDescription：設定遠端（對方）SDP 資訊
• addIceCandidate：新增 ICE 候選位址
{% endcolorquote %}

## **ICE 框架（Interactive Connectivity Establishment）**

用於在網路中穿透 NAT 的技術框架，ICE 整合了 STUN 與 TURN 協定，用於確定兩個設備之間最佳的通訊路徑，進而達成端點間連線。

{% colorquote info %}
**NAT（Network Address Translation）網路位置轉譯**
用來將私有 IP 轉換（映射）為公有 IP，由於 NAT 不允許外網主機直接訪問內網，因此會造成 P2P 直接連線的困難，NAT 跟防火牆詳細說明可以參考 [這篇文章](https://ithelp.ithome.com.tw/articles/10209590)
{% endcolorquote %}

{% colorquote info %}
**STUN（Session Traversal Utilities for NAT）協定**
用來穿越 NAT 的一種協議，協助在 NAT 內的使用者找到自己的公網 IP和 Port，並將資訊傳給其他使用者，讓雙方瞭解彼此的資訊，繞過 NAT 限制，達成 P2P 連線，不過在 Symmetric NAT 不可穿透的情況下則不適用
{% endcolorquote %}

{% colorquote info %}
**TURN（Traversal Using Relay NAT）協定**
穿越 NAT 的另一種協議，使用「中繼」的方式來進行，當 STUN 的候選位址都無法連線時，將資料透過中間伺服器傳輸
{% endcolorquote %}

#### **ICE 建立連線流程**

1. 先嘗試用本地網路 IP 直接連線

2. 如果失敗，再使用 STUN 協定穿透 NAT 建立連線

3. 如果失敗，最後才使用 TURN 協定將資料透過中繼伺服器傳輸

## **Signaling Server 信號伺服器**

WebRTC 通訊時雖然不需透過伺服器中轉，使用 P2P 達到即時影音串流，但在此之前還是需要先透過伺服器來進行媒體串流和 ICE 候選位址協調，以確定通訊的對方和方式，這個伺服器就稱為 Signaling Server。

WebRTC 沒有特別規範 Signaling Server，常用的服務為 WebSocket 或是 HTTP 伺服器。

#### **Signaling Server 建立流程**

1. 透過 WebSocket 或是 HTTP 伺服器建立連線
2. 發送端與接收端分別使用 SDP（Session Description Protocol）offer-answer 模型描述媒體串流訊息，傳遞給 Signaling Server，Signaling Server 再傳遞給對方
3. 發送端與接收端分別將其收集到的本地網路位址和 ICE 候選位址通知 Signaling Server，Signaling Server 再將訊息傳遞給對方
4. 控制訊息（通話開始、暫停、結束等）與數據傳輸（聊天訊息、檔案）

{% colorquote info %}
**SDP（Session Description Protocol）協定**
用於描述多媒體會話的格式化文字協定，像是會議的參與者、媒體類型、媒體流的傳輸協定、編碼方式、解碼方式、封包大小等訊息。也可以包含其他相關訊息，例如時區、媒體串流的 port 號、會話持續時間等。
SDP offer-answer 模型中，發送端會建立一個 SDP offer，並向接收端傳送該 SDP offer。接收端收到 SDP offer 後，會建立 SDP answer 並回傳給發送端，交換 SDP 的過程，被稱為**媒體協商**
{% endcolorquote %}

## **P2P 連線如何進行視訊通話？**

P2P 連線成功後，需透過 [MediaDevice API](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia) 裡的 `getUserMedia()` 取得本地媒體串流，媒體串流為包含視訊、音訊和資料的數據串流，WebRTC 使用多種技術來達成媒體串流傳輸，包括即時傳輸協定（RTP）和即時傳輸控制協定（RTCP）等，並使用前面提到的 WebRTC API 來達成 P2P 連線與資料傳遞。

### **試著使用 getUserMedia 在瀏覽器輸出影音**

呼叫 `getUserMedia()` 取得使用者的音訊和視訊，會回傳一個 Promise 物件，constraints 用來設定擷取條件（影像或聲音，至少要有一項為 true），以下範例會在瀏覽器上輸出本地端影音：

```html
<video id="video" autoplay></video>
```

```jsx
const constraints = {
    audio: true,
    video: true
};
const video = document.querySelector("#video");

const createStream = async () => {
    try {
        const stream = await navigator.mediaDevices.getUserMedia(constraints);
        video.srcObject = stream;
    } catch (err) {
        throw err;
    }
}
```

視訊影像是顛倒的，我們可以透過 CSS 設定反轉

```css
#video {
    transform: scaleX(-1);
}
```

---

## **WebRTC 一對一視訊流程解析**

#### **視訊情境**

裝置 A 開啟了一個聊天室，接著裝置 B 連接該聊天室進行視訊聊天

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 600px;" src="https://i.imgur.com/1l46aJb.png">
</div>

<small>圖片參考：[MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Connectivity#the_entire_exchange_in_a_complicated_diagram)
</small>

#### **流程解析（搭配 STUN 協定）**

1. <span style="font-weight: 500; color: rgb(91, 155, 213)">裝置 A 透過 STUN server 取得本地私有 IP 以及公有 IP</span>
2. <span style="font-weight: 500; color: rgb(237, 125, 49)">裝置 B 透過 STUN server 取得本地私有 IP 以及公有 IP</span>
3. <span style="font-weight: 500; color: rgb(91, 155, 213)">裝置 A 發送 offer SDP，並設定為本地 SDP</span>
4. <span style="font-weight: 500; color: rgb(91, 155, 213)">裝置 B 收到 offer SDP，並設定為遠端 SDP</span>
5. <span style="font-weight: 500; color: rgb(237, 125, 49)">裝置 B 發送 answer SDP，並設定為本地 SDP</span>
6. <span style="font-weight: 500; color: rgb(237, 125, 49)">裝置 A 收到 answer SDP，並設定為遠端 SDP</span>
7. <span style="font-weight: 500; color: rgb(91, 155, 213)">裝置 A 傳送 ICE 候選位址</span>
8. <span style="font-weight: 500; color: rgb(91, 155, 213)">裝置 B 寫入裝置 A ICE 候選位址</span>
9. <span style="font-weight: 500; color: rgb(237, 125, 49)">裝置 B 傳送 ICE 候選位址</span>
10. <span style="font-weight: 500; color: rgb(237, 125, 49)">裝置 A 寫入裝置 B ICE 候選位址</span>
11. <span style="font-weight: 500; color: rgb(112, 173, 71)">P2P 連線建立，進行即時媒體串流（音訊、視訊）</span>

WebRTC 基本觀念說明就到這，下一篇將會進行實作，一起完成一對一視訊聊天室吧！

---

參考文章：

[https://www.wowza.com/blog/webrtc-signaling-servers](https://www.wowza.com/blog/webrtc-signaling-servers)

[https://ithelp.ithome.com.tw/articles/10209725](https://ithelp.ithome.com.tw/articles/10209725)

[https://ithelp.ithome.com.tw/articles/10284640](https://ithelp.ithome.com.tw/articles/10284640)

[https://ithelp.ithome.com.tw/articles/10266739](https://ithelp.ithome.com.tw/articles/10266739)