# Assert 斷言 - assertj-android (fest-android), truth

## Square AssertJ for Android

* *註: 2011 釋出*

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

## Google truth

* *註: 筆者是在 2015/2 留意到它*
* *註:  2014/12 由 google testing blog 消息釋出*

仿效 square/assertj-android (square/fest-android)。

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

## See Also

* https://github.com/google/truth
* https://github.com/square/assertj-android
* http://joel-costigliola.github.io/assertj
* https://github.com/google/truth/issues/43
* http://googletesting.blogspot.tw/2014/12/testing-on-toilet-truth-fluent.html
