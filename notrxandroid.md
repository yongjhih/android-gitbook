# NotRxAndroid

依據 Android 各項元件出發，重新披上 RxJava 的皮。

RxAndroid Before:

```java
ViewObservable.bind(TextView).clicks().subscribe(ev -> {});
```

NotRxAndroid After:

```java
RxTextView.textChanged().subscribe(ev -> {});
```
