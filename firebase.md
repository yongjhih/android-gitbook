# Firebase

## 前言

Firebase 成立於 2011 年，主要提供 BaaS 的服務，當時的競爭對手有非常戲劇化落幕的 Parse 等。
Firebase 於 2014 年加入了 Google 旗下，繼續他的 BaaS 服務，不過在 2016 Google IO 大會上，Google 給予了新的生命 - 幫助開發者可以 **更容易的開發，成長以及賺錢**。

這有什麼了不起的？為什麼我需要他？

在看完了 Firebase 的官網介紹之後，他非常強調一件事情，就是**讓數字來幫助我們做決策**

下圖為 Firebase 的新藍圖，由許多服務結合而成，可以看到 Analytics 是最重要的核心，而 BaaS 只是其中的一部分

![Firebase](https://lh3.googleusercontent.com/you5Qm6B9GhkBvQ-A25p2p3iDsRCzwbqupJ-H4wJWAnkl2O0jOgar4zhY31e0RUAw40P47jkfDg24T3KHDRFSFFRGUXn6a8=s888)

好，大家或許會好奇，這些服務現在都有現成的，而且大部分 Google Service Api 也都有提供，他為何要造一樣的輪子？我們真的有必要去使用它嗎？

先告訴大家，有，而且 Google 給了一個無法拒絕的 Offer

![alt](./firebase_update.jpg)

上圖可以看到 App Indexing 已經交給 Firebase 來處理了，假如想要使用最新的 Google Play Service API，Firebase 是避不了的！

當然除了上述理由外，Firebase 最重要的是提供許多實用的功能來幫助 App 成長茁壯，而且所有模組都圍繞在最核心的功能：Analytics，
用數字來幫助我們決策，不再只是一昧的去猜想修改程式，讓數字來說話吧！

PS1. 本篇文章會專注在介紹 Firebase 為我們開發上帶來什麼好處，假如想了解 Firebase 如何導入，建議直接上官網了解！

PS2. 因為不是每樣功能都有使用，所以會先針對目前使用的服務做分享

## Analytics
// TODO

## Cloud Messaging & Notifications
// TODO

## Crash Reporting
和許多知名 Crash 工具如 Crashlytics 串接上差不多，不過比較令人驚艷的地方是
他不只顯示基本的 Crash 資訊，
還很貼心的將使用者如何點擊的 Event 呈現出來（不過前提是你有埋到這些 Event），
更方便我們去思考相對應的解法（不過有依然很多無解啊 XD）

![alt](https://1.bp.blogspot.com/-OR6aq1Rtwzo/WEmfXoIrSJI/AAAAAAAAAps/ig171K0d960Hf8YZl0C5s2xO4DZICxQeACLcB/s1600/image03.png)

有興趣的也可以參考這篇[官方文章](https://firebase.googleblog.com/2016/12/firebase-crash-reporting-full-release.html)

## Remote Config
// TODO

## App Indexing
// TODO

## Dynamic Links
// TODO

## Invites
// TODO

