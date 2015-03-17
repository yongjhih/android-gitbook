# AutoValue


一個透過 Annotation Programming 簡化撰寫 identity Model 的編譯時期函式庫。

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
  
  @Override
  public String toString() {
    return "Foo{"
      + "text=" + text
      + ", number=" + number
      + "}";
  }
  
  @Override
  public boolean equals(Object o) {
    if (o == this) {
      return true;
    }
    if (o instanceof Foo) {
      Foo that = (Foo) o;
      return (this.text.equals(that.text()))
        && (this.number == that.number());
    }
    return false;
  }
  
  @Override int hashCode() {
    return Objects.hashCode(text, number);
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
Foo andrew1 = Foo.create("Andrew", 1);
Foo andrew2 = Foo.create("Andrew", 2);
System.out.println(andrew.equals(andrew1));
System.out.println(andrew.equals(andrew2));
```

## Android Parcelable

另外推薦 [frankiesardo/auto-parcel](https://github.com/frankiesardo/auto-parcel) :

```java
@AutoParcel
abstract class SomeModel implements Parcelable {
  abstract String name();
  abstract List<SomeSubModel> subModels();
  abstract Map<String, OtherSubModel> modelsMap();

  static SomeModel create(String name, List<SomeSubModel> subModels, Map<String, OtherSubModel> modelsMap) {
    return new AutoParcel_SomeModel(name, subModels, modelsMap);
  }
}
```
