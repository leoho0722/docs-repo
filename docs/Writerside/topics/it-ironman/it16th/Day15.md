# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 15 - 實作 WebAuthn Authentication

昨天我們實作了 WebAuthn Registration，今天我們要繼續實作 WebAuthn Authentication 的部分

一樣先開啟 controller 資料夾中的 `assertion.go` 檔案

## 在 Controller 中進行對應 Authentication 流程處理

### 產生登入資訊 (WebAuthn 第三支 API) {id="webauthn-api_1"}

首先，我們先將 http request 解碼成定義好的 `api.CredentialGetOptionsRequest` 物件

接著，透過 request 中的 `Username` 欄位在資料庫中進行搜尋，找出對應的使用者 (`foundUser`)

接著呼叫 `go-webauthn` 提供的 `BeginLogin` 開始產生登入資訊

產生登入資訊後，接著將從資料庫中找出的使用者的 Challenge 更新為產生出來的 challenge

可以看到這邊有一個 sessionData，這個是當次進行 WebAuthn Authentication 產生的 session，在驗證註冊資訊時，會需要用來跟 Authenticator 回傳的資訊進行驗證

最後將產生出來的登入資訊進行回傳

```go
var assertionSessionData *goWebAuthn.SessionData

func StartAssertionHandler(ctx *gin.Context) {
	fmt.Println("call /assertion/options")

	var request *api.CredentialGetOptionsRequest
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

	foundUser, err := database.GetUserByName(request.Username)
	if err != nil {
		ctx.JSON(
			http.StatusInternalServerError,
			api.CommonResponse{
				Status:       "failed",
				ErrorMessage: "failed to get user by name, error: " + err.Error(),
			},
		)
		return
	}
	fmt.Println("get user by name success")

	options, sessionData, err := webauthn.WebAuthn.BeginLogin(foundUser)
	if err != nil {
		ctx.JSON(
			http.StatusInternalServerError,
			api.CommonResponse{
				Status:       "failed",
				ErrorMessage: "failed to begin login, error: " + err.Error(),
			},
		)
		return
	}
	fmt.Println("begin login success")

	assertionSessionData = sessionData

	err = database.UpdateUser(
		foundUser, &database.User{
			Challenge: options.Response.Challenge.String(),
		},
	)
	if err != nil {
		ctx.JSON(
			http.StatusInternalServerError,
			api.CommonResponse{
				Status:       "failed",
				ErrorMessage: "failed to update user, error: " + err.Error(),
			},
		)
		return
	}
	fmt.Println("update user success")

	ctx.JSON(
		http.StatusOK,
		api.CredentialGetOptionsResponse{
			CommonResponse: api.CommonResponse{
				Status:       "success",
				ErrorMessage: "",
			},
			PublicKeyCredentialRequestOptions: options.Response,
		},
	)
}
```

### 驗證登入資訊 (WebAuthn 第四支 API)

經過 Authenticator 進行驗證後，接著就要來驗證 Authenticator 回傳的資訊

這邊一樣先將 http request 解碼成定義好的 `api.AuthenticatorAssertionResponseRequest` 物件

接著先將 Authenticator 回傳的 ClientDataJSON 進行 base64 URL no padding 解碼，這邊是透過 Go 內建的 encoding/base64 函式庫進行解碼

接下來再將解碼出來的內容，透過 json.Unmarshal 轉換成 map，大概會長得像下面這樣

```json
{"challenge":"NxyZopwVKbFl7EnnMae_5Fnir7QJ7QWp1UFUKjFHlfk","origin":"https://0a9c-2001-b011-2015-1886-24ec-4ea0-b6e4-1aea.ngrok-free.app","type":"webauthn.get"}
```

可以看到 challenge 也在其中，那麼我們就可以透過 challenge 在資料庫中找到對應的使用者 (`foundUser`)

接著使用 request body 來建立 `protocol.CredentialAssertionResponse` 物件，並進行解析，得到解析後的 `protocol.ParsedCredentialAssertionData` (`pca`)

再來使用 `foundUser`、`sessionData`、`pca` 驗證 WebAuthn Authentication

```go
func FinishAssertionHandler(ctx *gin.Context) {
	fmt.Println("call /assertion/result")

	var request *api.AuthenticatorAssertionResponseRequest
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
	fmt.Println("Parse request success")
	reqBody := utils.PrintJSON(request)
	fmt.Println("Request body: ", reqBody)

	var authenticatorClientDataJSON []byte
	_, err := base64.RawURLEncoding.Decode(authenticatorClientDataJSON, request.Response.ClientDataJSON)
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
		fmt.Println("get user by challenge success")

		car := protocol.CredentialAssertionResponse{
			PublicKeyCredential: protocol.PublicKeyCredential{
				Credential: protocol.Credential{
					ID:   request.Id,
					Type: request.Type,
				},
				RawID:                  []byte(request.Id),
				ClientExtensionResults: request.GetClientExtensionResults,
			},
			AssertionResponse: request.Response,
		}
		pca, err := car.Parse()
		if err != nil {
			ctx.JSON(
				http.StatusInternalServerError,
				api.CommonResponse{
					Status:       "failed",
					ErrorMessage: "failed to parse assertion response, error: " + err.Error(),
				},
			)
			return
		}
		fmt.Println("parse assertion response success")

		_, err = webauthn.WebAuthn.ValidateLogin(foundUser, *assertionSessionData, pca)
		if err != nil {
			ctx.JSON(
				http.StatusInternalServerError,
				api.CommonResponse{
					Status:       "failed",
					ErrorMessage: "failed to validate login, error: " + err.Error(),
				},
			)
			return
		}
		fmt.Println("validate login success")

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

今天實作了 WebAuthn Authentication 流程～

還記得昨天在實作 WebAuthn Registration 的時候有提到 iOS 和 Android 嗎？
如果要讓 iOS 和 Android App 可以進行 WebAuthn 流程的話，會需要在 Server 端的 `.well-known` 路徑下，放置對應的檔案，供對應平台系統讀取

明天就來介紹說，如何在 Server 端處理對應系統的 `.well-known` 檔案～