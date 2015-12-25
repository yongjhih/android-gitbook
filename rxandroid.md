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

沒法更換 Activity/Fragment 繼承時，可參考 [RxAppCompatActivity](https://github.com/trello/RxLifecycle/blob/master/rxlifecycle-components/src/main/java/com/trello/rxlifecycle/components/support/RxAppCompatActivity.java)/[RxFragment](https://github.com/trello/RxLifecycle/blob/master/rxlifecycle-components/src/main/java/com/trello/rxlifecycle/components/RxFragment.java)

舉例寫一個 SimpleFragmentLifecycleProvider 類別：

```java
public class SimpleFragment extends Fragment {

    SimpleFragmentLifecycleProvider lifecycle = new SimpleFragmentLifecycleProvider();

    @Override
    @CallSuper
    public void onResume() {
        super.onResume();
        lifecycle.onResume();

        helloObservable
            .compose(lifecycle.bindUntil(FragmentEvent.DESTROY))
            .subscribe();

        helloObservable2
            .compose(lifecycle.bind())
            .subscribe();
    }

    @Override
    @CallSuper
    public void onPause() {
        lifecycle.onPause();
        super.onPause();
    }

    @Override
    @CallSuper
    public void onDestroy() {
        lifecycle.onDestroy();
        super.onDestroy();
    }

    // ...
}

public class SimpleFragmentLifecycleProvider implements FragmentLifecycleProvider {
    private final BehaviorSubject<FragmentEvent> lifecycleSubject = BehaviorSubject.create();

    @Override
    public final Observable<FragmentEvent> lifecycle() {
        return lifecycleSubject.asObservable();
    }

    @Override
    public final <T> Observable.Transformer<T, T> bindUntil(FragmentEvent event) {
        return RxLifecycle.bindUntilFragmentEvent(lifecycleSubject, event);
    }

    @Override
    public final <T> Observable.Transformer<T, T> bind() {
        return RxLifecycle.bindFragment(lifecycleSubject);
    }

    public void onAttach(android.app.Activity activity) {
        lifecycleSubject.onNext(FragmentEvent.ATTACH);
    }

    public void onCreate(Bundle savedInstanceState) {
        lifecycleSubject.onNext(FragmentEvent.CREATE);
    }

    public void onViewCreated(View view, Bundle savedInstanceState) {
        lifecycleSubject.onNext(FragmentEvent.CREATE_VIEW);
    }

    public void onStart() {
        lifecycleSubject.onNext(FragmentEvent.START);
    }

    public void onResume() {
        lifecycleSubject.onNext(FragmentEvent.RESUME);
    }

    public void onPause() {
        lifecycleSubject.onNext(FragmentEvent.PAUSE);
    }

    public void onStop() {
        lifecycleSubject.onNext(FragmentEvent.STOP);
    }

    public void onDestroyView() {
        lifecycleSubject.onNext(FragmentEvent.DESTROY_VIEW);
    }

    public void onDestroy() {
        lifecycleSubject.onNext(FragmentEvent.DESTROY);
    }

    public void onDetach() {
        lifecycleSubject.onNext(FragmentEvent.DETACH);
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
