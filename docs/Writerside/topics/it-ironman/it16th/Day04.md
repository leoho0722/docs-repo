# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 04 - 認識 WebAuthn (2)

昨天簡單介紹了 WebAuthn，今天要來介紹在 WebAuthn 標準中所制定的 JavaScript API 以及用實際例子來介紹 WebAuthn 註冊流程

## WebAuthn 標準 JavaScript API

在 WebAuthn 標準中，制定了四支 JavaScript API，分為註冊和登入兩種情境

* 註冊 (Registration)
    * Credential Creation Options
    * Authenticator Attestation Response
* 驗證 (Authentication)
    * Credential Get Options
    * Authenticator Assertion Response

註冊和登入兩種情境所進行的流程基本上是相似的
都是先向 RP Server 請求產生資訊，並透過 Client 端的 Authenticator 進行運算後
將 Client 端運算後的資料回傳給 RP Server 進行驗證

這四隻 WebAuthn 標準 JavaScript API 的詳細內容，後面幾天會在進行介紹
這邊大家先有 WebAuthn 定義了四隻標準 API 的概念就可以了

## WebAuthn 註冊流程

![WebAuthn 註冊流程](https://www.w3.org/TR/webauthn/images/webauthn-registration-flow-01.svg)

▲ 圖取自 [WebAuthn Spec](https://www.w3.org/TR/webauthn/#sctn-api)

首先看到註冊，這邊以 Web 作為 Platform 來進行說明

整個流程主要有五個角色，從上往下看，分別為伺服器端的 RP Server、Client 端具有呼叫 WebAuthn API 功能的 Web 應用、WebAuthn API、瀏覽器，最後是 Client 端的 Authenticator

舉例來說，今天要註冊一個網站的會員，這個網站提供使用 WebAuthn 的會員註冊方式，使用者點擊網站上的 WebAuthn 註冊按鈕進行註冊

這時候，網站前端會與網站後端的 RP Server 進行請求 (圖例 ⓪)，接著後端的 RP Server 會隨機產生一個 Challenge 並將 RP Server 的相關資訊回傳給網站前端 (圖例 ①)

網站前端就會透過 WebAuthn API 請瀏覽器呼叫 Client 端的 Authenticator 讓使用者進行驗證 (在這過程中，瀏覽器會提供 RP Server、使用者資訊與 clientDataHash 資訊給 Authenticator) (圖例 ②)

透過 Client 端的 Authenticator 針對 RP Server 產生的隨機 Challenge 使用公開金鑰加密產生出公私鑰 Pair 以及產生出 attestation (圖例 ③)，並回傳由公鑰和 attestation 組成的 attestationObject 給瀏覽器 (圖例 ④)

瀏覽器這時會產生出 clientDataJSON，再透過 WebAuthn API 將 attestationObject 與 clientDataJSON 組成 Credential 提供給網站前端 (圖例 ⑤)

最後網站前端就會將 Credential 傳給 RP Server 進行註冊驗證，並將其中的公鑰儲存起來供登入時驗證。 (圖例 ⑥)

上面用一個日常生活中常見的例子來對 WebAuthn 註冊流程進行說明

明天再接著介紹 WebAuthn 驗證流程～

