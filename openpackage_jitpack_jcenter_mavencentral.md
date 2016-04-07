# 發布套件

## 引用套件

要把編譯好的 jar/aar 套件，上傳到套件管理中心，方便別人下載使用：

```java
dependencies {
    compile 'com.github.yongjhih:RetroFacebook:1.0.0'
}
```

那麼你要就上到一個 maven 套件伺服器。目前知名的伺服器中心有三家：

* mavenCentral
* jcenter
* jitpack

## mavenCentral 發布套件

由 sonatype 組織所維護, 最早的一家。
必須要開源才能被收錄。最嚴苛的一家。

申請方式，首先，你必須要有一個開源專案。

註冊 sonatype 帳號後，發 ticket ，在 ticket 內容寫你的開源網址，通常是 github repository 網址，如：https://github.com/yongjhih/RetroFacebook ，這樣你就會拿到 `com.github.yongjhih` 的 group 。以後你都可以往這個群組裡面丟 jar/aar 。

如果是公司行號要申請，就必須在 ticket 內容寫明你是域名的擁有者，例如：`com.infstory` ，這樣你才能拿到 `com.infstory` Group。

mavenCentral 發布指南:

* [OSSRH](http://central.sonatype.org/pages/ossrh-guide.html)

~/.gradle/gradle.properties 設定 mavenCentral 帳密與 gpg key

在你的專案安裝 https://github.com/chrisbanes/gradle-mvn-push

設定專案套件資訊 gradle.properties：

```
VERSION_NAME=0.9.2-SNAPSHOT
VERSION_CODE=92
GROUP=com.github.chrisbanes.actionbarpulltorefresh

POM_DESCRIPTION=A modern implementation of the pull-to-refresh for Android
POM_URL=https://github.com/chrisbanes/ActionBar-PullToRefresh
POM_SCM_URL=https://github.com/chrisbanes/ActionBar-PullToRefresh
POM_SCM_CONNECTION=scm:git@github.com:chrisbanes/ActionBar-PullToRefresh.git
POM_SCM_DEV_CONNECTION=scm:git@github.com:chrisbanes/ActionBar-PullToRefresh.git
POM_LICENCE_NAME=The Apache Software License, Version 2.0
POM_LICENCE_URL=http://www.apache.org/licenses/LICENSE-2.0.txt
POM_LICENCE_DIST=repo
POM_DEVELOPER_ID=chrisbanes
POM_DEVELOPER_NAME=Chris Banes
```

上傳套件：

```
./gradlew uploadArchives
```

上傳後，會先進到 stage 暫存區，做掃描行為後，你進到網頁後台確定發布。不過通常半個到一個工作天才能夠真正進到 mavenCentral。

## jcenter

由 bintray 公司所維護。原則上也是需要開源。jcenter 會鏡像 mavenCentral 。而目前 jcenter 是 android studio 預設的套件中心。加上目前由於網頁設計的比較便民，所以大多已經改由 jcenter 發布了。當然很多其他系統還是只有 mavenCentral 所以 jcenter 也提供同步到 mavenCentral ，不過你要給它 OSSRH 的帳密就是了，當然 jcenter 也表明不會儲存你的帳密。

安裝 https://github.com/novoda/bintray-release

上傳套件：

```
./gradlew bintrayUpload
```

去 bintray.com 後台一鍵申請發布到 jcenter ，通常一天內就會審核通過。(未來這部份應該透過上傳程式就直接提出申請，這樣省事多了)

## jitpack

大約是 2015 年初的時候成立的。最大的特點是，直接支援 github/bitbucket repository 。所以不用特別申請套件中心帳號。因為不是你去放套件，而是它自己來拉你的源碼來編譯 jar/aar。

### jitpack 引用套件

引用方 build.gradle:

```gradle
repositories {
    // ...
    maven { url "https://jitpack.io" }
}

dependencies {
    compile 'com.github.{user}:{repo}:{version}'
}
```

### jitpack 發布套件

因為是引用時編譯，不需要自己編譯上傳發布，只要安裝配置好基本 script 提交到 github repo ，就沒事了。

在 module 內 build.gradle 加入：

```gradle
buildscript {
    repositories {
        jcenter()
    }

    dependencies {
        // ...
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.3'
    }
}

apply plugin: 'com.github.dcendents.android-maven'
apply from: 'android-javadoc.gradle'
```

以及新增 [android-javadoc.gradle](https://gist.github.com/yongjhih/3f1ed93c80e0b248cf55):

```gradle
task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}

task javadoc(type: Javadoc) {
    failOnError  false
    source = android.sourceSets.main.java.sourceFiles
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}

task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}

artifacts {
    archives sourcesJar
    archives javadocJar
}
```

如果你引用的了太多 jitpack 上面的套件，會需要很長的時間等待 jitpack 的編譯時間。故筆者作為一個函式庫貢獻者，盡可能還是採取 jitpack 與 jcenter 並行。甚者，如有閒暇會讓 jcenter 同步到上游 mavenCentral 。

*p.s. jitpack 未來或許營利除了私有套件之外，可能可以提供付費服務，如報表等。javadoc 生成塞廣告。結合 travis-ci, cycle-ci 。同步 mavenCentral/jcenter*

## 線上 javadoc

* javadoc.io for maven central, `http://javadoc.io/doc/{GROUP}{.SUBGROUP}/{ARTIFACT}/{VERSION}`
* jitpack, `https://jitpack.io/{GROUP}{/SUBGROUP}/{ARTIFACT}/{VERSION}/javadoc/`

## ref.

* https://github.com/chrisbanes/gradle-mvn-push
* https://github.com/novoda/bintray-release
