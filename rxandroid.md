# RxAndroid

https://github.com/ReactiveX/RxAndroid

生命周期的连动。

## AppObservable

```java
AppObservable.bindActivity()
AppObservable.bindFragment()
```

主要检查 `Fragment.isAdded()`, `Activity.isFinishing()`。

*注：笔者不是很清楚，为什么不用 overloading: `AppObservable.bind(Activity/Frgment/v4.Fragment)` 来取代 `AppObservable.bindFragment(Fragment)`,
`AppObservable.bindFragment(v4.Fragment)`,
`AppObservable.bindActivity(Activity)`*

## LifecycleObservable

```java
LifecycleObservable.bindActivityLifecycle()
LifecycleObservable.bindFragmentLifecycle()
```

当 Activty/Fragment 对应的生命周期结束时，自动 `unsubscribe()`。

`LifecycleObservable` “什么时候订阅什么时候取消”对照表

```java
CREATE -> LifecycleEvent.DESTROY;
START -> LifecycleEvent.STOP;
RESUME -> LifecycleEvent.PAUSE;
PAUSE -> LifecycleEvent.STOP;
STOP -> LifecycleEvent.DESTROY;
```

手动自己 `unsubscribe()`， 如果 Activity 要結束，把一些 subscriptions 取消：

```java
class SimpleActivity extends Activity {
    CompsotionSubscription mSubscriptions = new CompositeSubscription();

    @Override
    protected void onResume() {
        super.onResume();

        bind(Observable.just("Hello, world"), s -> textView.setText(s));
    }

    protected <T> void bind(Observable<T> obs, Action1<T> onNext) {
        mCompositeSubscription.add(AppObservable.bindActivity(this, obs).subscribe(onNext));
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        mCompositeSubscription.unsubscribe();
    }
}
```

`LifecycleObservable` + `RxActivity`:

```java
class SimpleActivity extends RxActivity {

    @Override
    public void onResume() {
        LifecycleObservable.bindActivityLifecycle(lifecycle(),
            AppObservable.bindActivity(this, Observable.just("Hello, world")))
        ).subscribe(s -> textView.setText(s));
    }
}
```

## ViewObservable, WidgetObservable

View 的连动. 当 View 显示时 `subscribe()` 离开时 `unsubscribe()`

```java
ViewObservable.bindView()
```

Event 的连动.

```java
ViewObservable.clicks()
```

## ogaclejapan/RxBinding

https://github.com/ogaclejapan/RxBinding

类似于 ViewObservable。主要以 MVVM 双向连动作努力。

Before：

```java
class HogeActivity extends Activity {

    @InjectView(R.id.text)
    private TextView mTextView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.inject(this);

        AppObservable.bindActivity(this, Observable.just("hoge"))
            .subscribeOn(Schedulers.io())
            .subscribe(setTextAction());
    }

    Action1<String> setTextAction() {
        return text -> mTextView.setText(text);
    }
}
```

After:

```java
class HogeActivity extends Activity {
    // ...

    private Rx<TextView> mRxTextView

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.inject(this);

        mRxTextView = RxView.of(mTextView);
        Subscription s = mRxTextView.bind(Observable.just("hoge"), RxActions.setText());
    }
}
```

*RxAndroid 正在重新规划需留意: https://github.com/ReactiveX/RxAndroid/issues/172 重点在于模块化，看起来对于无限上纲的功能需要做点整理*

## RxLifecycle

## See Also

* [RxActivity](https://github.com/ReactiveX/RxAndroid/blob/master/rxandroid-framework/src/main/java/rx/android/app/RxActivity.java)
* [RxFragmentActivity](https://github.com/ReactiveX/RxAndroid/blob/master/rxandroid-framework/src/main/java/rx/android/app/support/RxFragmentActivity.java)
* [RxFragment](https://github.com/ReactiveX/RxAndroid/blob/master/rxandroid-framework/src/main/java/rx/android/app/support/RxFragment.java)
* [ogaclejapan/RxBinding](https://github.com/ogaclejapan/RxBinding
)
* [JakeWharton/RxBinding](https://github.com/JakeWharton/RxBinding)
* https://github.com/trello/RxLifecycle

*注：并没有 RxAppCompatActivity*
