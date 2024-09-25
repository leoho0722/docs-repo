# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 23 - 開始進行前後端串接 (3)

昨天通過了第二關之後，今天要來接續第三關

## 串接之旅 Part 3

### 開始測試囉

#### 串接第三關：failed to parse credential creation response, error: Parse error for Registration

![https://ithelp.ithome.com.tw/upload/images/20240922/20140363p916DDmu0c.png](https://ithelp.ithome.com.tw/upload/images/20240922/20140363p916DDmu0c.png)

從錯誤來看，是在解析 `protocol.CredentialCreationResponse` 的時候發生錯誤所導致的

那我們看一下 go-webauthn source code 來判斷有哪些可能性

![https://ithelp.ithome.com.tw/upload/images/20240922/201403631qduCi9ujn.png](https://ithelp.ithome.com.tw/upload/images/20240922/201403631qduCi9ujn.png)

看起來有四種可能性會導致 `Parse error for Registration` 的錯誤，我們來一一除錯
下面是我們要進行 parse 的 `protocol.CredentialCreationResponse` 轉成 JSON 格式後的樣子

```json
{"id":"Ig2aOCXeBzERa5C2mtkNRI4U1Ww=","type":"public-key","rawId":"SWcyYU9DWGVCekVSYTVDMm10a05SSTRVMVd3PQ","response":{"clientDataJSON":"eyJ0eXBlIjoid2ViYXV0aG4uY3JlYXRlIiwiY2hhbGxlbmdlIjoiWXkxS1NsUkhWM1ZaVjAxQ1pXTjBYek00VkVKV2F6VlBVMU5QT1dWVVZXaEpWVFJFT0c5TWIwbDFPQSIsIm9yaWdpbiI6Imh0dHBzOi8vZmVlOC0yMTEtMjAtNy0xMTUubmdyb2stZnJlZS5hcHAifQ","authenticatorData":null,"publicKey":null,"publicKeyAlgorithm":0,"attestationObject":"bzJObWJYUmtibTl1WldkaGRIUlRkRzEwb0doaGRYUm9SR0YwWVZpWWkySHVIWXgtMzZrWUd0cFFoajdjNmMyZUxwdFpjR1c4WFhKdXZmdVRYcXBkQUFBQUFQdjhNQWNWVGs3TWpBdHVBZ1ZYMTcwQUZDSU5tamdsM2djeEVXdVF0cHJaRFVTT0ZOVnNwUUVDQXlZZ0FTRllJUGFhdmtMbm9xdGtMTFFRcGJ6MjVpYUdtZlFxZWxseHZ5QXZ0YllySnJwaUlsZ2d3ck1qUHo3THVvNXpIVmxfck8xeWR4U1RMV0hJQlIxRWx0VlhEQzZZTzVr"}}
```

我們的 credential.id 不為空，所以不會是 `Missing ID` 這個可能性
我們的 credential.id 有帶 padding (`=`)，跟 Raw 不吻合，所以有可能是 `ID not base64.RawURLEncoded` 這個導致的
我們的 credential.type 不為空，所以不會是 `Missing type` 這個可能性
我們的 credential.type 是 `public-key`，所以不會是 `Type not public-key` 這個可能性

看起來最有可能導致錯誤的就是 `ID not base64.RawURLEncoded` 這個

那這個錯誤，我們可以從前端來進行修改，因為 credential.id 是由前端提供給後端的

我們看到前端 `PasskeysViewController.swift` 中的 `passkeysFinishRegistration` Function
可以看到 credential.id 是轉換成 base64URL 編碼，而不是 base64RawURL 編碼

![https://ithelp.ithome.com.tw/upload/images/20240922/20140363FRPS2QtYR2.png](https://ithelp.ithome.com.tw/upload/images/20240922/20140363FRPS2QtYR2.png)

在這邊，我們改用 base64RawURL 的編碼方式，對 credential.id 進行編碼，像是下面這樣

```swift
let base64RawURLEncodedCredentialID = credentialID.base64EncodedString().base64EncodedToBase64RawURLEncoded()
```

而 `base64EncodedToBase64RawURLEncoded` 這個 Function 裡面寫了什麼呢？

其實就是將 base64 與 base64URL 對應的字串進行對換，以及將 padding (`=`) 改為空字串

```swift
extension String {
    // ...

    /// 將 Base64 編碼字串轉換為 Raw Base64URL 編碼字串
    func base64EncodedToBase64RawURLEncoded() -> String {
        return self
            .replacingOccurrences(of: "+", with: "-")
            .replacingOccurrences(of: "/", with: "_")
            .replacingOccurrences(of: "=", with: "")
    }
}
```

修改後，會像是下面這樣

![https://ithelp.ithome.com.tw/upload/images/20240922/20140363WY8rrVtpPH.png](https://ithelp.ithome.com.tw/upload/images/20240922/20140363WY8rrVtpPH.png)

再重新執行一次，就成功通過第三關了～

明天我們再繼續勇闖第四關！