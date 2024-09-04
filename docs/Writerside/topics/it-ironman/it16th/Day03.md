# 【Go！帶你探索 FIDO2 資安技術全端應用】Day03 - 認識 WebAuthn (1)

昨天簡單介紹了 FIDO 聯盟，以及目前的 FIDO 無密碼身份驗證標準分別有 FIDO 跟 FIDO2

今天要來介紹的是組成 FIDO2 無密碼身份驗證標準之一的 WebAuthn

## 補充一下昨天少提到的內容

![FIDO2 組成](https://fidoalliance.org/wp-content/uploads/FIDO2-Graphic-v2.png)

FIDO2 旨在讓使用者可以使用常見設備 (如：手機、電腦) 在行動和桌面環境中對線上服務進行身份驗證

上圖為整個 FIDO2 的組成，打星號的為在 FIDO2 中所規範的

## WebAuthn

WebAuthn 全名為 Web Authentication，最早於 2016 年被提出
並在 2019 年正式被全球資訊網協會 W3C 列入正式標準中

WebAuthn 標準中制定了一套標準的 JavaScript API，用於進行 Web 無密碼身份驗證

在伺服器端，也就是 Relying Party Server，需使用 https 且具有合法公開的 domain (很重要x3)
如下圖紅線畫的地方所說的

![https://ithelp.ithome.com.tw/upload/images/20240904/20140363qTZ0u7et6J.png](https://ithelp.ithome.com.tw/upload/images/20240904/20140363qTZ0u7et6J.png)

為什麼是使用 domain 呢？因為 WebAuthn 透過將身份憑證綁定在 domain 上，讓身份憑證都只能在特定 domain 上使用

今天先簡單介紹 WebAuthn 到這邊～
明天再來介紹在 WebAuthn 標準中制定的 JavaScript API 以及用實際例子介紹 WebAuthn 註冊流程