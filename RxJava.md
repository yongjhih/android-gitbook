# RxJava

\#Promise \#RxJava \#Reactive \#Functional

一個 FRP 的實現，透過 FRP(Functional Reactive Programming) 概念。短期效用：有效避免巢狀 callback 增加可讀性以及減少 ```List<item>``` 的轉換成本。

鑑於 RxJava 太少範例可以參考，才開始撰寫這篇文章，而這篇文章的技術來源大多是閱讀 RxJava 源碼、測試程式源碼以及其 github Wiki 而來。如果有謬誤之處，歡迎用各種方式聯絡我。基本上，本人極其懶惰，採取開放共筆。

1. *注意：這邊直接使用 lambda &lambda; 的表達式，如果你還不清楚，請跳轉到 [Lambda](lambda.md)*
2. *java8.stream 也實現了 Reactive。*

## 有效解決重複的 loop 增進效能，維持同個 loop

只列出安裝同個 app 的朋友：

Before:

```java
List<Profile> getInstalledFriendList(/* @Writable */List<Profile> friends) {
    List<Profile> installedFriendList = friends;
    Iterator<Profile> it = friends.iterator();

    while (it.hasNext()) {
        if (!f.getInstalled()) it.remove();
    }

    return installedFriendList;
}
```

After:

```java
Observable<Profile> getInstalledFriendObs(List<Profile> friends) {
    return Observable.from(friends).filter(p -> p.getInstalled());
}

// 如果你堅持一定要傳遞 List
List<Profile> getInstalledFriendList(List<Profile> friends) {
    return getInstalledFriendObs(friends).toList().toBlocking().single();
}
```

列出朋友名字：

Before:

```java
List<String> getFriendNameList(List<Profile> friends) {
    List<String> friendNameList = new ArrayList<>();

    for (Profile p : friends) {
        friendNames.add(p.getDisplayName());
    }

    return friendNameList;
}
```

After:

```java
Observable<String> getFriendNameObs(List<Profile> friends) {
    return Observable.from(friends).map(p -> p.getDisplayName());
}

// 如果你堅持一定要傳遞 List
List<String> getFriendNameList(List<Profile> friends) {
    return getFriendNameObs(friends).toList().toBlocking().single();
}
```

一次達成的寫法，列出安裝同個 app 朋友的名字：


Before:

```java
// 如果不改變寫法，會整整跑完兩個 loop
List<String> getInstalledFriendNameList(List<Profile> friends) {
    return getFriendNameList(getInstalledFriendList());
}
```

```java
// 你可改變寫法，以沿用 loop
List<String> getInstalledFriendNameList(List<Profile> friends) {
    List<String> installedFriendNameList = new ArrayList<>();
    for (Profile p : friends) {
        if (p.getInstalled()) friendNames.add(p.getDisplayName());
    }
    return installedFriendNameList;
}
```

After:

```java
// 而 Observable 不用刻意改變寫法，直接組起來就好：
List<String> getInstalledFriendNameList(List<Profile> friends) {
    return Observable.from(friends)
        .filter(p -> p.getInstalled())
        .map(p -> p.getDisplayName())
        .toList().toBlocking().single(); // 如果你堅持一定要傳遞 List
}
```

首先，你可以發現你可以維持一樣的寫法，再來如果你把界面都維持 Observable 來傳遞，你可以決定哪時候才去開跑，以及拿幾筆才作幾筆過濾與轉換，有效避免無謂的全數過濾與轉換。

把界面維持 Observable 傳遞：

```java
Observable<Profile> getInstalledFriendObs(Observable<Profile> friendObs) {
    return friendObs.filter(p -> p.getInstalled());
}

Observable<Profile> getFriendNameObs(Observable<Profile> friendObs) {
    return friendObs.map(p -> p.getDisplayName());
}

Observable<Profile> getInstalledFriendNameObs(List<Profile> friends) {
    return getFriendNameObs(getInstalledFriendObs(Observable.from(friends)));
}
```

只做 100 筆過濾與轉換：

```java
getInstalledFriendNameObs(friends)
    .take(100) // 拿個 100 筆
    .toList().toBlocking().single();
```

## 拉平巢狀 callback 增加易讀性

Before:

```java
void login(Activity activity, LoginListenr loginListener) {
    loginFacebook(activity, fbUser -> {
        getFbProfile(fbUser, fbProfile -> {
            loginParse(fbProfile, parseUser -> {
                getParseProfile(fbProfile, parseProfile -> {
                    loginListener.onLogin(parseProfile);
                });
            });
        });
    });
}
```

After:

