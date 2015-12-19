# Android 實務

可以吃嗎？

這邊主要抽一些實務上常見的建議與指南。

## 不要用 Thread 或 HandlerThread 推薦使用 AsyncTask

如果你要背景下載檔案，你應該使用 IntentService 而不是自己去操作 Thread。

如果是這個 Activity 內的長時間存取，可以使用 AsyncTask 。如有必要請搭配 LoaderManager 使用。

## 不要用 Handle Message 推薦使用 `post(Runnable)`

通常你需要的是 `post(Runnable)`

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

## 不再 findViewBindId 推薦使用 ButterKnife

After:

```java
@BindView(R.id.username)
public TextView usernameView;


// ...

@Override public void onCreate(Bundle savedInstanceState) {
   // ...
   ButterKnife.bind(this);
}
```

Before:

```java
public TextView usernameView;

@Override public void onCreate(Bundle savedInstanceState) {
   // ...
   usernameView = (TextView) findViewById(R.id.username);
}
```

## 使用 @Nullable/@NonNull 標注

以及各種 support-annotations

...


