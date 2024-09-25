# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 10 - 建立 WebAuthn RP Server API Route

昨天準備好 Go 的開發環境後，今天就要來建立 WebAuthn RP Server 啦

## 建立 WebAuthn RP Server API Route

### 建立專案

使用 Visual Studio Code 建立一個資料夾，來存放 RP Server 的 source code

```shell
go mod init <GO MODULE 名稱>
touch main.go
```

這邊我以 `leoho.io/it16th-webauthn-rp-server` 作為 `GO MODULE 名稱`

![go mod init](https://ithelp.ithome.com.tw/upload/images/20240911/20140363zMEbvZlSA1.png)
▲ go mod init

接著新增 `main.go` 的檔案

![新增 main.go](https://ithelp.ithome.com.tw/upload/images/20240911/20140363n4vGGBqbRQ.png)
▲ 新增 main.go

### 安裝相關套件

建立 API Route 我們會需要使用 [gin](https://github.com/gin-gonic/gin) 這個套件
我們透過 go get 來進行安裝，`go get -u github.com/gin-gonic/gin`

> 可參考去年的 [【하나, 둘, ready, get set, go】Day 24 - 使用 Gin 撰寫第一支 Web Backend API](https://ithelp.ithome.com.tw/articles/10317339)

### 建立 route、controller 資料夾

接著建立用來存放 API Route 和處理 Request 請求的 Controller 資料夾

![建立 route、controller 資料夾](https://ithelp.ithome.com.tw/upload/images/20240911/20140363wGk6HrkwgN.png)
▲ 建立 route、controller 資料夾

### 建立 API Route

我們先看到 route 資料夾，在裡面建立一個 `route.go` 的檔案，其 package 為 route，建立好之後，會像下面這樣

![建立 route.go](https://ithelp.ithome.com.tw/upload/images/20240911/20140363hZDXp0qoz9.png)
▲ 建立 route.go

接著新增一個用來設定 API Route 的 Function，這邊我取名為 `NewRoute`，這個 Function 會在 `main.go` 裡的 `main()` 裡面呼叫

接著在 `NewRoute` Function 中新增下面的程式碼，新增後整個 `route.go` 會長得像下面這樣

```go
package route

import (
	"github.com/gin-gonic/gin"
    
	"leoho.io/it16th-webauthn-rp-server/controller"
)

func NewRoute() {
    // 1.
	app := gin.Default()

    // 2.
	attestation := app.Group("/attestation")
	{
		attestation.POST("/options", controller.StartAttestationHandler)
		attestation.POST("/result", controller.FinishAttestationHandler)
	}
    
    // 3.
	assertion := app.Group("/assertion")
	{
		assertion.POST("/options", controller.StartAssertionHandler)
		assertion.POST("/result", controller.FinishAssertionHandler)
	}

    // 4.
	app.Run("0.0.0.0:8080")
}
```

1. 建立一個 gin 預設路由物件
2. 建立 attestation 路由群組，管理 WebAuthn 註冊流程的 API Route
3. 建立 assertion 路由群組，管理 WebAuthn 驗證流程的 API Route
4. 使用 `0.0.0.0:8080` 來建立 API Route，在區網內，可以使用主機的 IP:8080 來呼叫到 API Route

上面看到的 controller，是實際要處理 Request 的地方，接下來我們來繼續新增！

### 建立 Controller

在剛剛建立的 controller 資料夾內，新增兩個檔案，分別為 `attestation.go` 和 `assertion.go`，兩個檔案的 package 都是 controller

`attestation.go` 負責處理 WebAuthn 註冊流程
`assertion.go` 負責處理 WebAuthn 驗證流程

![建立對應的 controller go 檔案](https://ithelp.ithome.com.tw/upload/images/20240911/2014036321i6ENAfHN.png)
▲ 建立對應的 controller go 檔案

#### attestation.go

我們新增兩個 Function 來處理 WebAuthn 註冊流程，分別為 `StartAttestationHandler` 和 `FinishAttestationHandler`

```go
package controller

import "github.com/gin-gonic/gin"

func StartAttestationHandler(ctx *gin.Context) {
    // 1.
}

func FinishAttestationHandler(ctx *gin.Context) {
    // 2.
}
```

1. StartAttestationHandler 負責處理 WebAuthn 產生註冊資訊的請求，也就是 `Credential Creation Options`，可以透過呼叫 `http://<主機 IP:8080/attestation/options>` 來與 API 互動
2. FinishAttestationHandler 負責處理 WebAuthn 驗證註冊資訊的請求，也就是 `Authenticator Attestation Response`，可以透過呼叫 `http://<主機 IP:8080/attestation/result>` 來與 API 互動

#### assertion.go

我們新增兩個 Function 來處理 WebAuthn 驗證流程，分別為 `StartAssertionHandler` 和 `FinishAssertionHandler`

```go
package controller

import "github.com/gin-gonic/gin"

func StartAssertionHandler(ctx *gin.Context) {
    // 1.
}

func FinishAssertionHandler(ctx *gin.Context) {
    // 2.
}
```

1. StartAssertionHandler 負責處理 WebAuthn 產生登入資訊的請求，也就是 `Credential Get Options`，可以透過呼叫 `http://<主機 IP:8080/assertion/options>` 來與 API 互動
2. FinishAssertionHandler 負責處理 WebAuthn 驗證登入資訊的請求，也就是 `Authenticator Assertion Response`，可以透過呼叫 `http://<主機 IP:8080/assertion/result>` 來與 API 互動

今天我們將 WebAuthn 註冊 / 驗證流程的四支 API Route 都先建立好了，那麼明天就可以來撰寫邏輯的部份了～