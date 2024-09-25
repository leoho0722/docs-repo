# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 25 - 開始進行前後端串接 (5)

昨天通過了第四關之後，今天要來接續第五關

## 串接之旅 Part 5

### 開始測試囉

#### 串接第五關：failed to create credential, error: Error validating challenge

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363BjMdMXVnC4.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363BjMdMXVnC4.png)

從錯誤來看，是在建立 WebAuthn Credential 時，無法驗證 challenge 所導致的

那我們看一下 go-webauthn source code 來判斷有哪些可能性會導致這個錯誤

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363MIEPdwXigN.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363MIEPdwXigN.png)

看起來錯誤是在這邊拋出的，那 `storedChallenge` 是什麼呢？
我們回到 `CreateCredential` 這個 method 來看一下

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363K5HhRatRzM.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363K5HhRatRzM.png)

可以看到 `storedChallenge` 就是 `session.Challenge`，也就是在整個 WebAuthn Registration session 中的 challenge，而要進行驗證的 challenge 則是在 clientDataJSON 中的 challenge

在前面第二關的時候，也有遇到相似的問題，那時是 `challenge mismatch` 跟這次的有點像

那跟第二關一樣，我們從後端進行處理

我們再看一次前端提供的 clientDataJSON 的內容長怎樣

```json
{"type":"webauthn.create","challenge":"WFJ4OFMxMjJwZkpTS2hfT0dvOEVZMVl1N1lOY1NTeUlWdGVOanZVclJyWQ","origin":"https://ab15-211-20-7-115.ngrok-free.app"}
```

而 session 中的 challenge 又長怎樣

```text
XRx8S122pfJSKh_OGo8EY1Yu7YNcSSyIVteNjvUrRrY
```

剛好又是經過 base64 編碼過後的，那麼這次我們在 `StartAttestationHandler` Function 中，將 session 中的 challenge 也改為 base64 編碼的

```go
sessionData.Challenge = base64.RawStdEncoding.EncodeToString([]byte(sessionData.Challenge))
```

像是下面這樣

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363J8gPXMUJyf.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363J8gPXMUJyf.png)

接著，我們將勇闖第二關時，所寫的 `decodedChallenge` 那段程式碼，修改到當 clientDataJSON 中的 challenge 與 session 中的 challenge 相等時要處理的地方

![https://ithelp.ithome.com.tw/upload/images/20240924/20140363lIRWrUGdX8.png](https://ithelp.ithome.com.tw/upload/images/20240924/20140363lIRWrUGdX8.png)

再重新執行一次，恭喜成功完成 WebAuthn Registration 了！

![https://ithelp.ithome.com.tw/upload/images/20240924/2014036394rR27SUsr.png](https://ithelp.ithome.com.tw/upload/images/20240924/2014036394rR27SUsr.png)

明天我們再繼續勇闖第六關！