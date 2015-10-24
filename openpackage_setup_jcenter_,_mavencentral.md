# 開源套件設置

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

## mavenCentral

由 sonatype 組織所維護, 最早的一家。
必須要開源才能被收錄。最嚴苛的一家。

申請方式，首先，你必須要有一個開源專案。

註冊 sonatype 帳號後，發 ticket ，在 ticket 內容寫你的開源網址，通常是 github repository 網址，如：https://github.com/yongjhih/RetroFacebook ，這樣你就會拿到 `com.github.yongjhih` 的 group 。以後你都可以往這個群組裡面丟 jar/aar 。

如果是公司行號要申請，就必須在 ticket 內容寫明你是域名的擁有者，例如：`com.infstory` ，這樣你才能拿到 `com.infstory` Group。

很多上傳的方法，這裡就先不介紹。

上傳後，會先進到 stage 暫存區，做掃描行為後，你進到後台確定發布。不過通常半個到一個工作天才能夠真正進到 mavenCentral。

See Also:

* [OSSRH](http://central.sonatype.org/pages/ossrh-guide.html)

## jcenter

由 bintray 公司所維護。原則上也是需要開源。jcenter 會鏡像 mavenCentral 。而目前 jcenter 是 android studio 預設的套件中心。加上目前由於網頁設計的比較便民，所以大多已經改由 jcenter 發布了。當然很多其他系統還是只有 mavenCentral 所以 jcenter 也提供同步到 mavenCentral ，不過你要給它 OSSRH 的帳密就是了，當然 jcenter 也表明不會儲存你的帳密。

## jitpack

大約是 2015 年初的時候成立的。最大的特點是，直接支援 github/bitbucket repository 。所以不用特別申請套件中心帳號。因為不是你去放套件，而是它自己來拉你的源碼來編譯 jar/aar。(託運算與儲存的廉價)

基本上，筆者很看好這個套件中心。

不過目前還沒不是預設套件中心，使用者需要自行新增套件中心。故筆者目前的作法採取 jitpack 與 jcenter 並行。如有閒暇會讓 jcenter 同步到 mavenCentral 。

*p.s. jitpack 未來或許營利除了私有套件之外，可能可以提供付費服務，如報表等。javadoc 生成塞廣告。結合 travis-ci, cycle-ci 。同步 mavenCentral/jcenter*

## 線上 javadoc

* javadoc.io for maven central, http://javadoc.io/doc/GROUP.SUBGROUP/ARTIFACT/VERSION
* jitpack, https://jitpack.io/GROUP/SUBGROUP/ARTIFACT/VERSION/javadoc/

## ref.

* https://github.com/chrisbanes/gradle-mvn-push
* https://github.com/novoda/bintray-release
