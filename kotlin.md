# Kotlin

卡特琳

特點：

* null-safety / Optional 最佳解決方案。 `?:`
* 語法簡潔 `for ((key, value) in map)`
* AutoValue?
* lambdas `setOnClickListener({ finish() })`
* Jake Wharton 加持 (誤

Kotlin 開始知名的時候，大概可以追溯到 2014 年中旬登上 Android 開發週報的：http://blog.gouline.net/2014/08/31/kotlin-the-swift-of-android/ ，剛開始看到是覺得確實很敏捷，但是對於成熟度抱著遲疑得態度。

在這之後，筆者是在今年一月份 Jake Wharton 在 G+ 發表了一篇貼文才認真看待這個語言： https://plus.google.com/+JakeWharton/posts/WSCoqkJ5MBj
經過這之後確實很多人跟筆者一樣，較為積極的看待這個語言。


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

## See Also

* https://github.com/ReactiveX/RxKotlin

* [Using Project Kotlin for Android](https://docs.google.com/document/d/1ReS3ep-hjxWA8kZi0YqDbEhCqTt29hG8P44aA9W0DM8) @JackWharton
* https://github.com/JetBrains/anko