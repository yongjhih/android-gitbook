# Data Stream

[RxJava](RxJava) 已經介紹了 Data Stream 的概念。如果你不需要 RxJava  提供的執行緒管理、錯誤處理等又覺得 RxJava 不夠輕量。或許你可以採用輕量，只針對資料流的函式庫。

* https://github.com/kgmyshin/Marray
* https://github.com/konmik/solid

特點：

* Marray 可修改。
* SolidList 主要以 ImuttableList 出發，所以沒實現修改能力。
* SolidList 支援 Parcelable 傳遞。

另外還有其他 Promise 選用:

* Bolt-Android

另一個知名 https://github.com/goldmansachs/gs-collections 不過似乎不夠輕。

http://www.goldmansachs.com/gs-collections/documents/GS%20Collections%20Training%20Session%20and%20Kata%205.0.0.pdf