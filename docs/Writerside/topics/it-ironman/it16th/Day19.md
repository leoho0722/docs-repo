# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 19 - 實作 Apple Passkeys API (2)

昨天我們實作好 Apple Passkeys API 後，今天要來將 Passkeys API 回傳的 Authenticator 資料進行接值，以及設計前端的網路呼叫功能

# 處理 Authenticator 回傳的資料

昨天已經實作完呼叫 Apple Passkeys API 的部分，接下來需要將 Authenticator 回傳的資料進行處理

這邊透過 Delegation 的方式進行處理，所以下面就來定義相關的 Delegation

## PasskeysManagerDelegate

這個 Delegate 負責處理當 Passkeys API 發生錯誤時，要進行錯誤處理的部分
並作爲下面 `PasskeysRegistration` 和 `PasskeysAuthentication` 的父 Delegate

```swift
protocol PasskeysManagerDelegate: NSObjectProtocol {
    
    func passkeysManager(controller: ASAuthorizationController, 
                         didCompleteWithError error: Error)
}
```

並在 `PasskeysManager` 中宣告 `delegate` 弱引用變數，型別為 `PasskeysManagerDelegate?`

```swift
import AuthenticationServices
import Foundation

class PasskeysManager: NSObject {

    private let domain: String
    
    private let logger = Logger()
    
    weak var delegate: PasskeysManagerDelegate?
    
    init(domain: String) {
        self.domain = domain
    }
    
    // ...
}
```

### 呼叫點

```swift
extension PasskeysManager: ASAuthorizationControllerDelegate {

    // ...
    
    func authorizationController(controller: ASAuthorizationController,
                                 didCompleteWithError error: Error) {
        guard let authorizationError = error as? ASAuthorizationError else {
            logger.error("Unexpected authorization error: \(error.localizedDescription)")
            return
        }
        
        delegate?.passkeysManager(controller: controller,
                                  didCompleteWithError: authorizationError)
    }
}
```

## PasskeysRegistration

`PasskeysRegistration` 是用來將 Authenticator 回傳給 App 的 attestation 資料，進行資料傳遞的 Delegate

```swift
protocol PasskeysRegistration: PasskeysManagerDelegate {
    
    func passkeysManager(with credentialRegistration: ASAuthorizationPlatformPublicKeyCredentialRegistration)
}
```

### 呼叫點

接著在 `func authorizationController(controller: ASAuthorizationController, didCompleteWithAuthorization authorization: ASAuthorization)` 中進行呼叫，如下

```swift
extension PasskeysManager: ASAuthorizationControllerDelegate {
    
    func authorizationController(controller: ASAuthorizationController,
                                 didCompleteWithAuthorization authorization: ASAuthorization) {
        switch authorization.credential {
        case let credentialRegistration as ASAuthorizationPlatformPublicKeyCredentialRegistration:
            logger.log("A new passkey was registered: \(credentialRegistration)")
            
            (delegate as! PasskeysRegistration).passkeysManager(with: credentialRegistration)
        default:
            fatalError("Received unknown authorization type.")
        }
    }
    // ...
}
```

## PasskeysAuthentication

`PasskeysAuthentication` 是用來將 Authenticator 回傳給 App 的 assertion 資料，進行資料傳遞的 Delegate

```swift
protocol PasskeysAuthentication: PasskeysManagerDelegate {
    
    func passkeysManager(with credentialAssertion: ASAuthorizationPlatformPublicKeyCredentialAssertion)
}
```

### 呼叫點

接著在 `func authorizationController(controller: ASAuthorizationController, didCompleteWithAuthorization authorization: ASAuthorization)` 中進行呼叫，如下

```swift
extension PasskeysManager: ASAuthorizationControllerDelegate {
    
    func authorizationController(controller: ASAuthorizationController,
                                 didCompleteWithAuthorization authorization: ASAuthorization) {
        switch authorization.credential {
        case let credentialRegistration as ASAuthorizationPlatformPublicKeyCredentialRegistration:
            logger.log("A new passkey was registered: \(credentialRegistration)")
            
            (delegate as! PasskeysRegistration).passkeysManager(with: credentialRegistration)
        case let credentialAssertion as ASAuthorizationPlatformPublicKeyCredentialAssertion:
            logger.log("A passkey was used to sign in: \(credentialAssertion)")
            
            (delegate as! PasskeysAuthentication).passkeysManager(with: credentialAssertion)
        default:
            fatalError("Received unknown authorization type.")
        }
    }
    
    // ...
}
```

# 設計前端的網路呼叫功能

