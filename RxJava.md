# RxJava

\#Promise \#RxJava \#Reactive \#Functional

## 前言

RxJava, Reactive Java,
一個 Java FRP (Functional Reactive Programming) 的實現。可以接龍 的 callback 。

短期效用：有效避免巢狀 callback 增加可讀性以及減少 ```List<item>``` 的轉換成本。

鑑於 RxJava 太少範例可以參考，才開始撰寫這篇文章，而這篇文章的技術來源大多是閱讀 RxJava 源碼、測試程式源碼以及其 github Wiki 而來。如果有謬誤之處，歡迎用各種方式聯絡我。基本上，本人極其懶惰，採取開放共筆。

1. *注意：這邊直接使用 lambda &lambda; 的表達式，如果你還不清楚，請跳轉到 [Lambda](lambda.md)*
2. *java8.stream 也實現了 Reactive。*

先看一些範例對照後，了解樣貌之後，我們再來討論 RxJava 基本使用概念與方法。

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
    return Observable.create(sub -> { // Observable.OnSubscriber
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

另一種方法，Subject ：

```java
Observable<ParseUser> loginParseWithFacebook(Activity activity) {
    ReplaySubject<ParseUser> subject = ReplaySubject.create();
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

p.s. *這邊的 Subject 方法 與 Observable.create() 方法其實有差異， Observable.create() 內的 OnSubscriber 直到 subscribe() 才會執行。但 Subject 方法會馬上跑，所以這邊用 ReplaySubject 來記住進貨。*

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

我們經常把 `List<A>` 轉成 `List<B>`，如： `List<TextView>` 轉成 `List<String>`，你可能會把整個 textViews 一一取出 `toString()` 然後抄一份：

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

這是我們自己寫一個 MapList 類別來達成，而現在有了 RxJava： 
```
List<String> strings = new IteratorOnlyList(Observable.from(textViews)
    .map(textView -> textView.getText().toString())
    .toBlocking()
    .getIterator());
```

如果你想維持 List 界面，為了維持 lazy ，又 RxJava 這邊只有提供到 Iterator ，所以我們沒有使用 `toList().toBlocking().single()`，你可以寫一個 IteratorOnlyList 把這個 iterator 包起來，方便傳遞，雖然很多操作都殘缺。

盡可能還是改用 Observable 作為界面吧。

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

## 多方合併 merge(), concatWith()

```java
Observable<User> getActivityUsers(Observable<Post> posts, Observable<Comment> comments) {
    return Observable.merge(posts.map(post -> post.getUser()),
        comments.map(comment -> comment.getUser()))
        .distinct(user -> user.getObjectId());
}
```

## 如何使用

在我們看過一些對照組之後，大致上瞭解未來在使用上會呈現什麼樣貌。所以我們開始回頭學學，如何開始使用 RxJava 。

首先你要認識基本操作元件：Observable<Result> ， 如果你知道 AsyncTask<Input, Progress, Result> 或者 FutureTask<Result>  了話，基本上差不多。先看這個例子：

```java
// AsyncTask 版本
AsyncTask<Void, Void, String> helloAsync = new AsyncTask<>() {
    @Override public String doInBackground(Void... voids) {
        return "Hello, world!";
    }
}
```

```java
// FutureTask 版本
FutureTask<String> helloFuture =
    new FutureTask<>(new Callable<String>() {
        @Override public String call() {
            return "Hello, world!";
        }
    });
```

```java
// Observable 版本
Observable<String> helloObs = Observable.create(
    new Observable.OnSubscribe<String>() {
        @Override
        public void call(Subscriber<? super String> sub) {
            sub.onNext("Hello, world!");
            sub.onCompleted(); // 因為 Observable 支援複數的關係，所以需要 onCompleted() ，到時候再細說。
        }
    }
);

// Observable lambda 版本：
Observable<String> helloObs = Observable.create(sub -> {
        sub.onNext("Hello, world!");
        sub.onCompleted();
    });
```

這些 AsyncTask, FutureTask, Observable 都是生產者，定義出資料的產生，接下來，當產品生出來的時候通知你。所以我們補上 listener ，RxJava 稱之為 Subscriber：

```java
Subscriber<String> helloSubscriber = new Subscriber<>() {
    @Override public void onNext(String string) { System.out.println(string); }
    @Override public void onCompleted() { }
    @Override public void onError(Throwable e) { }
};

helloObs.subscribe(helloSubscriber); // 你可以下訂(subscribe()) ，產品出產時就會通知你了(Subscriber)。
```

通常我們會用 `subscribe(Action1<? super T> onNext, Action1<Throwable> onError), Action1<? super T> onCompleted)` 來搭配 lambda ，寫起來會比較簡便：

```java
helloObs.subscribe(string -> System.out.println(string));
helloObs.subscribe(string -> System.out.println(string), e -> e.printStackTrace());
helloObs.subscribe(string -> System.out.println(string), e -> e.printStackTrace(), () -> System.out.println("onCompleted"));
```

我們再稍微回到 Observable.create() , 如果你的原料早就準備好了，我們可以寫成：

```java
Observable.just("Hello, world!").subscribe(string -> System.out.println(string));
```

接下來是簡單的加工：

```java
Observable.just("Hello, world!")
    .map(string -> "andrew: " + string) // "andrew: Hello, world!"
    .map(string -> string.length())
    .subscribe(length -> System.out.println(length)); // 21, 請不要真的去算長度, 此處僅示意.
```

所以從原物料的進貨作業(create(), just())到加工(map())到下訂(subscribe()) 有了流程上的基本認識。我們可以討論幾個站點的作用與概念:

首先，生產過程(進貨與加工)會被認定不可預期的處理時間，很有可能會很久的意思。

另一個是，JIT(Just In Time)，零庫存概念，也就是說你可以定義很多道加工手續，但是再沒有下訂之前，通通都不會開跑的，包括進貨作業。

接下來，換一個比較實際的例子：

```java
Observable.just("http://yongjhih.gitbooks.io/feed/content/RxJava.html")
    .map(url -> download(url))
    .subscribeOn(Schedulers.io()) // 把生產加工過程丟到背景去做
    .subscribe(file -> System.out.println(file));
```

如果你有很多網址要下載，你可能會這樣做：

```java
Observable.just(Arrays.asList("http://yongjhih.gitbooks.io/feed/content/RxJava.html",
    "http://yongjhih.gitbooks.io/feed/content/README.html"))
    .map(urls -> {
        List<File> files = new ArrayList<>();
        for (String url : urls) files.add(download(url));
        return files;
    })
    .subscribeOn(Schedulers.io())
    .subscribe(files -> {
        for (File file : files) System.out.println(file);
    });
```

但是這種情況，你應該使用 Observable.from() ：

```java
Observable.from(Arrays.asList("http://yongjhih.gitbooks.io/feed/content/RxJava.html",
    "http://yongjhih.gitbooks.io/feed/content/README.html"))
    .map(url -> download(url))
    .subscribeOn(Schedulers.io())
    .subscribe(file -> System.out.println(file));
```

如果原料是複數，但是加工時，要單數一個一個處理，改用 Observable.from() 會比較好操作，如果你用 Observable.just() 那就會拿到一個 List 。其實有個方法可以途中攤平， flatMap(list -> Observable.from(list)) 來攤平轉成單數：

```java
Observable.just(Arrays.asList("http://yongjhih.gitbooks.io/feed/content/RxJava.html",
    "http://yongjhih.gitbooks.io/feed/content/README.html"))
    .map(urls -> {
        List<File> files = new ArrayList<>();
        for (String url : urls) files.add(download(url));
        return files;
    })
    .flatMap(files -> Observable.from(files)) // 可以這裡才攤平
    .subscribeOn(Schedulers.io())
    .subscribe(file -> System.out.println(file));
```

## toBlocking().single()

這行會卡住，直到拿到一個為止。

與 toBlocking().take(1) 有何不同。single() 與 take(1) 最大的不同是，single() 只能使用在單數的 Observable 上，否則會噴 exception 。

怎樣會是單數的 Observable ？例如：

```java
Observable<List<Integer>> = Observable.range(1, 10).toList(); // 這就是一個單數的 Observable
```

```java
List<Integer> integers = Observable.range(1, 10).toList().toBlocking().single(); // pass
```

```java
Integer i = Observable.range(1, 10).toBlocking().take(1); // pass
```

```java
Integer i = Observable.range(1, 10).toBlocking().single(); // exception
```


## 名詞解釋

描述這是怎樣的工作，以及中間需要的製程，希望產生出什麼產品。

Observable<T> 一份工作 task 一個未來 future , T 產品. 相當於 AsyncTask<INPUT, PROGRESS, T>, Future<T>

Subscriber/Observer<T> onEvent, Listener. 提貨券.

subscribe 下訂。

Subscription 訂單, subscribe 下訂之後產生出來的訂單，這個訂單可以用來取消訂單來中止生產。

## 動手玩

```bash
git clone https://github.com/yongjhih/RxJava-retrofit-github-sample
cd RxJava-retrofit-github-sample
./gradlew execute
```

修改 src/main/java/com/github/yongjhih/Main.java 內容就可以自己玩了。

## Subject

碼頭，多方進貨與多方出貨，例如做一條 EventBus 廣播系統。

## Exception 處理

如果發生 exception 重試.

最常用的是 `retry(Func2<Integer, Throwable, Boolean> predicate)`

如果是 NullPointerException 才重試:

```java
retry((c, e)) -> e instanceof NullPointerException);
```

一直重試：

```java
retry() 
```

重試 3 次：

```java
retry(3)
```

使用 Handler `retryWhen(final Func1<? super Observable<? extends Throwable>, ? extends Observable<?>> notificationHandler)`，例如，隨著重試次數延後重試時間:

```java
Observable.create((Subscriber<? super String> s) -> {
    s.onError(new RuntimeException("always fails"));
}).retryWhen(attempt -> { // 每次的 exception 都會進來 ，當作 Observable<Throwable>.subscribe(e -> {}) 來看待
    return attempt.zipWith(Observable.range(1, 3), (n, i) -> i).flatMap(i -> {
        System.out.println("delay retry by " + i + " second(s)");
        return Observable.timer(i, TimeUnit.SECONDS);
    });
}).toBlocking().subscribe(System.out::println);
```

忽略 exception：

`.onErrorResumeNext(e -> Observable.empty());`

或傳個替代資料：

```java
.flatMap(parseUser -> {
    return Observable.zip(createChannel(parseUser), createFriendRole(parseUser), (user, user2) -> user)
        .onErrorResumeNext(e -> Observable.just(parseUser));
})
```

## 排序 toSortedList()

## 分組 groupBy()

## 分段 window()

## 快取 cache()

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

* https://speakerdeck.com/jakewharton/2014-1?slide=13
* http://slides.com/yaroslavheriatovych/frponandroid

一些 Rx 開源：

* https://github.com/yongjhih/RxParse
* https://github.com/ogaclejapan/RxBinding
* https://github.com/mcharmas/Android-ReactiveLocation
* https://github.com/square/sqlbrite
* https://github.com/pardom/Ollie
* https://github.com/eccyan/RxJava-Optional
* https://github.com/frankiesardo/ReactiveContent
* https://github.com/evant/rxloader
* https://github.com/vyshane/rex-weather
* https://github.com/ReactiveX/RxNetty
 
* 小抄：https://gist.github.com/yongjhih/bbe3b528873c7eb671c6