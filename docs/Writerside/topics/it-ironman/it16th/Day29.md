# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 29 - 開始進行前後端串接 (9)

昨天通過了第八關之後，今天要來接續第九關

## 串接之旅 Part 9

### 開始測試囉

#### 串接第九關：failed to validate login, error: User does not own the credential returned

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363qTQVkLerJQ.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363qTQVkLerJQ.png)

從錯誤來看，是當前要進行 WebAuthn Authentication 的使用者，在後端沒有找到對應的 Credential 可以進行驗證

我們從 go-webauthn source code 來判斷有哪些可能性

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363FF5VyrSkLT.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363FF5VyrSkLT.png)

看起來是 `protocol.CredentialAssertionResponse` 中的 `RawID` 與 session 中的 allowCredentials 裡的 credential id 對不起來所導致的

那這個我們從後端進行修改

我們新增一個資料夾，叫做 `utils`，並在裡面新增一個檔案 `base64.go`
接著在裡面新增下面的程式碼

```go
package utils

import (
	"encoding/base64"
	"strings"
)

func convertToBase64StdEncoding(base64URLEncoded string) string {
	base64StdEncoded := base64URLEncoded
	base64StdEncoded = strings.Replace(base64StdEncoded, "-", "+", -1)
	base64StdEncoded = strings.Replace(base64StdEncoded, "_", "/", -1)

	if len(base64StdEncoded)%4 != 0 {
		base64StdEncoded += strings.Repeat("=", 4-len(base64StdEncoded)%4)
	}

	return base64StdEncoded
}

func DecodeToBase64StdEncoding(src string) ([]byte, error) {
	base64StdEncoded := convertToBase64StdEncoding(src)
	base64StdDecoded, err := base64.StdEncoding.DecodeString(base64StdEncoded)

	return []byte(string(base64StdDecoded)), err
}
```

接下來再到 `controller/assertion.go` 中的 `FinishAssertionHandler`

我們將前端提供的 Credential ID 使用上面寫的 Function 轉換成 Credential Raw ID，像是下面這樣

```go
credentialRawID, err := utils.DecodeToBase64StdEncoding(request.Id)
if err != nil {
    ctx.JSON(
        http.StatusBadRequest,
        api.CommonResponse{
            Status:       "failed",
            ErrorMessage: "failed to encode credential raw id, error: " + err.Error(),
        },
    )
    return
}
```

接著用轉換出來的 `credentialRawID` 取代原本在 `protocol.CredentialAssertionResponse` 中的 `RawID`

```go
car := protocol.CredentialAssertionResponse{
    PublicKeyCredential: protocol.PublicKeyCredential{
        Credential: protocol.Credential{
            ID:   request.Id,
            Type: request.Type,
        },
        RawID:                  protocol.URLEncodedBase64(credentialRawID),
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

再重新執行一次，就成功通過第九關了～但又遇到最終大魔王，第十關！

#### 串接最終關：failed to validate login, error: userHandle and User ID does not match

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363PvVubDtfhB.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363PvVubDtfhB.png)

從錯誤來看，感覺是 Authenticator 回傳的 UserHandle 跟 User ID 不吻合

那我們看一下 go-webauthn source code 來判斷有哪些可能性

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363qImYUANa2I.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363qImYUANa2I.png)

看起來有可能資料庫那邊實作的 `WebAuthnID()` 跟 `protocol.CredentialAssertionResponse` 裡的 UserHandle 不吻合
那我們看一下 `protocol.CredentialAssertionResponse` 裡的 UserHandle 是什麼

```json
{"ID":"HeZ5B367bXiKOWqfrtC92_OePlg","Type":"public-key","rawId":"HeZ5B367bXiKOWqfrtC92/OePlg=","Response":{"CollectedClientData":{"type":"webauthn.get","challenge":"N3hrWk56WmFudHlnS2VxUlROWjZ2dDdiSXMyMkY4S0NQVW1xZ1BMNDF5MA","origin":"https://ab15-211-20-7-115.ngrok-free.app"},"AuthenticatorData":{"rpid":"XPS8nD4bj97jBqQcXhhmjwu+nLSR5/WOIRyKrL+1xR0=","flags":29,"sign_count":0,"att_data":{"aaguid":null,"credential_id":null,"public_key":null},"ext_data":null},"Signature":"MEUCIEs/427hEij5MNigWtUEZm4jFHB6DUMOF1pujUkqGTayAiEAwbfgAiOULh0NbxbDhqJjNNfeHEszK8qGeBv6cK5Mw54=","UserHandle":"bGVvaG8="},"Raw":{"id":"HeZ5B367bXiKOWqfrtC92_OePlg","type":"public-key","rawId":"HeZ5B367bXiKOWqfrtC92_OePlg","response":{"clientDataJSON":"eyJ0eXBlIjoid2ViYXV0aG4uZ2V0IiwiY2hhbGxlbmdlIjoiTjNocldrNTZXbUZ1ZEhsblMyVnhVbFJPV2paMmREZGlTWE15TWtZNFMwTlFWVzF4WjFCTU5ERjVNQSIsIm9yaWdpbiI6Imh0dHBzOi8vYWIxNS0yMTEtMjAtNy0xMTUubmdyb2stZnJlZS5hcHAifQ","authenticatorData":"XPS8nD4bj97jBqQcXhhmjwu-nLSR5_WOIRyKrL-1xR0dAAAAAA","signature":"MEUCIEs_427hEij5MNigWtUEZm4jFHB6DUMOF1pujUkqGTayAiEAwbfgAiOULh0NbxbDhqJjNNfeHEszK8qGeBv6cK5Mw54","userHandle":"bGVvaG8"}}}
```

可以看到 UserHandle 是 `bGVvaG8`

而資料庫那邊的 `WebAuthnID()` 回傳的是 `184e6301-6770-462b-8ee9-f0f50577914c`
看來知道問題點在哪裡了

那我們嘗試將 UserHandle 進行 base64RawURL 解碼看看，可以看到會是跟我們輸入的 username 相同

```go
userHandle := "bGVvaG8"
base64RawURLDecodedUserHandle, err := base64.RawURLEncoding.DecodeString(userHandle)
if err != nil {
    fmt.Println(err.Error())
}
fmt.Println(string(base64RawURLDecodedUserHandle))

// 輸出：leoho
```

所以我們可以將 `WebAuthnID()` 那邊修改成下面這樣

```go
func (u *User) WebAuthnID() []byte {
	return []byte(u.Name)
}
```

再重新執行一次，就成功通過最終關了～

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363u7GoXpBUS5.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363u7GoXpBUS5.png)

成功完成 WebAuthn Authentication 了！