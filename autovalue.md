# AutoValue


一個透過 Annotation Programming 簡化撰寫 Model 的編譯時期函式庫。

Before:

```java
public class Foo {
  public String text;
  public int number;
  
  public String text() {
    return text;
  }

  public int number() {
    return numbert;
  }
  
  public Foo(String text, int number) {
    this.text = text;
    this.number = number;
  }
}
```

```java
Foo andrew = new Foo("Andrew", 1);
```

After:

```java
import com.google.auto.value.AutoValue;

@AutoValue
public abstract class Foo {
  public abstract String text();
  public abstract int number();
  
  public static Foo create(String text, int number) {
    return new AutoValue_Foo(text, number);
  }
}
```

```java
Foo andrew = Foo.create("Andrew", 1);
```