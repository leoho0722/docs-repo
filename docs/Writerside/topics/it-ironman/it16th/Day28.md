# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 28 - 開始進行前後端串接 (8)

昨天通過了第七關之後，今天要來接續第八關

## 串接之旅 Part 8

### 開始測試囉

#### 串接第八關：challenge mismatch

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363UT8Dfcbc09.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363UT8Dfcbc09.png)

這個錯誤跟前面第二關遇到的錯誤相同，所以我們可以參考前面的方式來進行修改

首先看到 `controller/assertion.go` 中的 `StartAssertionHandler`
我們一樣將 session 中的 challenge 進行 base64 編碼

```go
sessionData.Challenge = base64.RawStdEncoding.EncodeToString([]byte(sessionData.Challenge))
assertionSessionData = sessionData
```

像是下面這樣

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363skmRHK6Gq3.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363skmRHK6Gq3.png)

接著看到 `FinishAssertionHandler`

我們一樣用在前面 WebAuthn Registration 的 `decodedChallenge` 方式來做修改

```go
decodedChallenge, err := base64.RawURLEncoding.DecodeString(challenge)
if err != nil {
    ctx.JSON(
        http.StatusBadRequest,
        api.CommonResponse{
            Status:       "failed",
            ErrorMessage: "failed to decode challenge, error: " + err.Error(),
        },
    )
    return
}
challenge = string(decodedChallenge)
```

像是下面這樣

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363q8hD2WC1Ih.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363q8hD2WC1Ih.png)

再重新執行一次，就成功通過第八關了～

明天我們再繼續勇闖第九關！