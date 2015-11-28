# Proguard

build.gradle:

```gradle
proguardFiles gson.pro
proguardFiles joda-time.pro
...
```

或者 proguard-rules.pro:

```gradle
-include gson.pro
-include joda-time.pro
...
```

## 啟用 proguard

build.gradle:

```gradle
minifyEnabled true
```

## 一行搞定所有知名函式庫 proguard 設定

build.gradle：

```gradle
dependencies {
    compile 'com.infstory:proguard-snippets:1.0.0'
}
```

就可以搞定所有知名的函式庫，如：gson、Joda-Time、RxJava。

proguard-snippets 它的作用是：

* 蒐集所有知名函式庫的 *.pro
* 利用 aar 可以包裝 *.pro 的特性
* 利用 consumerProguardFiles 幫你來套用 *.pro

```gradle
consumerProguardFiles fileTree(dir: ‘libraries’, include: ‘*.pro’)
```

* 源碼：https://github.com/yongjhih/proguard-snippets

目前支援的知名函式庫：

* ACRA 4.5.0
* ActionBarSherlock 4.4.0
* ActiveAndroid
* Amazon Web Services 1.6.x / 1.7.x
* Amazon Web Services 2.1.x
* AndroidAnnotations
* android-gif-drawable
* Apache Avro
* Butterknife 5.1.2
* Crashlytics 1.+
* Crittercism
* EventBus 2.0.2
* Facebook 3.2.0
* Facebook Conceal
* Flurry 3.4.0
* Google Analytics 3.0+
* Google Guava
* Google Play Services 4.3.23
* GreenDao 1.3.x
* GSON 2.2.4
* Jackson 2.x
* Joda-Convert 1.6
* Joda-Time 2.3
* New Relic
* Parse
* Realm
* RxJava 0.21
* Support Library v7
* Sqlite
* Square OkHttp
* Square Okio
* Square Otto
* Square Picasso
* Square Retrofit
* Square Wire
* Icepick
