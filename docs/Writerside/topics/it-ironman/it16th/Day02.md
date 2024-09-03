# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 02 - 認識 FIDO 聯盟

## 已經使用已久的登入方式

在開始介紹 FIDO2 與 WebAuthn 之前，我們先來回顧一下，已經使用已久的登入方式，也就是帳密登入

![帳密登入方式](https://ithelp.ithome.com.tw/upload/images/20240903/20140363teNgsMQ4bs.png)

▲ 圖取自 [https://blog.techbridge.cc/2019/08/17/webauthn-intro](https://blog.techbridge.cc/2019/08/17/webauthn-intro)

不論今天是要購物、查看信件、瀏覽社群等，都有一個共通點，就是需要先註冊一個帳號！
這件事每個人都已經習以為常了
但使用帳密登入，有一個最大的問題就是伺服器端可以拿到你的帳號密碼
能拿到帳密就代表著伺服器有能力私下儲存起來
當發生資料外洩、網路釣魚等資安事件時，可能就會被外流了

## FIDO 聯盟

![FIDO 聯盟 logo](https://fidoalliance.org/wp-content/uploads/2023/12/FIDO_Alliance_Passkey_logo%E2%84%A2-1024x512.jpg)

▲ 圖取自 [FIDO Alliance](https://fidoalliance.org/overview/legal/logo-usage/) 官網

為了解決這個問題，由 PayPal 和 Lenovo 為首的企業在 2012 年成立了 FIDO 聯盟，其核心宗旨為減少全世界對於密碼的依賴，並制定相關的身份驗證與設備證明標準。
在近幾年，各科技公司像是 Apple、Google、Intel、Microsoft 等也加入並領導著 FIDO 聯盟推動相關規範，目前 FIDO 版本與其包含的相關規範簡單整理後如下：

* FIDO (2014 年提出)
    * UAF (Universal Authentication Framework)
    * U2F (Universal Two Factor)

UAF 像是使用生物辨識來替代原先輸入密碼的環節
U2F 則是在原先輸入完帳密後使用如 NFC、BLE、USB 金鑰這類實體的驗證器作為第二驗證因子，進行驗證

* FIDO2 (2018 年提出)
    * WebAuthn (Web Authentication)
    * CTAP (Client to Authenticator Protocol)
        * CTAP 1.0 (原 FIDO 中的 UAF、U2F)
        * CTAP 2.0

FIDO2 則是在 2018 年提出，並由 WebAuthn、CTAP 兩大協定所組成，
原本在 FIDO 中包含的 UAF 和 U2F 在 FIDO2 中的 CTAP 協定則稱為 CTAP 1.0。

接下來幾天就來簡單介紹 FIDO2 中的 WebAuthn 與 CTAP。