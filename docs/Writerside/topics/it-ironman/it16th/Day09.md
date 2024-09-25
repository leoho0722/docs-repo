# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 09 - 前後端實作環境準備

前面簡單認識了 FIDO2、WebAuthn、CTAP 之後，接下來就要開始來進行實作了～

既然要開始實作，那麼第一步肯定是要先把環境安裝好，畢竟俗話說的好，工欲善其事，必先利其器

## 前端環境準備

前端我會使用 `iOS App` 來進行範例實作，所以會需要先將 `Xcode` 安裝好

Xcode 的安裝教學可以參考我在 2021 年鐵人賽參加的文章

> [【在 iOS 開發路上的大小事－Day01】先裝個 Xcode 開發環境壓壓驚](https://ithelp.ithome.com.tw/articles/10259406)

以及這次會使用到 Apple 提供的 Passkeys API，這是需要 Apple 付費開發者帳號才能使用的功能
所以也會需要準備好一個 `Apple 付費開發者帳號`

前端環境的準備，大致如上，接下來是後端環境的準備

## 後端環境

後端我會使用 Go 來進行範例實作，關於 Go 的開發環境可以參考去年 2023 鐵人賽參加的文章

> [【하나, 둘, ready, get set, go】Day 2 - 開發環境](https://ithelp.ithome.com.tw/articles/10314137)

在 Day 03 的時候有提到說，RP Server 會需要一個 Public domain 供外部存取
所以也會需要準備好一個 Public domain 給 RP Server 使用
(PS：如果沒有 Public domain 的話，後面也會告訴大家可以怎麼暫時替代)

在 WebAuthn Library 的部分，我會使用 [go-webauthn](https://github.com/go-webauthn/webauthn) 來實際示範

這個 Library 也在 [webauthn.io](https://webauthn.io) 中列出，算是比較多人使用的

![WebAuthn Library](https://ithelp.ithome.com.tw/upload/images/20240910/20140363BsRc3vNOA1.png)

▲ 圖截自 [webauthn.io](https://webauthn.io)

### 安裝方式

只要在 Terminal 中輸入下面指令，就可以安裝好 `go-webauthn` 了

```shell
go get -u github.com/go-webauthn/webauthn
```

將前後端環境都準備好之後，後面就可以實作前後端了～