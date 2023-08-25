---
title: Three.js 3D Library (1) 基本介紹
date: 2023-04-06
tags: [ three.js ]
category: Javascript
description: Three.js 是一個基於 WebGL 開發的 JavaScript 函式庫，可在網頁上建立 3D 圖形。Three.js 提供比 WebGL 更簡單、高效且功能豐富的方式來創建互動性的 3D 場景，並支援多種渲染器，包括 WebGL、Canvas 和 SVG
image: https://imgur.com/5QCBIK5.png
---

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
    <img style="width: 100%; max-width: 550px;" src="https://imgur.com/5QCBIK5.png">
</div>

[Three.js](https://threejs.org/) 是一個基於 WebGL 開發的 JavaScript 函式庫，可在網頁上建立 3D 圖形。Three.js 提供比 WebGL 更簡單、高效且功能豐富的方式來創建互動性的 3D 場景，並支援多種渲染器，包括 WebGL、Canvas 和 SVG

### **套件安裝**

```bash
npm i three
```

```jsx
import * as THREE from 'three'
```

<!-- more -->

## **基本元素**

- 場景 Scene
- 相機 Camera
- 物體 Objects
- 光源 Light
- 渲染器 Renderer

## **場景 Scene**

場景是一個物件容器，用來存放所有的物體、光源和相機等元素。網頁上的 3D 圖形都是由場景中的元素所組成，一個網頁可以有多個場景

```jsx
const scene = new THREE.Scene()
scene.background = new THREE.Color(0xffffff) // 設定背景設為白底
```

建立場景後，可以使用 `scene.add(object)` 方法將物體加入到場景，也可以使用 `scene.remove(object)` 方法將物體從場景移除

## **相機 Camera**

相機決定了在場景中看到的位置和角度。Three.js 支援多種類型的相機，包括：

- PerspectiveCamera 透視相機
- OrthographicCamera 正交相機

### **1. PerspectiveCamera 透視相機**

透視相機用於模擬人眼觀察物體的效果，建立透視相機基本參數：

- FOV (Field of View)：視野角度，單位為度數，通常設定為 45 度
- Aspect Ratio：畫面的寬高比，通常設定為 `window.innerWidth / window.innerHeight`
- Near：相機到近端的距離，通常設定為 0.1
- Far：相機到遠端的距離，通常設定為 1000

```jsx
const camera = new THREE.PerspectiveCamera(
    45,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);
```

### **2. OrthographicCamera 正交相機**

正交相機無論物體距離遠近，看起來大小都相同，適合用於模擬透視效果不明顯的場景，例如平面地圖。建立正交相機基本參數：

- Left：畫面左邊界的位置
- Right：畫面右邊界的位置
- Top：畫面上邊界的位置
- Bottom：畫面下邊界的位置
- Near：相機到近端的距離
- Far：相機到遠端的距離

```jsx
const camera = new THREE.OrthographicCamera(
    -window.innerWidth / 2,
    window.innerWidth / 2,
    window.innerHeight / 2,
    -window.innerHeight / 2,
    0.1,
    1000
);
```

建立相機後，可以使用 `camera.position.set(x, y, z)` 方法設定相機的位置，也可以使用 `camera.lookAt(object)` 方法讓相機對準另一個物體。

## **物體 Objects**

物體是在場景中顯示的 3D 圖形元素，有以下幾種：

- Mesh 網格：由幾何形狀和材質組成的物體
- Points：由點和材質組成的物體，通常用於顯示粒子效果
- Line：由線段和材質組成的物體，通常用於顯示簡單的線條效果
- Sprite：只有 2D 圖片的物體，可以隨著相機位置旋轉和縮放

### **1. 網格 Mesh**

Three.js 中最常用的物體類別，由 **幾何體 Geometry** 和 **材質 Material** 組合而成

- **幾何體 Geometry：**定義物體的形狀、大小和位置
- **材質 Material：**定義物體的材質和外觀，可以設定顏色、紋理、透明度…

```jsx
// 建立幾何體
const geometry = new THREE.SphereGeometry(1, 32, 32);

// 建立材質
const material = new THREE.MeshBasicMaterial({ color: 0xffffff });

// 建立網格
const sphere = new THREE.Mesh(geometry, material);

// 將網格加入到場景中
scene.add(sphere);
```

### **2. 粒子 Points**

通常用於顯示粒子效果

```jsx
// 建立幾何體
const geometry = new THREE.BufferGeometry();
const vertices = [];
for (let i = 0; i < 1000; i++) {
    const x = 2000 * Math.random() - 1000;
    const y = 2000 * Math.random() - 1000;
    const z = 2000 * Math.random() - 1000;
    vertices.push(x, y, z);
}
geometry.setAttribute('position', new THREE.Float32BufferAttribute(vertices, 3));

// 建立材質
const material = new THREE.PointsMaterial({size: 35, sizeAttenuation: true, alphaTest: 0.5, transparent: true});

// 建立粒子
const points = new THREE.Points(geometry, material);

// 將粒子加入到場景中
scene.add(points);
```

### **3. 線條 Line**

由一些連接的線段和材質組成的物體，通常用於顯示簡單的線條效果

以下範例為一個由三個點組成的線段

```jsx
// 建立幾何體
const geometry = new THREE.BufferGeometry();
const vertices = new Float32Array([
    -1, 0, 0,
    0, 1, 0,
    1, 0, 0
]);
geometry.setAttribute('position', new THREE.BufferAttribute(vertices, 3));

// 建立材質
const material = new THREE.LineBasicMaterial({ color: 0xffffff });

// 建立線條
const line = new THREE.Line(geometry, material);

// 將線條加入到場景中
scene.add(line);
```

## **光源 Light**

光源用於模擬光照效果，可以讓場景中的物體更加真實。有些材質不受光源影響，例如`MeshBasicMaterial` 跟 `MeshNormalMaterial` 是沒有陰影的材質。如果需要使用光源，可以嘗試使用 `MeshPhongMaterial` 、 `MeshStandardMaterial` 或 `MeshLambertMaterial`

光源有幾下幾種：

- 方向光 DirectionalLight：模擬太陽光射向物體的效果，用來產生陰影效果
- 環境光 AmbientLight：模擬室內的照明效果，對場景中物體的亮度沒有方向性的影響
- 點光源 PointLight：模擬燈泡發出的光線，可以有位置的光源
- 聚光燈 SpotLight：模擬射燈效果，可以有方向和位置的光源
- 半球光 HemisphereLight：模擬天空環境光的光源，為球形光源

### **1. 方向光 DirectionalLight**

為平行光，只有方向沒有位置，類似太陽光，來自於一個方向，可以用來模擬太陽光射向物體的效果

```jsx
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(0, 1, 0);
scene.add(light);
```

### **2. 環境光 AmbientLight**

一種均勻的光源，對場景中物體的亮度沒有方向性的影響，用來模擬室內的照明效果

```jsx
const light = new THREE.AmbientLight(0x404040);
scene.add(light);
```

### **3. 點光源 PointLight**

一種有位置的光源，從一個點向四面八方發射光線，用來模擬燈泡光

```jsx
const light = new THREE.PointLight(0xffffff, 1, 100); // 光源顏色、強度和距離限制
light.position.set(0, 0, 50);
scene.add(light);
```

### **4. 聚光燈 SpotLight**

為有方向和位置的光源，類似手電筒，可以聚焦光線，用來模擬射燈效果

```jsx
const light = new THREE.SpotLight(0xffffff, 1, 100, Math.PI / 4); // 光源顏色、強度、距離限制和照射角度
light.position.set(0, 0, 50);
scene.add(light);
```

### **5. 半球光 HemisphereLight**

一種模擬天空環境光的光源，為球形光源，用來模擬天空和地面反射光互相作用的效果

```jsx
const hemisphereLight = new THREE.HemisphereLight(0xffffbb, 0x080820, 1); // 天空和地面的顏色以及強度
scene.add(hemisphereLight);
```

## **渲染器 Renderer**

前面提到的幾種元素，最後會透過渲染器將場景中的物件轉換成 2D 圖像，顯示在網頁上，通常使用 **WebGLRenderer 渲染器**，WebGLRenderer 使用 WebGL 進行渲染，可以讓網頁上的 3D 圖形更加流暢和精細

```jsx
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);
```

使用 `renderer.setSize(width, height)` 方法設定渲染器的寬度和高度，使用 `renderer.domElement` 屬性取得渲染器的 DOM 元素，並加入網頁 DOM 元素內

### **1. 場景渲染**

加入場景跟相機進行渲染

```jsx
renderer.render(scene, camera);
```

### **2. 動畫循環渲染 Render Loop**

如果想加入動態效果，可以搭配瀏覽器原生方法 `requestAnimationFrame`，自動建立每秒 60 次的循環渲染，`requestAnimationFrame` 的優點為：

1. 切換瀏覽器分頁時會自動暫停，可以提升網頁效能
2. 流暢的動畫效果

```jsx
...
const cube = new THREE.Mesh( geometry, material );

function animate() {
    requestAnimationFrame(animate);

    cube.rotation.x += 0.01;
    cube.rotation.y += 0.01;

    renderer.render(scene, camera);
}
animate();
```

以上基本概念理解後，下一篇將組合以上元素，進行**『模型實作』**囉！

---

參考資源：

[https://ithelp.ithome.com.tw/users/20107572/ironman/1782](https://ithelp.ithome.com.tw/users/20107572/ironman/1782)

[https://threejs.org/](https://threejs.org/)