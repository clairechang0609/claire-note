---
title: WebRTC 學習筆記 (4) 實作多方視訊聊天室
date: 2023-05-10 17:00
tags: [ webrtc ]
category: WebRTC
description: 本篇說明如何使用 WebRTC 實作「多方」視訊聊天室，並解析運用原理
image: https://i.imgur.com/GqA532q.png
---

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 500px;" src="https://i.imgur.com/GqA532q.png">
</div>

本篇是視訊聊天室的進階版本，必須先掌握 [一對一視訊聊天室](https://clairechang.tw/2023/05/02/webrtc/webrtc-video-chat/) 介紹的相關知識，才能更快理解本篇的實作內容～

## **視訊情境**

多個裝置 A、B、C …… 想要進行多方視訊聊天，可以隨時進入聊天室，不需要特定的順序

<!-- more -->

## **流程解析**

多方視訊可以透過 `RTCPeerConnection()` 建立多個連線來達成。跟一對一的連線類似，**多方視訊中的每個 P2P 連線 發送端（Offer）與 接收端（Answer）都需要分別建立一個 `RTCPeerConnection()` 物件，每個物件分別管理一個連線，並透過 Signaling Server 交換 ICE 和 SDP 以建立連線。**

連線建立後，每個裝置可以將自己的影像和聲音傳送到其他裝置的 `RTCPeerConnection()` 中，以達成多方視訊的效果。此外，也可以透過使用媒體伺服器（如 SFU 或 MCU）來進一步優化多方視訊的效能和品質。

在一對一視訊情境，有明確的發送端與接收端，以 A、B 為例：

1. 裝置 A 發送 Offer SDP，並設定為本地 SDP（setLocalDescription）
2. 裝置 B 收到 Offer SDP，並設定為遠端 SDP（setRemoteDescription）
3. 裝置 B 發送 Answer SDP，並設定為本地 SDP（setLocalDescription）
4. 裝置 A 收到 Answer SDP，並設定為遠端 SDP（setRemoteDescription）

那麽在多方視訊情況，該怎麼定義發送端與接收端呢？

每個新加入聊天室的裝置，向已加入聊天室的所有裝置發送 Offer SDP，而其他裝置收到 Offer SDP 後，再向該裝置發送 Answer SDP，完成媒體協商並建立 P2P 連接。

以 A、B、C 為例：

1. 裝置 A 加入聊天室
2. 裝置 B 加入聊天室，向 A 發送 Offer SDP
3. 裝置 A 收到 B 的 Offer SDP，向 B 發送 Answer SDP
4. 裝置 B 收到 A 的 Answer SDP
5. 裝置 C 加入聊天室，向 A、B 發送 Offer SDP
6. 裝置 A、B 收到 C 的 Offer SDP，向 C 發送 Answer SDP
7. 裝置 C 收到 A、B 的 Answer SDP

> 試著思考看看：當有三台裝置進行視訊連線，總共會產生幾個 `RTCPeerConnection()` 物件呢？
> 

## **視訊實作**

#### **範例頁面：**[https://webrtc-socket.herokuapp.com/chat](https://webrtc-socket.herokuapp.com/chat)

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/7WD56Nw.png">
</div>

分為 **Server（Signaling Server）**以及 **Client（視訊聊天室）**進行

檔案結構如下：

```html
server.js
chat.html
chat.js
```

{% colorquote info %}
**重點說明：**
由於每個 P2P 連線的發送端與接收端都需要各別建立 `RTCPeerConnection()`，因此會使用 Socket.IO 為每個連線產生的唯一識別碼 `socket.id` 作為身份識別
{% endcolorquote %}

## **Server（server.js）**

### **套件安裝（NPM）**

- Node.js 框架 Express
- Websocket 套件 Socket.io

### **套件引入／啟動伺服器**

```jsx
const express = require('express');
const app = express();
const http = require('http');
const socket = require('socket.io');

const server = http.createServer(app).listen('3000');
```

### **連接 Client 頁面（轉換為絕對路徑 __dirname）**

```jsx
// html 路徑轉換
app.get('/chat', function(req, res) {
    res.sendfile(`${__dirname}/chat.html`);
});

// js 路徑轉換
app.get(/(.*)\.(jpg|gif|png|ico|css|js|txt)/i, function(req, res) {
    res.sendfile(`${__dirname}/${req.params[0]}.${req.params[1]}`);
});
```

### **加入 Socket.io**

{% colorquote info %}
**Socket.io API 說明：**
`io.in(room).emit('joined')`：向所有裝置發送訊息（包含自己）
`socket.to(room).emit('joined')`：向所有裝置發送訊息（不包含自己）
`socket.to(id).emit('joined')`：向特定裝置發送訊息
{% endcolorquote %}

```jsx
const io = socket(server);

io.on('connection', (socket) => {
    console.log('connection');

    // 裝置加入聊天室
    socket.on('join', (room) => {
        console.log('join');
        socket.join(room);
        // 取得加入聊天室裝置的 socket.id
        const members = Array.from(socket.adapter.rooms.get(room));
        // 向所有裝置告知有新裝置加入（包含自己）
        io.in(room).emit('joined', socket.id, members);
    });

    // 轉傳 Offer
    socket.on('offer', (room, desc, remoteId, localId) => {
        socket.to(localId).emit('offer', desc, remoteId);
    });

    // 轉傳 Answer
    socket.on('answer', (room, desc, remoteId, localId) => {
        socket.to(localId).emit('answer', desc, remoteId);
    });

    // 交換 ice candidate
    socket.on('ice_candidate', (room, data, remoteId, localId) => {
        socket.to(localId).emit('ice_candidate', data, remoteId);
    });

    // 裝置離開聊天室
    socket.on('disconnect_socket', () => {
        console.log('disconnect');
        socket.disconnect();
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
    - #remoteVideos 接收多個遠端媒體串流
- CDN WebRTC Adapter 以及 Socket.io-client
- 引入 chat.js

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <link rel="icon" type="image/x-icon" href="./favicon.ico">
    <meta name="viewport" content="width=device-width, user-scalable=yes, initial-scale=1, maximum-scale=1">
    <title>chat</title>
    <style lang="scss">
        .local-video {
            transform: scaleX(-1);
        }
    </style>
</head>
<body>
    <div class="container">
        <video muted="true" autoplay="true" playsinline="true" id="localVideo" class="local-video"></video>
        <div id="remoteVideos"></div>
        
        <div class="btn-group">
            <button type="button" id="enter">enter</button>
            <button type="button" id="leave">leave</button>
        </div>
    </div>
    <script src="https://webrtc.github.io/adapter/adapter-latest.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/socket.io-client@3.0.4/dist/socket.io.js"></script>
    <script src="chat.js" type="module" async></script>
</body>
</html>
```

### **Javascript（chat.js）**

#### **步驟拆分**

1. 開啟視窗時，先透過 `getUserMedia()` 取得本地媒體串流
2. 點擊進入聊天室按鈕：連接 Socket Server（Signaling Server），透過 Offer／Answer SDP 和遠端裝置交換 ICE 候選位址與 SDP 資訊，建立點對點連線
3. 點擊離開聊天室按鈕：移除事件監聽、關閉 RTCPeerConnection 連線、釋放記憶體，並傳送離開事件給 Socket Server

#### **定義常數與變數**

```jsx
const localVideo = document.querySelector('#localVideo')
const remoteVideos = document.querySelector('#remoteVideos')
const enterBtn = document.querySelector('#enter');
const leaveBtn = document.querySelector('#leave'); 
let peerConnectionList = {};
let socket;
let localStream;
const room = 'room1'; // 房間先預設為 room1
```

#### **開啟視窗時，先透過 getUserMedia 取得本地媒體串流**

1. 取得本地影音串流
2. Dom 設置本地串流
3. 傳出媒體串流

```jsx
const createStream = async () => {
    const constraints = { audio: true, video: true };
    // 1. 取得本地影音串流
    const stream = await navigator.mediaDevices.getUserMedia(constraints);
    // 2. Dom 設置本地媒體串流
    localVideo.srcObject = stream;
    // 3. 傳出媒體串流
    localStream = stream;
}

createStream(); // 畫面一開啟即呼叫
```

#### **點擊進入聊天室按鈕**

##### **事件觸發**

```jsx
enterBtn.addEventListener('click', socketConnect);
```

##### **連接 Socket Server（Signaling Server），透過 Offer／Answer SDP 和遠端裝置交換 ICE 候選位址與 SDP 資訊，建立點對點連線**

1. Socket Server 連線
2. 加入房間
3. 監聽有裝置加入房間：如果是自己，建立 RTCPeerConnection，設定本地 SDP（`setLocalDescription`），並發送 Offer SDP
4. 監聽收到 Offer SDP：建立 RTCPeerConnection，，設定遠端 SDP（`setRemoteDescription`）與本地 SDP（`setLocalDescription`），並發送 Answer SDP
5. 監聽收到 Answer SDP：設定遠端 SDP（`setRemoteDescription`）
6. 監聽收到 ICE 候選位址：使用 RTCIceCandidate 定義 ICE 候選位址

```jsx
const socketConnect = () => {
    // 1. Socket Server 連線
    socket = io('ws://localhost:3000');

    // 2. 加入房間
    socket.emit('join', room);

    // 3. 監聽有裝置加入房間
    socket.on('joined', (id, roomMembers) => {
        // 檢查 server 傳來的 socket.id 是否等於自己的 socket.id
        // 且聊天室裝置大於一，依序發送 Offer SDP
        if (id === socket.id && roomMembers.length > 1) {
            console.log('發送 offer');
            // 3-1. 發送 Offer SDP
            roomMembers.forEach(remoteId => {
                if (remoteId !== socket.id) {
                    setOfferSDP(remoteId);
                }
            });
        }
    });

    // 4. 監聽收到 Offer SDP
    socket.on('offer', async (desc, remoteId) => {
        console.log('收到 offer');
        // 4-1. 發送 Answer SDP
        console.log('發送 answer');
        await sendAnswerSDP(remoteId, desc);
    })

    // 5. 監聽收到 Answer SDP
    socket.on('answer', (desc, remoteId) => {
        console.log('收到 answer');
        // 設定遠端媒體串流
        peerConnectionList[remoteId].setRemoteDescription(desc);
    })

    // 6. 監聽收到 ICE 候選位址
    socket.on('ice_candidate', (data, remoteId) => {
        console.log('收到 ice_candidate');
        // RTCIceCandidate 定義 ICE 候選位址
        const candidate = new RTCIceCandidate({
            sdpMLineIndex: data.label,
            candidate: data.candidate,
        });
        // 加入候選位址
        peerConnectionList[remoteId].addIceCandidate(candidate);
    })
}
```

**3-1. 發送 Offer SDP**

1. 建立 RTCPeerConnection
2. 建立本地 SDP
3. 設定本地 SDP
4. 發送 Offer SDP

```jsx
const setOfferSDP = async (remoteId) => {
    // 1. 建立 RTCPeerConnection
    const peerConnection = await createPeerConnection(remoteId);

    const offerOptions = {
        offerToReceiveAudio: true, // 是否傳送聲音流給對方
        offerToReceiveVideo: true // 是否傳送影像流給對方
    };

    // 2. 建立本地 SDP
    const localSDP = await peerConnection.createOffer(offerOptions);

    // 3. 設定本地 SDP
    await peerConnection.setLocalDescription(localSDP);

    // 4. 發送 Offer SDP
    socket.emit('offer', room, peerConnection.localDescription, socket.id, remoteId);
}
```

**4-1. 發送 Answer SDP**

1. 建立 RTCPeerConnection
2. 設定遠端 SDP
3. 建立本地 SDP
4. 設定本地 SDP
5. 發送 Answer SDP

```jsx
const sendAnswerSDP = async (remoteId, desc) => {
    // 1. 建立 RTCPeerConnection
    const peerConnection = await createPeerConnection(remoteId);

    // 2. 設定遠端 SDP
    await peerConnection.setRemoteDescription(desc);

    const answerOptions = {
        offerToReceiveAudio: true, // 是否傳送聲音流給對方
        offerToReceiveVideo: true // 是否傳送影像流給對方
    };

    // 3. 建立本地 SDP
    const localSDP = await peerConnection.createAnswer(answerOptions);

    // 4. 設定本地 SDP
    await peerConnection.setLocalDescription(localSDP);

    // 5. 發送 Answer SDP
    socket.emit('answer', room, peerConnection.localDescription, socket.id, remoteId);
}
```

**3-1. 4-1. 建立 RTCPeerConnection**

1. 設定 iceServer
2. 建立 RTCPeerConnection
3. 增加本地媒體串流
4. 監聽找到本地的 ICE 候選位址
5. 監聽 ICE 連接狀態
6. 監聽遠端裝置的串流傳入
    {% colorquote info %}
    **video 標籤屬性：**
    **playsinline：**控制影片在行動裝置的播放行為，避免某些裝置（如 IOS）彈出視窗播放器
    **controls：**是否顯示播放控制器，在某些裝置（如 IOS Safari）設定為 false 畫面可能會被隱藏，可以先將屬性設為 true，接著移除該屬性：
    `video.setAttribute('controls', true)` 
    `video.removeAttribute('controls')`
    {% endcolorquote %}
    
7. 將 P2P 連線存入 peerConnectionList

```jsx
const createPeerConnection = async (remoteId) => {
    // 1. 設定 iceServer
    const configuration = {
        iceServers: [{
            urls: 'stun:stun.l.google.com:19302' // google 提供免費的 STUN server
        }]
    };

    // 2. 建立 RTCPeerConnection
    const peerConnection = new RTCPeerConnection(configuration);

    // 3. 增加本地媒體串流
    localStream.getTracks().forEach((track) => {
        peerConnection.addTrack(track, localStream);
    });

    // 4. 監聽找到本地的 ICE 候選位址
    peerConnection.onicecandidate = (e) => {
        console.log('找尋到 ICE 候選位址');
        if (e.candidate) {
            console.log('發送 ICE 候選位址');
            // 傳送 ICE 候選位址給遠端
            socket.emit('ice_candidate', room, {
                label: e.candidate.sdpMLineIndex,
                id: e.candidate.sdpMid,
                candidate: e.candidate.candidate,
            }, socket.id, remoteId);
        }
    };

    // 5. 監聽 ICE 連接狀態
    peerConnection.oniceconnectionstatechange = (e) => {
        if (e.target.iceConnectionState === 'disconnected') {
            console.log('有裝置斷線');

            // 移除事件監聽
            peerConnectionList[remoteId].onicecandidate = null;
            peerConnectionList[remoteId].onnegotiationneeded = null;
            peerConnectionList[remoteId].oniceconnectionstatechange = null;

            // 關閉 RTCPeerConnection 連線並釋放記憶體
            peerConnectionList[remoteId].close();
            delete peerConnectionList[remoteId];

            // 移除遠端 video
            document.getElementById(remoteId).remove();
        }
    };

    // 6. 監聽遠端裝置的串流傳入
    peerConnection.onaddstream = ({ stream }) => {
        console.log('監聽到串流');
        const video = document.createElement('video');
        video.srcObject = stream;
        video.setAttribute('controls', true);
        video.setAttribute('playsinline', true);
        video.setAttribute('autoplay', true);
        video.setAttribute('muted', true);
        video.setAttribute('volume', 0);
        video.removeAttribute('controls');
        video.classList.add('remote-video');
        video.id = remoteId;
        remoteVideos.append(video);
    };

    // 7. 將 P2P 連線存入 peerConnectionList
    peerConnectionList[remoteId] = peerConnection;

    return peerConnection;
}
```

#### **點擊離開聊天室按鈕**

##### **事件觸發**

```jsx
leaveBtn.addEventListener('click', leave);
```

##### **離開聊天室**

1. 移除事件監聽
2. 關閉 RTCPeerConnection 連線
3. 釋放記憶體
4. 移除遠端 video
5. 傳遞離開聊天室事件給 Socket Server

```jsx
const leave = () => {
    console.log('離開聊天室');

    // 檢查是否有連線
    if (Object.keys(peerConnectionList).length) {
        Object.keys(peerConnectionList).forEach(key => {
            // 1. 移除事件監聽
            peerConnectionList[key].onicecandidate = null;
            peerConnectionList[key].onnegotiationneeded = null;
            peerConnectionList[key].oniceconnectionstatechange = null;
            // 2. 關閉 RTCPeerConnection 連線
            peerConnectionList[key].close();
        })
        // 3. 釋放記憶體
        peerConnectionList = {};

        // 4. 移除遠端 video
        remoteVideos.innerHTML = null;
    }

    // 5. 傳遞離開聊天室事件
    socket.emit('disconnect_socket');
    socket = null;
}
```

這樣就大功告成囉，上述說明有試著簡化程式碼，避免細節繁瑣難以理解

> 試著優化看看：
    1. enter button 跟 leave button 加入 disabled 屬性，避免重複操作
    2. 將 socket.id 做為識別碼，加入聊天視窗（也更容易除錯！）
    3. 不同裝置的螢幕寬高比不同，利用 css 優化，避免畫面破版
    4. async function 加入 try / catch 捕捉錯誤情境
> 

#### **範例頁面：**[https://webrtc-socket.herokuapp.com/chat](https://webrtc-socket.herokuapp.com/chat)

---

參考資源：

[https://github.com/muaz-khan/RTCMultiConnection/blob/master/dist/RTCMultiConnection.js](https://github.com/muaz-khan/RTCMultiConnection/blob/master/dist/RTCMultiConnection.js)

[https://iter01.com/489722.html](https://iter01.com/489722.html)

[https://socket.io/docs/v3/rooms/](https://socket.io/docs/v3/rooms/)