# RxJava

\#Promise \#Reactive \#Functional \#Monad \#Stream

## 前言

RxJava, Reactive Java,
一個 Java FRP (Functional Reactive Programming) 的實現。

可以接龍的 callbacks。

類似於 command line 的 `|` 的概念。

```sh
ls | wc -l
```

RxJava 大約從 2013 年中一場[研討會](http://www.infoq.com/presentations/netflix-functional-rx)之後漸漸嶄露頭角。大約一年後 2014 年中後開始許多文章出現(英文)。2014 年底 trello 甚至應聘熟悉 RxJava 的開發者。

有誰在用？先從有貢獻來看：

* Square
* SoundCloud
* NetFlix

短期效用：有效避免巢狀 callback 增加可讀性以及減少 `List<item>` 的轉換成本。

鑑於 RxJava 太少範例可以參考，才開始撰寫這篇文章偏向範例而不是從學術性角度介紹起，而這篇文章的技術來源大多是閱讀 RxJava 源碼、測試程式源碼以及其 github Wiki 而來。如果有謬誤之處，歡迎用各種方式聯絡我。

1. *注意：這邊直接使用 lambda &lambda; 的表達式，如果你還不清楚，請跳轉到 [Lambda](lambda.md)*
2. *java8.stream 也實現了 Reactive。*

先看一些範例對照後，了解樣貌之後，我們再來討論 RxJava 基本使用概念與方法。


## 有效解決重複的 loop 增進效能，維持同個 loop

假設我們有一萬名使用者 `List<User> users`，其中有五千名女性使用者。

列出使用者年齡：

Before:

```java
List<Integer> getAgeList(List<User> users) {
    List<Integer> ageList = new ArrayList<>();

    for (User user : users) {
        ageList.add(user.getAge());
    }

    return ageList;
}
```

After:

```java
Observable<Integer> getAgeObs(List<User> users) {
    return Observable.from(users).map(user -> user.getAge());
}

// 如果你堅持一定要傳遞 List
List<Integer> getAgeList(List<User> users) {
    return getAgeObs(users).toList().toBlocking().single();
}
```

列出女性使用者：

Before:

```java
List<User> getFemaleList(/* @Writable */List<User> users) {
    List<User> femaleList = users;
    Iterator<User> it = users.iterator();

    while (it.hasNext()) {
        if (it.getGender() != User.FEMALE) it.remove();
    }

    return femaleList;
}
```

After:

```java
Observable<User> getFemaleObs(List<User> users) {
    return Observable.from(users).filter(user -> user.getGender() == User.FEMALE);
}

// 如果你堅持一定要傳遞 List
List<User> getFemaleList(List<User> users) {
    return getFemaleObs(users).toList().toBlocking().single();
}
```

組合一下就可以列出女性使用者年齡：

Before:

```java
List<Integer> getFemaleAgeList(List<User> users) {
    getAgeList(getFemaleList(users)); // 如果不改變寫法，會整整跑完兩個 loop
}
```

你可以發現原本的寫法有個瑕疵，就是會先繞完一萬使用者，找出來五千名女性後，為了詢問年紀，只好再繞一次這五千名女性。

可以第一次找出女性使用者時，就順便問一下年紀嗎？

為了避免重複迴圈，你可改變寫法，以沿用 loop ：

```java
List<Integer> getFemaleAgeList(List<User> users) {
    //getAgeList(getFemaleList(users));
    List<Integer> ageList = new ArrayList<>();

    for (User user : users) {
        if (user.getGender() == User.FEMALE) {
            ageList.add(user.getAge());
        }
    }

    return ageList;
}
```

而 Observable 不用刻意改變寫法，直接組起來就好：

After:

```java
Observable<Integer> getFemaleAgeObs(List<User> users) {
    return getFemaleObs(users).map(user -> user.getAge());
}
```

你可以發現維持一樣的寫法，它會同時做兩件事情：找出女性順便詢問年紀(過濾與轉換)，避免重複的迴圈。



## 提前打斷迴圈的能力，避免不必要的過濾與轉換

列出一百名女性使用者：

Before:

```java
List<User> getFemaleList(List<User> users, int limit) {
    return getFemaleList(users).subList(0, limit);
}

// 這裡會完整繞完萬名使用者，找出千名女性使用者後，才抽出百名。
getFemaleList(users, 100);
```

After:

```java
Observable<User> getFemaleObs(List<User> users, int limit) {
    return getFemaleObs(users).take(limit);
}

// 這裡會蒐集到百名女性使用者後，即停止。
getFemaleObs(users, 100);
```

我們可以從這裡看到差異，儘管你只要找出百名女性使用者，原本的寫法卻會繞完萬名使用者，找出所有女性使用者，再分割前一百名。
而 Observable 會聰明的找到第一百名女性使用者就馬上停止。原本的寫法要做到提前停止，就必須改寫：

```java
List<User> getFemaleList(/* @Writable */List<User> users) { ... }

List<User> getFemaleList(List<User> users, int limit) {
    //return getFemaleList(users).subList(0, limit);
    List<User> femaleList = new ArrayList<>();

    int i = 0;
    for (User user : users) {
        if (i >= limit) break;
        if (user.getGender() == User.FEMALE) femaleList.add(user);
        i++;
    }

    return femaleList;
}

// 比較靈活一點的寫法，提供 predicate function
List<User> getFemaleList(List<User> users, Func2<Boolean, User, Integer> predicate) {
    List<User> femaleList = new ArrayList<>();

    int i = 0;
    for (User user : users) {
        if (!predicate.call(user, i)) break;
        if (user.getGender() == User.FEMALE) femaleList.add(user);
        i++;
    }

    return femaleList;
}

getFemaleList(users, 100);
getFemaleList(users, (user, i) -> i <= 100); // predicate Func2
```

這種靈活的方法套用在各個資料流身上，也就是 RxJava 所提供的 operators 。

接下來，開始一點組合應用：

列出前百名女性使用者年齡：

Before:

```java
List<Integer> getFemaleAgeList(List<User> users, int limit) {
    return getAgeList(getFemaleList(users, limit));
}
```

After:

```java
List<Integer> getFemaleAgeList(List<User> users, int limit) {
    return Observable.from(users)
        .filter(user -> user.getGender() == User.FEMALE)
        .take(limit)
        .map(user -> user.getAge())
        .toList().toBlocking().single(); // 如果你堅持一定要傳遞 List
}
```

你可以之後才決定選幾筆，Observable 選幾筆才作幾筆過濾與轉換，有效避免無謂的全數過濾與轉換。

你可以把界面維持 Observable 傳遞，維持一樣的撰寫方法，直接組裝起來就好：

```java
Observable<User> getFemaleObs(Observable<User> userObs) {
    return userObs.filter(user -> user.getGender() == User.FEMALE);
}

Observable<Integer> getAgeObs(Observable<User> userObs) {
    return userObs.map(user -> p.getAge());
}

Observable<Integer> getFemaleAgeObs(List<User> users) {
    return getAgeObs(getFemaleObs(Observable.from(users)));
}
```

只做 100 筆過濾與轉換(女性、年齡)：

```java
getFemaleAgeObs(users)
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
void login(Activity activity, LoginListener loginListener) {
    Observable.just(activity)
        .flatMap(activity -> loginFacebook(activity))
        .flatMap(fbUser -> getFbProfile(fbUser))
        .flatMap(fbProfile -> loginParse(fbProfile))
        .flatMap(parseUser -> getParseProfile(parseUser))
        .subscribe(parseProfile -> loginListener.onLogin(parseProfile));
}

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
```



## 如何導入套用與改變撰寫

### 既有長時間存取的函式改成 Observable

...

```java

File download(String url) { ... return file; }

Observable<File> downloadObs(String url) {
    return Observable.defer(() -> Observable.just(download(url)));
}
```

...

```java
downloadObs(url).subscribeOn(Schedulers.io()).subscribe(file -> {
  System.out.println(file);
});
```

### 既有的 callback 改成 Observable

...

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

...

```java
loginParseWithFacebook(activity).subscribe(parseUser -> {
  System.out.println(parseUser);
});
```

## 轉換 map()

我們經常把 `List<T>` 轉成 `List<R>`，如： `List<TextView>` 轉成 `List<String>`，你可能會把整個 textViews 一一取出 `toString()` 然後抄一份：

```java

List<String> strings = new ArrayList<>();

for (TextView textView : textViews) {
   strings.add(textView.getText().toString());
}
```

萬一 textViews 有一萬筆，最終你其實在存取 strings 通常不會全部都用到，這樣就太浪費了。所以我們拿出牛仔精神 `Cow - Copy-On-Write`(Lazy/CallByNeed)，先寫好轉換程式，當拿到那筆再去轉換，當然缺點是 textViews 要一直拿著，要稍微留意一下。我們先想像一下，寫一個名稱叫做 MapList 的類別，先把 textViews 拿著，在取出的時候 (@Override public E get(int index))，再去跑轉換程式。這裡我們開放一個 `map(Mappable)` 好把轉換程式交給我們 (Mappable)。

```java
MapList<T, R> extends ArrayList<R> { // @Unmodifitable
    List<T> list;
    Mappable<T, R> mapper;
    
    public MapList(List<T> list) {
        super();
        this.list = list;
    }
    
    @Override public R get(int i) {
        return mapper.map(list.get(i));
    }
    
    public MapList<T, R> map(Mappable<T, R> mapper) {
        this.mapper = mapper;
        return this;
    }
    
    public interface Mappable<T, R> {
        R map(T t);
    }
}
```

```java
List<String> strings = new MapList<TextView, String>(textViews).map(textView -> textView.getText().toString());
```

這是我們自己寫一個 MapList 類別來達成，而現在有了 RxJava： 

```java
List<String> strings = new IteratorOnlyList(Observable.from(textViews)
    .map(textView -> textView.getText().toString())
    .toBlocking()
    .getIterator());
```

如果你想維持 List 界面，為了維持 lazy ，又 RxJava 這邊只有提供到 Iterator ，所以我們沒有使用 `toList().toBlocking().single()`，你可以寫一個 IteratorOnlyList 把這個 iterator 包起來，方便傳遞，雖然很多操作都殘缺。

盡可能還是改用 Observable 作為界面吧。



## 去除重複資料 distinct()

列出有發文的使用者：

Before:

```java
List<User> getPostedUsers(List<Post> posts) {
    Map<String, User> users = new HashMap<>();
    for (Post post : posts) {
        User user = post.getUser();
        users.put(user.getObjectId(), user);
    }
    return new ArrayList<>(users.values());
}
```

After:

```java
Observable<User> getPostedUsers(List<Post> posts) {
    return Observable.from(posts).map(post -> post.getUser())
        .distinct(user -> user.getObjectId());
}
```



## 多方合併 merge(), concatWith()

列出貼文以及留言的使用者：

Before:

```java
List<User> getActivityUsers(Collection<Post> posts, Collection<Comment> comments) {
    Map<String, User> users = new HashMap<>();
    for (Post post : posts) {
        User user = post.getUser();
        users.put(user.getObjectId(), user);
    }
    for (Comment comment : comments) {
        User user = comment.getUser();
        users.put(user.getObjectId(), user);
    }
    return new ArrayList<>(users.values());
}
```

After:

```java
Observable<User> getActivityUsers(Observable<Post> posts, Observable<Comment> comments) {
    return Observable.merge(posts.map(post -> post.getUser()),
        comments.map(comment -> comment.getUser()))
        .distinct(user -> user.getObjectId());
}
```

`merge()` 與 `concatWith()` 最大的差異是， `merge()` 是併發同時進貨 ，所以會交錯，而 `concatWith` 則是排隊等到前面進貨完才換下一位。

優先列出貼文的使用者後，才列出留言的使用者：

```java
Observable<User> getActivityUsers(Observable<Post> posts, Observable<Comment> comments) {
    return posts.map(post -> post.getUser())
        .concatWith(comments.map(comment -> comment.getUser()))
        .distinct(user -> user.getObjectId());
}
```

## 如何使用

在我們看過一些對照組之後，大致上瞭解未來在使用上會呈現什麼樣貌。

我們開始回頭學學 What/Why/How，什麼是 RxJava 、為什麼要 RxJava 、如何開始使用 RxJava 。

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

如果你有很多網址要下載，你可能會想到：

```java
Observable<List<String>> urlList = Observable.just(Arrays.asList("http://yongjhih.gitbooks.io/feed/content/RxJava.html");
```

你可以留意到這時 Observable 產線上是 `List<String>` 了。所以你可能會這樣做：

```java
    urlList.map(list -> {
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
Observable<String> urls = Observable.from(Arrays.asList("http://yongjhih.gitbooks.io/feed/content/RxJava.html",
    "http://yongjhih.gitbooks.io/feed/content/README.html"));
```

```java
urls.map(url -> download(url))
    .subscribeOn(Schedulers.io())
    .subscribe(file -> System.out.println(file));
```

或者

```java
Observable.just("http://yongjhih.gitbooks.io/feed/content/RxJava.html",
    "http://yongjhih.gitbooks.io/feed/content/README.html"));
```

如果原料是 List，但是加工時，想要一個一個處理，用 Observable.from() 會比較好操作，如果你用 Observable.just() 那就會拿到一個 List 。其實有個方法可以展開： `.flatMap(list -> Observable.from(list))`：

```java
Observable.just(Arrays.asList("http://yongjhih.gitbooks.io/feed/content/RxJava.html",
    "http://yongjhih.gitbooks.io/feed/content/README.html"))
    .map(urls -> {
        List<File> files = new ArrayList<>();
        for (String url : urls) files.add(download(url));
        return files;
    })
    .flatMap(files -> Observable.from(files)) // 可以這裡才展開
    .subscribeOn(Schedulers.io())
    .subscribe(file -> System.out.println(file));
```

`map()` 是轉換產線上的物件 ， flatMap() 是轉換 `Observable<Object>` 的方法，舉個例子：

```java
Observable.just("http://yongjhih.gitbooks.io/feed/content/RxJava.html")
    .flatMap(url -> downloadObs(url))
    .map(file -> file.size())
    .susbscirbe(size -> Systen.out.println(size));
```

```java
Observable<File> downloadObs(String url) { ... }
```

## 產生器與流量控制

原料無中生有。

印出 1 到 10:

Before:

```java
for (int i = 1; i <= 10; i++) {
    System.out.println(i);
}
```

After:

```java
Observable.range(1, 10).forEach(i -> System.out.println(i));
```

這邊第一次使用 `forEach()` ，僅差 `subscribe()` 會回傳 `Subscription` ，`Subscription` 是用來停止生產的。

秒針：

```java
// 計秒器
Subscription subscription = Observable.interval(1, TimeUnit.SECONDS).subscribe(l -> System.out.println(l));

Observable.just(sub).delay(10, TimeUnit.SECONDS).subscribe(s -> s.unsubscribe()); // 10 秒後停掉計秒器
```



## 片語 toBlocking().single()

這行會卡住，直到拿到一個為止。



## single(), take(1), first()

single() 與 take(1) 最大的不同是，single() 只能使用在單數的 Observable 上，否則會噴 exception 。

例如：


```java
List<Integer> integers = Observable.range(1, 2).toList().single(); // pass
```

```java
Integer i = Observable.range(1, 2).toBlocking().take(1); // pass
```

```java
Observable<Integer> singleInteger = Observable.range(1, 1).single(); // pass
```

```java
Observable<Integer> singleInteger = Observable.range(1, 2).single(); // exception
```



## 名詞解釋

Observable 觀測所
Subscriber 訂閱者
Observer 觀察員

描述這是怎樣的工作，以及中間需要的製程，希望產生出什麼產品。

Observable<T> 一份工作 task 一個未來 future , T 產品. 相當於 AsyncTask<INPUT, PROGRESS, T>, Future<T>

Subscriber/Observer<T> onEvent, Listener. 提貨券.

subscribe 下訂。

Subscription 訂單, subscribe 下訂之後產生出來的訂單，這個訂單可以用來取消訂單來中止生產。

Eager vs. Lazy

## 動手玩

```bash
git clone https://github.com/yongjhih/RxJava-retrofit-github-sample
cd RxJava-retrofit-github-sample
./gradlew execute
```

修改 src/main/java/com/github/yongjhih/Main.java 內容就可以自己玩了。



## 組合資料 zip()

```java
Observable<User> getUser(Activity activity) {
    Observable.zip(getFbUser(activity), getParseUser(activity),
        (fbUser, parseUser) -> getUser(fbUser, parseUser));
}
```

## Subject

相當於 Event Bus，多方進貨，多方出貨。開放式，俗稱 Hot ，對應之前封閉式的 Observable 為 Cold 。

多方進貨，相當於把 Observable.OnSubscriber ，讓其他人可以塞資料。

Before:

```java
Observable.just("hello, world!").subscribe(System.out::println);
```

After:

```java
Subject<String, String> subject = PublishSubject.create();
subject.asObservable().subscribe(System.out::println);

subject.onNext("hello, world!");
```

找一個實際點的例子：

```java
Subject<View> mLikeCountSubject = PublishSubject.create();

@OnClick(R.id.like_button)
public void onLikeClick(View view) {
    mLikeCountSubject.onNext(view);
}

@Override
public void onResume() {
    super.onResume();
    
    mLikeCountSubject.asObservable().map(view -> 1)
        .scan((count, i) -> count + i)
        .subscribe(count -> likeText.setText(count.toString()));
}
```

http://reactivex.io/documentation/subject.html



## Exception 處理

如果發生 exception 重試，會重新進貨，從頭再跑一輪。

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
}).retryWhen(attempt -> {
    return attempt.zipWith(Observable.range(1, 3), (n, i) -> i).flatMap(i -> {
        System.out.println("delay retry by " + i + " second(s)");
        return Observable.timer(i, TimeUnit.SECONDS);
    });
}).toBlocking().subscribe(System.out::println);
```

忽略 exception：

```java
.onErrorResumeNext(e -> Observable.empty()); // onCompleted()
```

或傳個替代資料：

```java
.flatMap(parseUser -> {
    return Observable.zip(createChannel(parseUser), createFriendRole(parseUser), (user, user2) -> user)
        .onErrorResumeNext(e -> Observable.just(parseUser));
})
```

## flatMap()

p.s. 似乎很多讀者對於 `flatMap()` 有理解的困難，所以這裡特別解釋一下 `flatMap`

...

## concatMap() 與 flatMap()

```java
Observable<File> downloadObs(String url) { ... }
```

```java
Observable.just("https://raw.githubusercontent.com/yongjhih/android-gitbook/master/README.md", https://raw.githubusercontent.com/yongjhih/android-gitbook/master/RxJava.md)
    .flatMap(s -> downloadObs(s).subscribeOn(Schedulers.io()))
    .subscribe(f -> System.out.println("downloaded: " + f));
```

這裡會同時開兩個 threads 在下載。避免這種情況，希望一個一個跑：

```java
Observable.just("https://raw.githubusercontent.com/yongjhih/android-gitbook/master/README.md", https://raw.githubusercontent.com/yongjhih/android-gitbook/master/RxJava.md)
    .concatMap(s -> downloadObs(s).subscribeOn(Schedulers.io()))
    .subscribe(f -> System.out.println("downloaded: " + f));
```


## 排序 toSortedList()



## 分組 groupBy()



## 分段 window()



## 快取 cache()


## 取樣 debounce()/throttleLast()

```java
  .throttleLast(100, TimeUnit.MILLISECONDS)
  .debounce(200, TimeUnit.MILLISECONDS)
  .onBackpressureLatest()
```

## 利用 compose(Transformer) 重用常用的流程組合

Compose/Transformer 0.20 版本(2014.8)

先丟背景等等回來前景:

Before:

```java
.observeOn(AndroidScheduler.mainThread())
.subscribeOn(Schedulers.io))
```

After:

```java
.compose(mainAsync())
```

```java
<T> Transformer<T, T> mainAsync() {  
    return obs -> obs.subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread());
}
```

計數器:

Before:

```java
.map(o -> 1)
.scan((count, i) -> count + i)
```

After:

```java
.compose(amount())
```

```java
<T> Transformer<T, T> amount() {  
    return obs -> obs.map(o -> 1)
        .scan((count, i) -> count + i);
}
```

```java
public class Transformers {
    @SuppressWarnings("unchecked")
    private static final Transformer MAIN_ASYNC = obs -> obs.subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread());
        
    @SuppressWarnings("unchecked")
    public static final <T> Transformer<T, T> mainAsync() {
        return MAIN_ASYNC;
    }
    
    @SuppressWarnings("unchecked")
    private static final Transformer AMOUNT = obs -> obs.map(o -> 1)
        .scan((count, i) -> count + i);
        
    @SuppressWarnings("unchecked")
    public static final <T> Transformer<T, T> amount() {
        return AMOUNT;
    }
}
```



## 附錄：Android View Observable 範例

寫一個讚計數器:

```java
ViewObservable.clicks(findViewById(R.id.like_button))
    .map(clickEvent -> 1)
    .scan((count, i) -> count + i)
    .subscribe(likes -> {
        TextView likesView = (TextView) findViewById(R.id.likes_view);
        textView.setText(likes.toString());
    });
