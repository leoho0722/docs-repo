# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 07 - 認識 Passkeys

前面幾天簡單認識了組成 FIDO2 的 WebAuthn 和 CTAP 後

今天要來認識的是基於 FIDO2 的 Passkeys

## Passkeys {id="passkeys_1"}

Passkeys 是基於 FIDO2 的無密碼登入方式，在前面幾天介紹到 WebAuthn 註冊 / 驗證流程

在 Registration 時，Authenticator 會產生出公私鑰的 key-pair 與 attestation，並提供給 Platform 作為 Registration Credential

在 Authentication 時，Authenticator 會產生出 Signature 與 AuthenticatorData，提供給 Platform 作為 Authentication Credential

而這兩種 Credential 可以通稱為 Passkeys，下面是使用 Passkeys 進行驗證的示意圖

![使用 Passkeys 進行驗證的示意圖](https://ithelp.ithome.com.tw/upload/images/20240908/20140363JtpsoXUPG8.png)

▲ 圖取自 [FIDO Alliance](https://fidoalliance.org/how-fido-works/)

使用者可以透過將 Passkeys 儲存在裝置上的雲端服務 (如：Apple iCloud、Google Password Manager)，進行跨裝置同步

而當裝置中不具有對應 Passkeys 時，則可以透過在鄰近的其他裝置中所儲存的 Passkeys 進行使用

### 舉例來說

* 跨裝置同步
  你的 iPhone 上擁有 網站 A 的 Passkeys，但你的 Mac 電腦中並沒有 網站 A 的 Passkeys
  iPhone 與 Mac 都登入同一個 Apple ID，今天你要在 Mac 電腦上使用 Passkeys 登入 網站 A
  這時因為都是使用同一個 Apple ID，透過 iCloud 鑰匙圈進行跨裝置同步 Passkeys，可以直接在 Mac 上使用儲存在 iCloud 鑰匙圈中 網站 A 的 Paskeys 進行登入

* 透過其他裝置上的 Passkeys 進行驗證
  你的 iPhone 上擁有 網站 A 的 Passkeys，但你的 Windows 電腦中並沒有 網站 A 的 Passkeys
  今天你要在 Windows 電腦上使用 Passkeys 登入 網站 A
  這時可以使用 iPhone 掃描 網站 A 生成的 QR Code，透過儲存在 iPhone 上 網站 A 的 Passkeys

## 現在有哪些網路服務支援 Passkeys

以下列出的網路服務主要是我有用過或是聽過的，如果有漏的話，歡迎在底下留言補充～

### Microsoft

Microsoft 是今年 5 月左右才開放可以使用 Passkeys 進行驗證

![Microsoft 使用 Passkeys 驗證](https://ithelp.ithome.com.tw/upload/images/20240908/20140363ikXTTxWhfn.png)

### Google

Google 是在去年 5 月左右就開放可以使用 Passkeys 進行驗證

![Google 使用 Passkeys 驗證](https://ithelp.ithome.com.tw/upload/images/20240908/20140363NJ0mh36cOQ.png)

### GitHub

GitHub 可以使用 Passkeys 進行登入驗證，或是用於雙重驗證

![GitHub 使用 Passkeys 驗證](https://ithelp.ithome.com.tw/upload/images/20240908/20140363weiqqecX7Z.png)

### GitLab

GitLab 相較於 GitHub，目前只能用於雙重驗證，尚無法用來進行登入驗證

![GitLab 使用 Passkeys 驗證](https://ithelp.ithome.com.tw/upload/images/20240908/20140363wVPypxM9Kt.png)

### 1Password

1Password 作為主流常見的密碼管理器，早在 2022 年底就開始針對 Passkeys 進行[相關支援](https://www.future.1password.com/passkeys/)，並在 2023 年正式支援 Passkeys

![1Password 使用 Passkeys 驗證](https://ithelp.ithome.com.tw/upload/images/20240908/20140363vEQ3nvhfUP.png)

▲ 圖取自 [1Password Blog](https://blog.1password.com/1password-passkeys-apps-and-devices/)

今天認識了 Passkeys，並介紹了現在有哪些網路服務支援使用 Passkeys 進行註冊 / 驗證
明天來說說，FIDO2 實際上解決了什麼問題～