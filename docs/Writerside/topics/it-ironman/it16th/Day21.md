# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 21 - 開始進行前後端串接 (1)

在前面 10 天，我們已經將 WebAuthn 前後端設計完成了，接下來就是要進行前後端串接了

會分成好幾天來分享串接過程中遇到的問題點，以及前面設計時，沒有考慮到或是有寫錯的地方

開始我們的串接之旅吧～

## 串接之旅 Part 1

### 將環境準備好

#### 後端

在開始串接前，要先記得將後端的 WebAuthn RP Server、ngrok 跟 PostgreSQL 資料庫 run 起來

##### Run WebAuthn RP Server

可以透過 Visual Studio Code 的偵錯功能或是在 Terminal 輸入下面指令

```sh
go run main.go
```

WebAuthn RP Server run 起來之後，接著使用 ngrok 取得一個免費公開的測試用 domain

```sh
ngrok http 8080 # 8080 換成在 route.go 中設定的 port 號
```

##### Run PostgreSQL 資料庫

PostgreSQL 資料庫我們是使用 Docker Compose 執行的，所以就使用 docker compose 的指令來執行即可

```sh
docker compose up -d # -d 是為了讓他在背景跑，不會阻塞住當前 Terminal
```

#### 前端

##### 設定 Signing & Capability 中的 Associated Domains

將 Signing & Capability 中的 Associated Domains 設為從 ngrok 取得的 domain，像是下面這樣

![https://ithelp.ithome.com.tw/upload/images/20240922/2014036395LLsYThEu.png](https://ithelp.ithome.com.tw/upload/images/20240922/2014036395LLsYThEu.png)

##### 設定 `RequestConfiguration.Host` 中 rpServer 的值

接著將 `RequestConfiguration.Host` 中的 `rpServer` 設為從 ngrok 取得的 domain

```swift
extension RequestConfiguration {

    // ...

    enum Host: String {

        case rpServer = "<設為從 ngrok 取得的 domain>"
    }
    
    // ...
}
```

### 開始測試囉

#### 串接第一關：AttestationOptionsResponse 不合

![https://ithelp.ithome.com.tw/upload/images/20240922/20140363u8zM9R228s.png](https://ithelp.ithome.com.tw/upload/images/20240922/20140363u8zM9R228s.png)

從錯誤來看是找不到 `excludeCredentials`，我們從後端來看的話，會發現是因為資料庫中，沒有已經完成註冊驗證的 Credential，所以會找不到需要排除的 Credential，也就沒有出現在後端回傳的 response body 中

那這個錯誤，我們可以從前端的 Response body 來下手，看到 `AttestationOptionsResponse` 這個 struct，裡面有一個 `excludeCredentials` 欄位，我們在型別那邊加上 `?` 讓他變成是一個 Optional 的欄位，像是下面這樣

```swift
struct AttestationOptionsResponse: Decodable {
    
    let status: String
    
    let errorMessage: String
    
    let rp: RelyingParty
    
    let user: UserEntity
    
    let challenge: String
    
    let pubKeyCredParams: [PubKeyCredParam]
    
    let timeout: Int
    
    let excludeCredentials: [ExcludeCredential]? // 這邊加上 ? 讓他變成是 Optional 的
    
    let authenticatorSelection: AuthenticatorSelectionCriteria
    
    let attestation: String
    
    // ...
}
```

修改完之後，原本的錯誤已經解決了，但又遇到新的錯誤了

![https://ithelp.ithome.com.tw/upload/images/20240922/20140363Yi6BCHxt9x.png](https://ithelp.ithome.com.tw/upload/images/20240922/20140363Yi6BCHxt9x.png)

從錯誤來看，這次是在 `authenticatorSelection` 欄位中找不到 `authenticatorAttachment` 欄位
從後端的 Response 來看，確實是沒有 `authenticatorSelection` 這個欄位的

![https://ithelp.ithome.com.tw/upload/images/20240922/201403633z3maWKab5.png](https://ithelp.ithome.com.tw/upload/images/20240922/201403633z3maWKab5.png)

那這個我們可以從後端來處理

首先，我們看到 `api/attestation.go` 中的 `CredentialCreationOptionsRequest`
將 `AuthenticatorSelection` 從原先定義的 `map[string]interface{}` 更改為 `protocol.AuthenticatorSelection`

```go
type CredentialCreationOptionsRequest struct {
	Username               string                          `json:"username"`
	DisplayName            string                          `json:"displayName"`
	AuthenticatorSelection protocol.AuthenticatorSelection `json:"authenticatorSelection,omitempty"`
	Attestation            string                          `json:"attestation"`
}
```

接著再看到 `controller/attestation.go` 的 `StartAttestationHandler`
我們多設定一個 go-webauthn 的 Registration Option，叫做 `authenticatorSelectionOption`

```go
authenticatorSelectionOption := goWebAuthn.WithAuthenticatorSelection(request.AuthenticatorSelection)
```

接著在呼叫 `BeginRegistration` 的時候傳入

```go
options, sessionData, err := webauthn.WebAuthn.BeginRegistration(user, excludeCredentialsOption, authenticatorSelectionOption)
```

修改完之後，原本的錯誤已經解決了，但接著又遇到新的錯誤了

![https://ithelp.ithome.com.tw/upload/images/20240922/20140363p372EBWNl9.png](https://ithelp.ithome.com.tw/upload/images/20240922/20140363p372EBWNl9.png)

從錯誤來看，這次是找不到 `attachment` 欄位
從後端的 Response 來看，確實是沒有 `attachment` 這個欄位的

![https://ithelp.ithome.com.tw/upload/images/20240922/20140363xR8vqANLbv.png](https://ithelp.ithome.com.tw/upload/images/20240922/20140363xR8vqANLbv.png)

那這個我們一樣可以從後端進行處裡，同樣看到 `StartAttestationHandler`
我們多設定一個 go-webauthn 的 Registration Option，叫做 `attachmentOption`

```go
attestationOption := goWebAuthn.WithConveyancePreference(protocol.ConveyancePreference(request.Attestation))
```

接著在呼叫 `BeginRegistration` 的時候傳入

```go
options, sessionData, err := webauthn.WebAuthn.BeginRegistration(
    user,
    excludeCredentialsOption,
    authenticatorSelectionOption,
    attestationOption,
)
```

在上面寫了三個 go-webauthn 的 Registration Option，那有沒有辦法可以合在一起，不用分開寫呢？

答案是有的！

我們只要改成像下面這樣宣告，並將原先的三個 Registration Option 在這邊一起進行設定，就可以了

```go
registrationOptions := func (options *protocol.PublicKeyCredentialCreationOptions) {
    options.CredentialExcludeList = user.CredentialExcludeList()
    options.AuthenticatorSelection = request.AuthenticatorSelection
    options.Attestation = protocol.ConveyancePreference(request.Attestation)
}
```

然後一樣在 `BeginRegistration` 傳入

```go
options, sessionData, err := webauthn.WebAuthn.BeginRegistration(user, registrationOptions)
```

為什麼可以這樣改寫呢～這是因為 go-webauthn 的 Registration Option 本身就是一個 typealias
從 go-webauthn 的 source code 中可以看到

```go
type RegistrationOption func(*protocol.PublicKeyCredentialCreationOptions)
```

今天通過了第一關，已經可以與 Apple Passkeys API 進行互動了，但後面還存在著未知挑戰

我們明天繼續勇闖第二關！