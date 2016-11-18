# Retrofit 1

ps. We adapted retrofit since 2014, 1.5.0+.

rest api 直接對應成 java interface ，且搭配 Json Mapper 直接轉換。

一個類似於 JAX-RS/JSR-311) 的實現，基於[一些理由](https://github.com/square/retrofit/issues/573)並沒有完全按照 JAX-RS 實現。

Interface:

```java
    interface GitHub {
        @GET("/repos/{owner}/{repo}/contributors")
        Observable<List<Contributor>> contributors(
            @Path("owner") String owner,
            @Path("repo") String repo);

        @GET("/users/{user}")
        Observable<User> user(
            @Path("user") String user);

        @GET("/users/{user}/starred")
        Observable<List<Repo>> starred(
            @Path("user") String user);
    }
```

Usage:

```java
        RestAdapter restAdapter = new RestAdapter.Builder()
            .setEndpoint("https://api.github.com")
            .build();

        GitHub github = restAdapter.create(GitHub.class);

        github.contributors("ReactiveX", "RxJava")
            .flatMap(list -> Observable.from(list))
            .forEach(c -> System.out.println(c.login + "\t" + c.contributions));
```

Models:

```java
    static class Contributor {
        String login;
        int contributions;
    }

    static class User {
        String name;
    }

    static class Repo {
        String full_name;
    }
```

## Retrofit 1 網路處理

retrofit 內部運作也是使用 `Request` 概念來運行。

Retrofit 很顯然的，希望讓開發者專注 Restful 介面，e.g. `List<Repo> repos = github.repos();`

處理的介面不再是網路相關的 Request ，反倒是刻意隱含了 Request ，導致網路細部操作看似無法處理，所以提供了錯誤處理註冊，`Client` 配置等接口。

2. 常見的網路處理：

* Cache
* Retry
* Time Out
* Cancel
* Priority

使用 OkHttpClient 大多可以處理。(AOSP 4.4 之後其實也內建，不確定有沒有包進 SDK)

如果沒有用 RxJava Observable 了話，確實 Retry policy 的部份確實有點殘念，也不是無解，只是要自己辛苦點寫 ErrorHandler 重發。

使用 RxJava `retry()` 達到重試次數效果：

```java
int retryLimit = 3;
Observable<Repo> repos = github.repos()
    .retry(3);
```

使用 RxJava `retryWhen()` 達到 backoff 效果：

```java
int retryLimit = 3;
float backoff = 1.3f

Observable<Repo> repos = github.repos()
    .retryWhen(attempt -> attempt.zipWith(Observable.range(1, retryLimit), (n, i) -> i)
        .flatMap(i -> Observable.timer(i * backoff, TimeUnit.SECONDS));
```

Cancel 也是，如果沒有 RxJava 你只能寫 ExecutorService 處理或者操作 OkHttpClient。

使用 Rxjava `unsubcribe()` 取消：

```java
Subscription reposSubscription = repos.subscribe();

reposSubscription.unsubscribe(); // 取消
```

其中比較困難的是 Priority ，需要處理 OkHttpClient executor 。

基本上，「RxJava + Retrofit + OkHttpClient(supports SPDY)」一起用，應該是沒什麼太大的問題。

## NotRetrofit

由於 retrofit 是執行時期處理 annotations 效能有改善的空間。NotRetrofit 是改用編譯時期處理。如同 google fork square/dagger 專案的理由一樣: google/dagger2。

https://github.com/yongjhih/NotRetrofit

## Retrofit 2

```java
interface GitHub {
    @GET("/repos/{owner}/{repo}/contributors")
    Call<List<Contributor>> contributors(
        @Path("owner") String owner,
        @Path("repo") String repo);
}
```

解決 Retrofit 1 長久以來存在的一些問題。不直接回傳物件，而是透過一個中介 `Call`/`Response` 來包裝，使得可以保留網路資訊 Headers 。

### 如何取得分頁？

```java
interface GitHub {
    // ...
    @GET
    Call<List<Contributor>> contributorsPaginate(@Url String url);
}
```

```java
Call<List<Contributor>> contributorsCall = github.contributors("square", "retrofit");
Response<List<Contributor>> contributorsResponse = contributorsCall.execute();
String nextLink = contributorsResponse.headers().get("Link");
Call<List<Contributor>> contributorsNextCall = github.contributorsPaginate(nextLink);
Response<List<Contributor>> contributorsNextResponse = contributorsNextCall.execute();
```

## Retrofit 1 分析 (1.9.0)

```java
private class RestHandler implements InvocationHandler {
  // ...
  public Object invoke(Object proxy, Method method, final Object[] args) {
    // sync
    return invokeRequest(requestInterceptor, methodInfo, args);
    // Rx
    return rxSupport.createRequestObservable({
      (ResponseWrapper) invokeRequest(requestInterceptor, methodInfo, args);
    });
    // async
    Callback<?> callback = (Callback<?>) args[args.length - 1]; // last argument
    httpExecutor.execute({
      (ResponseWrapper) invokeRequest(interceptorTape, methodInfo, args);
    });
  }
}
```

## See Also

* https://github.com/yongjhih/RxJava-retrofit-github-sample
* https://github.com/yongjhih/NotRetrofit
* https://github.com/orhanobut/wasp
