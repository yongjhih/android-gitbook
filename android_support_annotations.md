# Annotations

常用的 Annotations 介紹.

google guava, jsr, apache common, google common, android support

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

```java
import android.support.annotation.StringRes;
...
    public abstract void setTitle(@StringRes int resId);
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
    public abstract int getNavigationMode();

    public abstract void setNavigationMode(@NavigationMode int mode);
```

```java
    @IntDef(flag=true, value={
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
