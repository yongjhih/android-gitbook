# Android 的日常

這邊主要抽一些日常的小撇步。

## 不要用 Thread 或 HandlerThread 推薦使用 AsyncTask

如果要背景下載檔案，應該使用 IntentService 而不是自己去操作 Thread。

如果是這個 Activity 內的長時間存取，可以使用 AsyncTask 。如有必要請搭配 LoaderManager 使用。

## 不要用 Handle Message 推薦使用 `post(Runnable)`

通常你其實需要的是 `post(Runnable)`

```java
post(() -> updateProgress());
```

這樣不用再管理編號了。

```java
static final int MSG_UPDATE_PROGRESS = 0;

// ...

    MSG_UPDATE_PROGRESS:
        updateProgress();
        break;

// ...
```

## 推薦使用 `postDelayed(milliseconds, Runnable)` 來做延遲呼叫

## 防彈跳推薦 `removeCallback(Runnable)` + `postDelayed(Runnable)` 不再量測時間 timeout

```java
removeCallback(updateProgressRunnable);
postDelayed(300, updateProgressRunnable);
```

## 不再 findViewById 推薦使用 ButterKnife

After:

```java
@BindView(R.id.username)
TextView usernameView;


// ...

@Override public void onCreate(Bundle savedInstanceState) {
   // ...
   ButterKnife.bind(this);
}
```

Before:

```java
TextView usernameView;

@Override public void onCreate(Bundle savedInstanceState) {
   // ...
   usernameView = (TextView) findViewById(R.id.username);
}
```

## 使用 @Nullable/@NonNull 標注

以及各種 support-annotations

* @IntDef
* @Keep
* etc.

## 宣告型別應盡可能抽象型別，應 `List` 非 `ArrayList`

* `Map` 非 `HashMap`

```java
List<String> getNames(List<User> users) {
    List<String> names = new ArrayList<>();
    for (User user : users) names.add(user.name());
    return names;
}
```

## 使用泛型建置技巧 `new ArrayList<>()`

java7 之後可省略型別，直接推定型別

```java
List<String> names = new ArrayList<>();
```

java6 沒有內建推定型別，可透過推定型別函式來包裝，常見的 apache common-lang 或者 guava 函式庫也有此範例：

```java
List<String> names = newArrayList();

public static <T> ArrayList<T> newArrayList() {
    return new ArrayList<T>();
}
```

## IDE 推薦使用 Android Studio 而不是 Eclipse/ADT

...

## 推薦使用 gradle 建置系統而不是 ant

還有一些其他選項:

* buck
* maven
* sbt
* kobalt

有個基本原則，就是支援 maven 套件中心的建置系統

## 該傳遞 Context 就不要傳遞 Activity，避免記憶體浪費

```java
ImageView imageView = new ImageView(activity.getApplicationContext());
```

不該：

```java
ImageView imageView = new ImageView(activity);
```

同樣的，函式庫的開發者，盡可能不要紀錄 activity:

```java
public class ImageLoader {
    Context context;
    public ImageLoader(Activity activity) { this(activity.getApplicationContext()); }
    public ImageLoader(Context context) { this.context = context; }
}
```

## 常見的取得 LayoutInflater

```java
LayoutInflater.from(Context);
```

## 常見的取得 context

```java
view.getContext();
```

## UI 執行緒執行

```java
activity.runOnUiThread(runnable);
```

```java
view.post(runnable);
```

```java
new Handler(Looper.getMainLooper()).post(runnable);
```

## 使用 Objects 工具類別來簡化 null 檢查

Before:

```java
if (!mBuildConfigFields.equals(that.mBuildConfigFields)) {
    return false;
}
if (!mConsumerProguardFiles.equals(that.mConsumerProguardFiles)) {
    return false;
}
if (!mManifestPlaceholders.equals(that.mManifestPlaceholders)) {
    return false;
}
if (mMultiDexEnabled != null ? !mMultiDexEnabled.equals(that.mMultiDexEnabled) :
        that.mMultiDexEnabled != null) {
    return false;
}
if (mMultiDexKeepFile != null ? !mMultiDexKeepFile.equals(that.mMultiDexKeepFile) :
        that.mMultiDexKeepFile != null) {
    return false;
}
if (mMultiDexKeepProguard != null ? !mMultiDexKeepProguard.equals(that.mMultiDexKeepProguard) :
        that.mMultiDexKeepProguard != null) {
    return false;
}
if (!mProguardFiles.equals(that.mProguardFiles)) {
    return false;
}
if (!mResValues.equals(that.mResValues)) {
    return false;
}
return true;
```

After:

