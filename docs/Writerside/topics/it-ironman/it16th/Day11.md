# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 11 - 定義 WebAuthn API Request 和 Response body

在昨天，我們已經寫好的 API Route，今天我們要來定義 WebAuthn API 的 Request 和 Response body

## 定義 WebAuthn API Request 和 Response body

這邊我們參考 [FIDO Alliance Conformance Test Tools API Documentation](https://github.com/fido-alliance/conformance-test-tools-resources/blob/main/docs/FIDO2/Server/Conformance-Test-API.md) 來定義 body 內容

### 建立 api model 資料夾

我們先在 source code 資料夾建立一個 `api` 資料夾，來存放下面要定義的 api model
並在資料夾中新增 `attestation.go`、`assertion.go` 和 `common.go`  三個檔案，package 都是 api

![建立 api model 資料夾](https://ithelp.ithome.com.tw/upload/images/20240912/20140363cHZVvnO0vH.png)
▲ 建立 api model 資料夾

那麼該怎麼將 WebAuthn API Spec 轉換成 Go 的 struct 物件呢？讓我們接著看下去

### WebAuthn Registration

#### Credential Creation Options (對應到 `attestation.go`)

![Credential Creation Options](https://ithelp.ithome.com.tw/upload/images/20240912/20140363CbvbU8F7Zx.png)
▲ 圖截自 [FIDO Alliance Conformance Test Tools API Documentation](https://github.com/fido-alliance/conformance-test-tools-resources/blob/main/docs/FIDO2/Server/Conformance-Test-API.md)

```go
type CredentialCreationOptionsRequest struct {
	Username               string                 `json:"username"`
	DisplayName            string                 `json:"displayName"`
	AuthenticatorSelection map[string]interface{} `json:"authenticatorSelection"`
	Attestation            string                 `json:"attestation"`
}
```

轉換成 Go struct 物件後會長得像上面這樣

#### Authenticator Attestation Response (對應到 `attestation.go`)

![Authenticator Attestation Response](https://ithelp.ithome.com.tw/upload/images/20240912/20140363xKe5PWfdWo.png)
▲ 圖截自 [FIDO Alliance Conformance Test Tools API Documentation](https://github.com/fido-alliance/conformance-test-tools-resources/blob/main/docs/FIDO2/Server/Conformance-Test-API.md)

```go
package api

import "github.com/go-webauthn/webauthn/protocol"

type CredentialCreationOptionsRequest struct {
	Username               string                 `json:"username"`
	DisplayName            string                 `json:"displayName"`
	AuthenticatorSelection map[string]interface{} `json:"authenticatorSelection"`
	Attestation            string                 `json:"attestation"`
}

type CredentialCreationOptionsResponse struct {
	CommonResponse
	protocol.PublicKeyCredentialCreationOptions
}

type AuthenticatorAttestationResponseRequest struct {
	Id                        string                           `json:"id"`
	Response                  AuthenticatorAttestationResponse `json:"response"`
	GetClientExtensionResults map[string]interface{}           `json:"getClientExtensionResults"`
	Type                      string                           `json:"type"`
}

type AuthenticatorAttestationResponse struct {
	AttestationObject string `json:"attestationObject"`
	ClientDataJSON    string `json:"clientDataJSON"`
}
```

轉換成 Go struct 物件後會長得像上面這樣

### WebAuthn Authentication

#### Credential Get Options (對應到 `assertion.go`)

![Credential Get Options](https://ithelp.ithome.com.tw/upload/images/20240912/20140363kvPDh9xAYS.png)
▲ 圖截自 [FIDO Alliance Conformance Test Tools API Documentation](https://github.com/fido-alliance/conformance-test-tools-resources/blob/main/docs/FIDO2/Server/Conformance-Test-API.md)

```go
type CredentialGetOptions struct {
	Username         string `json:"username"`
	UserVerification string `json:"userVerification"`
}
```

轉換成 Go struct 物件後會長得像上面這樣

#### Authenticator Assertion Response (對應到 `assertion.go`)

![Authenticator Assertion Response](https://ithelp.ithome.com.tw/upload/images/20240912/201403638fX2eWbx5F.png)
▲ 圖截自 [FIDO Alliance Conformance Test Tools API Documentation](https://github.com/fido-alliance/conformance-test-tools-resources/blob/main/docs/FIDO2/Server/Conformance-Test-API.md)

```go
package api

import "github.com/go-webauthn/webauthn/protocol"

type CredentialGetOptionsRequest struct {
	Username         string `json:"username"`
	UserVerification string `json:"userVerification"`
}

type CredentialGetOptionsResponse struct {
	CommonResponse
	protocol.PublicKeyCredentialRequestOptions
}

type AuthenticatorAssertionResponseRequest struct {
	Id                        string                                  `json:"id"`
	Response                  protocol.AuthenticatorAssertionResponse `json:"response"`
	GetClientExtensionResults map[string]interface{}                  `json:"getClientExtensionResults"`
	Type                      string                                  `json:"type"`
}
```

轉換成 Go struct 物件後會長得像上面這樣

### Common (對應到 `common.go`)

因為有共用的 struct 物件，所以將他們寫在 common 裡面

![Common Response body](https://ithelp.ithome.com.tw/upload/images/20240912/20140363ALVJua7S48.png)
▲ 圖截自 [FIDO Alliance Conformance Test Tools API Documentation](https://github.com/fido-alliance/conformance-test-tools-resources/blob/main/docs/FIDO2/Server/Conformance-Test-API.md)

```go
type CommonResponse struct {
	Status       string `json:"status"`
	ErrorMessage string `json:"errorMessage"`
}
```

轉換成 Go struct 物件後會長得像上面這樣

## 參考資料

> [WebAuthn Spec 5.4 Options for Credential Creation](https://www.w3.org/TR/webauthn-3/#dictionary-makecredentialoptions)
> [WebAuthn Spec 5.5. Options for Assertion Generation](https://www.w3.org/TR/webauthn-3/#dictionary-assertion-options)

今天將 WebAuthn API 的 Request、Response body 定義好之後，明天就可以來繼續實作了～