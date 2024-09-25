# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 14 - 實作 WebAuthn Registration

昨天設計好 Repository 後，今天就要來實作 WebAuthn Registration 了

要實作 WebAuthn Registration 有幾個部分要進行

* 建立 WebAuthn 物件
* 實作在 `go-webauthn` Library 中定義的 User method
* 在 Controller 中進行對應 Registration 流程處理

下面就來一一帶大家實作

## 建立 WebAuthn 物件

我們新增一個資料夾叫做 webauthn，並在資料夾中新增一個檔案叫做 `webauthn.go`，package 為 webauthn

在檔案中新增一個 Function 叫做 `NewRPServer`，用來初始化 WebAuthn RP Server

可以從下面的 sample code 看到，我們需要透過一個 WebAuthn Config 來建立出 WebAuthn 物件

在 WebAuthn Config 物件中，有一個比較重要的欄位，叫做 `RPOrigins`，這個欄位指的是，RP Server 可以驗證的來源，像是 iOS 跟 Android 的 origin 就會不一樣。

iOS 的會是 RP Server 的 origin
而 Android 的如果是使用 Google Play services 的 [Fido2ApiClient](https://developers.google.com/android/reference/com/google/android/gms/fido/fido2/Fido2ApiClient)，則會是 `android:apk-key-hash:` 開頭的。
不確定如果是使用 Android Developers Documentation 中的 [CredentialManager API](https://developer.android.com/identity/sign-in/credential-manager) 會不會也是 `android:apk-key-hash:` 開頭，如果有大神知道的話，歡迎在底下留言補充！

所以說，如果要同時讓 RP Server 可以支援來自 iOS 與 Android 的 WebAuthn 請求的話，就需要在 RPOrigins 這個欄位將兩個平台的 origin 進行新增

```go
package webauthn

import (
	"fmt"

	"github.com/go-webauthn/webauthn/webauthn"
)

var WebAuthn *webauthn.WebAuthn

func NewRPServer() {
	c := &webauthn.Config{
		RPID:          "0a9c-2001-b011-2015-1886-24ec-4ea0-b6e4-1aea.ngrok-free.app",
		RPDisplayName: "it16th-webauthn-rp-server",
		RPOrigins:     []string{"https://0a9c-2001-b011-2015-1886-24ec-4ea0-b6e4-1aea.ngrok-free.app"},
	}

	webAuthn, err := webauthn.New(c)
	if err != nil {
		fmt.Println(err)
	}

	WebAuthn = webAuthn
}
```

在 `webauthn.go` 檔案中新增好上面的程式碼後，接著在 `main.go` 中進行呼叫

```go
package main

import (
	"leoho.io/it16th-webauthn-rp-server/database"
	"leoho.io/it16th-webauthn-rp-server/route"
	"leoho.io/it16th-webauthn-rp-server/webauthn"
)

func main() {
	database.Connect()
	webauthn.NewRPServer()
	route.NewRoute()
}
```

## 實作在 `go-webauthn` Library 中定義的 User method

建立好 WebAuthn 物件後，接著就來實作在 `go-webauthn` Library 中定義的相關 User method

我們要進行實作的一共有四個 method

* `func WebAuthnID() []byte`
* `func WebAuthnName() string`
* `func WebAuthnDisplayName() string`
* `func WebAuthnCredentials() []Credential`

下面就來進行一一實作

首先，先開啟 database 資料夾中的 `model.go`

### WebAuthnID

第一個要實作的是 `WebAuthnID` method
我們在資料表中有定義 ID 欄位，所以就直接轉成 []byte 後回傳即可

```go
func (u *User) WebAuthnID() []byte {
	return []byte(u.ID)
}
```

### WebAuthnName

第二個要實作的是 `WebAuthnName` method
我們在資料表中有定義 Name 欄位，一樣直接回傳即可

```go
func (u *User) WebAuthnName() string {
	return u.Name
}
```

### WebAuthnDisplayName

第三個要實作的是 `WebAuthnDisplayName` method
我們在資料表中有定義 DisplayName 欄位，也是一樣直接回傳即可

```go
func (u *User) WebAuthnDisplayName() string {
	return u.DisplayName
}
```

### WebAuthnCredentials

最後一個要實作的是 `WebAuthnCredentials` method

這個 method 相較於其他需實作的 method 而言，算是比較複雜的，因為要將資料庫中所有使用者的 Credential 取出並回傳，用在產生註冊資訊時，要排除掉的 Credentials，避免重複註冊

我們先取得資料庫中的所有使用者，並進行遍歷
接著，判斷使用者的 Credential 是否以 `` ` `` 開頭，或是為 `` `{}` ``，這是因為在後面將 Credential 儲存進資料庫時，會用此格式轉成字串進行儲存

再透過 Go 內建函式庫中的 `strconv.Unquote` 將 `` ` `` 去除後，進行一連串的 json marshal / unmarshal 編解碼轉換，就可以得到所有使用者的 Credential 了

```go
func (u *User) WebAuthnCredentials() []webauthn.Credential {
	credentials := []webauthn.Credential{}

	allUser, err := GetUsers()
	if err != nil {
		fmt.Println(err.Error())
		return credentials
	}

	for _, user := range allUser {
		if !strings.HasPrefix(user.Credential, "`") || user.Credential == "`{}`" {
			continue
		}

		s, err := strconv.Unquote(string(user.Credential))
		if err != nil {
			fmt.Println(err.Error())
			return credentials
		}
		var credsMap map[string]interface{}
		err = json.Unmarshal([]byte(s), &credsMap)
		if err != nil {
			fmt.Println(err.Error())
			return credentials
		}
		credsJson, err := json.Marshal(credsMap)
		if err != nil {
			fmt.Println(err.Error())
			return credentials
		}
		var cred webauthn.Credential
		err = json.Unmarshal(credsJson, &cred)
		if err != nil {
			fmt.Println(err.Error())
			return credentials
		}
		credentials = append(credentials, cred)
	}

	return credentials
}
```

### CredentialExcludeList

還有一個不是在 `go-webauthn` 中定義的 User method，但後續進行註冊時，會呼叫到的 User method
也就是 `CredentialExcludeList` method，主要用來取得要被排除掉不能再進行註冊的 Credential

透過呼叫上面實作的 `WebAuthnCredentials` method 將 `[]webauthn.Credential` 來轉換成 `[]protocol.CredentialDescriptor` 物件

```go
func (u *User) CredentialExcludeList() []protocol.CredentialDescriptor {
	var credentialExcludeList []protocol.CredentialDescriptor
	for _, credential := range u.WebAuthnCredentials() {
		descriptor := credential.Descriptor()
		credentialExcludeList = append(credentialExcludeList, descriptor)
	}
	return credentialExcludeList
}
```

## 在 Controller 中進行對應 Registration 流程處理

再來要在 Controller 中進行流程處理，所以開啟 controller 資料夾中的 `attestation.go` 檔案

### 產生註冊資訊 (WebAuthn 第一支 API) {id="webauthn-api_1"}

首先我們先將 http request 解碼成定義好的 `api.CredentialCreationOptionsRequest` 物件

接著先建立出 `database.User` 物件，再將 excludeCredential 設為 `user.CredentialExcludeList()`，再將其作為 WebAuthn Registration Options

接著呼叫 `go-webauthn` 提供的 `BeginRegistration` 開始產生註冊資訊

產生註冊資訊後，接著將 `user.Challenge` 更新為產生出來的 challenge，並先將 `user.Credential` 設為 `` `{}` ``，後續在驗證註冊資訊時，再進行更新為 Authenticator 提供的 Credential

可以看到這邊有一個 `sessionData`，這個是當次進行 WebAuthn Registration 產生的 session，在驗證註冊資訊時，會需要用來跟 Authenticator 回傳的資訊進行驗證

最後將產生出來的註冊資訊進行回傳

```go
var attestationSessionData *goWebAuthn.SessionData

