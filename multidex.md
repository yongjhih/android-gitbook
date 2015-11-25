# multidex

我們知道 android app 最多 65k 數量限制。如果不幸超過，你可以透過官方的 multidex 機制來避開。
或期望 proguard 之後可以壓進 65k 內。

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

一般透過靜態分析得知哪些需要必須靜態載入 class ，但是有些靜態分析是分析不出來的，像是 ActiveAndroid ORM 的 models 。

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

```
dexcount {apk}
```

or

```
dexcount {jar}
```

or

```
dexcount {dex}
```

坊間一堆 dex counter 這裡就不贅述了

## 編譯順便計算

* https://github.com/KeepSafe/dexcount-gradle-plugin

```bash
./gradlew countDebugDexMethods
```

詳細：`build/outputs/dexcount/debug.txt`
