---
title: 什麼是 DNS，跟 Name Server 又有什麼關聯？
date: 2023-06-27
tags: [ dns ]
category: Web
description: 本篇文章說明什麼是 DNS 網域名稱系統、DNS 跟 Name Server 的關聯，以及 DNS 如何運作，是客製化網域前的必備知識
image: https://imgur.com/uGDMmQF.png
---

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 400px;" src="https://imgur.com/uGDMmQF.png">
</div>

## **網域名稱（Domain Name）**

要了解 DNS，首先必須知道域名的結構。域名用於識別和定位網路上特定資源的可讀性字串，通常由以下部分組成，以「點」分隔：

- **頂級網域（Top-level domain，TLD）**：頂級網域是域名結構中的最高層次，用於表示域名的類別或類型。常見包括 com、org、net、edu、gov 等，還有國家和地區的頂級網域，例如 tw、jp、uk。以 `www.example.com` 為例，「com」就是頂級網域。
- **次級網域（Second-level domain）**：次級網域位於頂級網域之下的一級域名，用於識別組織、公司、個人等。以 `www.example.com` 為例，「example」就是次級網域。
- **主機網域（Host domain）**：主機網域位於次級網域之下的域名部分，用於指定特定網站、服務或主機的名稱。以 `www.example.com` 為例，「www」為主機網域。
- **完整網域名稱（Fully Qualified Domain Name，簡稱 FQDN）**：包含所有層級的域名，從頂級網域到主機網域。`www.example.com` 就是完整網域名稱。

<!-- more -->

---

## **DNS 網域名稱系統（Domain Name System）**

DNS 用於將人類可讀的域名（ex：`www.example.com`）轉換為對應的 IP 位址（ex: 93.184.216.34），讓使用者只需要記得域名，而不需要背下由數字組成的 IP 位址，就好像我們在稱呼一個人的時候，通常會叫對方的名字（網域名稱），而不是身份證字號（IP），DNS 相當重要，少了它網際網路基本上就會癱瘓。

因網際網路只認得 IP 位址，當使用者輸入網域名稱，瀏覽器會先去離他最近的 DNS Server 透過網域名稱查詢相對應的伺服器 IP，並回傳 IP 位址給瀏覽器，瀏覽器再透過 IP 跟網路伺服器建立連線，並取得網頁內容。

{% colorquote info %}
**利用終端機查詢域名 IP：**
ex: 輸入 `ping www.example.com`，得到 IP 位址 `93.184.216.34`
{% endcolorquote %}

---

## **NS 名稱伺服器（Name Server）**

名稱伺服器是 DNS 中一個重要元素，主要功能：

- **域名解析：**當使用者在瀏覽器中輸入一個域名，瀏覽器將向名稱伺服器發出查詢，名稱伺服器開始將域名轉換為與之相關聯的 IP 地址。這個過程被稱為域名解析。名稱伺服器存儲著該域名的 DNS 記錄（A 記錄、CNAME 記錄等）。
- **分散資料儲存：**一個網域至少要有兩組名稱伺服器，DNS 系統使用分散的名稱伺服器架構。當一個名稱伺服器無法回覆查詢時，其他名稱伺服器可以接手處理。
- **域名管理：**一旦變更名稱伺服器也會變更管理 DNS 的地方。舉例來說，假如我們在 GoDaddy 購買網域，並使用預設的名稱伺服器，那麼 DNS 區域檔案就在 GoDaddy 內，但是當網域變更為 Cloudflare 名稱伺服器，那麼 DNS 區域檔案就位在 Cloudflare。

### **常見 DNS 區域檔案類型：**

