# Dagger2

DI 工具

## 什麼是 DI

DI, Dependency Injection (相依性注入) ，筆者個人口語化稱之為「配件」

在沒有的 DI 概念下：

Before:

```java
new CoffeeMaker().brew(); // 沖泡
```

```java
class CoffeeMaker { // 咖啡機
    private final Heater heater; // 加熱器
    private final Pump pump; // 幫浦
    
    CoffeeMaker() {
        this.heater = new ElectricHeater(); // 電熱器
        this.pump = new Thermosiphon(heater); // 熱虹吸管
    }
    
    public void brew() { /* ... */ }
}
```

手動 DI:

After:

```java
Heater heater = new ElectricHeater();
Pump pump = new Thermosiphon(heater);
new CoffeeMaker(heater, pump).brew();
```

```java
class CoffeeMaker {
    private final Heater heater;
    private final Pump pump;
    
    CoffeeMaker(Heater heater, Pump pump) {
        this.heater = heater;
        this.pump = pump;
    }
    
    public void brew() { /* ... */ }
```

自動 DI:

```java
public class CoffeeMaker {
    protected Heater heater;
    protected Pump pump;
    
    @Inject
    public CoffeeMaker(Heater heater, Pump pump) {
        this.heater = heater;
        this.pump = pump;
    }
    
    public void brew() { // 沖泡
        heater.heat(); // 加熱
        pump.pump(); // 加壓
        System.out.println("CoffeeMaker", " [_]P coffee! [_]P ");
    }
}
```

## 動手玩

```bash
git clone https://github.com/yongjhih/dagger2-sample
cd dagger2-sample
./gradlew execute
```

## 範例

為了沿用 api 與 client 所以須從外部提供。

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

隱藏相依前置作業，透過 builder 來獲得末端物件。

After:

```java
ApiComponent apiComponent = Dagger_ApiComponent.create();

TwitterComponent twitterComponent = Dagger_TwitterComponent.builder()
    .apiComponent(apiComponent)
    .twitterModule(new TwitterModule("Andrew Chen"))
    .build();
    
Tweeter tweeter = twitterComponent.tweeter();
Timeliner timeline = twitterComponent.timeline();

for (Tweet tweet : timeline.get()) {
    System.out.println(tweet);
}

TwitterComponent component2 = Dagger_TwitterComponent.builder()
    .apiComponent(apiComponent)
    .twitterModule(new TwitterModule("Andrew Chen2"))
    .build();
    
Tweeter tweeter2 = component2.tweeter();
Timeliner timeline2 = component2.timeline();

for (Tweet tweet : timeline.get()) {
    System.out.println(tweet);
}
```

連帶修改:

```java
@Module
public class NetworkModule {
    @Provides @Singleton // @Singleto 註明沿用同一個, @Provides 註明可提供 OkHttpClient
    OkHttpClient provideOkHttpClient() {
        return new OkHttpClient();
    }
}
```

```java
@Singleton
public class TwitterApi {
    private final OkHttpClient client;
    
    @Inject // 註明由相依提供 OkHttpClient
    public TwitterApi(OkHttpClient client) {
        this.client = client;
    }
}
```

```java
@Singleton
@Component(modules = NetworkModule.class) // 由哪些 modules 組成
public interface ApiComponent {
    TwitterApi api();
}
```

```java
@Module
public class TwitterModule {
    private final String user;

    public TwitterModule(String user) {
        this.user = user;
    }
    
    @Provides
    Tweeter provideTweeter(TwitterApi api) {
        return new Tweeter(api, user);
    }
    
    @Provides
    Timeline provideTimeline(TweeterApi api) {
        return new Timeline(api, user);
    }
}
```

```java
@Component(
    dependencies = ApiComponent.class,
    modules = TwitterModule.class
)
public interface TwitterComponent {
    Tweeter tweeter();
    Timeline timeline();
}
```

## See Also

* https://speakerdeck.com/jakewharton/dependency-injection-with-dagger-2-devoxx-2014