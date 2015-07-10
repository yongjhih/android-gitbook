# 經典問題 - 指引你「這該怎麼弄？」

這裡是從問題指引你到哪裡可以找到答案。

## 要怎麼寫非同步？

## 要怎麼避免 Callback Hell

## 按鈕不想讓別人連按，要怎麼寫 debounce？

## Restful client 要怎麼寫？用哪套？

## ImageLoader 要用哪套？

## Json 轉物件 要用哪套？

## 我該從哪得知 Android App 開發的資訊？

前端 UI 、美工 Art 、平面字體挑選、換場動畫設計。

App 工程開發。

函式庫資訊

## 沒有回傳值的 Callback 用 RxJava 要怎麼寫？

> 想要問一下，我有一個 init() function 可能會做很久，所以我裡面開了一個 thread，然後 ui 傳入一個 callback 當我 init 完成之後我 callback 他。

> 如果想要改成 RxJava 的架構，所以我 init 回傳了 Observable<Boolean>，這裡 Boolean 其實有點多餘，因為我只是要單純的通知 ui 說我做完了，感覺有點硬是要用 Observable 來做。


如果你用 AsyncTask<I, N, O> 也會有類似的問題。

```java
Observable<Void> initAsyncObs = Observable.defer(() -> Observable.just(initSync()))
  .subscribeOn(Schedulers.io())
  .observeOn(AndroidSchedulers.mainThread());

Void initSync() {
  // ...
  return null;
}
```

## Background Thread API 用 RxJava `.observeOn(AndroidSchedulers.mainThread());` 好嗎？

> api 回傳 Observable ，然後由 ui subscribe 來取得資料更新畫面
> 而 Observable 先在 api 層設定好要在 background thread 執行，讓 ui 不用去管這段
不知道這樣的觀念是否正確，或是有更好的架構？


Android Threading/Scheduling 很多不只是 background 的問題，還有 lifecycle 問題，LoaderManager 其實也有在處理 lifecycle 問題。

所以 Ui 還是要自己善用 `AppObservable.bindActivity(activity, obs)`, `AppObservable.bindFragment(fragment, obs)`, `ViewObservable.bindView(view, obs)` 這些就是負責處理在 Android 上的 LifeCycle 以及 Threading 問題。 (rxandroid)


## Ui Adapter 要用的，RxJava 回傳 `Observable<List<E>>` 還是 `Observable<E>`

> 感覺回傳 List 就很鳥，但是如果 ui 就是要收集到全部的資料在一次刷新，回傳 list 對 ui 來說好像比較方便

為了 UI adapater 方便，是否傳遞 `Observable<List<E>>`？

首先在函式庫角度來看，它應該只做到單純性、重用性、通用性、彈性、操作性的最大化。

* 傳遞 `Observable<List<E>>` 的劣勢，很明顯的是大部分的 Rx Operator 都無法直接使用，也就是我說的操作性很低。所以一般狀況下，不會傳遞無操作性的介面。

在這個前提下，自然會導出 -> 函式庫不提供 `Observable<Collection/Iteratable> `。

接著是下一層的問題 ，那 Ui 該怎麼辦？很簡單，請服用： `Observable.toList()`, `Observable.buffer()`, etc.

* See Also: 
https://kaif.io/z/compiling/debates/drg0Cf6oGj/dsCeOiO6Gz