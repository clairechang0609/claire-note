---
title: Vue.js 圖片上傳串接後端 API（轉換成base64格式）
date: 2022-11-03
tags: [ vue2 ]
category: Vue
---
> **版本：vue 2.6.14**
>

這次介紹的上傳步驟為，使用者填寫一份表單，在選取欲上傳圖片後，系統會先將檔案上傳至雲端資料庫，待使用者確認整份表單填寫無誤後，才送出表單。

**操作步驟：**

1. 點擊 input 選擇本地端圖片
2. 進行圖片轉換（轉成 base64 格式）
3. 呼叫後端 api 進行上傳（至雲端資料庫）
4. 成功回傳圖片路徑
5. 將該路徑顯示於畫面預覽

<!-- more -->

### **HTML**

說明：

- accept：檔案接受格式

```html
<input type="file" ref="image-input" accept="image/jpeg, image/png" @change="changeImage()" />
<img :src="image" :alt="image" />
```

### **Javascript**

點擊圖片觸發 changeImage 方法：

{% colorquote info %}
vue 語法糖 v-model 綁定的是 v-on:input 事件，因此 input[type=file] 不適用
{% endcolorquote %}

說明：

- FileReader 為 javascript 物件，能以非同步方式（在此使用到 onload 事件）讀取儲存在用戶端的檔案內容，讀取格式可為可 File 或 Blob 物件
- readAsDataURL 方法會讀取指定的 File 或 Blob 物件，讀取完成後將其轉換成 base64 格式

```jsx
changeImage() {
    const input = this.$refs['image-input'];
    const file = input.files[0];

    // reset input，避免使用者重複選同一檔案無法觸發 change 事件
    input.files = new DataTransfer().files;
        
    // 檢查圖片檔大小
    if (!this.checkFileSize(file)) {
        return;
    }
        
    // 上傳圖片
    this.uploadImage(file);
}
```

```jsx
checkFileSize(file) {
    // 限制 1 MB
    if (file.size > (1024 * 1024)) {
        this.$notify({
            type: 'danger',
            text: '檔案大小不可大於 1 MB'
        });
        return false;
    }
    return true;
}
```

**FileReader 正式上場！**

```jsx
uploadImage(file) {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.addEventListener('load', async () => {
        const apiUrl = '/api/v1/image'; // api url
        const configData = { file };
        const response = await this.$axios.$post(apiUrl, configData);
        this.image = response.image;
    });
}
```

最後就可以把 api 呼叫成功回傳回來的圖片路徑渲染在畫面上啦！
大功告成。