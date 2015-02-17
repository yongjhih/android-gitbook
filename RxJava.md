# RxJava

\#Promise \#RxJava \#Reactive \#Functional

一個 FRP 的實現，透過 FRP(Functional Reactive Programming) 概念。短期效用：有效避免巢狀 callback 增加可讀性以及減少 ```List<item>``` 的轉換成本。

1. *注意：這邊直接使用 lambda &lambda; 的表達式，如果你還不清楚，請跳轉到 [Lambda](lambda.md)*
2. *java8.stream 也實現了 Reactive。*

## 做中學

直接看 code 來對照，再來一一解釋運作原理。

只列出安裝同個 app 的朋友：

Before:

```java
List<Profile> friends = getFriends();
Iterator<Profile> it = friends.iterator();
while (it.hasNext()) {
    if (!f.getInstalled()) it.remove();
}
```

After:

```java
Observable<Profile> installedFriends = Observable.from(getFriends()).filter(f -> f.getInstalled());

List<Profile> installedFriendList = installedFriends.toList().toBlocking().single(); // 如果你堅持一定要傳遞 List
```

列出朋友名字：

Before:

```java
List<String> friendNames = new ArrayList<>();

for (Profile p : installedFriendList) {
    friendNames.add(p.getDisplayName());
}
```

After:

```java
List<String> friendNames = Observable.from(installedFriendList)
    .map(p -> p.getDisplayName())
    .toList().toBlocking.single();
```

一次達成的寫法，列出安裝同個 app 朋友的名字：


Before:

```java
// 這邊要改變寫法，不再沿用 List 來沿用 loop
List<String> friendNames = new ArrayList<>();
for (Profile p : getFriends()) {
    if (p.getInstalled()) friendNames.add(p.getDisplayName());
}
```

After:

```java
Observable<Profile> installedFriends = Observable.from(getFriends())
    .filter(f -> f.getInstalled());
Observable<String> installedFriendNames = installedFriends
    .map(p -> p.getDisplayName());
List<String> installedFriendNameList = installedFriendNames.toList().toBlocking.single(); // 這裡才開始做事
```

首先，你可以發現你可以維持一樣的寫法，再來如果你把界面都維持 Observable 來傳遞，你可以決定哪時候才去開跑，有效避免無謂的 loop 。(promise, lazy)

## 如何導入套用與改變撰寫

既有長時間存取的函式改成 Observable ：

```java
Observable<File> file = Observable.defer(() -> Observable.just(download()));
```

## Android 應該養成的習慣與注意事項

應該使用 ```AndroidObservable.bindFragment(fragment, observable)``` 來包裝你的 observable ，來避免操作 fragment 生命週期外的物件。例如：

```java
Observable.defer(() -> Observable.just(download())).subscribe(file -> {
    textView.setText(file);
});
```

如果你下載 download() 很久，然後離開了這個 fragment 後，才下載結束，這樣操作了 textView 就很有可能爆掉。你應該改成：

```java
AndroidObservable.bindFragment(fragment, Observable.defer(() -> Observable.just(download()))).subscribe(file -> {
    textView.setText(file);
});
```

## map() 介紹

我們最常用的是把 `List<TextView>` 轉成 `List<String>`，你可能會把整個 textViews 一一取出 `toString()` 然後抄一份：

```java

List<String> strings = new ArrayList<>();

for (TextView textView : textViews) {
   strings.add(textView.getText().toString());
}
```

萬一 textViews 有一萬筆，最終你其實在存取 strings 通常不會全部都用到，這樣就太浪費了。所以我們拿出牛仔精神 `Cow - Copy-On-Write`(Lazy/CallByNeed)，先寫好轉換程式，當拿到那筆再去轉換，當然缺點是 textViews 要一直拿著，要稍微留意一下。我們先想像一下，寫一個名稱叫做 MapList 的類別，先把 textViews 拿著，在取出的時候，再去跑轉換程式。這裡我們開放一個 ```map(Mappable)``` 好把轉換程式交給我們 (Mappable)。

```java
List<String> strings = new MapList<TextView, String>(textViews) // 先把 textViews 拿著
    .map(new Mappable<TextView, String> {
        @Override public String map(TextView view) { // 再取出時，會請我們轉換
           return textView.getText().toString();
        }
    });
```

這是我們自己寫一個 MapList 類別來達成，但是現在其實利用 RxJava 就可以辦到了。

```
List<String> strings = new IteratorOnlyList(Observable.from(textViews)
    .map(textView -> textView.getText().toString())
    .toBlocking()
    .getIterator());
```

不過為了維持 lazy ，又 RxJava 這邊只有提供到 Iterator ，所以我們沒有使用 `toList().toBlocking().single()`，你可以寫一個 IteratorOnlyList 把這個 iterator 包起來，方便傳遞，雖然很多操作都殘缺。


## 名詞解釋

Observable<T> 一份工作 task 一個未來 future , T 產品.

Observer<T> onEvent, Listener.

subscribe 下訂。

Subscription 訂單, 描述這是怎樣的工作，以及中間需要的製程，希望產生出什麼產品。下訂之後產生出來的訂單，這個訂單可以用來取消訂單來中止生產。



## 附錄：Android View 範例

寫一個讚計數器:

```java
        ViewObservable.clicks(findViewById(R.id.like_button))
            .map(clickEvent -> 1)
            .scan((increamnet, current) -> increament + current)
            .subscribe(likes -> {
                TextView likesView = (TextView) findViewById(R.id.likes_view);
                textView.setText(likes.toString());
            });
```

## See Also

小抄：https://gist.github.com/yongjhih/bbe3b528873c7eb671c6
