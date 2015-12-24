# 經典問題 - 指引你「這該怎麼弄？」

這裡是從問題指引你到哪裡可以找到答案。

## gradle 如何只測試單項?

```bash
./gradlew testDebug --tests='*.<testname>'
```

## 利用回傳空列表 `Collections.emptyList()` 來減少不必要的 null `List` 檢查

* 不再 `return new ArrayList<>();` 改用 `Collections.emptyList()`
* 不再 `return new HashSet<>();` 改用 `Collections.emptySet()`

不用檢查：

```java
List<ResolveInfo> infos = packageManager.queryIntentActivities(intent, flags);
// infos == null? 像是官方文件就寫清楚肯定會回傳一個空列表，所以我們可以不用檢查 null
for (info : infos) {
  System.out.println(info);
}
```

## 要怎麼寫非同步？

* Thread
* HandlerThread
* AsyncTask

## 要怎麼避免 Callback Hell

使用 RxJava 或者 Bolts 等具 promise 架構來整平

## 按鈕不想讓別人連按，要怎麼寫 debounce？

## Restful client 要怎麼寫？用哪套？

* Retrofit
* Volley

See Also: ...

## ImageLoader 要用哪套？

每套有其特性，一般短小精幹可用 picasso.

* 大量客製調整 AUIL
* Glide 效能考量

See Also: ...

## Json 轉物件 要用哪套？

* LoganSquare 目前是非 reflection 方式，所以應有較高的效能表現。
* Jackson
* Gson

See Also: ...

## 我該從哪得知 Android App 開發的資訊？

前端 UI 、美工 Art 、平面字體挑選、換場動畫設計。

App 工程開發。

函式庫資訊

## 沒有回傳值的 Callback 用 RxJava 要怎麼寫？

> 想要問一下，我有一個 init() function 可能會做很久，所以我裡面開了一個 thread，然後 ui 傳入一個 callback 當我 init 完成之後我 callback 他。

> 如果想要改成 RxJava 的架構，所以我 init 回傳了 Observable<Boolean>，這裡 Boolean 其實有點多餘，因為我只是要單純的通知 ui 說我做完了，感覺有點硬是要用 Observable 來做。


如果你用 `AsyncTask<I, N, O>` 也會有類似的問題。

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

* See Also: https://kaif.io/z/compiling/debates/drg0Cf6oGj/dsCeOiO6Gz

## multidex 跟 proguard 擇一執行哪一個速度比較快？

先不管題目的合理性。
答案是 proguard 比較快。

1. 理論上，經過 proguard 之後，code size 的縮減，所以啟動程式碼載入時間會縮短。
2. multidex 還會變慢，因為 65k 超出去的部份是啟動之後，動態載入的，你知道的，就是那個 reflection ，想當然爾會更慢。

而提問者其實後來要問的是「編譯」而不是「執行」，如果是編譯，multidex 會快些。而我們本來在開發上就是 release build 才會跑 proguard minify ，所以倒不用擔心。
