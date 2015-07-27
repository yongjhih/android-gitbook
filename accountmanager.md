# AccountManager

建立使用者以及登入使用者。行動裝置版本的 oauth provider 。

如果是扮演使用者中心：

* 對外提供登入器
* 以及其登入畫面
* 向 AccountManager 建立使用者

3rd-party application 只要依據你登入器的 uri 呼叫即可透過系統叫起你的登入畫面，提供授權。

## Retroauth

```java
GitHub github = GitHub.create(context);
github.login().flatMap(a -> github.getRepos());
```

```java
@Authentication(accountType = "accountType", tokenType = "tokenType")
public interface GitHub {
    @Authenticated
    Observable<Account> login();

    @Authenticated
    @GET("/{owner}/repos")
    Observable<Repo> getRepos(@Path String owner);
}
```