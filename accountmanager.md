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

從系統中取得帳號資料(userdata, 帳密與授權證書需要 app 擁有者才可以存取)：

```java
Account[] accounts = accountManager.getAccountsByType(accountType);
Account account = accounts.length != 0 ? accounts[0];
```

設定授權證書(accessToken/authToken):

```java
accountManager.setAuthToken(account, authTokenType, authToken);
```

取得授權證書(accessToken/authToken):

```java
String authToken = accountManager.peekAuthToken(account, authTokenType)
```

## GitHub App

我們可以寫一個 GitHub App ，GitHub 網頁登入後，建立一個 GitHub 帳號，把 access token 存進去。

```java
public class LoginActivity extends Activity {
  @Override public void onResume() {
    super.onResume();

    loginButton.onClick(() -> {
      final GitHub github = GitHub.create();

      // Populate username, password, clientId, clientSecret from UI
      // ..

      AppObservable.bindActivity(this, github.getAccessToken(username, password, clientId, clientSecret))
        .subscribeOn(Scheduler.io())
        .subscribe(accessToken -> {
          String accountType = "com.github";
          Account account = new Account(username, "com.github");
          AccountManager accountManager = AccountManager.get(context);
          accountManager.addAccountExplicitly(account, password, userdata);
          // 一般會寫 "user,user:email" 等，在 oauth 稱為 scope, 不過帳密授權應該就全部權限，這邊特別寫明是密碼登入，方便識別。
          String authType = "password"; // authType/scope/permissions
          String authToken = accessToken.accessToken;
          accountManager.setAuthToken(account, authType, authToken);

          finish();
        });
    });
  }
}
```

```java
@Retrofit("https://api.github.com")
public abstract class GitHub {
  @FormUrlEncoded
  @POST("/oauth/token?grant_type=password") Observable<AccessToken> getAccessToken(
      @Field("username") String email,
      @Field("password") String password,
      @Field("client_id") String clientId,
      @Field("client_secret") String clientSecret);
  public static GitHub create() { return new Retrofit_GitHub(); }
}
```

## 提供 3rd-party app 授權能力，充當行動裝置版本的 oauth provider 授權服務

3rd-party app 只要依據你帳號管理中心的 accountType 作為使用者中心/授權服務的識別名稱，呼叫即可透過系統叫起你的登入畫面，以提供授權證書。

```java
AccountManagerFuture<Bundle> authTokenFutureBundle = accountManager.getAuthToken(account, authTokenType, (Bundle) null, activity, (AccountManagerCallback<Bundle>) null, (Handler) null);
String authToken = authTokenFutureBundle.getString(AccountManager.KEY_AUTHTOKEN); // waiting for UI logon
```

GitHub App 就要比較辛苦的扮演使用者中心：

* 提供帳號管理中心
* 以及其登入畫面
* 向 AccountManager 建立使用者

先從剛熟悉的登入畫面開始，不只是單純的自己登入就好，要想辦法把 authToken 傳給委託人。

