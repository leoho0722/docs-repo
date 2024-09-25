# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 16 - 實作 iOS 和 Android 雙平台的 WebAuthn Server well-known

在前兩天帶大家實作了 WebAuthn Registration 和 WebAuthn Authentication 兩個流程

前面有提到說，前端會以 iOS App 作為示範，所以今天要來帶大家實作 WebAuthn Server well-known
下面會介紹 iOS 和 Android App 的設定方式，雙平台的設定方式大同小異，就讓我們繼續看下去吧～

## 實作 iOS 和 Android 雙平台的 WebAuthn Server well-known

### 設定 API Route

首先，設定雙平台對應的 API Route，先打開 route 資料夾中的 `route.go`
在 `NewRoute` Function 中新增下面這段

```go
	wellknown := app.Group("/.well-known")
	{
		wellknown.GET("/apple-app-site-association", controller.AppleWellKnownHandler)
		wellknown.GET("/assetlinks.json", controller.AndroidWellKnownHandler)
	}
```

新增後，整個 `NewRoute` 會長得像下面這樣

```go
func NewRoute() {
	app := gin.Default()

	wellknown := app.Group("/.well-known")
	{
		wellknown.GET("/apple-app-site-association", controller.AppleWellKnownHandler)
		wellknown.GET("/assetlinks.json", controller.AndroidWellKnownHandler)
	}

	attestation := app.Group("/attestation")
	{
		attestation.POST("/options", controller.StartAttestationHandler)
		attestation.POST("/result", controller.FinishAttestationHandler)
	}

	assertion := app.Group("/assertion")
	{
		assertion.POST("/options", controller.StartAssertionHandler)
		assertion.POST("/result", controller.FinishAssertionHandler)
	}

	app.Run("0.0.0.0:8080")
}
```

### iOS App

#### 設定 `apple-app-site-association`

接著看到 iOS App，如果要設定 Server 端的 well-known，需要將 `apple-app-site-association` 這個檔案放在 Server 的 `/.well-known` 路徑下

`apple-app-site-association` 是一個 json 格式內容的檔案，但是沒有副檔名
整個檔案可以參考 Apple Developer Documentation

> [Apple Developer Documentation](https://developer.apple.com/documentation/xcode/supporting-associated-domains)

在 WebAuthn well-known 需要的檔案內容會長得像下面這樣

```json
{
    "webcredentials": {
        "apps": [
            "<YOUR APPLE DEVELOPER TEAM ID>.com.example.it16th.fido2.frontend"
        ]
    }
}
```

撰寫好 `apple-app-site-association` 之後，我們將其放在 source code 目錄底下，待會在 Controller 中進行讀取

#### 設計對應的 Controller {id="controller_1"}

放置好 `apple-app-site-association` 之後，就要來設計對應的 Controller 了
我們在 controller 資料夾中，新增一個檔案叫做 `wellknown.go`，package 是 controller

首先，新增一個 Function 叫做 `func AppleWellKnownHandler(ctx *gin.Context)`

接著，透過 `os.ReadFile` 來讀取剛才寫好的 `apple-app-site-association`
再透過 json unmarshal 解碼成 map (`appleAppSiteAssociation`)

最後再將 `appleAppSiteAssociation` 作為 response body 進行回傳

```go
func AppleWellKnownHandler(ctx *gin.Context) {
	fmt.Println("call /.well-known/apple-app-site-association")

	appleAppSiteAssociationData, err := os.ReadFile("apple-app-site-association")
	if err != nil {
		ctx.JSON(
			http.StatusInternalServerError,
			api.CommonResponse{
				Status:       "failed",
				ErrorMessage: "failed to get apple-app-site-association",
			},
		)
		return
	}

	var appleAppSiteAssociation map[string]interface{}
	if err = json.Unmarshal(appleAppSiteAssociationData, &appleAppSiteAssociation); err != nil {
		ctx.JSON(
			http.StatusInternalServerError,
			api.CommonResponse{
				Status:       "failed",
				ErrorMessage: "failed to parse apple-app-site-association",
			},
		)
		return
	}

	ctx.JSON(
		http.StatusOK,
		appleAppSiteAssociation,
	)
}
```

### Android App

#### 設定 `assetlinks.json`

再來看到 Android App，如果要設定 Server 端的 well-known，需要將 `assetlinks.json` 這個檔案放在 Server 的 `/.well-known` 路徑下

`assetlinks.json` 是一個 json 格式內容的檔案
整個檔案可以參考 Google Developer Documentation

> [Google Developer Documentation](https://developers.google.com/identity/credential-sharing/example-multiple-websites-apps)

在 WebAuthn well-known 需要的檔案內容會長得像下面這樣

```json
[
    {
      "relation" : [
        "delegate_permission/common.handle_all_urls",
        "delegate_permission/common.get_login_creds"
      ],
      "target" : {
        "namespace" : "web",
        "site" : "YOUR WEBAUTHN RP SERVER ORIGIN"
      }
    },
    {
      "relation" : [
        "delegate_permission/common.handle_all_urls",
        "delegate_permission/common.get_login_creds"
      ],
      "target" : {
        "namespace" : "android_app",
        "package_name" : "com.example.it16th.fido2.frontend",
        "sha256_cert_fingerprints" : [
           "<GENERATED SHA256 CERT FINGERPRINTS>"
        ]
      }
    }
  ]
```

撰寫好 `assetlinks.json` 之後，我們將其放在 source code 目錄底下，待會在 Controller 中進行讀取

#### 設計對應的 Controller

放置好 `assetlinks.json` 之後，就要來設計對應的 Controller 了
我們繼續在 `wellknown.go` 中新增 Function，叫做 `func AndroidWellKnownHandler(ctx *gin.Context)`

接著，透過 `os.ReadFile` 來讀取剛才寫好的 `assetlinks.json`
再透過 json unmarshal 解碼成 map (`assetlinks`)

最後再將 `assetlinks` 作為 response body 進行回傳

```go
func AndroidWellKnownHandler(ctx *gin.Context) {
	fmt.Println("call /.well-known/assetlinks.json")

	assetlinksData, err := os.ReadFile("assetlinks.json")
	if err != nil {
		ctx.JSON(
			http.StatusInternalServerError,
			api.CommonResponse{
				Status:       "failed",
				ErrorMessage: "failed to get assetlinks.json",
			},
		)
		return
	}

	var assetlinks []map[string]interface{}
	if err = json.Unmarshal(assetlinksData, &assetlinks); err != nil {
		ctx.JSON(
			http.StatusInternalServerError,
			api.CommonResponse{
				Status:       "failed",
				ErrorMessage: "failed to parse assetlinks.json",
			},
		)
		return
	}
	ctx.JSON(
		http.StatusOK,
		assetlinks,
	)
}
```

今天新增了雙平台的 well-known 設定檔到我們設計的 WebAuthn RP Server 中
到目前為止，WebAuthn RP Server 的設計先告一段落，接下來幾天要來設計前端的 iOS App (**我的愛**)～