```



## 透過 Observable 實現 Java8 Optional

在沒有 Optional 的狀況下：

Before:

```java
String displayName(ParseUser parseUser) {
    String displayName = parseUser.getString("displayName");
    if (displayName == null) {
        displayName = "Unnamed";
    }
    return displayName;
}
```

After:

```java
String displayName(ParseUser parseUser) {
    return Optional.ofNullable(parseUser.getString("displayName")).orElse("Unnamed");
}
```

```java
// Optional 實現:

public class Optional<T> {
    Observable<T> obs;

    public Optional(Observable<T> obs) {
        this.obs = obs;
    }

    public static <T> Optional<T> of(T value) {
        if (value == null) {
            throw new NullPointerException();
        } else {
            return new Optional<T>(Observable.just(value));
        }
    }

    public static <T> Optional<T> ofNullable(T value) {
        if (value == null) {
            return new Optional<T>(Observable.empty());
        } else {
            return new Optional<T>(Observable.just(value));
        }
    }

    public T get() {
        return obs.toBlocking().single();
    }

    public T orElse(T defaultValue) {
        return obs.defaultIfEmpty(defaultValue).toBlocking().single();
    }
}
```

Oracle 官方比較複雜的 Optional 範例，當各類別傳遞已作成 Optional 界面：

Before:

```java
String getVersion(Computer computer) {
  String version = "UNKNOWN";
  if (computer != null) {
    Soundcard soundcard = computer.getSoundcard();
    if (soundcard != null) {
      USB usb = soundcard.getUsb();
      if (usb != null) {
        version = usb.getVersion();
      }
    }
  }
  return version;
}
```

After:

```java
// Optional 版本
String getVersion(Computer computer) {
  return computer.flatMap(Computer::soundcard) // soundcard()
    .flatMap(Soundcard::usb) // usb()
    .map(Usb::getVersion)
    .orElse("UNKNOWN");
}

