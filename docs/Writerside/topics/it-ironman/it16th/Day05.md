# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 05 - 認識 WebAuthn (3)

昨天使用一個日常生活的例子來對 WebAuthn 註冊流程進行說明後
今天繼續使用相同的例子來對 WebAuthn 驗證流程進行說明

## WebAuthn 驗證流程

![WebAuthn 驗證流程](https://www.w3.org/TR/webauthn/images/webauthn-authentication-flow-01.svg)

圖取自 [WebAuthn Spec](https://www.w3.org/TR/webauthn/#sctn-api)

跟註冊一樣以 Web 作為 Platform 來進行說明

整個流程主要有五個角色，從上往下看，分別為伺服器端的 RP Server、Client 端具有呼叫 WebAuthn API 功能的 Web 應用、WebAuthn API、瀏覽器，最後是 Client 端的 Authenticator

舉例來說，今天要登入一個網站的會員，這個網站提供使用 WebAuthn 的會員登入方式，使用者點擊網站上的 WebAuthn 登入按鈕進行登入

這時候，網站前端會與網站後端的 RP Server 進行請求 (圖例 ⓪)，接著後端的 RP Server 會隨機產生一個 Challenge 並將 RP Server 的相關資訊回傳給網站前端 (圖例 ①)

網站前端就會透過 WebAuthn API 請瀏覽器呼叫 Client 端的 Authenticator 讓使用者進行驗證 (在這過程中，瀏覽器會提供 RP Server 資訊與 clientDataHash 資訊給 Authenticator) (圖例 ②)

透過 Client 端的 Authenticator 針對瀏覽器提供的資訊，使用 Authenticator 中的私鑰加密產生出 Assertion Signature，如下圖所示 (後面簡稱為 Signature) (圖例 ③)，與 AuthenticatorData 一起傳給瀏覽器 (圖例 ④)

![Assertion Signature](https://ithelp.ithome.com.tw/upload/images/20240906/20140363zid1vemZ3j.png)

圖取自 [WebAuthn Spec](https://www.w3.org/TR/webauthn/#sctn-op-get-assertion)

瀏覽器此時會產生出 clientDataJSON 並透過 WebAuthn API 將 Signature、AuthenticatorData 提供給網站前端 (圖例 ⑤)

最後網站前端就會將以上資訊傳給 RP Server，RP Server 就會使用先前註冊時所儲存的公鑰進行解密，確認是否來自同一位使用者。(圖例 ⑥)

上面用一個日常生活中常見的例子來對 WebAuthn 驗證流程進行說明

明天再接著認識組成 FIDO2 的另外一個協議，CTAP (Client to Authenticator Protocol)～