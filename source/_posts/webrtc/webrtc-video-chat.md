---
title: WebRTC 學習筆記 (2) 實作一對一視訊聊天室
date: 2023-05-02
tags: [ webrtc ]
category: WebRTC
description: 如何透過 WebRTC 建立一個一對一視訊聊天室，藉由流程拆解，可以更清楚理解每個步驟的功能與目的
image: https://imgur.com/KgU7DNp.png
---

[上一篇](https://clairechang.tw/2023/04/25/webrtc/webrtc-intro/) 說明了完成聊天室必備的基礎觀念，接著就來進行實作吧！

## **視訊情境**

裝置 A 開啟了一個聊天室，接著裝置 B 連接該聊天室進行視訊聊天

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 600px;" src="https://i.imgur.com/1l46aJb.png">
</div>

<small>圖片參考：[MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Connectivity#the_entire_exchange_in_a_complicated_diagram)
</small>


### **流程解析（搭配 STUN 協定）**

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

<!-- more -->

---

## **視訊實作**

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/KgU7DNp.png">
</div>

實作分為 **Server（Signaling Server）**以及 **Client（視訊聊天室）**進行

檔案結構如下：

```html
server.js
chat.html
chat.js
```

## **Server（server.js）**

### **套件安裝（NPM）**

- Node.js 框架 Express
- Websocket 套件 Socket.io

### **套件引入**

```jsx
const express = require('express');
const app = express();
const http = require('http');
const socket = require('socket.io');
```

### **啟動伺服器**

port 設定 3000，因此伺服器連線網址為 http://localhost:3000

```jsx
const server = http.createServer(app).listen('3000');
```

### **連接 Client 頁面（轉換為絕對路徑 __dirname）**

```jsx
// html 路徑轉換
app.get('/chat', function(req, res) {
    es.sendfile(`${__dirname}/chat.html`);
});

// js 路徑轉換
app.get(/(.*)\.(jpg|gif|png|ico|css|js|txt)/i, function(req, res) {
    console.log(__dirname);
    res.sendfile(`${__dirname}/${req.params[0]}.${req.params[1]}`);
});
```

### **加入 Socket.io**

- `socket.on()` 監聽 join, offer, answer, ice_candidate, hangup 等 Client 自訂事件
- `socket.emit()` 傳遞事件給 Client

```jsx
const io = socket(server);

io.on('connection', (socket) => {
    console.log('connection');

    // 加入房間
    socket.on('join', (room) => {
        console.log('join');
        socket.join(room);
        socket.to(room).emit('ready', '準備通話');
    });

    // 轉傳 Offer
    socket.on('offer', (room, description) => {
        socket.to(room).emit('offer', description);
    });

    // 轉傳 Answer
    socket.on('answer', (room, desc) => {
        socket.to(room).emit('answer', description);
    });

    // 交換 ice candidate
    socket.on('ice_candidate', (room, data) => {
        socket.to(room).emit('ice_candidate', data);
    });

    // 關閉通話
    socket.on('hangup', (room) => {
        console.log('hangup');
        socket.leave(room);
    });
});
```

## **Client（視訊聊天室）**

### **套件安裝（CDN）**

- WebRTC Adapter（解決相容性問題）
- Socket.io-client

### **HTML（chat.html）**

- 加入 Video Dom
    - #localVideo 接收本地媒體串流
    - #remoteVideo 接收遠端媒體串流
- CDN WebRTC Adapter 以及 Socket.io-client
- 引入 chat.js

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, user-scalable=yes, initial-scale=1, maximum-scale=1">
    <title>chat</title>
    <style lang="scss">
        video {
            transform: scaleX(-1);
        }
    </style>
</head>
<body>
    <div>
        <video muted="false" width="320" autoplay playsinline id="localVideo"></video>
        <video width="320" autoplay playsinline id="remoteVideo"></video>
        
        <button type="button" id="call">call</button>
        <button type="button" id="hangup">hangup</button>
    </div>
    <script src="https://webrtc.github.io/adapter/adapter-latest.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/socket.io-client@3.0.4/dist/socket.io.js"></script>
    <script src="chat.js" type="module" async></script>
</body>
</html>
```

### **Javascript（chat.js）**

#### **任務步驟拆分**

1. 連接 Socket Server（Signaling Server）
透過 Socket 監聽 Offer SDP、Answer SDP、ICE Candidate
2. getUserMedia 取得本地媒體串流
3. RTCPeerConnection 建立 P2P 連線，與 RTCPeerConnection 事件監聽

#### **定義常數與變數**

```jsx
const localVideo = document.querySelector('#localVideo');
const remoteVideo = document.querySelector('#remoteVideo');
const callBtn = document.querySelector('#call');
const hangupBtn = document.querySelector('#hangup');
let peerConnection;
let socket;
let localStream;
const room = 'room1'; // 房間先預設為 room1
```

#### **連接 Socket Server（Signaling Server）**
透過 Socket 監聽 Offer SDP、Answer SDP、ICE Candidate

`socket.emit()` 以及 `socket.on()` 傳遞與接收資料給 Server

```jsx
const socketConnect = () => {
    // 伺服器連線網址：http://localhost:3000
    socket = io('ws://localhost:3000');

    // 發送房間資訊
    socket.emit('join', room);

    // 監聽加入房間
    socket.on('ready', (msg) => {
        // 發送 Offer SDP
        sendSDP('offer');
    });

    // 監聽收到 Offer
    socket.on('offer', async (desc) => {
        // 設定對方的媒體串流
        await peerConnection.setRemoteDescription(desc);
        // 發送 Answer SDP
        await sendSDP('answer');
    });

    // 監聽收到 Answer
    socket.on('answer', (desc) => {
        // 設定對方的媒體串流
        peerConnection.setRemoteDescription(desc)
    });

    // 監聽收到 ICE 候選位址
    socket.on('ice_candidate', (data) => {
        // RTCIceCandidate 用以定義 ICE 候選位址
        const candidate = new RTCIceCandidate({
            sdpMLineIndex: data.label,
            candidate: data.candidate
        });
        // 加入 ICE 候選位址
        peerConnection.addIceCandidate(candidate);
    });
};
```

處理 Offer SDP / Answer SDP

```jsx
/**
 * @param {String} type offer/answer
 */
const sendSDP = async (type) => {
    try {
        if (!peerConnection) {
            console.log('尚未開啟視訊');
            return;
        }

        const method = type === 'offer' ? 'createOffer' : 'createAnswer';
        const offerOptions = {
            offerToReceiveAudio: true, // 是否傳送聲音流給對方
            offerToReceiveVideo: true // 是否傳送影像流給對方
        };

        // 建立 SDP
        const localSDP = await peerConnection[method](offerOptions);

        // 設定本地 SDP
        await peerConnection.setLocalDescription(localSDP);

        // 發送 SDP
        socket.emit(type, room, peerConnection.localDescription);
    } catch (err) {
        console.log('error: ', err);
    }
};
```

#### **getUserMedia 取得本地媒體串流**

```jsx
const createStream = async () => {
    try {
        const constraints = { audio: true, video: true };

        // getUserMedia 取得本地影音串流
        const stream = await navigator.mediaDevices.getUserMedia(constraints);

        // Dom 設置本地媒體串流
        localVideo.srcObject = stream;

        // 傳出媒體串流
        localStream = stream;
    } catch (err) {
        console.log('getUserMedia error: ', err.message, err.name);
    }
};
```

#### **RTCPeerConnection 建立 P2P 連線，與 RTCPeerConnection 事件監聽**

- RTCPeerConnection 內的 `iceServers` 用來建立兩台裝置點對點連線的伺服器
- `getTracks()`：取得媒體串流（MediaStream）中媒體軌道（MediaStreamTrack）陣列
- `addTrack(MediaStreamTrack, MediaStream)`：增加媒體軌道（MediaStreamTrack）到 RTCPeerConnection 中指定的媒體串流（MediaStream）

```jsx
const createPeerConnection = () => {
    // 設定 iceServer
    const configuration = {
        iceServers: [{
            urls: 'stun:stun.l.google.com:19302' // google 提供免費的 STUN server
        }]
    };
    // 建立 RTCPeerConnection
    peerConnection = new RTCPeerConnection(configuration);

    // 增加本地串流
    localStream.getTracks().forEach((track) => {
        peerConnection.addTrack(track, localStream);
    });

    // 找尋到 ICE 候選位址後，送去 Server 與另一位配對
    peerConnection.onicecandidate = (e) => {
        if (e.candidate) {
            // 發送 ICE
            socket.emit('ice_candidate', room, {
                label: e.candidate.sdpMLineIndex,
                id: e.candidate.sdpMid,
                candidate: e.candidate.candidate,
            });
        }
    };

    // 監聽 ICE 連接狀態
    peerConnection.oniceconnectionstatechange = (e) => {
        // 若連接已斷，執行掛斷相關動作
        if (e.target.iceConnectionState === 'disconnected') {
            hangup();
        }
    };

    // 監聽是否有媒體串流傳入
    peerConnection.onaddstream = ({ stream }) => {
        // Dom 加入遠端串流
        remoteVideo.srcObject = stream;
    };
};
```

#### **點擊按鈕開始連線／關閉連線**

```jsx
// 建立本地媒體串流
createStream();

// 開始連線
const call = () => {
    socketConnect(); // socket 連線
    createPeerConnection(); // 建立 P2P 連線
};

// 關閉連線
const hangup = () => {
    // 移除事件監聽
    peerConnection.onicecandidate = null;
    peerConnection.onnegotiationneeded = null;

    // 關閉 RTCPeerConnection 連線並釋放記憶體
    peerConnection.close();
    peerConnection = null;

    // 傳遞掛斷事件給 Server
    socket.emit('hangup', room);
    socket = null;

    // 移除遠端 video src
    remoteVideo.srcObject = null; // 移除遠端媒體串流
};

callBtn.addEventListener('click', call);
hangupBtn.addEventListener('click', hangup);
```

這樣一對一聊天室就完成囉，後續會說明如何達成多方視訊功能

---

參考文章：

[https://ithelp.ithome.com.tw/articles/10251454](https://ithelp.ithome.com.tw/articles/10251454)

[https://ithelp.ithome.com.tw/articles/10209193](https://ithelp.ithome.com.tw/articles/10209193)

[https://ithelp.ithome.com.tw/articles/10278727](https://ithelp.ithome.com.tw/articles/10278727)

[https://medium.com/@jedy05097952/初探-webrtc-手把手建立線上視訊-1-5e9d4702e8e8](https://medium.com/@jedy05097952/%E5%88%9D%E6%8E%A2-webrtc-%E6%89%8B%E6%8A%8A%E6%89%8B%E5%BB%BA%E7%AB%8B%E7%B7%9A%E4%B8%8A%E8%A6%96%E8%A8%8A-1-5e9d4702e8e8)