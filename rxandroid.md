# RxAndroid

## RxBinding

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
    
    Rx<TextView> mRxTextView

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.inject(this);

        mRxTextView = RxView.of(mTextView);
        Subscription s = mRxTextView.bind(Observable.just("hoge"), RxActions.setText());
}
```
