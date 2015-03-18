# Dagger2

DI 工具

Before:

```java
OkHttpClient client = new OkHttpClient();
TwitterApi api = new TwitterApi(client);

String user = "Andrew Chen";

Tweeter tweeter = new Tweeter(api, user);
tweeter.tweet("Hello, world!");

Timeline timeline = new Timeline(api, user);
for (Tweet tweet : timeline.get()) {
    System.out.println(tweet);
}

String user2 = "Andrew Chen2";

Tweeter tweeter2 = new Tweeter(api, user2);
tweeter.tweet("Hello, world!");

Timeline timeline2 = new Timeline(api, user2);
for (Tweet tweet : timeline.get()) {
    System.out.println(tweet);
}
```

為了沿用 api 與 client 所以須從外部提供。

After:

```java
TwitterComponent component = Dagger_TwitterComponent.builder()
    .twitterModule(new TwitterModule("Andrew Chen"))
    .build();
    
Tweeter tweeter = component.tweeter();
Timeliner timeline = component.timeline();

for (Tweet tweet : timeline.get()) {
    System.out.println(tweet);
}

TwitterComponent component2 = Dagger_TwitterComponent.builder()
    .twitterModule(new TwitterModule("Andrew Chen2"))
    .build();
    
Tweeter tweeter2 = component.tweeter();
Timeliner timeline2 = component.timeline();

for (Tweet tweet : timeline.get()) {
    System.out.println(tweet);
}
```

連帶修改:

```java
@Module
public class NetworkModule {
    @Provides @Singleton
    OkHttpClient provideOkHttpClient() {
        return new OkHttpClient();
    }
}
```

```java
@Singleton
public class TwitterApi {
    private final OkHttpClient client;
    
    @Inject
    public TwitterApi(OkHttpClient client) {
        this.client = client;
    }
}
```

```java
public class TwitterModule {
    private final String user;

    public TwitterModule(String user) {
        this.user = user;
    }
    
    @Provides @Singleton
    Tweeter provideTweeter(TwitterApi api) {
        return new Tweeter(api, user);
    }
    
    @Provides @Singleton
    Timeline provideTimeline(TweeterApi api) {
        return new Timeliner(api, user);
    }
}
```

```java
@Singleton
@Component(modules = {
    NetworkModule.class,
    TwitterModule.class
})
public interface TwitterComponent {
    Tweeter tweeter();
    Timeline timeline();
}
```

## See Also

* https://speakerdeck.com/jakewharton/dependency-injection-with-dagger-2-devoxx-2014