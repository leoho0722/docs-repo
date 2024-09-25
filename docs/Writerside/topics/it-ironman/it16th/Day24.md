# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 24 - 開始進行前後端串接 (4)

昨天通過了第三關之後，今天要來接續第四關

## 串接之旅 Part 4

### 開始測試囉

#### 串接第四關：failed to parse credential creation response, error: Error parsing attestation response

![https://ithelp.ithome.com.tw/upload/images/20240922/20140363sVU9TEJEyI.png](https://ithelp.ithome.com.tw/upload/images/20240922/20140363sVU9TEJEyI.png)

從錯誤來看，是在解析 `protocol.CredentialCreationResponse` 中的 `AttestationResponse` 時發生錯誤所導致的

還記得前端在提供 attestationObject 給後端時，有用 base64RawURL 進行編碼嗎？
後端收到後，我們少做了 base64RawURL 解碼，就直接傳給 `protocol.CredentialCreationResponse`，所以才會出現這個錯誤

下面就來針對 attestationObject 進行 base64RawURL 解碼

```go
authenticatorAttestationObject, err := base64.RawURLEncoding.DecodeString(request.Response.AttestationObject)
if err != nil {
    ctx.JSON(
        http.StatusBadRequest,
        api.CommonResponse{
            Status:       "failed",
            ErrorMessage: "failed to decode attestationObject, error: " + err.Error(),
        },
    )
    return
}
```

並修改 `protocol.CredentialCreationResponse` 中的 `AttestationObject`

```go
ccr := protocol.CredentialCreationResponse{
    PublicKeyCredential: protocol.PublicKeyCredential{
        Credential: protocol.Credential{
            ID:   request.Id,
            Type: request.Type,
        },
        RawID:                  []byte(request.Id),
        ClientExtensionResults: request.GetClientExtensionResults,
    },
    AttestationResponse: protocol.AuthenticatorAttestationResponse{
        // 改成用上面 base64RawURL 解碼後的 authenticatorAttestationObject
        AttestationObject: protocol.URLEncodedBase64(authenticatorAttestationObject),
        AuthenticatorResponse: protocol.AuthenticatorResponse{
            ClientDataJSON: authenticatorClientDataJSON,
        },
    },
}
```

修改後，會像是下面這樣

![https://ithelp.ithome.com.tw/upload/images/20240922/20140363x79LSfzvkp.png](https://ithelp.ithome.com.tw/upload/images/20240922/20140363x79LSfzvkp.png)

![https://ithelp.ithome.com.tw/upload/images/20240922/20140363M6SKEkPdaC.png](https://ithelp.ithome.com.tw/upload/images/20240922/20140363M6SKEkPdaC.png)

再重新執行一次，就成功通過第四關了～

明天我們再繼續勇闖第五關！