func StartAttestationHandler(ctx *gin.Context) {
	fmt.Println("call /attestation/options")

	var request *api.CredentialCreationOptionsRequest
	if err := ctx.ShouldBindJSON(&request); err != nil {
		ctx.JSON(
			http.StatusBadRequest,
			api.CommonResponse{
				Status:       "failed",
				ErrorMessage: "failed to parse request body, error: " + err.Error(),
			},
		)
		return
	}

	user := &database.User{
		ID:          uuid.New().String(),
		Name:        request.Username,
		DisplayName: request.DisplayName,
	}
	excludeCredentialsOption := goWebAuthn.WithExclusions(user.CredentialExcludeList())
	options, sessionData, err := webauthn.WebAuthn.BeginRegistration(user, excludeCredentialsOption)
	if err != nil {
		fmt.Println("begin registration failed, error: ", err.Error())
		ctx.JSON(
			http.StatusInternalServerError,
			api.CommonResponse{
				Status:       "failed",
				ErrorMessage: "failed to create credential creation options, error: " + err.Error(),
			},
		)
		return
	}
	fmt.Println("begin registration success")

	user.Challenge = options.Response.Challenge.String()
	user.Credential = "`" + "{}" + "`"

	if err := database.CreateUser(user); err != nil {
		ctx.JSON(
			http.StatusInternalServerError,
			api.CommonResponse{
				Status:       "failed",
				ErrorMessage: "failed to create user, error: " + err.Error(),
			},
		)
		return
	}
	fmt.Println("create user success")

	attestationSessionData = sessionData

	ctx.JSON(
		http.StatusOK,
		api.CredentialCreationOptionsResponse{
			CommonResponse: api.CommonResponse{
				Status:       "success",
				ErrorMessage: "",
			},
			PublicKeyCredentialCreationOptions: options.Response,
		},
	)
}
```

### 驗證註冊資訊 (WebAuthn 第二支 API)

經過 Authenticator 進行驗證後，接著就要來驗證 Authenticator 回傳的資訊

這邊一樣先將 http request 解碼成定義好的 `api.AuthenticatorAttestationResponseRequest` 物件

接著先將 Authenticator 回傳的 `ClientDataJSON` 進行 base64 URL no padding 解碼，這邊是透過 Go 內建的 `encoding/base64` 函式庫進行解碼

接下來再將解碼出來的內容，透過 `json.Unmarshal` 轉換成 map，大概會長得像下面這樣

```json
{"challenge":"NxyZopwVKbFl7EnnMae_5Fnir7QJ7QWp1UFUKjFHlfk","origin":"https://0a9c-2001-b011-2015-1886-24ec-4ea0-b6e4-1aea.ngrok-free.app","type":"webauthn.create"}
```

可以看到 challenge 也在其中，那麼我們就可以透過 challenge 在資料庫中找到對應的使用者 (`foundUser`)

接著使用 request body 來建立 `protocol.CredentialCreationResponse` 物件，並進行解析，得到解析後的 `protocol.ParsedCredentialCreationData` (`pcc`)

再來使用 `foundUser`、`sessionData`、`pcc` 建立 WebAuthn Credential

Credential 建立好之後，透過 json marshal 編碼成 []byte 並轉成 string 型別，再以 `` `<Credential>` `` 的形式更新原先資料庫中的預設值

```go
func FinishAttestationHandler(ctx *gin.Context) {
	fmt.Println("call /attestation/result")

	var request *api.AuthenticatorAttestationResponseRequest
	if err := ctx.ShouldBindJSON(&request); err != nil {
		ctx.JSON(
			http.StatusBadRequest,
			api.CommonResponse{
				Status:       "failed",
				ErrorMessage: "failed to parse request body, error: " + err.Error(),
			},
		)
		return
	}

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
	fmt.Println("Decode clientDataJSON success")
	if challenge, ok := clientDataJSON["challenge"].(string); !ok || challenge != attestationSessionData.Challenge {
		ctx.JSON(
			http.StatusBadRequest,
			api.CommonResponse{
				Status:       "failed",
				ErrorMessage: "challenge mismatch",
			},
		)
		return
	} else {
		foundUser, err := database.GetUserByChallenge(challenge)
		if err != nil {
			ctx.JSON(
				http.StatusInternalServerError,
				api.CommonResponse{
					Status:       "failed",
					ErrorMessage: "failed to get user by challenge, error: " + err.Error(),
				},
			)
			return
		}
		fmt.Println("Get user by challenge success")

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
				AttestationObject: protocol.URLEncodedBase64(request.Response.AttestationObject),
				AuthenticatorResponse: protocol.AuthenticatorResponse{
					ClientDataJSON: authenticatorClientDataJSON,
				},
			},
		}

		pcc, err := ccr.Parse()
		if err != nil {
			ctx.JSON(
				http.StatusBadRequest,
				api.CommonResponse{
					Status:       "failed",
					ErrorMessage: "failed to parse credential creation response, error: " + err.Error(),
				},
			)
			return
		}
		fmt.Println("Parse credential creation response success")

		credential, err := webauthn.WebAuthn.CreateCredential(foundUser, *attestationSessionData, pcc)
		if err != nil {
			ctx.JSON(
				http.StatusInternalServerError,
				api.CommonResponse{
					Status:       "failed",
					ErrorMessage: "failed to create credential, error: " + err.Error(),
				},
			)
			return
		}
		fmt.Println("Create credential success")

		credentialJSON, err := json.Marshal(credential)
		if err != nil {
			ctx.JSON(
				http.StatusInternalServerError,
				api.CommonResponse{
					Status:       "failed",
					ErrorMessage: "failed to marshal credential, error: " + err.Error(),
				},
			)
			return
		}
		fmt.Println("Marshal credential success")

		if err = database.UpdateUser(
			foundUser, database.User{
				Credential: "`" + string(credentialJSON) + "`",
			},
		); err != nil {
			ctx.JSON(
				http.StatusInternalServerError,
				api.CommonResponse{
					Status:       "failed",
					ErrorMessage: "failed to update user, error: " + err.Error(),
				},
			)
			return
		}
		fmt.Println("Update user credential success")

		ctx.JSON(
			http.StatusOK,
			api.CommonResponse{
				Status:       "success",
				ErrorMessage: "",
			},
		)
	}
}
```

今天實作了 WebAuthn Registration 流程，明天再接著繼續實作 WebAuthn Authentication 流程～