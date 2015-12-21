# Android 實務

可以吃嗎？

這邊主要抽一些實務上常見的建議與指南。

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
List<String> getNames() {
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

## 該傳遞 Context 就不要傳遞 Activity

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
