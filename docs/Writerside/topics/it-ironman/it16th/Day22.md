# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 22 - 開始進行前後端串接 (2)

昨天通過了第一關之後，今天要來接續第二關

## 串接之旅 Part 2

### 開始測試囉

#### 串接第二關：challenge mismatch

![https://ithelp.ithome.com.tw/upload/images/20240922/20140363f1GUwa8SpB.png](https://ithelp.ithome.com.tw/upload/images/20240922/20140363f1GUwa8SpB.png)

從錯誤訊息來看，是 challenge mismatch 所導致的，那麼我們到後端看一下是哪邊會拋出這個錯誤

![https://ithelp.ithome.com.tw/upload/images/20240922/20140363cMObYlnfBR.png](https://ithelp.ithome.com.tw/upload/images/20240922/20140363cMObYlnfBR.png)

從後端程式碼中來看，是前端回傳的 clientDataJSON 中的 challenge 與在 WebAuthn Registration session 中記載的 challenge 不吻合

那我們將 clientDataJSON 進行 base64URL 解碼，可以看到會是下面這樣

```json
{"type":"webauthn.create","challenge":"TXVjN3ZMZ0JWc01XYVE4NV9zZFlwYVZHWk9CZWk4VURGSllZYVUxeERwRQ","origin":"https://fee8-211-20-7-115.ngrok-free.app"}
```

那我們看一下 WebAuthn Registration session 中記載的 challenge 是什麼

```json
{"challenge":"Muc7vLgBVsMWaQ85_sdYpaVGZOBei8UDFJYYaU1xDpE","rpId":"fee8-211-20-7-115.ngrok-free.app","user_id":"MzNjMmRmMjAtMGEyYS00NTg4LTgwZWQtMTcwNmM1NWE4ZTY2","expires":"0001-01-01T00:00:00Z","userVerification":"preferred"}
```

兩者的 challenge 確實是不吻合，那我們看一下在產生註冊資訊 API 回傳的 challenge 是什麼

```json
{"rp":{"name":"it16th-webauthn-rp-server","id":"fee8-211-20-7-115.ngrok-free.app"},"user":{"name":"leoho","displayName":"Leo Ho","id":"MzNjMmRmMjAtMGEyYS00NTg4LTgwZWQtMTcwNmM1NWE4ZTY2"},"challenge":"Muc7vLgBVsMWaQ85_sdYpaVGZOBei8UDFJYYaU1xDpE","pubKeyCredParams":[{"type":"public-key","alg":-7},{"type":"public-key","alg":-35},{"type":"public-key","alg":-36},{"type":"public-key","alg":-257},{"type":"public-key","alg":-258},{"type":"public-key","alg":-259},{"type":"public-key","alg":-37},{"type":"public-key","alg":-38},{"type":"public-key","alg":-39},{"type":"public-key","alg":-8}],"timeout":300000,"authenticatorSelection":{"authenticatorAttachment":"platform","requireResidentKey":false,"residentKey":"preferred","userVerification":"preferred"},"attestation":"direct"}
```

可以看到 WebAuthn Registration session 中的 challenge 與 產生註冊資訊 API 回傳的 challenge 是相同的，那麼就是要針對 clientDataJSON 進行處理了

如果我們將 clientDataJSON 中的 challenge 再進行一次 base64URL 解碼，會得到什麼呢

```text
Muc7vLgBVsMWaQ85_sdYpaVGZOBei8UDFJYYaU1xDpE
```

登登～是原本的 challenge！

> 這邊可以使用線上工具進行轉換
> 但只有支援 base64 沒有支援 base64URL，不過我們可以替換掉對應的字串，來解決這個問題
> https://www.convertstring.com/zh_TW/EncodeDecode/Base64Decode

那我們應該怎麼修改呢～

看到 `controller/attestation.go` 的 `FinishAttestationHandler`

在使用 base64URL 對 clientDataJSON 進行解碼後，新增下面的程式碼

```go
challenge, ok := clientDataJSON["challenge"].(string)
if !ok {
    ctx.JSON(
        http.StatusBadRequest,
        api.CommonResponse{
            Status:       "failed",
            ErrorMessage: "challenge not found",
        },
    )
    return
}

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

修改後，會像是下面這樣

![https://ithelp.ithome.com.tw/upload/images/20240922/20140363GcaMTJ4eek.png](https://ithelp.ithome.com.tw/upload/images/20240922/20140363GcaMTJ4eek.png)

再重新執行一次，就成功通過第二關了～

明天我們再繼續勇闖第三關！