- **SOA 記錄（Start of Authority）：**區域檔案中的第一個記錄，指定該區域的管理者和其他基本設定，包含主要名稱服務器（Primary Name Server）和負責人電子郵件地址（Responsible Person Email Address）等。
- **A 記錄（Address，IPv4 地址）：**是用於名稱解析的重要記錄，將網域名稱對應到 IPv4 的 32 位元位址。
- **AAAA 記錄（IPv6 地址）：**將網域名稱對應到 IPv6 的 32 位元位址。
- **CNAME 記錄（Canonical Name，別名）：**用於將某個網域或子網域指向到某個 A 記錄上，這樣，當我們需要將一個域名指向另一個域名或主機，就不需要再為新的別名建立一個 A 記錄。
    
    > 例如：要將 `www.example.com` 指向 `example.com`，可以在 `www.example.com` 的 CNAME 記錄中指定 `example.com`，當使用者輸入 `www.example.com` 時，DNS 將解析並回傳 `example.com` 的相應 A 記錄，從而將流量導向到正確的主機。
    > 
- **NS 記錄（Name Server）：**指定該區域使用的名稱服務器（Name Server），定義了負責提供該區域 DNS 解析的名稱服務器的伺服器名稱。
- **SRV 記錄（Service Record，服務）：**用於指定特定服務的伺服器位置和連接資訊，包含了服務的協議、服務名稱、域名、優先級、權重、埠和目標（伺服器）等。
- **MX 記錄（Mail Exchanger，郵件交換）：**指定特定域名電子郵件導向至郵件伺服器，所有要送往伺服器的郵件都要經過 MX 轉送。需設定該伺服器郵件傳遞時的優先次序，值越低表示有越高的處理優先權。
- **TXT 記錄（文字）：**記錄任意文字訊息，可用於驗證域名所有權、提供人類可讀的訊息等。

---

# **DNS 如何運作呢？**

<div style="display: flex; justify-content: center; margin: 30px 0;">
    <img style="width: 100%; max-width: 650px;" src="https://imgur.com/B9LX388.png">
</div>

1. 當使用者在網頁瀏覽器中輸入網域名稱 `www.example.com`，瀏覽器會先查看本地 DNS 暫存（DNS Cache），這是先前訪問過該網站保存的紀錄，如果有紀錄就直接進入該網站，若無則繼續以下步驟
2. 傳輸到網際網路，**DNS 遞迴解析伺服器（DNS Resolver）**接收該網域名稱查詢請求
3. 遞迴解析器查詢 **根名稱伺服器（Root Name Server）**的 IP 位址，並向根網域發出請求
4. 根名稱伺服器回傳儲存其網域資訊的 **頂級網域名稱伺服器（TLD Name Server）**的 IP 位址給遞迴解析器。搜尋 `www.example.com` 時會指向 .com TLD
5. 遞迴解析器向 .com TLD 發出請求
6. TLD 名稱伺服器回傳使用該網域的名稱伺服器 IP 位址給遞迴解析器
7. 遞迴解析器向該 **權威名稱伺服器（Authoritative Name Server）**發出請求
8. 名稱伺服回傳 `www.example.com` 的 IP 位址給遞迴解析器
9. 遞迴解析器回傳正確的 IP 位址給瀏覽器
10. 瀏覽器帶著 IP 去訪問網站
11. 位於該 IP 的 Web 伺服器回傳網頁內容給瀏覽器

由上述可以得知，網頁載入涉及 4 個 DNS 伺服器：

- **DNS 遞迴解析伺服器（DNS Resolver）**
- **根名稱伺服器（Root Name Server）**
- **頂級網域名稱伺服器（TLD Name Server）**
- **權威名稱伺服器（Authoritative Name Server）**

---

參考資源：

https://www.cloudflare.com/zh-tw/learning/dns/what-is-dns/

https://tw.godaddy.com/help/what-is-dns-665

[https://its-okay.medium.com/搞懂-ip-fqdn-dns-name-server-鼠年全馬鐵人挑戰-05-aa60f45496fb](https://its-okay.medium.com/%E6%90%9E%E6%87%82-ip-fqdn-dns-name-server-%E9%BC%A0%E5%B9%B4%E5%85%A8%E9%A6%AC%E9%90%B5%E4%BA%BA%E6%8C%91%E6%88%B0-05-aa60f45496fb)