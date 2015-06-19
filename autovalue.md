# AutoValue

一個透過 Annotation Programming 簡化撰寫 identity model 的編譯時期函式庫。

Before:

```java
public class User {
  public String name;
  public int id;
  
  public String name() {
    return name;
  }

  public int id() {
    return id;
  }
  
  public User(String name, int id) {
    this.name = name;
    this.id = id;
  }
  
  @Override
  public String toString() {
    return "User{"
      + "name=" + name
      + ", id=" + id
      + "}";
  }
  
  @Override
  public boolean equals(Object o) {
    if (o == this) {
      return true;
    }
    if (o instanceof User) {
      User that = (User) o;
      return (this.name.equals(that.name()))
        && (this.id == that.id());
    }
    return false;
  }
  
  @Override int hashCode() {
    return Objects.hashCode(name, id);
  }
}
```

```java
User andrew = new User("Andrew", 1);
User andrew1 = new User("Andrew", 1);
User andrew2 = new User("Andrew", 2);
System.out.println(andrew.equals(andrew1));
System.out.println(andrew.equals(andrew2));
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
User andrew = User.builder().name("Andrew").id(1).build();
User andrew1 = User.builder().name("Andrew").id(1).build();
User andrew2 = User.builder().name("Andrew").id(2).build();
System.out.println(andrew.equals(andrew1));
System.out.println(andrew.equals(andrew2));
```

## Android Parcelable

[frankiesardo/auto-parcel](https://github.com/frankiesardo/auto-parcel):

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
        public Builder subModels(List<SomeSubModel> x);
        public Builder modelsMap(Map<String, OtherSubModel> x);
        public SomeModel build();
    }
}
```

或者 https://github.com/johncarl81/parceler

```java
@AutoValue
@Parcel
public abstract class AutoValueParcel {

    @ParcelProperty("value") public abstract String value();

    @ParcelFactory
    public static AutoValueParcel create(String value) {
        return new AutoValue_AutoValueParcel(value);
    }
}
```

parceler 的產生器, 有點獨特..

## See Also

* https://github.com/vbauer/jackdaw