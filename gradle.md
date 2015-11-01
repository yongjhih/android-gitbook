# gradle

大多數的樣貌 build.gradle:

```gradle
buildscript { // 建置設定區 - 引入建置相關插件庫
    repositories {
        jcenter() // 建置套件庫
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3' // 插件庫
    }
}

apply plugin: 'com.android.application'

repositories {
    jcenter() // 函式套件庫
}

dependencies {
    //compile '{group}:{artifact}:{version}'
    //compile project('{module}')
}

android {} // com.android.application 插件設定區
```

多模組的目錄結構：

```
-a-project
|--build.gradle // 一般空檔, 除非需要子模組共用的設定，可以在這裡設定
|--a-module/build.gradle
|--b-module/build.gradle
```

設定預設編譯哪些 module：

settings.gradle:

```gradle
include ':a-module'
include ':b-module'
// or include ':a-module', 'b-module'
```

設定外部路徑：

```gradle
// ...
include ':b-c-module'
project(':b-c-module').projectDir = new File(settingsDir, '../b-project/c-module')
```

build.gradle:

```gradle
// ...
dependencies {
    // ...
    compile project(':b-c-module')
}
// ...
```

## 設定快取有效時間

預設 24 小時，每天一開始的編譯都會比較久。為了避免這種情形，可以拉長時間，如有必要再透過強制刷新來解決。

寫到專案設定：

```gradle
configuration.all {
  resolutionStrategy {
    cacheDynamicVersionsFor 30, 'days'
    cacheChangingModulesFor 30, 'days'
  }
}
```

## 強制刷新套件

如果有些套件像是 SNAPSHOT.jar 剛更新，可透過 `--refresh-dependencies` 來刷到新的版本：

```gradle
./gradlew --refresh-dependencies assembleDebug
```

## 顯示詳細的測試項目通過與失敗

```gradle
tasks.withType(Test) {
  testLogging {
    exceptionFormat "full"
    events "passed", "skipped", "failed", "standardOut", "standardError"
    showStandardStreams = true
  }
}
```

## 一般測試

```bash
./gradlew testDebug
```

## 測試單項

```bash
./gradlew testDebug --tests='*.<testname>'
```

## 顯示更多 lint 警告

```gradle
tasks.withType(JavaCompile) {
  options.compilerArgs << "-Xlint:deprecation" << "-Xlint:unchecked"
}
```

## 安裝 gradle wrapper

```gradle
task wrapper(Wrapper) {
  gradleVersion = "2.4"
}
```

```bash
gradle wrapper
```

## 升級 gradle wrapper

修改 gradle/wrapper/gradle-wrapper.properties:

```gradle
...
distributionUrl=https\://services.gradle.org/distributions/gradle-2.4-all.zip
```

## ref.

* https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.ResolutionStrategy.html