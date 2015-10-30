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

## 顯示測試項目通過與失敗

```gradle
tasks.withType(Test) {
  testLogging {
    exceptionFormat "full"
    events "passed", "skipped", "failed"
  }
}
```

```bash
./gradlew testDebug
```

## gradle 只測試單項

```bash
./gradlew testDebug --tests='*.<testname>'
```