public class Computer {
  private Soundcard soundcard;  
  public Optional<Soundcard> soundcard() { return Optional.ofNullable(soundcard); }
  public Soundcard getSoundcard() { return soundcard; }
}

public class Soundcard {
  private Usb usb;
  public Optional<Usb> usb() { return Optional.ofNullable(usb); }
  public Usb getUsb() { return usb; }
}

public class Usb {
  String version;
  public String getVersion() { return version; }
}
```

*P.S. Groovy 語言: `String version = computer?.getSoundcard()?.getUSB()?.getVersion();`*

如果真的要用 RxJava Optional 請用專門的專案  https://github.com/eccyan/RxJava-Optional

這邊僅是教學需要。 (https://gist.github.com/yongjhih/25017ac41efb4634c2ab)

## Threading, Scheduler 執行序控制

```
Observable.just("http://yongjhih.gitbooks.io/feed/content/RxJava.html").map(url -> download(url))
.subscribeOn(Schedulers.io)) // 在 io Thread() 跑 (60 thread pool, 60s timeout)
.observeOn(AndroidScheduler.mainThread()) // 在 mainThread 回來印
.subscirbe(System.out::println);
```

## 改善 Cache 流程

```java
Observable.merge(ParseObservable.getPostsFromLocalDatabase(), ParseObservable.getPosts()).distinct(o -> o.getObjectId())
```

或者用 `amb()`, `first()` 等方式作


## Notification

## Backpressure

## Testing

### TestSubscriber

### Assertions

Using assertj-rx:

```java
assertThat(observable.toBlocking())
    .completes()
    .emitsSingleValue("hello");