```java
return Objects.equal(mBuildConfigFields, that.mBuildConfigFields) &&
        Objects.equal(mConsumerProguardFiles, that.mConsumerProguardFiles) &&
        Objects.equal(mManifestPlaceholders, that.mManifestPlaceholders) &&
        Objects.equal(mMultiDexEnabled, that.mMultiDexEnabled) &&
        Objects.equal(mMultiDexKeepFile, that.mMultiDexKeepFile) &&
        Objects.equal(mMultiDexKeepProguard, that.mMultiDexKeepProguard) &&
        Objects.equal(mProguardFiles, that.mProguardFiles) &&
        Objects.equal(mResValues, that.mResValues);
```

## 使用 Objects 工具類別來產生 hashcode

Before:

```java
int result = mBuildConfigFields.hashCode();
result = 31 * result + mResValues.hashCode();
result = 31 * result + mProguardFiles.hashCode();
result = 31 * result + mConsumerProguardFiles.hashCode();
result = 31 * result + mManifestPlaceholders.hashCode();
result = 31 * result + (mMultiDexEnabled != null ? mMultiDexEnabled.hashCode() : 0);
result = 31 * result + (mMultiDexKeepFile != null ? mMultiDexKeepFile.hashCode() : 0);
result = 31 * result + (mMultiDexKeepProguard != null ? mMultiDexKeepProguard.hashCode() : 0);
return result;
```

After:

```java
return java.util.Objects.hashCode(
        mBuildConfigFields,
        mResValues,
        mProguardFiles,
        mConsumerProguardFiles,
        mManifestPlaceholders,
        mMultiDexEnabled,
        mMultiDexKeepFile,
        mMultiDexKeepProguard);
```

## Activity/Fragment 應該善用 @CallSuper

```java
@CallSuper
@Override
public void onResume() {
    super.onResume();
}
```

## 盡可能使用 `private final mMember`

## 成員變數名稱開頭，慣用小寫 "m" 開頭：`mMember`

## 不要直接 Handler 成員變數，避免記憶體浪費，可改用 WeakReference 包裝

```java
private final Handler mHandler = new Handler();
```

## static final 常數慣用大寫

```java
static final float PI = 3.14f;
```

## static 變數慣用小寫 "s" 開頭

```java
private static Runtime sRuntime = new Runtime();
```

## 不要輕易使用 static 變數，避免記憶體浪費

```java
static Drawable sBackground;
```

非得要用了話，請用 WeakReference 裝起來：

```java
static WeakReference<Drawable> sBackground;
```


## 不要輕易使用 static 變數在 View 上，避免記憶體浪費

```java
static TextView sView;
```

因為 view 本身帶有 Context ，非得要用了話，請用 WeakReference 裝起來：

```java
static WeakReference<TextView> sView;
```

## 工具類別應不給繼承且不給建構子


## 回傳空 List 應該用 `return Collections.emptyList();` 而不是 `return new ArrayList<>();`

```java
List<String> getNames(List<User> users) {
    if (users == null) return Collections.emptyList();

    List<String> names = new ArrayList<>();
    for (User user : users) names.add(user.name());
    return names;
}
```

回傳空 Map 應該用 `return Collections.emptyMap();` 而不是 `return new HashMap<>();`

```java
Map<String, User> getNames(List<User> users) {
    if (users == null) return Collections.emptyMap();

    Map<String, User> map = new HashMap<>();
    for (User user : users) map.put(user.id(), user);
    return map;
}
```

## 大量的字串串接，請用 StringBuilder 來避免不必要的低消

```java
System.out.println("Hello" + ", " + "world" + "!");
```

```java
new StringBuilder().append("Hello").append(", ").append("world").append("!");
```

## SystemService 取得方法，如 NotificationManager 請改用 `context.getSystemService(NotificationManager.class)`

Before:

```java
NotificationManager notificationManager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
```

After:

```java
NotificationManager notificationManager = context.getSystemService(NotificationManager.class);
```

好處是不用強轉型了。

```java
// 另外推薦函式庫 com.github.yongjhih:android-system-services:1.0.0
NotificationManager notificationManager = SystemServices.from(context).getNotificationManager();
```

## Listener/Callback 設計三兩事

就算我們只想要知道 `onPageSelected(position)` ，但是所有的方法還是都要假實現：

```java
pager.setOnPageChangeListener(new ViewPager.OnPageChangeListener() {
    @Override
    public void onPageScrollStateChanged(int state) {
    }

    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
    }

    @Override
    public void onPageSelected(int position) {
       System.out.println(position);
    }
});
```

所以一般我們都習慣寫一個空類別來偷懶一下：

```java
pager.setOnPageChangeListener(new SimpleOnPageChangeListener() {
    @Override
    public void onPageSelected(int position) {
       System.out.println(position);
    }
});
```

由於已經有 lambda 的幫助下，我們可以想到這個用法：

