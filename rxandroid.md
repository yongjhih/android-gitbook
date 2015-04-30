# RxAndroid

`AppObservable` 一般常見用法：

```java
class HogeActivity extends Activity {

    private TextView mTextView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mTextView = (TextView) findViewById(R.id.text);

        AppObservable.bindActivity(this, Observable.create(sub -> {
            SystemClock.sleep(1000);
            subscriber.onNext("hoge");
            subscriber.onCompleted();
        }))
        .subscribeOn(Schedulers.io())
        .subscribe(setTextAction());
    }

    Action1<String> setTextAction() {
        return text -> mTextView.setText(text);
    }
}
```