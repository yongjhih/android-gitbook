# Kotlin

卡特琳

特點：

* null-safety / Optional 最佳解決方案。 `?:`
* 語法簡潔 `for ((key, value) in map)`
* AutoValue?
* lambdas `setOnClickListener({ finish() })`
* Jake Wharton 加持 (誤

對於 android 來說，Kotlin 開始知名的時候，大概可以追溯到 2014 年中旬登上 Android 開發週報的：http://blog.gouline.net/2014/08/31/kotlin-the-swift-of-android/ ，剛開始看到是覺得確實很敏捷，但是對於成熟度抱著遲疑得態度。

在這之後，筆者是在 2015 年一月份 Jake Wharton 在 G+ 發表了一篇[貼文](https://plus.google.com/+JakeWharton/posts/WSCoqkJ5MBj)之後，確實很多人跟筆者一樣，較為積極的看待這個語言。

除了這些特性之外，對於 android 來說，滿大的優勢在於 symbol size 以及 code size 相較於其他語言，十分羽量。(kotlin: 6k~, scala: 50k~, groovy: 30k~)

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
NonNull (類 @NonNull):

```kotlin
var a: String = "bar"
// `a = null` throws NPE
// `a.length()` is ok
```

Elvis operator, before:

```kotlin
val l = if (a != null) a.length() else -1
```

After:

```kotlin
val l = a?.length() ?: -1
```

## Immutable: val

* var: variable
* val: immutable variable, final variable

## switch case: when

## range

```kotlin
for (i in 1..10) println(i)
```

## 集合運算子

### any(其中)

其中一個項目成立，就回傳真。

語法：

```kotlin
list.any(() -> Boolean)
```

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
// 任何一個數字能整除 2
assertTrue(list.any { it % 2 == 0 })
// 任何一個數字大於 10
assertFalse(list.any { it > 10 })
```

### all(都)

其中一個項目不成立，就回傳假。(所有的項目都成立，就回傳真。)

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
// 都小於 10
assertTrue(list.all { it < 10 })
// 都整除 2
assertFalse(list.all { it % 2 == 0 })
```

### count(共有幾個)

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
// 整除 2 的項目共有 3 個
assertEquals(3, list.count { it % 2 == 0 })
```

### reduce

不解釋

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
assertEquals(21, list.reduce { total, next -> total + next })
```

### reduceRight (倒著 reduce)

不解釋

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
assertEquals(21, list.reduceRight { total, next -> total + next })
```

### fold (類似有初始值的 reduce)

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
// 4 + 1+2+3+4+5+6 = 25
assertEquals(25, list.fold(4) { total, next -> total + next })
```

### foldRight (同 fold ，只是倒著)

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
// 4 + 6+5+4+3+2+1 = 25
assertEquals(25, list.foldRight(4) { total, next -> total + next })
```

### forEach

不解釋

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
list forEach { println(it) }
```

### forEachIndexed

不解釋

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
list forEachIndexed { index, value
       -> println("$index : $value") }
```

### max

不解釋

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
assertEquals(6, list.max())
```

### maxBy

在算最大之前，先運算一次。

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
// 所有數值負數後，最大的那個是 1
assertEquals(1, list.maxBy { -it })
```

### min

不解釋

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
assertEquals(1, list.min())
```
### minBy

不解釋

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
assertEquals(6, list.minBy { -it })
```

### none (沒有一個)

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
// 沒有一個整除 7
assertTrue(list.none { it % 7 == 0 })
```

### sumBy

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
// 2 餘數的總和
assertEquals(3, list.sumBy { it % 2 })
```

## 導入方法

* 可利用 Android Studio kotlin plugin 轉換程式碼 (轉完不一定可動，大多稍微改一下就好了)
* buildscript.dependencies: `classpath "org.jetbrains.kotlin:kotlin-gradle-plugin`
* `apply plugin: 'kotlin-android'`
* dependencies: `compile 'org.jetbrains.kotlin:kotlin-stdlib:0.12.200'`

## 動手玩

```
git clone git@github.com:yongjhih/gradle-init.kt.git
cd gradle-init.kt
./gradlew run
```

主程式： `src/main/kotlin/hello/HelloWorld.kt`


另一種玩法 kotlin-cli：

```sh
wget https://github.com/JetBrains/kotlin/releases/download/build-1.0.0-beta-1103/kotlin-compiler-1.0.0-beta-1103.zip
unzip kotlin-compiler-1.0.0-beta-1103.zip
```

HelloWorld.kts:

```kotlin
println("Hello world!")
```

執行：

```sh
./kotlinc/bin/kotlinc-jvm -script HelloWorld.kts
```

* 線上玩: http://try.kotlinlang.org/koans

## FAQ

* 如果多方繼承(`class`/`interface`) 時，`super.XXX()` 就會不清楚你要呼叫哪位 parent ，所以改成 `super<>.XXX` 即可。

## RxKotlin

```kotlin
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

## Anko - kotlin template engine

\#data-binding, \#reactjs, \#angularjs, \#jax

捨棄 xml 直接用 kotlin 語言來配置 UI。

```kotlin
verticalLayout {
    val name = editText()
    button("Say Hello") {
        onClick { toast("Hello, ${name.text}!") }
    }
}
```

* Android Studio 預覽插件：Anko Preview Plugin for idea

議題：

* 重用性 - 如何重用 layout?
* 獨立 layout
* 如何引用獨立 layout
* qualifier

## 擴充 Cursor

Before:

```java
String firstName = cursor.getString(cursor.getColumnIndexOrThrow("first_name"));
```

After:

```kotlin
val firstName = cursor.getString("first_name");
fun Cursor.getString(columnName: String): String {
  return getString(getColumnIndexOrThrow(columnName))
}
```

避免 `NullPointerException`:

Before:

```java
String firstName = null;
int firstNameColumnIndex = cursor.getColumnIndexOrThrow("first_name");
if (!cursor.isNull(firstNameColumnIndex)) {
  firstName = cursor.getString(firstNameColumnIndex);
}
firstNameView.setText(firstName != null ? firstName : "Andrew");
```

After:

```kotlin
val firstName = cursor.getStringOrNull("first_name")
firstNameView.setText(firstName ?: "Andrew")

fun Cursor.getStringOrNull(columnName: String): String? {
  val index = getColumnIndexOrThrow(columnName)
  return if (isNull(index)) null else getString(index)
}
```

## findViewById?

Before:

```java
@InjectView(R.id.first_name)
TextView firstNameTextView;

@Override
public void onCreate(...) {
  super.onCreate(...);
  setContentView(R.layout.activity_main);
  ButterKnife.inject(this);

  firstNameTextView.setText("Andrew");
}
```

After:

```kotlin
import kotlinx.android.synthetic.activity_main.first_name as firstNameTextView

firstNameTextView.setText("Andrew");
```

## 擴充資料庫 Transaction

Before:

```java
db.beginTransaction();
try {
  db.delete("users", "first_name = ?", new String[] { "Andrew" });
  db.setTransactionSuccessful();
} finally {
  db.endTransaction()
}
```

or

```java
Databases.from(db).inTransaction(db -> {
  db.delete("users", "first_name = ?", new String[] { "Andrew" });
});
```

After:

```kotlin
db.inTransation {
  delete("users", "first_name = ?", arrayOf("Andrew"))
}

// inflix, literal
inline fun SQLiteDatabase.inTransaction(func: SQLiteDatabase.() -> Unit) {
  beginTransaction()
  try {
    func()
    setTransactionSuccessful()
  } finally {
    endTransaction()
  }
}
```

## 擴充 SharedPreferences

Before:

```java
SharedPreferences.Editor editor = preferences.edit();

editor.putString("first_name", "Andrew");
editor.putString("last_name", "Chen");
editor.remove("age");

editor.apply();
```

or

```java
SharedPreferencesUtils.from(preferences).apply(editor -> {
  editor.putString("first_name", "Andrew");
  editor.putString("last_name", "Chen");
  editor.remove("age");
});
```

After:

```kotlin
preferences.edit {
  putString("first_name", "Andrew")
  putString("last_name", "Chen")
  remove("age")
}

inline fun SharedPreferences.edit(func: SharedPreferences.Editor.() -> Unit) {
  val editor = edit()
  editor.func()
  editor.apply()
}
```

## 擴充 Notification Builder

Before:

```java
Notification notification = new NotificationCompat.Builder(context)
  .setContentTitle("Hello")
  .setSubText("World")
  .build();
```

After:

```kotlin
val notification = Notification.build(context) {
  setContentTitle("Hello")
  setSubText("World")
}

inline fun Notification.build(context: Context, func: NotificationCompat.Builder.() -> Unit): Notification {
  val builder = NotificationCompat.Builder(context)
  builder.func()
  return builder.build()
}
```

## kovenant - Promise

* https://github.com/mplatvoet/kovenant

```kotlin
async {
    //some (long running) operation, or just:
    1 + 1
} then { 
    i -> "result: $i"
} success {
    msg -> println(msg)
}
```

```kotlin
async { "world" } and async { "Hello" } success {
    println("${it.second} ${it.first}!")
}
```

## injekt - DI

* https://github.com/kohesive/injekt

## funKtionale - functional (Deprecated)

* https://github.com/MarioAriasC/funKTionale

<!--
Before:

```kotlin
val prefixAndPostfix = { prefix: String, x: String, postfix: String ->
    "${prefix}${x}${postfix}"
}
val prefixAndBang = prefixAndPostfix(p3 = "!")
val hello = prefixAndBang(p1 = "Hello, ")

println(hello("world"))
```

After:

```kotlin
val curriedPrefixAndPostifx = prefixAndPostfix.curried()
val curriedHello = curriedPrefixAndPostifx("hello")("world")
curriedHello.uncurried<>()
```
-->

```kotlin
val sum = { x: Int, y: Int -> x + y }
val curriedSum: (Int) -> (Int) -> Int = sum.curried()

assertEquals(curriedSum(2)(4), 6)

val add3 = curriedSum(3)
assertEquals(add3(5), 8)
```

* ref: 2015/3 mid [1]

## Closure

Before:

```kotlin
add(1) { 2 }

fun add(x: Int, func: () -> Int): Int {
    return x + func()
}
```

After:

```kotlin
add(1)(2)

fun add(x: Int): (Int) -> Int {
    return { y -> x + y }
}
```

## Fuel - networking

* https://github.com/kittinunf/Fuel

```kotlin
"http://github.com/yongjhih/".httpGet().responseString { request, response, either ->
    //do something with response
    when (either) {
        is Left -> // left means failure
        is Right -> // right means success
    }
}
```

## 對照表

(origin from [Using Project Kotlin for Android](https://docs.google.com/document/d/1ReS3ep-hjxWA8kZi0YqDbEhCqTt29hG8P44aA9W0DM8))

| Library                 | Jar Size | Dex Size | Method Count | Field Count |
|-------------------------|----------|----------|--------------|-------------|
| kotlin-runtime-0.10.195 | 354 KB   | 282 KB   | 1071         | 391         |
| kotlin-stdlib-0.10.195  | 541 KB   | 835 KB   | 5508         | 458         |

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
* http://try.kotlinlang.org/
* https://medium.com/@octskyward/kotlin-fp-3bf63a17d64a
* http://blog.zuehlke.com/en/android-kotlin/
* http://antonioleiva.com/collection-operations-kotlin/
* https://docs.google.com/presentation/d/1XTm-9WnwoiYhyHGamt-dHJBKmkEr3WajDCOGxgfIRsc
* https://github.com/importre/popular
* https://kotlinlang.org/docs/tutorials/command-line.html
* https://speakerdeck.com/jakewharton/advancing-development-with-kotlin-gdg-devfest-dublin-2015
* https://github.com/mplatvoet/kovenant-android-demo

[1]: https://medium.com/@octskyward/kotlin-fp-3bf63a17d64a