改繼承 AccountAuthenticatorActivity 讓它幫你把一些你不想知道的東西塞好，如果你是用 AppCompatActivity 之類的，請參考 [AccountAuthenticatorActivity.java](https://github.com/android/platform_frameworks_base/blob/master/core/java/android/accounts/AccountAuthenticatorActivity.java) 源碼，自己塞。

```java
public class LoginActivity extends AccountAuthenticatorActivity { // 1.
  @Override public void onResume() {
    super.onResume();

    loginButton.onClick(() -> {
      final GitHub github = GitHub.create();

      // ..

      AppObservable.bindActivity(this, github.getAccessToken(username, password, clientId, clientSecret))
        .subscribeOn(Scheduler.io())
        .subscribe(accessToken -> {
          String accountType = "com.github";
          Account account = new Account(username, "com.github");
          AccountManager accountManager = AccountManager.get(context);
          accountManager.addAccountExplicitly(account, password, userdata);
          String authType = "password";
          String authToken = accessToken.accessToken;
          accountManager.setAuthToken(account, authType, authToken);

          // 2.
          Intent intent = new Intent();
          intent.putExtra(AccountManager.KEY_ACCOUNT_NAME, username);
          intent.putExtra(AccountManager.KEY_ACCOUNT_TYPE, accountType);
          intent.putExtra(AccountManager.KEY_AUTHTOKEN, authToken);
          ((AccountAuthenticatorActivity) this).setAccountAuthenticatorResult(intent.getExtras());
          setResult(RESULT_OK, intent);

          finish();
        });
    });
  }
}
```

委託登入授權服務(Authenticator) 實現所有 AccountManager 相關的代理操作。

3rd-party 對 AccountManager.getAuthToken() 其實會委託給我們的登入授權服務 Authenticator 受理 ，這部份我們再叫出剛配置好的登入畫面:

```java
// 登入授權服務
public class GitHubAuthenticator extends AbstractAccountAuthenticator {
  // 取得授權
  @Override
  public Bundle getAuthToken(AccountAuthenticatorResponse response, Account account,
        String authTokenType, Bundle options) throws NetworkErrorException {
    // 1. 你可以無條件提供授權
    // 2. 或者你可以提供一個畫面提醒使用者，是否同意提供授權

    // 如果還沒登入請使用者登入完再跳回來，繼續授權。

    AccountManager accountManager = AccountManager.get(context);
    // 拿拿看既有 GitHub Service 的權限授權
    String authToken = accountManager.peekAuthToken(account, authTokenType);

    String accountType = "com.github";
    if (authToken != null) {
      Bundle bundle = new Bundle();
      bundle.putString(AccountManager.KEY_ACCOUNT_NAME, account.name);
      bundle.putString(AccountManager.KEY_ACCOUNT_TYPE, accountType);
      bundle.putString(AccountManager.KEY_AUTHTOKEN, authToken);
      return bundle;
    }
    String authType = "password";

    // 拿不到 authToken 就當作登入吧 (由於目前我們的政策是，登入後，授權已經塞好在 bundle 了，所以我們可以直接回傳)
    return addAccount(response, accountType, authType, (String[]) null, (Bundle) null);

  }

  // 建立帳號
  @Override
  public Bundle addAccount(AccountAuthenticatorResponse response, String accountType,
        String authTokenType, String[] requiredFeatures, Bundle options)
        throws NetworkErrorException {

      // 登入畫面
      Intent intent = new Intent(context, LoginActivity.class);
      intent.putExtra(AccountManager.KEY_ACCOUNT_AUTHENTICATOR_RESPONSE, response);

      final Bundle bundle = new Bundle();
      bundle.putParcelable(AccountManager.KEY_INTENT, intent);
      return bundle;
  }
}
```

公開授權服務 AndroidManifest.xml:

```xml
<service
    android:name=".GitHubAuthenticatorService"
    android:exported="false">
    <intent-filter>
        <action android:name="android.accounts.AccountAuthenticator" />
    </intent-filter>
    <meta-data
        android:name="android.accounts.AccountAuthenticator"
        android:resource="@xml/authenticator" />
</service>

<activity
  android:name=".LoginActivity"
  android:exported="true"
  android:label="@string/app_name" />
```

res/xml/authenticator.xml:

```xml
<account-authenticator xmlns:android="http://schemas.android.com/apk/res/android"
    android:accountType="com.github"
    android:icon="@drawable/ic_launcher"
    android:smallIcon="@drawable/ic_launcher"
    android:label="@string/account_label" />
```

為了聽 AccountManagerService 發出來的委託 Service:

```java
public class GitHubAuthenticatorService extends Service {
    @Override
    public IBinder onBind(Intent intent) {
        return new GitHubAuthenticator(this).getIBinder();
    }
}
```

## 搭配 retrofit2 (retrofit-android) 自動授權

```java
GitHub github = GitHub.create(context);
github.repos().subscribe(System.out::println);
```

```java
public interface GitHub {
    @RequestInterceptor(GitHubAuthInterceptor.class)
    @GET("/{owner}/repos")
    Observable<Repo> repos(@Path String owner);
}
```

在呼叫 `GitHub.repos()` 時，會先呼叫 `accountManager.getAuthToken()` ，拿到 token 後，才憑證發出 `GET /{owner}/repos` request 。

```java
@Singleton
public class GitHubAuthInterceptor extends AuthenticationInterceptor {

    @Override
    public String accountType() {
        return "com.github";
    }

    @Override
    public String authTokenType() {
        return "com.github";
    }

    @Override
    public void intercept(String token, RequestFacade request) {
        if (token != null) request.addHeader("Authorization", "Bearer " + token);
    }

}
```

## 搭配 retroauth 自動授權

```java
@Authentication(accountType = R.string.auth_account_type, tokenType = R.string.auth_token_type)
public interface GitHub {
    @Authenticated
    @GET("/{owner}/repos")
    Observable<Repo> repos(@Path String owner);
}
```

## retroauth 分析

[AuthInvoker.java#L59](https://github.com/Unic8/retroauth/blob/master/retroauth/src/main/java/eu/unicate/retroauth/AuthInvoker.java#L59)

```java
getAccountName()
    .flatMap(this::getAccount)
    .flatMap(this::getAuthToken) // 取得授權證
    .flatMap(this::authenticate) // 設定憑證
    .flatMap(o -> request)
    .retry((c, e) -> retryRule.retry(c, e));
```
