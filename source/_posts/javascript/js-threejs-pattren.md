---
title: Three.js 3D Library (2) 模型實作
date: 2023-04-11
tags: [ three.js ]
category: Javascript
description: 本篇將介紹如何透過 Three.js 製作出立體行星自轉效果，透過步驟拆解，能更快速理解每個方法與流程
image: https://i.imgur.com/Vhd77Pi.png
---

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
    <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/Vhd77Pi.png">
</div>

依照上一篇 [Three.js 3D Library (1) 基本介紹](https://clairechang.tw/2023/04/06/javascript/js-threejs-intro/) 介紹了製造 3D 圖形的基本元素，接下來進行實作吧！

### **座標系統**

開始實作之前，必須先了解 Three.js 使用 **右手座標系統 Right-Hand Coordinate** 來定義 3D 空間：

**X軸：**向右為正方向，**Y軸：**向上為正方向，**Z軸：**向外為正方向

<!-- more -->

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img style="width: 100%; max-width: 400px;" src="https://i.imgur.com/up4duGF.png">
</div>

如果對此概念不熟悉也沒關係，Three.js 提供座標輔助：

```jsx
const axesHelper = new THREE.AxesHelper(10); // 自訂座標大小
scene.add(axesHelper);
```

實作行星自轉畫面，步驟如下：

1. **建立場景**
2. **建立相機**
3. **建立渲染器**
4. **建立物體**
5. **建立光源**
6. **渲染場景**

以下為進階功能：

1. **加入軌道控制器**
2. **打造 RWD 響應式畫面**

## **建立場景**

設定**背景底色**

```jsx
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x000000); // 背景色
```

## **建立相機**

使用**透視相機**，設定**相機位置**跟**相機焦點**

```jsx
const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);
camera.position.set(120, 120, 120); // 相機位置
camera.lookAt(scene.position); // 相機焦點
```

## **建立渲染器**

```jsx
const renderer = new THREE.WebGLRenderer({
    antialias: true,
    alpha: true
});
renderer.setSize(window.innerWidth, window.innerHeight);
// 將canvas內容加入DOM
document.getElementById('container').appendChild(renderer.domElement);
```

## **建立物體**

結構如下：

- 行星體（網格物體）
    - 球型體
    - 行星環
- 群星背景（粒子物體）

{% colorquote info %}
再次複習一下建立**網格（Mesh）**或**粒子（Points）**物體順序：
**建立幾何體 → 建立材質 → 建立物體 → 將物體加入場景**
{% endcolorquote %}

### **行星體（網格物體）**

分為**球型體**跟行**星環**，軸心旋轉（X軸-30度、Y軸-10度）

<div style="display: flex; justify-content: center; margin: 0;">
    <img style="width: 100%; max-width: 600px;" src="https://i.imgur.com/BB5mKxf.png">
</div>

#### **輔助座標**

首先建立輔助座標，可以協助我們快速理解3D空間

```jsx
const axesHelper = new THREE.AxesHelper(50);
axesHelper.rotation.x = THREE.MathUtils.degToRad(-30); // X軸旋轉-30度
axesHelper.rotation.y = THREE.MathUtils.degToRad(-10); // Y軸旋轉-10度
scene.add(axesHelper);
```

使用 `THREE.MathUtils.degToRad()` 運算函式，將角度（degrees）轉換成弧度（radians）

#### **球型體**

使用 `IcosahedronGeometry(radius, detail)` 建立多面體，詳細說明見[官方文件](https://threejs.org/docs/?q=IcosahedronGeometry#api/en/geometries/IcosahedronGeometry)

```jsx
// 建立幾何體
const sphereGeometry = new THREE.IcosahedronGeometry(20, 1);
// 建立材質
const sphereMaterial = new THREE.MeshPhongMaterial({
    color: 0xffffff,
    emissive: 0x000000,
    specular: 0xffffff,
    shininess: 100,
    flatShading: true
});
// 建立網格
const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
```

#### **行星環**

使用 `RingGeometry()` 建立圓環幾何體，詳細說明見[官方文件](https://threejs.org/docs/?q=RingGeometry#api/en/geometries/RingGeometry)

```jsx
// 建立幾何體
const RingGeometry = new THREE.RingGeometry(42, 28, 30, 30);
// 建立材質
const RingMaterial = new THREE.MeshPhongMaterial({
    color: 0xffffff,
    emissive: 0x000000,
    specular: 0xffffff,
    shininess: 100,
    flatShading: true,
    transparent: true,
    opacity: 0.7
});
// 建立網格物體
const ring = new THREE.Mesh(RingGeometry, RingMaterial);
// X軸旋轉90度，讓圓環呈現水平排列
ring.rotation.x = THREE.MathUtils.degToRad(90);
```

#### **組合行星體**

使用 `Group()` 建立群組，軸心旋轉（同前面說明：X軸-30度、Y軸-10度）

```jsx
const planet = new THREE.Group();
planet.add(sphere, ring); // 加入球形體&行星環
planet.rotation.x = THREE.MathUtils.degToRad(-30);
planet.rotation.y = THREE.MathUtils.degToRad(-10);

// 將物體加入場景
scene.add(planet);
```

### **群星背景（粒子物體）**

![](https://i.imgur.com/6XrmQ7m.png)

使用 `BufferGeometry()` 建立幾何體，可以減少 GPU 的消耗，詳細說明見[官方文件](https://threejs.org/docs/#api/en/core/BufferGeometry)

```jsx
// 建立幾何體
const geometry = new THREE.BufferGeometry();
const particleAmount = 2000; // 群星數
const vertices = [];
for (let i = 0; i < particleAmount; i++) {
    const x = 2000 * Math.random() - 1000;
    const y = 2000 * Math.random() - 1000;
    const z = 2000 * Math.random() - 1000;
    vertices.push(x, y, z);
}
geometry.setAttribute('position', new THREE.Float32BufferAttribute(vertices, 3));

// 建立材質
// 載入圖片來建立紋理
const particleTexture = new THREE.TextureLoader().load('https://i.imgur.com/9TwBBHH.png');
const material = new THREE.PointsMaterial({
    size: 5,
    sizeAttenuation: true,
    map: particleTexture,
    transparent: true,
    alphaTest: 0.5,
    color: 0xf3f3af
});

// 建立粒子物體
const particles = new THREE.Points(geometry, material);

// 將物體加入場景
scene.add(particles);
```

## **建立光源**

依序建立三種光源，讓光線看起來更自然：

- 環境光 AmbientLight
- 點光源 PointLight
- 聚光燈 SpotLight

### **環境光**

```jsx
const ambientLight = new THREE.AmbientLight(0x222222);
```

### **點光源**

```jsx
const pointLight = new THREE.PointLight(0x777777, 1, 0);
pointLight.position.set(100, 100, 0);
```

### **聚光燈**

```jsx
const spotLight = new THREE.SpotLight(0xffffff, 3, 150, Math.PI / 6, 1, 1);
spotLight.position.set(50, 100, -80);
spotLight.castShadow = true;
spotLight.shadow.mapSize.width = 1024;
spotLight.shadow.mapSize.height = 1024;
spotLight.shadow.camera.near = 10;
spotLight.shadow.camera.far = 200;
spotLight.shadow.focus = 1;
```

聚光燈輔助可以更明確地看到光線位置

```jsx
const lightHelper = new THREE.SpotLightHelper(spotLight);
scene.add(lightHelper);
```

最後將光源加入場景

```jsx
scene.add(ambientLight, pointLight, spotLight);
```

## **渲染場景**

搭配前一篇說明過的 [動畫循環渲染 Render Loop](https://clairechang.tw/2023/04/06/javascript/js-threejs-intro/#2-%E5%8B%95%E7%95%AB%E5%BE%AA%E7%92%B0%E6%B8%B2%E6%9F%93-Render-Loop) 方法

```jsx
function animate() {
    requestAnimationFrame(animate);
    // 星球體Y軸旋轉，創造自轉效果
    planet.rotation.y += 0.01;
    // 將場景渲染到渲染器
    renderer.render(scene, camera);
}
animate();
```

到這裡畫面就完成囉！👏👏👏

接著說明進階的功能

## **加入軌道控制器**

**軌道控制器 OrbitControls** 讓使用者可以透過滑鼠或是觸控面板來縮放和移動相機，改變場景中的視角和位置，軌道圍繞著前面提到的 `camera.lookAt()` 相機焦點

軌道控制器也可以啟用自動旋轉功能，詳細請見[官方文件](https://threejs.org/docs/#examples/en/controls/OrbitControls.update)

```jsx
const control = new OrbitControls(camera, renderer.domElement);
```

## **打造 RWD 響應式畫面**

監聽螢幕尺寸變化，讓畫面自適應

```jsx
window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
});
```

### **完整範例程式碼**

<iframe height="470" style="width: 100%;" scrolling="no" title="Three.js-Planet Animation" src="https://codepen.io/claire-chang-the-bashful/embed/eYPNdJy?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/claire-chang-the-bashful/pen/eYPNdJy">
  Three.js-Planet Animation</a> by Claire Chang (<a href="https://codepen.io/claire-chang-the-bashful">@claire-chang-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

---

參考資源：

[https://ithelp.ithome.com.tw/users/20107572/ironman/1782](https://ithelp.ithome.com.tw/users/20107572/ironman/1782)

[https://threejs.org/](https://threejs.org/)

[https://medium.com/小彥彥的前端五四三/threejs-內建幾何模型-二-3df675ba8315](https://medium.com/%E5%B0%8F%E5%BD%A5%E5%BD%A5%E7%9A%84%E5%89%8D%E7%AB%AF%E4%BA%94%E5%9B%9B%E4%B8%89/threejs-%E5%85%A7%E5%BB%BA%E5%B9%BE%E4%BD%95%E6%A8%A1%E5%9E%8B-%E4%BA%8C-3df675ba8315)