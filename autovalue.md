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
    return number;
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
public abstract class User {
    public abstract String name();
    public abstract int id();

    public static User builder() {
        return new AutoValue_User.Builder();
    }
    
    @AutoValue.Builder
    interface Builder {
        Builder name(String s);
        Builder id(int n);
        User build();
    }
}
```

```java
User andrew = Foo.builder().name("andrew").id(1).build();
User andrew1 = Foo.create("Andrew", 1);
User andrew2 = Foo.create("Andrew", 2);
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

    public static Builder builder() {
        return new AutoParcel_SomeModel.Builder();
    }   

    @AutoParcel.Builder
    public interface Builder {
        public Builder name(String x);
        public Builder subModels(<SomeSubModel> x);
        public Builder modelsMap(Map<String, OtherSubModel> x);
        public SomeModel build();
    }
}
```

## See Also

* https://github.com/vbauer/jackdaw