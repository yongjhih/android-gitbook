# Kotlin

卡特琳

特點：

* null-safety / Optional 最佳解決方案。 `?:`
* 語法簡潔 `for ((key, value) in map)`
* AutoValue?
* lambdas `setOnClickListener({ finish() })`
* Jake Wharton 加持 (誤

Kotlin 開始知名的時候，大概可以追溯到 2014 年中旬登上 Android 開發週報的：http://blog.gouline.net/2014/08/31/kotlin-the-swift-of-android/ ，剛開始看到是覺得確實很敏捷，但是對於成熟度抱著遲疑得態度。

在這之後，筆者是在 2015 年一月份 Jake Wharton 在 G+ 發表了一篇[貼文](https://plus.google.com/+JakeWharton/posts/WSCoqkJ5MBj)
經過這之後，確實很多人跟筆者一樣，較為積極的看待這個語言。

除了這些特性之外，對於 android 來說，滿大的優勢在於 symbol size 以及 code size 相較於其他語言，十分羽量。(kotlin: 6k~, scala: 50k~)

## POJO

Before:

```java
@AutoValue
public abstract class Money {
  public abstract String currency();
  public abstract int amount();

  public static Money of(String currency, int amount) {
    return new AutoValue_Money(currency, amount);
  }
}
```

After:

```kotlin
data class Money(val currency: String, val amount: Int)
```

## Lambda

Before:

```java
thing.setListener(new Listener() {
  @Override public void onThing() {
    System.out.println("Thing!");
  }
});
```

After:

```kotlin
t.setListener(Listener { println("Thing!") })
```

## Multi-assignment for loop

Before:

```java
for (Map.Entry<String, String> entry : map.entrySet()) {
  System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

After:

```kotlin
for ((key, value) in map) {
  println(key + ": " + value)
}
```

## 簡便 getter 與 setter

```kotlin
public var context: Context? = null
  get
  set (value) {
    $context = value
  }
```

# Null Safety

Nullable (類 @Nullable) :

```kotlin
var a: String? = "bar"
// `a = null` is ok
// `a.length()` throws exception
// `a?.length()` is ok
// `a!!.length()` // throws NPE if a is null
```

```kotlin
var a: String = "bar" // `a = null` throws NPE
```

Elvis operator, before:

```kotlin
val l = if (a != null) a.length() else -1
```

After:

```kotlin
val l = a?.length() ?: -1
```

## 導入方法

* 可利用 Android Studio kotlin plugin 轉換程式碼 (轉完不一定可動，大多稍微改一下就好了)
* buildscript.dependencies: `classpath "org.jetbrains.kotlin:kotlin-gradle-plugin`
* `apply plugin: 'kotlin-android'`
* dependencies: `compile 'org.jetbrains.kotlin:kotlin-stdlib:0.12.200'`

## FAQ

* 如果多方繼承(`class`/`interface`) 時，`super.XXX()` 就會不清楚你要呼叫哪位 parent ，所以改成 `super<>.XXX` 即可。

## RxKotlin

```java
observable<String> { subscriber ->
        subscriber.onNext("H")
        subscriber.onNext("e")
        subscriber.onNext("l")
        subscriber.onNext("")
        subscriber.onNext("l")
        subscriber.onNext("o")
        subscriber.onCompleted()
    }.filter { it.isNotEmpty() }.
    fold (StringBuilder()) { sb, e -> sb.append(e) }.
    map { it.toString() }.
    subscribe { result ->
      a.received(result)
    }

    verify(a, times(1)).received("Hello")
```

## Anko

捨棄 xml 直接用 kotlin 語言來配置 UI。 立意良好。但是來誰作一下視覺化預覽阿？！ 

## 對照表

(origin from [Using Project Kotlin for Android](https://docs.google.com/document/d/1ReS3ep-hjxWA8kZi0YqDbEhCqTt29hG8P44aA9W0DM8))

| Library                   | Jar Size | Dex Size | Method Count | Field Count |
|---------------------------|----------|----------|--------------|-------------|
| rxjava-1.0.4              | 678 KB   | 513 KB   | 3557         | 1668        |
| support-v4-21.0.3         | 745 KB   | 688 KB   | 6721         | 1886        |
| play-services-base-6.5.87 | 773 KB   | 994 KB   | 5212         | 2252        |
| okio-1.2.0                | 54 KB    | 55 KB    | 508          | 76          |
| okhttp-2.2.0              | 304 KB   | 279 KB   | 1957         | 882         |
| retrofit-1.9.0            | 119 KB   | 93 KB    | 766          | 228         |
| picasso-2.4.0             | 112 KB   | 97 KB    | 805          | 342         |
| dagger-1.2.2              | 59 KB    | 54 KB    | 400          | 119         |
| butterknife-6.0.0         | 48 KB    | 50 KB    | 307          | 73          |
| wire-runtime-1.6.1        | 71 KB    | 71 KB    | 471          | 147         |
| gson-2.3.1                | 206 KB   | 170 KB   | 1231         | 390         |
| Total                     | 2963 KB  | 2894 KB  | 21935        | 8063        |

| Library              | Jar Size | Dex Size | Method Count | Field Count |
|----------------------|----------|----------|--------------|-------------|
| scala-library-2.11.5 | 5.3 MB   | 4.9 MB   | 50801        | 5820        |
| groovy-2.4.0-grooid  | 4.5 MB   | 4.5 MB   | 29636        | 8069        |
| guava-18.0           | 2.2 MB   | 1.8 MB   | 14833        | 3343        |

## See Also

* https://github.com/ReactiveX/RxKotlin

* [Using Project Kotlin for Android](https://docs.google.com/document/d/1ReS3ep-hjxWA8kZi0YqDbEhCqTt29hG8P44aA9W0DM8) @JackWharton
* https://github.com/JetBrains/anko
* http://kotlinlang.org/docs/reference/
