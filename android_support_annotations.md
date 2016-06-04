# Annotations 註記

常用的註記介紹.

一堆常見的工具庫都有提供註記：google guava, jsr, apache common, google common, android support

## NonNull/Nullable

`@NonNull` 與 `@Nullable` 在一些工具庫也有提供，不過 Android 主流還是使用 Android Support：

```java
import android.support.annotation.NonNull;
import android.support.annotation.Nullable;
...

    /**
     * Add support for inflating the <fragment> tag.
     */
    @Nullable
    @Override
    public View onCreateView(String name, @NonNull Context context, @NonNull AttributeSet attrs) {
        ...
```

這樣的好處是防呆以及省防禦性檢查：

```java
public class MyFragment extends Fragment {
    @Nullable
    @Override
    public View onCreateView(String name, @NonNull Context context, @NonNull AttributeSet attrs) {
        if (TextUtils.isEmpty(name)) return null; // 這行編譯就會報錯

        if (TextUtils.isEmpty(name)) nam = context.getResources().getString(R.id.app_name); // 不用一一檢查 context != null
        // 因為 @NonNull 註記已經確保不讓塞 null
    }
}
```

以及其他：

```java
import android.support.annotation.StringRes;
...
    public abstract void setTitle(@StringRes int resId); // 只允許塞 R.string.xxx
```

```java
import android.support.annotation.IntDef;
...
public abstract class ActionBar {
    ...
    @IntDef({NAVIGATION_MODE_STANDARD, NAVIGATION_MODE_LIST, NAVIGATION_MODE_TABS})
    @Retention(RetentionPolicy.SOURCE)
    public @interface NavigationMode {}

    public static final int NAVIGATION_MODE_STANDARD = 0;
    public static final int NAVIGATION_MODE_LIST = 1;
    public static final int NAVIGATION_MODE_TABS = 2;

    @NavigationMode
    public abstract int getNavigationMode(); // 不用新增型別 NavigationMode class，直接標記枚舉防呆

    public abstract void setNavigationMode(@NavigationMode int mode);
```

```java
    @IntDef(flag=true, value={ // flag = true 是給 1, 2, 4, 8 等這種旗標性數值使用，所以就算不是枚舉的數字 10 也是合法的，因為 2 + 8
            DISPLAY_USE_LOGO,
            DISPLAY_SHOW_HOME,
            DISPLAY_HOME_AS_UP,
            DISPLAY_SHOW_TITLE,
            DISPLAY_SHOW_CUSTOM
    })
    @Retention(RetentionPolicy.SOURCE)
    public @interface DisplayOptions {}
```

## @VisibleForTesting

```java
@VisibleForTesting
public void setLogger(ILogger logger) {...}
```

## @Keep

> [@Keep](http://tools.android.com/tech-docs/support-annotations#TOC-Keep)

> We've also added @Keep to the support annotations. Note however that > that annotation hasn't been hooked up to the Gradle plugin yet (though > it's [in progress](https://android-review.googlesource.com/#/c/152983/).) When finished this will let you annotate methods and > > classes that should be retained when minimizing the app.

看起來還在進行中，可以先用筆者的專案：https://github.com/yongjhih/proguard-annotations


## See Also

* ref. http://tools.android.com/tech-docs/support-annotations
* https://plus.google.com/+StephanLinzner/posts/GBdq6NsRy6S
