# Kotlin

卡特琳

特點：

* null-safety / Optional 最佳解決方案。 `?:`
* 語法簡潔 `for ((key, value) in map)`
* AutoValue?
* lambdas `setOnClickListener({ finish() })`
* Jake Wharton 加持 (誤

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