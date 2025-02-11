---
title: CSS 製作固定比例寬高區塊（Aspect Ratio / Height Based on Width）
date: 2023-02-03 
tags: [ css ]
category: CSS
description: 利用 CSS Aspect Ratio 製作固定寬高比區塊，不管螢幕尺寸如何縮放，圖片都能保持在固定比例，達到響應式版面設計
image: https://i.imgur.com/iOgfFHV.png
---

<img style="width: 100%; max-width: 100%;" src="https://i.imgur.com/iOgfFHV.png">

在開發響應式網站（Responsive Web Design）常會需要製作等比例寬高圖片，不管螢幕尺寸如何縮放，圖片都能保持在固定比例，做法蠻多，以下列出其中兩種做法：

<!-- more -->

## **aspect-ratio**

Chrome 88（2021） 開始支援的屬性，設置方式也很簡單，直接輸入寬高比就可以了

```css
aspect-ratio: width / height;
```

{% colorquote info %}
缺點：`aspect-ratio` 屬性瀏覽器支援度較低，可以參考 [can I use](https://caniuse.com/?search=Aspect%20Ratio)
{% endcolorquote %}

## **padding-bottom**

如果在意瀏覽器支援度，可以使用 `padding-bottom` 建立外層容器

| 寬高比 | padding-bottom 代入值 |
| --- | --- |
| 16:9 | 56.25% |
| 4:3 | 75% |
| 5:4 | 80% |

```css
padding-bottom: 56.25%;
```

通常會搭配 `position` 定位使用，如下：

```html
<div class="image-wrap">
    <p>some description</p>
</div>
```

```css
.image-wrap {
    position: relative;
    width: 100%;
    padding-bottom: 56.25%;
}
p {
    position: absolute;
    width: 100%;
}
```

範例程式碼：

<iframe height="470" style="width: 100%;" scrolling="no" title="CSS Aspect Ratio / Height Based on Width" src="https://codepen.io/claire-chang-the-bashful/embed/qByLGjg?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/claire-chang-the-bashful/pen/qByLGjg">
  CSS Aspect Ratio / Height Based on Width</a> by Claire Chang (<a href="https://codepen.io/claire-chang-the-bashful">@claire-chang-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

---

參考文章：

[https://stackoverflow.com/questions/1495407/maintain-the-aspect-ratio-of-a-div-with-css](https://stackoverflow.com/questions/1495407/maintain-the-aspect-ratio-of-a-div-with-css)
[https://developer.mozilla.org/en-US/docs/Web/CSS/aspect-ratio](https://developer.mozilla.org/en-US/docs/Web/CSS/aspect-ratio)