```java
pager.setOnPageChangeListener(new SimplerOnPageChangeListener().onPageSelected(position -> {
   System.out.println(position);
}));
```

附錄:

```java
public interface ViewPager.OnPageChangeListener() {
    void onPageScrollStateChanged(int state);
    void onPageScrolled(int position, float positionOffset, int positionOffsetPixels);
    void onPageSelected(int position);
}

public class SimpleOnPageChangeListener implements OnPageChangeListener {
    @Override
    public void onPageScrollStateChanged(int state) {
    }

    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
    }

    @Override
    public void onPageSelected(int position) {
    }
}

public class SimplerOnPageChangeListener implements OnPageChangeListener {
    @Override
    public void onPageScrollStateChanged(int state) {
        if (onPageScrollStateChanged == null) return;
        onPageScrollStateChanged.call(state);
    }

    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
        if (onPageScrolled == null) return;
        onPageScrolled.call(position, positionOffset, positionOffsetPixels);
    }

    @Override
    public void onPageSelected(int position) {
        if (onPageSelected == null) return;
        onPageSelected.call(position);
    }

    Action1<Integer> onPageScrollStateChanged;
    Action3<Integer, Float, Integer> onPageScrolled;
    Action1<Integer> onPageSelected;

    public SimplerOnPageChangeListener onPageScrollStateChanged(Action1<Integer> onPageScrollStateChanged) {
        this.onPageScrollStateChanged = onPageScrollStateChanged;
        return this;
    }

    public SimplerOnPageChangeListener onPageScrolled(Action3<Integer, Float, Integer> onPageScrolled) {
        this.onPageScrolled = onPageScrolled;
        return this;
    }

    public SimplerOnPageChangeListener onPageSelected(Action1<Integer> onPageSelected) {
        this.onPageSelected = onPageSelected;
        return this;
    }

    public interface Action1<T> {
        void call(T t);
    }

    public interface Action3<T, T2, T3> {
        void call(T t, T2 t2, T3 t3);
    }
}
```

## 泛型 Generic 應用於 Callback

如果你是設計一個 Github RESTful API 客戶端，可能會是這樣寫：

```java
interface RequestListener {
    void onComplete(String json);
    void onException(Exception e);
}

class GitHub {
    public void request(String endpoint, RequestListener listener) {
        // ...
        // listener.onComplete(json); or listener.onException(e);
    }
}

github.request("yongjhih/rxparse", new RequestListener() {
    @Override public void onComplete(String json) {
        List<Contributor> contributors = Contributors.parse(json);
        // ...
    }
    @Override public void onException(Exception e) {
    }
  });
```

我們可以透過泛型來寫更通用一點介面：

```java
interface SimpleRequestListener<T> {
    void onComplete(List<T> list);
    void onException(Exception e);
}

class GitHub {
    // ...
    public void contributors(String endpoint, SimpleRequestListener<Contributor> listener) {
        request(endpoint, new RequestListener() {
            @Override public void onComplete(String json) {
                listener(Contributors.parse(json));
            }
            @Override public void onException(Exception e) {
                listener(e);
            }
        });
    }

    public void repositories(String endpoint, SimpleRequestListener<Repository> listener) {
        request(endpoint, new RequestListener() {
            @Override public void onComplete(String json) {
                listener(Repositories.parse(json));
            }
            @Override public void onException(Exception e) {
                listener(e);
            }
        });
    }
}

github.contributors("yongjhih/rxparse", new SimpleRequestListener<>() {
    @Override public void onComplete(List<Contributor> contributors) {
        // ...
    }
    @Override public void onException(Exception e) {
    }
  });

github.repositories("yongjhih", new SimpleRequestListener<>() {
    @Override public void onComplete(List<Repository> repositories) {
        // ...
    }
    @Override public void onException(Exception e) {
    }
  });
```

如果有很多 API 要寫，複製貼上的程式碼片段仍然大了一點，所以再設計一個通用的建構方法：

```java
interface Parsable<T> {
    T parse(String);
}

public class Contributor implements Parsable {
    public Contributor() {}
    public List<Contributor>(String json) {
        // ...
        return contributors;
    }
}

class GitHub {
    // ...
    public <T extends Parsable> void request(String endpoint, Class<T> clazz, SimpleRequestListener<T> listener) {
        request(endpoint, new RequestListener() {
            @Override public void onComplete(String json) {
                Parsable parser = clazz.newInstance();
                listener(parser.parse(json));
            }
            @Override public void onException(Exception e) {
                listener(e);
            }
        });
    }

    public void contributors(String endpoint, SimpleRequestListener<Contributor> listener) {
        request(endpoint, Contributor.class, listener);
    }

    public void repositories(String endpoint, SimpleRequestListener<Repository> listener) {
        request(endpoint, Repository.class, listener);
    }
}
```

