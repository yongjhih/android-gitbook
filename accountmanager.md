# AccountManager

使用者帳號服務。

你可以以你的 app 名義透過 AccountManager 建立屬於你的使用者。

建立帳號(Top Down)：

```java
accountManager.addAccountExplicitly((Account) account, "password", (Bundle) userdata); // nullable userdata

Account account = new Account(username, accountType);

String accountType = "com.infstory"; // 帳號識別類型
String username = "yongjhih@example.com"; // 帳號名稱
```

相當然爾，你可以提供登入畫面，透過網路服務確定登入成功後，再執行上方程式來建立本地使用者。個資以及帳密都會存在系統裡，僅有 app 擁有者本身才可以存取密碼。(*當然已經鮮少 app 會頻繁的使用密碼，一般後端的使用者授權服務，大多登入成功後，會提供一組暫時授權證書，大多操作皆使用此授權證書*)

從系統中取得帳號(userdata, 帳密與授權證書需要 app 擁有者才可以存取)：

```java
Account[] accounts = accountManager.getAccountsByType(accountType);
Account account = accounts.length != 0 ? accounts[0];
```

設定授權證書(token):

```java
accountManager.setAuthToken(account, accountType, accessToken);
```

我們可以看

提供 3rd-party app 授權能力，充當行動裝置版本的 oauth provider 。


如果是扮演使用者中心：

* 對外提供登入器
* 以及其登入畫面
* 向 AccountManager 建立使用者

3rd-party 只要依據你登入器的 accountType 作為使用者中心的識別名稱，呼叫即可透過系統叫起你的登入畫面，以提供授權。

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