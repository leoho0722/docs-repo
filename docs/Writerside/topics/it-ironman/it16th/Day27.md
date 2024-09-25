# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 27 - 開始進行前後端串接 (7)

昨天通過了第六關之後，今天要來接續第七關

## 串接之旅 Part 7

### 開始測試囉

#### 串接第七關：failed to decode clientDataJSON, error: illegal base64 data at input byte 0

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363hI7kuB5WVI.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363hI7kuB5WVI.png)

從錯誤看起來，是後端在針對前端提供的 clientDataJSON 進行 base64RawURL 解碼時發生錯誤

那這個錯誤，我們從後端進行修正！

首先看到 `controller/assertion.go` 中的 `FinishAssertionHandler`

接著我們修改對 clientDataJSON 進行解碼的方式

首先是 `api/assertion.go` 中的 `AuthenticatorAssertionResponseRequest.Response`
原先是使用 `protocol.AuthenticatorAssertionResponse`，這邊我們改成自建一個 `AuthenticatorAssertionResponse` 物件，像是下面這樣

```go
type AuthenticatorAssertionResponse struct {
	ClientDataJSON    string `json:"clientDataJSON"`
	AuthenticatorData string `json:"authenticatorData"`
	Signature         string `json:"signature"`
	UserHandle        string `json:"userHandle,omitempty"`
}
```

再來我們回到 `controller/assertion.go` 繼續修改 clientDataJSON 的解碼方式
與前面 WebAuthn Registration 的 `FinishRegistration` 使用相同方式進行解碼

```go
authenticatorClientDataJSON, err := base64.RawURLEncoding.DecodeString(request.Response.ClientDataJSON)
if err != nil {
    ctx.JSON(
        http.StatusBadRequest,
        api.CommonResponse{
            Status:       "failed",
            ErrorMessage: "failed to decode clientDataJSON, error: " + err.Error(),
        },
    )
    return
}
var clientDataJSON map[string]interface{}
if err := json.Unmarshal(authenticatorClientDataJSON, &clientDataJSON); err != nil {
    ctx.JSON(
        http.StatusBadRequest,
        api.CommonResponse{
            Status:       "failed",
            ErrorMessage: "failed to unmarshal clientDataJSON, error: " + err.Error(),
        },
    )
    return
}
```

接著也將 `AuthenticatorAssertionResponseRequest.Response` 中的其他欄位 (AuthenticatorData、Signature、UserHandle) 一起進行 base64RawURL 解碼

#### AuthenticatorData

```go
authenticatorData, err := base64.RawURLEncoding.DecodeString(request.Response.AuthenticatorData)
if err != nil {
    ctx.JSON(
        http.StatusBadRequest,
        api.CommonResponse{
            Status:       "failed",
            ErrorMessage: "failed to decode authenticatorData, error: " + err.Error(),
        },
    )
    return
}
```

#### Signature

```go
authenticatorSignature, err := base64.RawURLEncoding.DecodeString(request.Response.Signature)
if err != nil {
    ctx.JSON(
        http.StatusBadRequest,
        api.CommonResponse{
            Status:       "failed",
            ErrorMessage: "failed to decode signature, error: " + err.Error(),
        },
    )
    return
}
```

#### UserHandle

```go
authenticatorUserHandle, err := base64.RawURLEncoding.DecodeString(request.Response.UserHandle)
if err != nil {
    ctx.JSON(
        http.StatusBadRequest,
        api.CommonResponse{
            Status:       "failed",
            ErrorMessage: "failed to decode userHandle, error: " + err.Error(),
        },
    )
    return
}
```

因為上面將 Request body 中的 Response 從 `protocol.AuthenticatorAssertionResponse` 改成我們自己寫的 `AuthenticatorAssertionResponse`
所以下面 `protocol.CredentialAssertionResponse` 也需要進行修改

```go
car := protocol.CredentialAssertionResponse{
    PublicKeyCredential: protocol.PublicKeyCredential{
        Credential: protocol.Credential{
            ID:   request.Id,
            Type: request.Type,
        },
        RawID:                  []byte(request.Id),
        ClientExtensionResults: request.GetClientExtensionResults,
    },
    AssertionResponse: protocol.AuthenticatorAssertionResponse{
        AuthenticatorResponse: protocol.AuthenticatorResponse{
            ClientDataJSON: protocol.URLEncodedBase64(authenticatorClientDataJSON),
        },
        AuthenticatorData: protocol.URLEncodedBase64(authenticatorData),
        Signature:         protocol.URLEncodedBase64(authenticatorSignature),
        UserHandle:        protocol.URLEncodedBase64(authenticatorUserHandle),
    },
}
```

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363Jk9RGdO3o0.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363Jk9RGdO3o0.png)

再重新執行一次，就成功通過第七關了～

明天我們再繼續勇闖第八關！