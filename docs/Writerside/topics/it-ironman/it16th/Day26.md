# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 26 - 開始進行前後端串接 (6)

昨天成功 WebAuthn Registration 後，接下來要接續進行 WebAuthn Authentication

## 串接之旅 Part 6

### 開始測試囉

#### 串接第六關：CredentialGetOptionsResponse 不合

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363Cz3nTlbU8N.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363Cz3nTlbU8N.png)

從錯誤來看，看起來跟前面第一關的問題是相同的，找不到 `userVerification` 欄位，那麼我們可以嘗試使用相同的方法進行處理

首先，我們看到 `controller/assertion.go` 中的 `StartAssertionHandler`
新增一個 `authenticationOptions`，並將前端傳來的 Request body 中的 `userVerification` 進行設定

```go
authenticationOptions := func(options *protocol.PublicKeyCredentialRequestOptions) {
    options.UserVerification = protocol.UserVerificationRequirement(request.UserVerification)
}
```

接著，在提供給 `BeginLogin`，像是下面這樣

```go
options, sessionData, err := webauthn.WebAuthn.BeginLogin(foundUser, authenticationOptions)
```

再重新執行一次，可以與 Apple Passkeys API 進行互動了

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363nPRj25ofIn.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363nPRj25ofIn.png)

明天我們再繼續勇闖第七關！