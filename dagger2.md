# Dagger2

DI 工具

## 什麼是 DI

DI, Dependency Injection (相依性注入) ，Anti-DIY, Auto-DIY 自動組裝，行前準備/著裝/組件/要件

![CoffeeMaker](http://upload.wikimedia.org/wikipedia/commons/2/23/KBG741S-AO.jpg)

沖泡出一杯風味十足的咖啡之前，你需要一台濾泡式咖啡機：

在沒有的 DI 概念下：

Before:

```java
new CoffeeMaker().brew(); // 沖泡
new CoffeeMaker().brew(); // 沖泡
```

```java
class CoffeeMaker { // 咖啡機
    private final Heater heater; // 加熱器
    private final Pump pump; // 幫浦
    
    CoffeeMaker() {
        this.heater = new ElectricHeater(); // 電熱器
        this.pump = new Thermosiphon(heater); // 虹吸幫浦
    }
    
    public void brew() { /* ... */ }
}
```

缺點是每杯咖啡都要生產整台咖啡機、電熱器與幫浦，太浪費了，一點都不環保。

手動 DI:

After:

```java
Heater heater = new ElectricHeater();
Pump pump = new Thermosiphon(heater);
CoffeeMaker maker = new CoffeeMaker(heater, pump);
maker.brew();
maker.brew();
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
}
```

這樣至少電熱器與幫浦都重覆使用了。

但是這樣我們要泡咖啡前，都要先準備電熱器與幫浦然後組裝成咖啡機後才開始泡咖啡。

我們希望寫好咖啡機所需要的組件，請一個組裝工人幫我們組
，以後只要說我現在要用咖啡機就馬上組裝好了，我們都不用自己 DIY。

利用 Dagger2 自動 DI 來組裝那些要件，只要描述好要件相依後，就可以一直泡一直泡：

```java
Coffee coffee = Dagger_CoffeeApp_Coffee.create();
coffee.maker().brew(); // 一直泡
coffee.maker().brew(); // 一直泡
```

咖啡機要加熱加壓沖泡，相依要件關係圖：

```
CoffeeMaker -> DripCoffeeModule -----------------------> Heater
                  \-> PumpModule -> ThermosiphonPump -/
```

濾泡式咖啡機：

```java
@Singleton // 共用咖啡機
@Component(modules = DripCoffeeModule.class) // 濾泡裝置(安裝著高壓熱水沖泡裝置提供幫浦與加熱器)
public interface Coffee {
    CoffeeMaker maker();
}
```

咖啡機需要把水加熱、加壓後沖泡：

```java
class CoffeeMaker {
  private final Lazy<Heater> heater; // 加熱器
  private final Pump pump;

  // 準備幫浦與加熱器
  @Inject CoffeeMaker(Lazy<Heater> heater, Pump pump) {
    this.heater = heater;
    this.pump = pump;
  }

  // 沖泡
  public void brew() {
    heater.get().on(); // 加熱
    pump.pump(); // 加壓
    System.out.println(" [_]P coffee! [_]P "); // 熱騰騰的咖啡出爐囉！
    heater.get().off(); // 隨手關加熱器
  }
}
```

高壓熱水裝置需要加壓器具：

```java
@Module(includes = PumpModule.class) // 一同準備加壓器具
class DripCoffeeModule { // 高壓熱水裝置
  @Provides @Singleton Heater provideHeater() { // 提供共用的加熱器具
    return new ElectricHeater(); // 電熱器具
  }
}
```

```java
@Module(complete = false, library = true) // complete = false 需要借用加熱器具
class PumpModule { // 幫浦加壓器具
  @Provides Pump providePump(Thermosiphon pump) { // 利用熱虹吸管來提供幫浦器具
    return pump;
  }
}
```

```java
class Thermosiphon implements Pump {
  private final Heater heater;

  @Inject
  Thermosiphon(Heater heater) { // 索取加熱器來加壓
    this.heater = heater;
  }

  @Override public void pump() {
    if (heater.isHot()) {
      System.out.println("=> => pumping => =>");
    }
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
* http://google.github.io/dagger/