```

## Hooks

* RxJavaObservableExecutionHook
* RxJavaSchedulersHook
* RxJavaErrorHandler

## 後記

這裡鮮少討論一般常見的理論：無論是 FRP, monad, push/pull, cold/hot 。主要因為先以範例，也就是實際看得到的改善是什麼來作介紹。

http://yarikx.github.io/NotRxJava/ 也是從這種方式帶領你看發展進程，希望大家會喜歡

## See Also

* 章節：[輕量資料流處理](data_stream.md)

文獻:

* https://speakerdeck.com/jakewharton/2014-1?slide=13
* http://slides.com/yaroslavheriatovych/frponandroid
* http://en.wikipedia.org/wiki/Null_coalescing_operator
* http://www.oracle.com/technetwork/articles/java/java8-optional-2175753.html
* http://java.dzone.com/articles/java-8-elvis-operator
* http://blog.danlew.net/2015/06/22/loading-data-from-multiple-sources-with-rxjava/
* https://blog.growth.supply/party-tricks-with-rxjava-rxandroid-retrolambda-1b06ed7cd29c

一些 Rx 開源：

* https://github.com/yongjhih/RxParse
* https://github.com/ogaclejapan/RxBinding
* https://github.com/eccyan/RxJava-Optional
* https://github.com/square/sqlbrite
* https://github.com/pushtorefresh/storio
* https://github.com/mcharmas/Android-ReactiveLocation
* https://github.com/pardom/Ollie
* https://github.com/frankiesardo/ReactiveContent
* https://github.com/evant/rxloader
* https://github.com/vyshane/rex-weather
* https://github.com/ReactiveX/RxNetty
* https://github.com/vbauer/caesar
* https://gist.github.com/ivacf/874dcb476bfc97f4d555
* https://github.com/ribot/assertj-rx
 
* 小抄：https://gist.github.com/yongjhih/bbe3b528873c7eb671c6

[^1] `rx.android.observables.AndroidObservable` has changed to `rx.android.app.AppObservable` (Version 0.24 – January 3rd 2015)
