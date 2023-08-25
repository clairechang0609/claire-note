---
title: GitHub Pages 自訂域名與 HTTPS 設定（GoDaddy + Cloudflare）
date: 2023-06-28
tags: [ dns, github ]
category: Web
description: 本篇介紹 Github Pages 搭配 GoDaddy 購買客製化網域，並使用 CloudFlare DNS 網域託管，提升網路安全與效能優化，並增加網站專業度
image: https://imgur.com/4nmO3WX.png
---

<div style="display: flex; justify-content: center; margin: 0 0 30px;">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/4nmO3WX.png">
</div>

2022 年底開始在 GitHub 經營了個人部落格，使用免費域名一段時間後，覺得 SEO（搜尋引擎優化）的成效不如預期，因此決定購買自己的網域，來提升部落格的專業度和可信度。

以下介紹：

**Step 1、購買網域名稱：**使用 [GoDaddy](https://tw.godaddy.com/)

**Step 2、DNS 網域託管：**使用 [Cloudflare](https://www.cloudflare.com/zh-tw/)

**Step 3、Cloudflare DNS 設定**

**Step 4、GitHub 設定網域**

**Step 5、替網站加上 Https、設定 SSL 加密**

<!-- more -->

{% colorquote info %}
想了解 DNS 運作的流程，可以參考 [這篇文章](https://clairechang.tw/2023/06/27/web/dns-and-name-server/)
{% endcolorquote %}

---

## **Step 1、購買網域名稱**

1. 至 [GoDaddy](https://tw.godaddy.com/) 註冊會員，並選購域名。GoDaddy 會列出名稱相似的可選域名：

<div style="display: flex; justify-content: center; margin: 30px 0; border: 1px solid rgb(200, 200, 200);">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/lsIfx8O.png">
</div>

2. 付款完成後，可至 [whois](https://who.is/) 查詢，查詢網域是否購買成功，未被選購的網域，查詢後會顯示 `No Found` ，以下為網域已被使用的查詢結果

```
Domain Name: clairechang.tw
   Domain Status: clientRenewProhibited,clientUpdateProhibited,clientDeleteProhibited,clientTransferProhibited
   Registrant:
      (Redacted for privacy)

   Administrative Contact:
      (Redacted for privacy)

   Technical Contact:
      (Redacted for privacy)

   Record expires on 2025-06-25 12:03:28 (UTC+8)
   Record created on 2023-06-25 12:03:28 (UTC+8)

   Domain servers in listed order:
      carlane.ns.cloudflare.com     
      kellen.ns.cloudflare.com      

Registration Service Provider: GoDaddy
Registration Service URL: http://www.GoDaddy.com/

Provided by Registry Services, LLC. Registry Gateway Services

Information Updated: 2023-06-28 05:06:23
```

{% colorquote info %}
需注意：由於我購買的頂級域名是 `tw`，由台灣網路資訊中心（TWNIC）負責管理和監督，因此透過 [whois](https://who.is/) 查詢後，註冊人資料有被隱藏及保護（Registrant、Administrative Contact、Technical Contact），如果購買的域名，隱私資料未隱藏，建議購買**「全方位網域保護」**，否則註冊資料將會被完全公開
{% endcolorquote %}

---

## **Step 2、DNS 網域託管**

將域名的 DNS 管理權限交給第三方的服務提供商 Cloudflare。

其實 GoDaddy 也有提供 DNS 服務，那為什麼要使用 Cloudflare 呢？因為 Cloudflare 除了 DNS 代管，還提供 WAF（網絡防火牆，Web Application Firewall）、CDN（內容傳遞網絡，Content Delivery Network）服務、DDoS 攻擊防護等，提升網路安全與效能優化。

1. 至 [Cloudflare](https://www.cloudflare.com/zh-tw/) 註冊帳號，並新增網站，輸入剛才在 GoDaddy 購買的網域

<div style="display: flex; justify-content: center; margin: 30px 0; border: 1px solid rgb(200, 200, 200);">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/DbwPJHT.png">
</div>

2. 選擇方案：選擇免費即可，若之後有需求可再升級

<div style="display: flex; justify-content: center; margin: 30px 0; border: 1px solid rgb(200, 200, 200);">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/B1agYG7.png">
</div>

3. 接著看到 DNS 紀錄，這裡先跳過，Step 3 會進行設定
4. **變更名稱伺服器：**此步驟為重點，首先會看到指示需到網域註冊商（也就是 GoDaddy）移除他的名稱伺服器，並調整為 Cloudflare 的名稱伺服器

<div style="display: flex; justify-content: center; margin: 30px 0; border: 1px solid rgb(200, 200, 200);">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/j5FUSPQ.png">
</div>

回到 GoDaddy 進行調整，選擇「變更名稱伺服器」，並調整為 Cloudflare 的伺服器

<div style="display: flex; justify-content: center; margin: 30px 0; border: 1px solid rgb(200, 200, 200);">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/GnlULkB.png">
</div>

<div style="display: flex; justify-content: start; margin: 30px 0; border: 1px solid rgb(200, 200, 200);">
    <img style="width: 100%; max-width: 600px;" src="https://imgur.com/Hb8RCsK.png">
</div>

5. 回到 Cloudflare 確認是否託管成功，需要一段時間，成功後會看到以下訊息

<div style="display: flex; justify-content: start; margin: 30px 0; border: 1px solid rgb(200, 200, 200);">
    <img style="width: 100%; max-width: 500px;" src="https://imgur.com/jKUGxX2.png">
</div>

---

## **Step 3、Cloudflare DNS 設定**

1. 點開 DNS 設定，可以看到從 GoDaddy 抓回來的預設值，我們要將網域指向 GitHub Pages，先透過 [GitHub 文件](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site) 找到以下四組 IP

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

2. 接著到 Cloudflare DNS 進行設定，先刪除預設 Type 為 A 的設定，並加入這四組 GitHub IP 到 A 紀錄

<div style="display: flex; justify-content: center; margin: 30px 0; border: 1px solid rgb(200, 200, 200);">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/1jzn7Hm.png">
</div>

---

## **Step 4、GitHub 設定網域**

1. 到 GitHub 專案儲存庫，點擊 Settings → Pages → Custom domain，將網域貼上

<div style="display: flex; justify-content: center; margin: 30px 0; border: 1px solid rgb(200, 200, 200);">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/3M0BoWI.png">
</div>

2. 設定成功後，回到專案首頁，可以看到自動新增了一支 CNAME 檔案，內容為剛才輸入的網域名稱

<div style="display: flex; justify-content: center; margin: 30px 0; border: 1px solid rgb(200, 200, 200);">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/ct71Omv.png">
</div>

<div style="display: flex; justify-content: center; margin: 30px 0; border: 1px solid rgb(200, 200, 200);">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/2IJptsf.png">
</div>

---

## **Step 5、替網站加上 Https、設定 SSL 加密**

1. 回到 Cloudflare，增加一組規則，使用 301 轉址將網域 HTTP url 自動轉址到 HTTPS，確保整個網站的訪問都是安全的

<div style="display: flex; justify-content: center; margin: 30px 0; border: 1px solid rgb(200, 200, 200);">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/DcpDzRR.png">
</div>

<div style="display: flex; justify-content: start; margin: 30px 0; border: 1px solid rgb(200, 200, 200);">
    <img style="width: 100%; max-width: 600px;" src="https://imgur.com/LczDuMT.png">
</div>

2. Cloudflare 預設的 SSL/TLS 為「彈性模式」，改為**「完整模式」**，這樣流量在瀏覽器跟 Cloudflare Proxy 間，以及 Cloudflare Proxy 與網站伺服器間，都是使用 SSL/TLS 加密，提升網頁安全

<div style="display: flex; justify-content: center; margin: 30px 0; border: 1px solid rgb(200, 200, 200);">
    <img style="width: 100%; max-width: 100%;" src="https://imgur.com/avJnjUa.png">
</div>

最後，點開網域若能順利看到畫面，就大功告成囉！

---

參考資源：

[https://medium.com/pm的生產力工具箱/如何替-github-pages-設定個人網域與-cloudflare-ssl-設定-1e61b9ff0909](https://medium.com/pm%E7%9A%84%E7%94%9F%E7%94%A2%E5%8A%9B%E5%B7%A5%E5%85%B7%E7%AE%B1/%E5%A6%82%E4%BD%95%E6%9B%BF-github-pages-%E8%A8%AD%E5%AE%9A%E5%80%8B%E4%BA%BA%E7%B6%B2%E5%9F%9F%E8%88%87-cloudflare-ssl-%E8%A8%AD%E5%AE%9A-1e61b9ff0909)

https://www.renokai.tw/2022/09/deploy-website/

https://israynotarray.com/other/20221206/2011045213/

https://gist.github.com/cvan/8630f847f579f90e0c014dc5199c337b