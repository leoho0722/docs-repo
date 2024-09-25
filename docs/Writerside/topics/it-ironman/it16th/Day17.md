# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 17 - 實作一個簡單的 iOS App UI

在前面幾天，我們實作了 WebAuthn 後端功能，接下來我們要繼續來實作前端的 iOS App UI

## 實作一個簡單的 iOS App UI

在昨天 (9/17) iOS 18 正式推出了
因此這邊會以最新版本的 Xcode 16 作為這次 iOS App 的開發環境 (如果要用 Xcode 15 也是可以的)
並以 SwiftUI 作為 UI 框架

### 建立專案

首先，先開啟 Xcode 來建立專案

![開啟 Xcode 建立專案](https://ithelp.ithome.com.tw/upload/images/20240918/20140363O3YWOHPDKu.png)
▲ 開啟 Xcode 建立專案

### iOS App 實作想法

這邊會以一個簡單的帳號登入來作為範例，並使用 Apple Passkeys API 進行 WebAuthn 註冊 / 驗證

### 設計一個簡單的 UI

這邊主要是著重在功能上，所以 UI 部分就簡單設計 (小小偷懶一下，其實是時間不夠 😂)

一個 TextField 用來輸入 WebAuthn 的使用者名稱
一個 Button 用來進行 WebAuthn Registration 流程
一個 Button 用來進行 WebAuthn Authentication 流程

![大概設計的簡單 UI](https://ithelp.ithome.com.tw/upload/images/20240918/20140363kvbrmPgal6.png)
▲ 大概設計的簡單 UI

今天設計好簡單的 UI 後，明天開始就要來一步一步在 iOS App 上實作出 WebAuthn 註冊 / 驗證流程了