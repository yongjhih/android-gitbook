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

UI 測試框架：

* Robotium
* uiautomator
* espresso
* Appium
* Calabash

一般內建的 espresso 即可

跨 iOS 則使用 Appium 或 Calabash

## assertj-android

* *註: 筆者 2013 下旬發現*
* *註: 2013 上旬仿 AssertJ 以 fest-android 名稱釋出，2014 中旬正式加入 AssertJ 家族，更名為 assertj-android*
* *註: assertj 2011 釋出*

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

## Hamcrest

## See Also

* http://developer.android.com/training/testing/unit-testing/local-unit-tests.html
* https://github.com/square/assertj-android
* https://github.com/google/truth
* http://joel-costigliola.github.io/assertj
* https://github.com/google/truth/issues/43
* http://googletesting.blogspot.tw/2014/12/testing-on-toilet-truth-fluent.html
* https://github.com/ribot/assertj-rx
* https://github.com/hamcrest/JavaHamcrest
