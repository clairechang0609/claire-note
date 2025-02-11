---
title: Three.js 3D Library (3) UnrealBloomPass 後製發光效果
date: 2023-04-15
tags: [ three.js ]
category: Javascript
description: 接續前一篇介紹如何透過 Three.js 製作出立體行星，本篇說明如何後製出發光效果，讓畫面更為吸睛
image: https://i.imgur.com/GxGd20A.png
---

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
    <img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/GxGd20A.png">
</div>

上一篇 [Three.js 3D Library (2) 模型實作](https://clairechang.tw/2023/04/11/javascript/js-threejs-pattren/) 我們建立了行星動畫，接下來試著加入發光效果，讓畫面更吸睛～

## **UnrealBloomPass 發光效果**

Three.js 提供了強大的渲染引擎和豐富的功能，其中後製（Post-processing）是在渲染到螢幕前對畫面進行一些處理，再將畫面渲染到螢幕上，以達到特定的視覺效果，**UnrealBloomPass** 是其中一個後製功能，可以打造出逼真的發光效果。

<!-- more -->

使用 **UnrealBloomPass** 通常會跟 **RenderPass** 以及 **EffectComposer（合成器）**搭配使用，到這裡先釐清一下**『渲染器渲染』**跟**『合成器渲染』**的差異：

#### **渲染器渲染（WebGLRenderer）**

- Three.js 中最基本的場景渲染方式
- 直接調用 WebGLRenderer 的 render 方法，將場景中的物體渲染到渲染器的畫面中
- 每次調用時，只會產生一個渲染畫面
- 不支援後製效果，結果直接顯示在畫面中

```jsx
const renderer = new THREE.WebGLRenderer();

// ...

renderer.render(scene, camera); // 渲染器渲染
```

#### **合成器渲染（EffectComposer）**

- 需要搭配 Three.js 中的 **EffectComposer（合成器）**跟 **RenderPass 使用**
- 調用 EffectComposer 的 render 方法，將場景渲染到合成器的畫面中
- 可以在合成器中增加多個後製效果，EX：色彩校正、模糊、光暈…
- 每次調用時，合成器會依序應用後製效果，生成最終的渲染畫面

```jsx
const renderer = new THREE.WebGLRenderer();
const composer = new EffectComposer(renderer); // 建立合成器
const renderScene = new RenderPass(scene, camera);
composer.addPass = new EffectComposer(renderScene); // 將 RenderPass 加入合成器

// ...

composer.render(); // 合成器渲染
```

---

## **基礎應用**

將整個場景內的物體都加上 UnrealBloomPass 效果，接下來用一個簡單的範例來說明，步驟如下：

1. **建立場景**
2. **建立相機**
3. **建立渲染器**
4. **建立物體**
5. <span style="color: #DD4A68">**建立 UnrealBloomPass 物件**</span>
6. <span style="color: #DD4A68">**建立合成器**</span>
7. <span style="color: #DD4A68">**將 UnrealBloomPass 加入合成器**</span>
8. **渲染場景**

```jsx
// 1. 建立場景
const scene = new THREE.Scene();

// 2. 建立相機
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.set(0, 0, 10);
camera.lookAt(scene.position) // 相機焦點

// 3. 建立渲染器
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.getElementById('container').appendChild(renderer.domElement);

// 4. 建立物體
const geometry = new THREE.BoxGeometry();
const material = new THREE.MeshBasicMaterial({ color: 0xffffff });
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

// 5. 建立 UnrealBloomPass 物件
const bloomPass = new UnrealBloomPass(new THREE.Vector2(window.innerWidth, window.innerHeight), 1.5, 0.4, 0.85);

// 6. 建立合成器
const composer = new EffectComposer(renderer);
composer.setSize(window.innerWidth, window.innerHeight);

// 7. 將 UnrealBloomPass 加入合成器
const renderScene = new RenderPass(scene, camera);
composer.addPass(renderScene);
composer.addPass(bloomPass);

// 8. 渲染場景
function animate() {
    requestAnimationFrame(animate);
    // 將場景渲染到合成器
    composer.render();
}
animate();
```

效果如下：

![](https://i.imgur.com/YAeBjZG.png)

---

## **進階應用-部分物體渲染**

以上一篇 [Three.js 3D Library (2) 模型實作](https://clairechang.tw/2023/04/11/javascript/js-threejs-pattren/) 接續實作，使用 **UnrealBloomPass** 搭配 **Shader（著色器）** 來實現部分區塊發光效果

那麼就開始實作吧！首先調整步驟如下：

1. **建立場景**
2. **建立相機**
3. **建立渲染器**
4. **建立物體**
5. **建立光源**
6. <span style="color: #DD4A68">**建立 UnrealBloomPass 物件**</span>
7. <span style="color: #DD4A68">**建立 bloom 合成器**</span>
8. <span style="color: #DD4A68">**將 UnrealBloomPass 加入合成器**</span>
9. <span style="color: #DD4A68">**建立 ShaderPass，並設定自定義著色器**</span>
10. <span style="color: #DD4A68">**建立著色器的合成器**</span>
11. <span style="color: #DD4A68">**將 ShaderPass 加入合成器**</span>
12. <span style="color: #DD4A68">**合成器渲染場景**</span>
13. <span style="color: #DD4A68">**監聽螢幕縮放 RWD**</span>

## **建立場景/相機/渲染器/物體/光源**

前一篇已介紹過，這裡簡單帶過

### **場景**

背景色必須調整為黑色，否則會受發光效果影響

```jsx
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x000000); // 背景色
```

### **相機**

```jsx
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.set(120, 120, 120); // 相機位置
camera.lookAt(scene.position); // 相機焦點
```

### **渲染器**

```jsx
const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.getElementById('container').appendChild(renderer.domElement);
```

### **物體**

- 行星體
    - 球型體
    - **行星環（後續加入發光效果）**
- 群星背景

{% colorquote info %}
了呈現行星環發光效果，將[圖層（Layer）](https://threejs.org/docs/index.html?q=layer#api/en/core/Layers)設為 1（預設為 0）：
`ring.layers.enable(1)`
{% endcolorquote %}

```jsx
// 行星體-球形體
const sphereGeometry = new THREE.IcosahedronGeometry(20, 1);
const sphereMaterial = new THREE.MeshPhongMaterial({
    color: 0xffffff,
    emissive: 0x000000,
    specular: 0xffffff,
    shininess: 100,
    flatShading: true
});
const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);

// 行星體-行星環
const RingGeometry = new THREE.RingGeometry(42, 28, 30, 30);
// 調整為 MeshBasicMaterial，只設定隨機顏色，後續會加入發光效果
const color = new THREE.Color().setRGB(Math.random(), Math.random(), Math.random());
const RingMaterial = new THREE.MeshBasicMaterial({ color: color });
const ring = new THREE.Mesh(RingGeometry, RingMaterial);
ring.layers.enable(1);
ring.rotation.x = THREE.MathUtils.degToRad(90);

// 組合行星體
const planet = new THREE.Group();
planet.add(sphere, ring);
planet.rotation.x = THREE.MathUtils.degToRad(-30);
planet.rotation.y = THREE.MathUtils.degToRad(-10);
scene.add(planet);

// 群星背景
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
const particleTexture = new THREE.TextureLoader().load('https://i.imgur.com/9TwBBHH.png');
const material = new THREE.PointsMaterial({ size: 5, sizeAttenuation: true, map: particleTexture, transparent: true, alphaTest: 0.5, color: 0xf3f3af });
const particles = new THREE.Points(geometry, material);
scene.add(particles);
```

### **光源**

```jsx
// 環境光
const ambientLight = new THREE.AmbientLight(0x222222);
// 點光源
const pointLight = new THREE.PointLight(0x777777, 1, 0);
pointLight.position.set(100, 100, 0);
// 聚光燈
const spotLight = new THREE.SpotLight(0xffffff, 3, 150, Math.PI / 15, 1, 1);
spotLight.position.set(50, 100, -80);
spotLight.castShadow = true;
spotLight.shadow.mapSize.width = 1024;
spotLight.shadow.mapSize.height = 1024;
spotLight.shadow.camera.near = 10;
spotLight.shadow.camera.far = 200;
spotLight.shadow.focus = 1;
scene.add(ambientLight, pointLight, spotLight);
```

## **建立 UnrealBloomPass 物件**

```jsx
const bloomPass = new UnrealBloomPass(
    new THREE.Vector2(window.innerWidth, window.innerHeight),
    3, // 強度
    0.5, // 射散
    0 // 縮放
);
```

## **建立 bloom 合成器**

`renderToScreen` 必須先設定為 false，待 ShaderPass 執行 render 時再一起渲染

```jsx
const bloomComposer = new EffectComposer(renderer);
bloomComposer.renderToScreen = false;
```

{% colorquote info %}
`renderToScreen` 是後製效果的選項之一，設定為 true 時，渲染結果將直接渲染到螢幕上；設定為 false 時，則渲染到目標紋理上，預設為 true
{% endcolorquote %}

## **將 UnrealBloomPass 加入合成器**

將前面新增的 UnrealBloomPass 物件跟 RenderPass 加入合成器

```jsx
bloomComposer.addPass(new RenderPass(scene, camera));
bloomComposer.addPass(bloomPass);
```

## **建立 ShaderPass，並設定自定義著色器**

ShaderPass 為渲染過程中應用自定義著色器效果的後製通道（Post-processing Pass）。

步驟如下：

1. 定義 ShaderMaterial
2. 將 ShaderMaterial 加入 ShaderPass

ShaderMaterial 可以定義著色器材質類型，用 GLSL（OpenGL Shading Language Language）語言編寫 Uniforms**、**頂點著色器（Vertex Shader）以及片段著色器（Fragment Shader）

#### **Uniforms**

用於將外部變數（例如紋理、時間等）傳遞給 Shader

#### **Shader 著色器**

WebGL 組成中包含著色器，基本上包含**頂點著色器（Vertex Shader）**及**片段著色器（Fragment Shader）**

- **頂點著色器（Vertex Shader）**

負責場景中頂點的運算，將頂點從模型空間（Model Space）轉換到螢幕空間（Screen Space）

- **片段著色器（Fragment Shader）**

對頂點處理生成的片元（像素）進行處理，例如材質貼圖、光照計算、透明度計算等，並生成最終的像素顏色

```jsx
const shaderMaterial = new THREE.ShaderMaterial({
    uniforms: {
        baseTexture: { value: null }, // base 紋理
        bloomTexture: { value: bloomComposer.renderTarget2.texture } // bloom 紋理
    },
    vertexShader: `
        varying vec2 vUv;
        void main() {
            vUv = uv;
            gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
    `,
    fragmentShader: `
        uniform sampler2D baseTexture;
        uniform sampler2D bloomTexture;
        varying vec2 vUv; // 紋理座標
        void main() {
            // 輸出最終顏色
            gl_FragColor = texture2D(baseTexture, vUv) + vec4(1.0) * texture2D(bloomTexture, vUv);
        }
    `
});
const finalPass = new ShaderPass(shaderMaterial, 'baseTexture');
```

## **建立 Shader 合成器**

```jsx
const finalComposer = new EffectComposer(renderer);
```

## **將 ShaderPass 加入合成器**

```jsx
finalComposer.addPass(new RenderPass(scene, camera));
finalComposer.addPass(finalPass);
```

## **合成器渲染場景**

建立一個圖層物件，並將圖層設定為 1（預設為 0），後面用來檢核使用

```jsx
const bloomLayer = new THREE.Layers();
bloomLayer.set(1);
```

利用 `bloomLayer` 判斷圖層是否為 1，前面行星環圖層設定為 1（`ring.layers.enable(1)`），因此除了行星環之外的物體，都設定為黑色（`color: 0x000000`）

```jsx
const materials = [];

const darkenMaterial = obj => {
    if (!bloomLayer.test(obj.layers)) {
        materials[obj.uuid] = obj.material
        obj.material = new THREE.MeshBasicMaterial({ color: 0x000000 })
    }
};
```

將暫時設定為黑色的物體還原為初始值

```jsx
const restoreMaterial = obj => {
    if (materials[obj.uuid]) {
        obj.material = materials[obj.uuid]
        delete materials[obj.uuid]
    }
};
```

執行渲染，步驟如下：

- `scene.traverse(darkenMaterial)`：[scene.traverse](https://threejs.org/docs/#api/en/core/Object3D.traverse) 是一個 callback 函式，第一個參數帶入物體本身，執行 `darkenMaterial()` 將行星環外的物體設定為黑色
- `bloomComposer.render()`：發光效果合成器渲染，因前一步驟將行星環外的物體設定為黑色，因此只有行星環會呈現發光效果，前面提到 `bloomComposer.renderToScreen` 設定為 false，將效果先渲染到紋理上
- `scene.traverse(restoreMaterial)`：將暫時設定為黑色的物體還原為初始值
- `finalComposer.render()`：ShaderPass 合成器渲染，將效果渲染到螢幕上（包含發光效果）

```jsx
const animate = () => {
    requestAnimationFrame(animate);
    sphere.rotation.y += 0.01;
        
    scene.traverse(darkenMaterial);
    bloomComposer.render();
    scene.traverse(restoreMaterial);
    finalComposer.render();
};
animate();
```

## **監聽螢幕縮放 RWD**

將 **bloomComposer** 跟 **finalComposer** 加入監聽

```jsx
window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
    bloomComposer.setSize(window.innerWidth, window.innerHeight)
    finalComposer.setSize(window.innerWidth, window.innerHeight)
});
```

### **完整範例程式碼**

<iframe height="470" style="width: 100%;" scrolling="no" title="Three.js-UnrealBloomPass" src="https://codepen.io/claire-chang-the-bashful/embed/MWPyJYz?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/claire-chang-the-bashful/pen/MWPyJYz">
  Three.js-UnrealBloomPass</a> by Claire Chang (<a href="https://codepen.io/claire-chang-the-bashful">@claire-chang-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

---

參考資源：

[https://ithelp.ithome.com.tw/users/20107572/ironman/1782](https://ithelp.ithome.com.tw/users/20107572/ironman/1782)

[https://threejs.org/](https://threejs.org/)

[https://threejs.org/examples/?q=bloom#webgl_postprocessing_unreal_bloom_selective](https://threejs.org/examples/?q=bloom#webgl_postprocessing_unreal_bloom_selective)