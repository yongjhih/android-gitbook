# RxBinding (RxNotAndroid)

依據 Android 各項元件出發，重新披上 RxJava 的皮。

RxAndroid Before:

```java
ViewObservable.clicks(textView).subscribe(ev -> {});
```

RxBinding After:

```java
RxTextView.textChanged(textView).subscribe(ev -> {});
```

基本上與 ViewObservable, ogaclejapan/RxBinding 都是處理 View 相關的連動。

## See Also

* Jack Wharton 的 RxJava 電台訪談： http://fragmentedpodcast.com/episodes/7/
* https://github.com/JakeWharton/RxBinding
* https://github.com/ogaclejapan/RxBinding