接著要來定義 Request body 與 Response body
下面是依照 [Day 11](https://ithelp.ithome.com.tw/articles/10349599) 設計的後端 API 規格來進行定義

## 定義 Request body 與 Response body

### Common

```swift
import Foundation

struct CommonResponse: Decodable {
    
    let status: String
    
    let errorMessage: String
}

struct Response<D>: Decodable where D: Decodable {
    
    let statusCode: Int
    
    let body: D
}
```

### Registration

#### AttestationOptions

```swift
import Foundation

struct AttestationOptionsRequest: Codable {
    
    var username: String
    
    var displayName: String
    
    var authenticatorSelection: AuthenticatorSelectionCriteria
    
    var attestation: String
}

struct AttestationOptionsResponse: Decodable {
    
    let status: String
    
    let errorMessage: String
    
    let rp: RelyingParty
    
    let user: UserEntity
    
    let challenge: String
    
    let pubKeyCredParams: [PubKeyCredParam]
    
    let timeout: Int
    
    let excludeCredentials: [ExcludeCredential]
    
    let authenticatorSelection: AuthenticatorSelectionCriteria
    
    let attestation: String
    
    struct RelyingParty: Decodable {
        
        let name: String
    }
    
    struct UserEntity: Decodable {
        
        let id: String
        
        let name: String
        
        let displayName: String
    }
    
    struct PubKeyCredParam: Decodable {
        
        let type: String
        
        let alg: Int
    }
    
    struct ExcludeCredential: Decodable {
        
        let type: String
        
        let id: String
    }
}
```

#### AttestationResults

```swift
import Foundation

struct AttestationResultsRequest: Codable {
    
    var id: String
    
    var response: AuthenticatorAttestationResponse
    
    var getClientExtensionResults: ClientExtensionResults
    
    var type: String
    
    struct AuthenticatorAttestationResponse: Codable {
        
        var clientDataJSON: String
        
        var attestationObject: String
    }
}
```

### Authentication

#### AssertionOptions

```swift
import Foundation

struct AssertionOptionsRequest: Codable {
    
    var username: String
    
    var userVerification: String
    
    init(username: String, userVerification: UserVerificationRequirement) {
        self.username = username
        self.userVerification = userVerification.rawValue
    }
}

struct AssertionOptionsResponse: Decodable {
    
    let status: String
    
    let errorMessage: String
    
    let challenge: String
    
    let timeout: Int
    
    let rpId: String
    
    let allowCredentials: [AllowCredential]
    
    let userVerification: String
    
    struct AllowCredential: Decodable {
        
        let id: String
        
        let type: String
    }
}
```

#### AssertionResults

```swift
import Foundation

struct AssertionResultsRequest: Codable {
    
    var id: String
    
    var response: AuthenticatorAssertionResponse
    
    var getClientExtensionResults: ClientExtensionResults
    
    var type: String
    
    struct AuthenticatorAssertionResponse: Codable {
        
        var authenticatorData: String
        
        var signature: String
        
        var userHandle: String
        
        var clientDataJSON: String
    }
}
```

### AuthenticatorSelectionCriteria

```swift
import Foundation

struct AuthenticatorSelectionCriteria: Codable {
    
    var authenticatorAttachment: String
    
    var residentKey: String
    
    var requireResidentKey: Bool
    
    var userVerification: String
    
    init(authenticatorAttachment: AuthenticatorAttachment,
         residentKey: String,
         requireResidentKey: Bool = false,
         userVerification: UserVerificationRequirement = .preferred) {
        self.authenticatorAttachment = authenticatorAttachment.rawValue
        self.residentKey = residentKey
        self.requireResidentKey = requireResidentKey
        self.userVerification = userVerification.rawValue
    }
}

enum AuthenticatorAttachment: String, Codable {
    
    case platform = "platform"
    
    case crossPlatform = "cross-platform"
}

enum UserVerificationRequirement: String, Codable {
    
    case required = "required"
    
    case preferred = "preferred"
    
    case discouraged = "discouraged"
}
```

### ClientExtensionResults

```swift
struct ClientExtensionResults: Codable {

}
```

## 使用 URLSession 撰寫網路呼叫功能

這邊使用我個人寫的 Swift Package ([SwiftHelpers](https://github.com/leoho0722/SwiftHelpers)) 來加速功能撰寫

#### NetworkConfiguration

```swift
import Foundation
import SwiftHelpers

struct NetworkConfiguration {
    
    var method: HTTP.Method
    
    var scheme: NetworkScheme
    
    var host: String
    
    var endpoint: String
    
    var headers: Dictionary<String, String>
    
    var body: Encodable
    
    enum NetworkScheme: String {
        
        case http = "http://"
        
        case https = "https://"
    }
}
```

### NetworkManager

```swift
import Foundation
import SwiftHelpers

class NetworkManager: NSObject {

    static let shared = NetworkManager()
    
    private let urlSessionConfiguration: URLSessionConfiguration
    private let urlSession: URLSession
    
    override init() {
        self.urlSessionConfiguration = .default
        self.urlSession = URLSession(configuration: self.urlSessionConfiguration)
    }
    
    func request<D>(with config: NetworkConfiguration) async throws -> Response<D> where D: Decodable {
        let request = try buildURLRequest(config: config)
        let (data, response) = try await urlSession.data(for: request)
        guard let httpResponse = (response as? HTTPURLResponse) else {
            throw URLError(.badServerResponse)
        }
        let decodedResponse: D = try decodeResponse(data: data)
        return Response(statusCode: httpResponse.statusCode, body: decodedResponse)
    }
    
    private func buildURLRequest(config: NetworkConfiguration) throws -> URLRequest {
        guard let url = URL(string: "\(config.scheme.rawValue)\(config.host)\(config.endpoint)") else {
            throw URLError(.badURL)
        }
        var request = URLRequest(url: url)
        request.httpMethod = config.method.rawValue
        request.allHTTPHeaderFields = config.headers
        
        switch config.method {
        case .post:
            request.httpBody = try JSON.toJsonData(data: config.body)
        default:
            break
        }
        
        return request
    }
    
    private func decodeResponse<D>(data: Data) throws -> D where D: Decodable {
        let decoder = JSONDecoder()
        return try decoder.decode(D.self, from: data)
    }
}
```

今天接續實作了將 Passkeys API 回傳的 Authenticator 資料進行接值，以及設計前端的網路呼叫功能

明天要來把各個功能串接起來～