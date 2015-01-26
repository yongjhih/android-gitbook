# Lambda

老樣子，直接看 code：

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

* 當 interface 只有一個 method 就可以整個省略。只剩下參數名稱要寫而已，如果怕參數型別有混淆之虞可寫上型別：

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