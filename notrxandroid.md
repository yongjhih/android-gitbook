# NotRxAndroid

依據 Android 各項元件出發，重新披上 RxJava 的皮。

RxAndroid Before:

```java
ViewObservable.bind(textView).clicks().subscribe(ev -> {});
```

NotRxAndroid After:

```java
rxTextView.textChanged().subscribe(ev -> {});
```

## See Also

* Jack Wharton 的 RxJava 電台訪談： http://fragmentedpodcast.com/episodes/7/
* https://github.com/JakeWharton/NotRxAndroid
