# Lambda &lambda;

一種簡化寫 callback 的表達式。

Before:

```java
view.setOnClickListener(new View.OnClickListener() {
  @Override public void onClick(View v) {
    println("yo");
  }
});
```

After:

```java
view.setOnClickListener(v -> println("yo"));
```

* 當 interface 只有一個 method 需要實作時，就不需要特別再說是哪個 method name 了。包含 interface name 就可以整個省略。只剩下參數名稱要寫而已，如果怕參數型別有混淆之虞可寫上型別：

```java
view.setOnClickListener((View v) -> println("yo"));
```

* 如果沒有參數：

Before:

```java
Runnable = new Runnable() {
  @Override public void run() {
    println("yo");
  }
};
```

After:

```java
Runnable = () -> println("yo");
```

如果你要在 Android 上使用，請參考 [gradle-retrolambda](https://github.com/evant/gradle-retrolambda) 。


## 進階：如果 abstract class 只有一個 abstract method 是否也能適用 lambda？

筆者測試過，目前是不行的。

這裡筆者有想到一招替代方案，寫個可以吃 lambda 的 abstract class creator 來幫忙生，例如：

Before:

```java
void save(ParseUser user) {
    // SaveCallback 是 abstract class 且只有一個 abstract method: "void done(ParseException e);"
    user.save(new SaveCallback() {
        @Override public void done(ParseException e) {
            e.printStackTrace();
        }
    });
}
```

After:

```java
void save(ParseUser user) {
    user.save(Callbacks.save(e -> e.printStackTrace()));
}
```

寫一個可以吃 lambda 的 Callbacks.save() 來幫忙生 abstract SaveCallback：

```java
public class Callbacks {
    public interface ISaveCallback {
        void done(ParseException e);
    }

    public static SaveCallback save(ISaveCallback callback) {
        return new SaveCallback() {
            @Override public void done(ParseException e) {
                callback.done(e);
            }
        };
    }
}
```