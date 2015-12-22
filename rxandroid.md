# RxAndroid

https://github.com/ReactiveX/RxAndroid

Android 相關的元件連動

## Activity/Fragment: AppObservable

```java
AppObservable.bindActivity()
AppObservable.bindFragment()
```

主要檢查 `Fragment.isAdded()`, `Activity.isFinishing()` 等狀態，排除一些不適當的執行。

## RxLifecycle

當 Activity/Fragment 對應的生命週期結束時 `unsubscribe()` 避免 leaks。

`LifecycleObservable` 哪時候訂閱哪時候取消對照表：

```java
CREATE -> DESTROY;
START -> STOP;
RESUME -> PAUSE;
PAUSE -> STOP;
STOP -> DESTROY;
```

手動 `unsubscribe()`:

```java
class SimpleActivity extends Activity {
    CompsotionSubscription mResumeSubscriptions = new CompositeSubscription();

    @Override
    protected void onResume() {
        super.onResume();

        bindResume(Observable.just("Hello, world"), s -> textView.setText(s));
    }

    protected <T> void bindResume(Observable<T> obs, Action1<T> onNext) {
        mResumeSubscriptions.add(AppObservable.bindActivity(this, obs).subscribe(onNext));
    }

    @Override
    protected void onPause() {
        super.onPause();

        mResumeSubscriptions.unsubscribe();
    }
}
```

`RxActivity`/`RxAppCompatActivity`:

```java
class SimpleActivity extends RxActivity {

    @Override
    public void onResume() {
        Observable.just("Hello, world")
        .compose(bindToLifecycle())
        .subscribe(s -> textView.setText(s));
    }
}
```

## ViewObservable, WidgetObservable

View 的連動. 當 View 顯示時 `subscribe()` 離開時 `unsubscribe()`

```java
ViewObservable.bindView()
```

Event 的連動.

```java
ViewObservable.clicks()
```

## See Also

* [RxActivity](https://github.com/ReactiveX/RxAndroid/blob/master/rxandroid-framework/src/main/java/rx/android/app/RxActivity.java)
* [RxFragmentActivity](https://github.com/ReactiveX/RxAndroid/blob/master/rxandroid-framework/src/main/java/rx/android/app/support/RxFragmentActivity.java)
* [RxFragment](https://github.com/ReactiveX/RxAndroid/blob/master/rxandroid-framework/src/main/java/rx/android/app/support/RxFragment.java)
* [ogaclejapan/RxBinding](https://github.com/ogaclejapan/RxBinding)
* [JakeWharton/RxBinding](https://github.com/JakeWharton/RxBinding)
* https://github.com/trello/RxLifecycle
* https://github.com/ReactiveX/RxAndroid/issues/172
* 註: *RxAndroid 1.0 後，把大部分的功能拆出去 JakeWharton/RxBinding(views 行為連動相關) 與 trello/RxLifecycle(生命週期連動相關)*