```java
// wrap callback functions in Observable<?>
Observable<FbUser> loginFacebook(Activity activity) {
    return Observable.create(sub -> {
        loginFacebook(activity, fbUser -> {
            sub.onNext(fbUser);
            sub.onCompleted();
        });
    });
}

Observable<FbProfile> getFbProfile(FbUser fbUser) { ... }
Observable<ParseUser> loginParse(FbProfile fbProfile) { ... }
Observable<ParseProfile> getParseProfile(ParseUser parseUser) { ... }

void login(Activity activity, LoginListener loginListener) {
    Observable.just(activity)
        .flatMap(activity -> loginFacebook(activity))
        .flatMap(fbUser -> getFbProfile(fbUser))
        .flatMap(fbProfile -> loginParse(fbProfile))
        .flatMap(parseUser -> getParseProfile(parseUser))
        .subscribe(parseProfile -> loginListener.onLogin(parseProfile));
}
```

## 如何導入套用與改變撰寫

### 既有長時間存取的函式改成 Observable

```java

File download(String url) { ... return file; }

Observable<File> downloadObs(String url) {
    return Observable.defer(() -> Observable.just(download(url)));
}
```

### 既有的 callback 改成 Observable

```java
Observable<ParseUser> loginParseWithFacebook(Activity activity) {
    return Observable.create(sub -> {
        ParseFacebookUtils.logIn(Arrays.asList("public_profile", "email"), activity, new LogInCallback() {
            @Override
            public void done(final ParseUser parseUser, ParseException err) {
                if (err != null) {
                    sub.onError(err);
                } else {
                    sub.onNext(parseUser);
                    sub.onCompleted();
                }
            }
        });
    })
}
```

另一種方法， Subject ，通常為了跨執行緒廣播，例如做一條 EventBus 。而這邊僅為舉例如何使用 subject 方式。

```java
Observable<ParseUser> loginParseWithFacebook(Activity activity) {
    //final Subject<ParseUser, ParseUser> subject = new SerializedSubject<>(PublishSubject.create()); // crossover thread
    final PublishSubject<ParseUser> = PublishSubject.create();
    ParseFacebookUtils.logIn(Arrays.asList("public_profile", "email"), activity, new LogInCallback() {
        @Override
        public void done(final ParseUser parseUser, ParseException err) {
            if (err != null) {
                subject.onError(err);
            } else {
                subject.onNext(parseUser);
                subject.onCompleted();
            }
        }
    });
    return subject.asObservable();
}
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

## 轉換 map()

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

如果你想維持 List 界面，為了維持 lazy ，又 RxJava 這邊只有提供到 Iterator ，所以我們沒有使用 `toList().toBlocking().single()`，你可以寫一個 IteratorOnlyList 把這個 iterator 包起來，方便傳遞，雖然很多操作都殘缺。

盡可能改用 Observable 作為界面。

## 組合資料 zip()

```java
Observable<User> getUser(Activity activity) {
    Observable.zip(getFbUser(activity), getParseUser(activity),
        (fbUser, parseUser) -> getUser(fbUser, parseUser));
}
```

## 去除重複資料 distinct()

```java
Observable<User> getPostedUsers(Observable<Post> posts) {
    return posts.map(post -> post.getUser())
        .distinct(user -> user.getObjectId());
}
```

## 多方合併 concat(), merge()

```java
Observable<User> getActivityUsers(Observable<Post> posts, Observable<Comment> comments) {
    return Observable.merge(posts.map(post -> post.getUser()),
        comments.map(comment -> comment.getUser()))
        .distinct(user -> user.getObjectId());
}
```

## 排序 toSortedList()

## 分組 groupBy()

## 分段 window()

## 快取 cache()

## 重試 retry()

## 名詞解釋

Observable<T> 一份工作 task 一個未來 future , T 產品.

Subscriber/Observer<T> onEvent, Listener. 提貨券.

subscribe 下訂。

Subscription 訂單, 描述這是怎樣的工作，以及中間需要的製程，希望產生出什麼產品。下訂之後產生出來的訂單，這個訂單可以用來取消訂單來中止生產。

## 動手玩

```bash
git clone https://github.com/yongjhih/RxJava-GroupByTest
cd RxJava-GroupByTest
./gradlew -PmainClass=com.github.yongjhih.GroupByTest execute
```

修改 src/main/java/com/github/yongjhih/GroupByTest.java 內容就可以自己玩了。

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

* 範例：https://speakerdeck.com/jakewharton/2014-1?slide=13
* 小抄：https://gist.github.com/yongjhih/bbe3b528873c7eb671c6


