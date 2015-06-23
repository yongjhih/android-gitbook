# Retrofit

```java
public class Main {
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

    interface GitHub {
        @GET("/repos/{owner}/{repo}/contributors")
        //List<Contributor> contributors(
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

    public static void main(String... args) {
        String token = (args.length > 1) ? args[1] : null;
        System.out.println("token: " + token);
        // Create a very simple REST adapter which points the GitHub API endpoint.
        RestAdapter restAdapter = new RestAdapter.Builder()
            .setEndpoint("https://api.github.com")
            .setRequestInterceptor(request -> {
                if (token != null && !"".equals(token)) {
                    // https://developer.github.com/v3/#authentication
                    request.addHeader("Authorization", "token " + token);
                }
            })
            .build();

        // Create an instance of our GitHub API interface.
        GitHub github = restAdapter.create(GitHub.class);

        // Fetch and print a list of the contributors to this library.
        github.contributors("ReactiveX", "RxJava")
            .flatMap(list -> Observable.from(list))
            .forEach(c -> System.out.println(c.login + "\t" + c.contributions));
}
```

## yongjhih/retrofit2

由於 retrofit 是執行時期處理 annotations 效能有改善的空間。retrofit2 是改用編譯時期處理。

## See Also

* https://github.com/yongjhih/RxJava-retrofit-github-sample
* https://github.com/yongjhih/retrofit2