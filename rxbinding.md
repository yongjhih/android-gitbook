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

## 範例 1 - 驗證過濾

```java
Observable<CharSequence> rxUsername = RxTextView.textChanges(tvUsername);
Observable<CharSequence> rxPassword = RxTextView.textChanges(tvPassword);
Observable<CharSequence> rxFullName = RxTextView.textChanges(tvFullName).mergeWith(Observable.just(""));
Observable.combineLatest(
  rxUsername, rxPassword, rxFullName, RegistrationModel::new)
  .filter(RegistrationModel::isValid)
  .subscribe(result -> enableSubmitButton());
```

## 範例 2 - 流量控制

```java
RxSearchView.queryTextChanges(searchView)
  .filter(charSequence -> !TextUtils.isEmpty(charSequence))
  .throttleLast(100, TimeUnit.MILLISECONDS)
  .debounce(200, TimeUnit.MILLISECONDS)
  .onBackpressureLatest()
  .concatMap(charSequence -> {
    return searchRepositories(charSequence);
  })
  .observeOn(AndroidSchedulers.mainThread())
  .subscribeOn(Schedulers.io())
  .onErrorResumeNext(throwable -> {
    return Observable.empty();
  })
  .subscribe(response -> {
    showRepositories(response.getItems());
});
```

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
