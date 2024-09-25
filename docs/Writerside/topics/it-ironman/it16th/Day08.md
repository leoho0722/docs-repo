# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 08 - FIDO2 解決了什麼

在前面幾天，我們認識了組成 FIDO2 的 WebAuthn、CTAP 兩大規範，以及基於 FIDO2 的 Passkeys

今天我們要來說說 FIDO2 究竟解決了什麼問題！

## 解決了什麼

FIDO2 主要解決了四個問題

1. 無需再記錄密碼
2. 防止重送攻擊
3. 防止網路釣魚
4. 隱私保護

### 無需再記錄密碼

透過使用 FIDO2，傳統的密碼已經轉換成一個個的 Credential 也就是 Passkeys
透過 Authenticator 進行生物辨識驗證等方式，即可證明是否與註冊時為同一位使用者

### 防止重送攻擊

由於每次進行 WebAuthn 時，RP Server 產生的 Challenge 都是隨機產生的，以及 RP Server 對於請求的來源也有限制，
因此當有中間人從中攔截時，只要重新產生一組新的 Challenge，中間人就算持有舊的 Challenge，從其他來源發起請求，也無法進行後續驗證，可有效防止重送攻擊

### 防止網路釣魚

WebAuthn 採用公私鑰 key-pair，攻擊者無法透過網路釣魚的方式取得儲存於使用者裝置上的私鑰
此外 WebAuthn 的每個 Credential 都綁定在特定 domain 上，因此每個 Credential 都是唯一的

### 隱私保護

承上點，在執行 WebAuthn 流程中，私鑰永遠會儲存在使用者裝置上，只有公鑰會傳送給伺服器，既使攻擊者拿到公鑰，也無法代表原使用者

好～透過這 8 天的認識，應該對於 FIDO2、WebAuthn、CTAP 這些名詞和概念有初步理解一點了
接下來會透過實際操作，來帶大家繼續探索 FIDO2 的前後端應用～