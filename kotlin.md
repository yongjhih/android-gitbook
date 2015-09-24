# Kotlin

卡特琳

特點：

* null-safety / Optional 最佳解決方案。 `?:`
* 語法簡潔 `for ((key, value) in map)`
* AutoValue?
* lambdas `setOnClickListener({ finish() })`
* Jake Wharton 加持 (誤

Kotlin 開始知名的時候，大概可以追溯到 2014 年中旬登上 Android 開發週報的：http://blog.gouline.net/2014/08/31/kotlin-the-swift-of-android/ ，剛開始看到是覺得確實很敏捷，但是對於成熟度抱著遲疑得態度。

在這之後，筆者是在 2015 年一月份 Jake Wharton 在 G+ 發表了一篇貼文： https://plus.google.com/+JakeWharton/posts/WSCoqkJ5MBj
經過這之後，確實很多人跟筆者一樣，較為積極的看待這個語言。

除了這些特性之外，對於 android 來說，滿大的優勢在於 symbol size 以及 code size 相繼於其他語言，十分羽量。(kotlin: 6k~, scala: 50k~)

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

## See Also

* https://github.com/ReactiveX/RxKotlin

* [Using Project Kotlin for Android](https://docs.google.com/document/d/1ReS3ep-hjxWA8kZi0YqDbEhCqTt29hG8P44aA9W0DM8) @JackWharton
* https://github.com/JetBrains/ank
* http://kotlinlang.org/docs/reference/
