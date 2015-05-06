# Assert 斷言 - assertj-android (fest-android)

## AssertJ

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

Before:

```
Expected:<[8]> but was:<[4]>
```

After:

```
Expected visibility <gone> but was <invisible>
```
