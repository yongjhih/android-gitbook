# RxJava

\#Promise \#RxJava \#Reactive \#Functional

可以接龍的 callback/listener.

寫一個讚計數器:

```java
        ViewObservable.clicks(findViewById(R.id.like_button))
            .map(new Func1<OnClickEvent, Integer>() {
                @Override
                public Integer call(OnClickEvent clickEvent) {
                    return 1;
                }
            })
            .scan(new Func2<Integer, Integer, Integer>() {
                @Override
                public Integer call(Integer increamnet, Integer current) {
                    return increament + current;
                }
            })
            .subscribe(new Action1<Integer>() {
                @Override
                public void call(Integer likes) {
                    TextView likesView = (TextView) findViewById(R.id.likes_view);
                    textView.setText(likes.toString());
                }
            });
```

## map() 介紹

我們最常用的是把 `List<TextView>` 轉成 `List<String>`，你可能會把整個 textViews 取出 text 然後抄一份：

```java

List<String> strings = new ArrayList<>();

for (TextView textView : textViews) {
   strings.add(textView.getText().toString());
}
```

萬一 textViews 有一萬筆，最終你其實在存取 strings 通常不會全部都用到，這樣就太浪費了。所以我們拿出牛仔精神 `Cow - Copy-On-Write`，先寫好轉換程式，當拿到那筆在去轉換，當然缺點是 textViews 要一直拿著，要稍微留意一下。我們先想像一下，寫一個名稱叫做 MapList 的類別，先把 textViews 拿著，然後人家跟 MapList 取出來的時候再去跑轉換程式。這裡我們開放一個 ```map(Mappable)``` 來讓人家時把轉換程式給我們 (Mappable)。

```java
List<String> strings = new MapList<TextView, String>(textViews) // 先把 textViews 拿著
    .map(new Mappable<TextView, String> {
        @Override public String map(TextView view) { // 再取出時，會請我們轉換
           return textView.getText().toString();
        }
    });
```

這是我們自己寫一個 MapList 類別來達成，但是現在其實利用 RxJava 就可以辦到了。

```java
Iterator<String> strings = Observable.from(textViews).map(new Func1<TextView, String>() {
    @Override public String call(TextView textView) {
        return textView.getText().toString();
    }.toBlocking().getIterator();
})
```

不過 RxJava 這邊只有提供到 ```Iterator``` ，你可以寫一個 IteratorOnlyList 把這個 iterator 包起來，方便傳遞，雖然很多操作都殘缺。
```





