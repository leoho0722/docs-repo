# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 06 - 認識 CTAP

前面簡單認識了 WebAuthn 以及 WebAuthn 的註冊、驗證流程

今天要來認識組成 FIDO2 的另外一個協議，[CTAP (Client to Authenticator Protocol)](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-errata-20220621.html)

## CTAP (Client to Authenticator Procotol)

CTAP 主要是用來規範 Client (也就是 Platform) 與 Authenticator 之間的通訊協議，主要規範了三大點

* Authenticator API
* Message Encoding
* Transport-specific Binding

### Authenticator API

在進行 WebAuthn 註冊/驗證流程中，比較常見的 Authenticator API 有兩個，分別為
`authenticatorMakeCredential` 與 `authenticatorGetAssertion`

* authenticatorMakeCredential
    * 用途：定義在註冊時 Authenticator 如何產生出 Credential

![authenticatorMakeCredential Payload](https://ithelp.ithome.com.tw/upload/images/20240907/20140363mBhM5ZTMQb.png)

▲ 圖取自 [WebAuthn Spec](https://www.w3.org/TR/webauthn-2/#sctn-op-make-cred)

* authenticatorGetAssertion
    * 定義在登入時 Authenticator 如何取得 Assertion Signature 向 RP Server 證明是同一位使用者

![authenticatorGetAssertion](https://ithelp.ithome.com.tw/upload/images/20240907/20140363zeepVIcaQp.png)

▲ 圖取自 [WebAuthn Spec](https://www.w3.org/TR/webauthn-2/#sctn-op-get-assertion)

而 Authenticator API 中還有定義其他相關內容，這邊就先不詳述。
有興趣的可以點擊連結，參考 [CTAP Spec](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-errata-20220621.html#authenticator-api)

### Message Encoding

而 Client 與 Authenticator 之間通訊的訊息編碼皆為 [CBOR (簡明二進制物件表示法)](https://cbor.io/) 編碼

相比使用 JSON 進行編碼，所需耗費的資源更低，適合用在像是 Authenticator 這種嵌入式硬體上

在 CTAP 中，CBOR 被用於表達使用者資訊、公私鑰 key-pair、Signature 等的資料結構

### Transport-specific Binding

在 Transport-specific Binding 中則是規範了 External Authenticator (如：USB、BLE、NFC) 在 CTAP 中的通訊方式

今天簡單認識了組成 FIDO2 的另一個協議，CTAP

明天要繼續來認識基於 FIDO2 的 Passkeys 以及現在有哪些網路服務是有支援 Passkeys 的～