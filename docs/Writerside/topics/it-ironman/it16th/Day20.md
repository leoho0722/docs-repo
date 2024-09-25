# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 20 - 實作 Apple Passkeys API (3)

昨天我們設計好了 `PasskeysManagerDelegate`、`PasskeysRegistration`、`PasskeysAuthentication` 這三個用來處理 PasskeysManager 物件事件的 delegate 以及網路呼叫功能

今天要來將各個功能串接組合起來

下面有部分使用的 Function 因礙於篇幅已經過長了 (笑😂)，所以就先略過不提，但可以在本次鐵人賽 30 天 Sample Code 的 GitHub Repository 中查看

> [本次鐵人賽 Sample Code 的 GitHub Repository](https://github.com/leoho0722/it16th)

# 串接！

## 從 Delegate 取出 Authenticator 回傳的資料

### PasskeysRegistration

使用 Apple Passkeys API 向 Authenticator 進行運算後，我們可以從 Passkeys API 取得以下至少四個資料

* clientDataJSON
* attestationObject
* credentialID
* attachment

讓我們可以將其回傳到 WebAuthn RP Server，進行後續的驗證註冊流程

```swift
extension PasskeysViewController: @preconcurrency PasskeysRegistration {
    
    func passkeysManager(with credentialRegistration: ASAuthorizationPlatformPublicKeyCredentialRegistration) {
        let clientDataJSON = credentialRegistration.rawClientDataJSON
        let attestationObject = credentialRegistration.rawAttestationObject
        let credentialID = credentialRegistration.credentialID
        let attachment = credentialRegistration.attachment
        
        passkeysFinishRegistration(clientDataJSON: clientDataJSON,
                                   attestationObject: attestationObject,
                                   credentialID: credentialID)
    }
}
```

### PasskeysAuthentication

使用 Apple Passkeys API 向 Authenticator 進行運算後，我們可以從 Passkeys API 取得以下至少六個資料

* clientDataJSON
* authenticatorData
* signature
* userID
* credentialID
* attachment

讓我們可以將其回傳到 WebAuthn RP Server，進行後續的驗證登入流程

```swift
extension PasskeysViewController: @preconcurrency PasskeysAuthentication {
    
    func passkeysManager(with credentialAssertion: ASAuthorizationPlatformPublicKeyCredentialAssertion) {
        let clientDataJSON = credentialAssertion.rawClientDataJSON
        let authenticatorData = credentialAssertion.rawAuthenticatorData
        let signature = credentialAssertion.signature
        let userID = credentialAssertion.userID
        let credentialID = credentialAssertion.credentialID
        let attachment = credentialAssertion.attachment
        
        passkeysFinishAuthentication(clientDataJSON: clientDataJSON,
                                     authenticatorData: authenticatorData,
                                     signature: signature,
                                     credentialID: credentialID,
                                     userID: userID)
    }
}
```

接著，我們在 `PasskeysViewController` 新增下面四個 Function，分別用來處理 WebAuthn 的四支 API

## WebAuthn Registration

### 產生註冊資訊

```swift
private func passkeysBeginRegistration(username: String, displayName: String) {
    Task {
        do {
            let authenticatorSelection = AuthenticatorSelectionCriteria(authenticatorAttachment: .platform,
                                                                        residentKey: .preferred)
            let request = AttestationOptionsRequest(username: username,
                                                    displayName: displayName,
                                                    authenticatorSelection: authenticatorSelection,
                                                    attestation: .direct)
            let requestConfiguration = RequestConfiguration(method: .post,
                                                            scheme: .https,
                                                            host: .rpServer,
                                                            endpoint: .beginRegistration,
                                                            body: request)
            let response: AttestationOptionsResponse = try await NetworkManager.shared.request(with: requestConfiguration)

            guard let window = self.view.window else {
                fatalError("The view was not in the app's view hierarchy!")
            }
            passkeysManager.registration(username: username, challenge: response.challenge, anchor: window)
        } catch {
            var errorMessage: String
            switch error {
            case let networkError as NetworkError:
                switch networkError {
                case .badRequest(let response), .internalServerError(let response):
                    let decoder = JSONDecoder()
                    let decodedResponse = try! decoder.decode(CommonResponse.self, from: response)
                    errorMessage = decodedResponse.errorMessage
                default:
                    errorMessage = error.localizedDescription
                }
            default:
                errorMessage = error.localizedDescription
            }
            Alert.showWith(title: "錯誤",
                           message: "WebAuthn 產生註冊資訊失敗！\n錯誤訊息為：\(errorMessage)",
                           confirmTitle: "確認",
                           vc: self)
        }
    }
}
```

### 驗證註冊資訊

```swift
private func passkeysFinishRegistration(clientDataJSON: Data,
                                        attestationObject: Data?,
                                        credentialID: Data) {
    Task {
        do {
            let base64URLEncodedClientDataJSON = clientDataJSON.base64EncodedString().base64EncodedToBase64RawURLEncoded()
            let base64URLEncodedAttestationObject = attestationObject?.base64EncodedString().base64EncodedToBase64RawURLEncoded()
            let base64URLEncodedCredentialID = credentialID.base64EncodedString().base64EncodedToBase64URLEncoded()

            let authenticatorAttestationResponse = AttestationResultsRequest.AuthenticatorAttestationResponse(clientDataJSON: base64URLEncodedClientDataJSON,
                                                                                                              attestationObject: base64URLEncodedAttestationObject)
            let request = AttestationResultsRequest(id: base64URLEncodedCredentialID,
                                                    response: authenticatorAttestationResponse,
                                                    getClientExtensionResults: .init(),
                                                    type: .publicKey)
            let requestConfiguration = RequestConfiguration(method: .post,
                                                            scheme: .https,
                                                            host: .rpServer,
                                                            endpoint: .finishRegistration,
                                                            body: request)
            let response: CommonResponse = try await NetworkManager.shared.request(with: requestConfiguration)

            if response.status == "ok" {
                Alert.showWith(title: "成功",
                               message: "WebAuthn Registration 已完成！",
                               confirmTitle: "確認",
                               vc: self)
            }
        } catch {
            var errorMessage: String
            switch error {
            case let networkError as NetworkError:
                switch networkError {
                case .badRequest(let response), .internalServerError(let response):
                    let decoder = JSONDecoder()
                    let decodedResponse = try! decoder.decode(CommonResponse.self, from: response)
                    errorMessage = decodedResponse.errorMessage
                default:
                    errorMessage = error.localizedDescription
                }
            default:
                errorMessage = error.localizedDescription
            }
            Alert.showWith(title: "錯誤",
                           message: "WebAuthn Registration 驗證註冊資訊失敗！\n錯誤訊息為：\(errorMessage)",
                           confirmTitle: "確認",
                           vc: self)
        }
    }
}
```

## WebAuthn Authentication

### 產生登入資訊

```swift
private func passkeysBeginAuthentication(username: String) {
    Task {
        do {
            let request = AssertionOptionsRequest(username: username, userVerification: .preferred)
            let requestConfiguration = RequestConfiguration(method: .post,
                                                            scheme: .https,
                                                            host: .rpServer,
                                                            endpoint: .beginAuthentication,
                                                            body: request)
            let response: AssertionOptionsResponse = try await NetworkManager.shared.request(with: requestConfiguration)

            guard let window = self.view.window else {
                fatalError("The view was not in the app's view hierarchy!")
            }

            passkeysManager.authentication(challenge: response.challenge,
                                           anchor: window,
                                           preferImmediatelyAvailableCredentials: true)
        } catch {
            var errorMessage: String
            switch error {
            case let networkError as NetworkError:
                switch networkError {
                case .badRequest(let response), .internalServerError(let response):
                    let decoder = JSONDecoder()
                    let decodedResponse = try! decoder.decode(CommonResponse.self, from: response)
                    errorMessage = decodedResponse.errorMessage
                default:
                    errorMessage = error.localizedDescription
                }
            default:
                errorMessage = error.localizedDescription
            }
            Alert.showWith(title: "錯誤",
                           message: "WebAuthn Authentication 產生登入資訊失敗！\n錯誤訊息為：\(errorMessage)",
                           confirmTitle: "確認",
                           vc: self)
        }
    }
}
```

### 驗證登入資訊

```swift
private func passkeysFinishAuthentication(clientDataJSON: Data,
                                          authenticatorData: Data?,
                                          signature: Data?,
                                          credentialID: Data,
                                          userID: Data?) {
    Task {
        do {
            let base64URLEncodedClientDataJSON = clientDataJSON.base64EncodedString().base64EncodedToBase64RawURLEncoded()
            let base64URLEncodedAuthenticatorData = authenticatorData?.base64EncodedString().base64EncodedToBase64RawURLEncoded()
            let base64URLEncodedSignature = signature?.base64EncodedString().base64EncodedToBase64RawURLEncoded()
            let base64URLEncodedCredentialID = credentialID.base64EncodedString().base64EncodedToBase64RawURLEncoded()
            let base64URLEncodedUserID = userID?.base64EncodedString().base64EncodedToBase64RawURLEncoded()

            let authenticatorAssertionResponse = AssertionResultsRequest.AuthenticatorAssertionResponse(authenticatorData: base64URLEncodedAuthenticatorData,
                                                                                                        signature: base64URLEncodedSignature,
                                                                                                        userHandle: base64URLEncodedUserID,
                                                                                                        clientDataJSON: base64URLEncodedClientDataJSON)

            let request = AssertionResultsRequest(id: base64URLEncodedCredentialID,
                                                  response: authenticatorAssertionResponse,
                                                  getClientExtensionResults: .init(),
                                                  type: .publicKey)
            let requestConfiguration = RequestConfiguration(method: .post,
                                                            scheme: .https,
                                                            host: .rpServer,
                                                            endpoint: .finishAuthentication,
                                                            body: request)
            let response: CommonResponse = try await NetworkManager.shared.request(with: requestConfiguration)

            if response.status == "ok" {
                Alert.showWith(title: "成功",
                               message: "WebAuthn Authentication 已完成！",
                               confirmTitle: "確認",
                               confirm: pushToHome,
                               vc: self)
            }
        } catch {
            var errorMessage: String
            switch error {
            case let networkError as NetworkError:
                switch networkError {
                case .badRequest(let response), .internalServerError(let response):
                    let decoder = JSONDecoder()
                    let decodedResponse = try! decoder.decode(CommonResponse.self, from: response)
                    errorMessage = decodedResponse.errorMessage
                default:
                    errorMessage = error.localizedDescription
                }
            default:
                errorMessage = error.localizedDescription
            }
            Alert.showWith(title: "錯誤",
                           message: "WebAuthn Authentication 驗證登入資訊失敗！\n錯誤訊息為：\(errorMessage)",
                           confirmTitle: "確認",
                           vc: self)
        }
    }
}
```
