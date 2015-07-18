# Retrofit

rest api 直接對應成 java interface ，且搭配 Json Mapper 直接轉換.

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

## yongjhih/retrofit2 (in progress)

由於 retrofit 是執行時期處理 annotations 效能有改善的空間。retrofit2 是改用編譯時期處理。

## retrofit 分析 (1.9.0)

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
* https://github.com/yongjhih/retrofit2