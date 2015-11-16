# RxBinding(JakeWharton/RxBinding)

讓每個 Android 的元件都包裝成 Rx 。

RxAndroid Before:

```java
ViewObservable.clicks(textView).subscribe(ev -> {});
```

RxBinding After:

```java
RxTextView.textChanged(textView).subscribe(ev -> {});
```

基本上與 ViewObservable, ogaclejapan/RxBinding 都是處理 View 相關的連動。

## 同名 ogaclejapan/RxBinding

https://github.com/ogaclejapan/RxBinding

主要以 MVVM 雙向連動為主要訴求

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

## See Also

* Jack Wharton 的 RxJava 電台訪談： http://fragmentedpodcast.com/episodes/7/
* https://github.com/JakeWharton/RxBinding
* https://github.com/ogaclejapan/RxBinding
