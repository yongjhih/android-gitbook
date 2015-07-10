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

## 沒有回傳值的 Callback 用 RxJava 要怎麼寫?

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
