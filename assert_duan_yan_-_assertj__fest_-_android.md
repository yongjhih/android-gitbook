# Assert 斷言 - assertj-android (fest-android), truth

## Square AssertJ for Android

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

*註: 筆者是在 2015/2 留意到它*

仿效 square/assertj-android (square/fest-android)

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