# Assert 斷言 - assertj, truth

## assertj-android

* *註: 筆者 2013 下旬發現*
* *註: 2013 仿 AssertJ 以 fest-android 名稱釋出，2014 中旬正式加入 AssertJ 家族，更名為 assertj-android*

針對 Android 的類別作語法撿結語錯誤訊息的強化。

語法的簡潔：

Before(JUnit):

```java
assertEquals(View.GONE, view.getVisibility());
```

After(AssertJ):

```java
assertThat(view).isGone();
```

錯誤訊息的強化：

Before(JUnit):

```
Expected:<[8]> but was:<[4]>
```

After:

```
Expected visibility <gone> but was <invisible>
```

## truth

* *註: 筆者是在 2015/2 留意到它*
* *註:  2014/12 由 google testing blog 消息釋出*
* 仿 AssertJ

在沒有特定的類別下，提供一個置換錯誤訊息的能力。

Before(JUnit):

```java
boolean buttonEnabled = false;
assertTrue(buttonEnabled);
```

After(truth):

```java
ASSERT.that(buttonEnabled).named("buttonEnabled").isTrue();
```

錯誤訊息的強化：

Before(JUnit):

```
<false> was expected be true, but was false
```

After(truth):

```
"buttonEnabled" was expected to be true, but was false
```

## assertj-rx

```java
assertThat(observable.toBlocking()).completes();
```

```java
assertThat(observable.toBlocking())
    .completes()
    .emitsSingleValue("hello");
```

```java
assertThat(observable.toBlocking()).fails();
```


## See Also

* https://github.com/square/assertj-android
* https://github.com/google/truth
* http://joel-costigliola.github.io/assertj
* https://github.com/google/truth/issues/43
* http://googletesting.blogspot.tw/2014/12/testing-on-toilet-truth-fluent.html
* https://github.com/ribot/assertj-rx
