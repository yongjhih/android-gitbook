# Assert 斷言 - assertj, truth

斷言：

* junit
* hamcrest ：結合 junit 內建的斷言，提供較好的斷言語義
* truth ：仿造 assertj 斷言訊息
* assertj 提供更好懂的斷言訊息

其中推薦 assertj 斷言家族

測試框架：

* junit
* testNg ：取代 junit

一般內建的 junit 即可

## assertj-android

* *註: 筆者 2013 下旬發現*
* *註: 2013 上旬仿 AssertJ 以 fest-android 名稱釋出，2014 中旬正式加入 AssertJ 家族，更名為 assertj-android*
* *註: assertj 2011 釋出*

針對 Android 強化斷言錯誤訊息以及語法簡潔。

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

## Hamcrest

*p.s. 2012 已無動靜*

口語化，但無法接龍

```java
List<String> list = Arrays.asList("Andrew", "Chen");
assertThat(list, is(not(empty())));
assertThat(list, is(contains("Andrew", "Chen")));
```

## 小道消息

Google 的 alexruiz 本來在 2011 年有一個 FEST ([fest-assert](https://github.com/alexruiz/fest-assert-2.x)) 主要是可接龍的斷言，2013 Q2 Square 就以此基礎開發了 fest-assert for android 叫做 fest-android
，但是後來 2014 Q2 改用 AssertJ Core 為基礎，改名 assertj-android 成為 AssertJ 家族成員。

fest-assert 的第二作者 joel-costigliola 2011 就分家 AssertJ 了，而 fest-assert 看起來 2013 Q2 都沒有什麼動靜了，直到 2014 Q4 Google 另外釋出 [truth](https://github.com/google/truth) ，也是可接龍的斷言，不過似乎也沒有太多明顯優勢。

以生態來說，仍以 AssertJ 為大宗

## See Also

* http://developer.android.com/training/testing/unit-testing/local-unit-tests.html
* https://github.com/square/assertj-android
* https://github.com/google/truth
* http://joel-costigliola.github.io/assertj
* https://github.com/google/truth/issues/43
* http://googletesting.blogspot.tw/2014/12/testing-on-toilet-truth-fluent.html
* https://github.com/ribot/assertj-rx
* https://github.com/ubiratansoares/rxassertions
* https://github.com/hamcrest/JavaHamcrest
* http://google.github.io/truth/comparison
