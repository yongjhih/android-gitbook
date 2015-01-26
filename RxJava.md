# RxJava

\#Promise \#RxJava \#Reactive \#Functional

透過 FRP(Functional Reactive Programming) 概念，有效避免巢狀 callback 增加可讀性以及減少 ```List<item>``` 的轉換成本。

可以接龍的 callback/listener.



## 做中學

直接看 code 來對照學習，先可以動，再來一一解釋運作原理。



## map() 介紹

我們最常用的是把 `List<TextView>` 轉成 `List<String>`，你可能會把整個 textViews 一一取出 `toString()` 然後抄一份：

```java

List<String> strings = new ArrayList<>();

for (TextView textView : textViews) {
   strings.add(textView.getText().toString());
}
```

萬一 textViews 有一萬筆，最終你其實在存取 strings 通常不會全部都用到，這樣就太浪費了。所以我們拿出牛仔精神 `Cow - Copy-On-Write`，先寫好轉換程式，當拿到那筆再去轉換，當然缺點是 textViews 要一直拿著，要稍微留意一下。我們先想像一下，寫一個名稱叫做 MapList 的類別，先把 textViews 拿著，在取出的時候，再去跑轉換程式。這裡我們開放一個 ```map(Mappable)``` 好把轉換程式交給我們 (Mappable)。

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


## 名詞解釋

Observable<T> 一份工作 task 一個未來 future , T 產品.

Observer<T> onEvent, Listener.

Subscription 訂單, 描述這是怎樣的工作，以及中間需要的製程，希望產生出什麼產品。

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

## See Also
我的小抄：https://gist.github.com/yongjhih/bbe3b528873c7eb671c6
