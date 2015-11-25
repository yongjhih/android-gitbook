# multidex

我們知道 android app 最多 65k 數量限制。如果不幸超過，你可以透過官方的 multidex 機制來避開。
或期望 proguard 之後可以壓進 65k 內。

multidex 工作原理:

透過靜態分析得知哪些需要必須靜態載入 class 放入 classes.dex 其他則依序放入 `classes{2..N}.dex` 用來動態載入，來有效避開 65k 限制。

有趣的是，靜態分析其實是透過 proguard 來分析的。

重要的依據 AndroidManifest.xml 是 app 程式進入點，Activity/Service/ContentProvider。

## 套用 multidex

```gradle
multiDexEnabled true
```


```java
public class SimpleApplication extends MultiDexApplication {
    // ...
}
```


## 隱藏的設定 multiDexKeepFile/multiDexKeepProguard

不過有些靜態分析是分析，像是 ActiveAndroid ORM 的 models 。啟動時就必須靜態載入。

透過 multiDexKeepFile/multiDexKeepProguard 設定，指名哪些要包在 classes.dex 來靜態載入。

```gradle
multiDexKeepProguard file('multiDexKepp.txt')
multiDexKeepFile file('multiDexKepp.pro')
```

multiDexKepp.txt:

```
com/bluelinelabs/logansquare/LoganSquare.class
```

multiDexKepp.pro:

```proguard
-keep public class * extends com.activeandroid.Model {
    <init>();
}
```

## 套用 proguard

```gradle
minifyEnabled true
```

## 計算

坊間一堆 dex counter：

* 大多透過 `cat classes.dex | head -c 92 | tail -c 4 | hexdump -e '1/4 "%d"'` 取得
* 透過 baksmali 比較精準

轉換 dex 流程：

* apk2dex - `unzip "$apk" classes.dex`
* jar2dex - `dx --dex --output="$dex" "$jar"`

https://github.com/yongjhih/rc/blob/master/bin/dexize

```
dexize {apk...}
```

or

```
dexize {jar...}
```

```
dexize {folder}
```

## 編譯順便計算

* https://github.com/KeepSafe/dexcount-gradle-plugin

```bash
./gradlew countDebugDexMethods
```

詳細：`build/outputs/dexcount/debug.txt`

* https://github.com/mutualmobile/gradle-dexinfo-plugin

```bash
./gradlew dexinfoDebug
```

## See also

* https://gist.github.com/JakeWharton/6002797
* http://inloop.github.io/apk-method-count
