# Activity 與 Fragment 的日常

相當於一篇網頁。

依據使用者的操作頁面的操作，定義了一些狀態，稱之 Activity 的生命週期。

Activity 生命週期：

```
|-onCreate()
||-onStart()
|||-onResume()
|||-onPause()
||-onStop()
|-onDestroy()
```

它們是對稱的存在。

```
|-onCreate() 第一次建立
||-onStart() 如果透過超連結回來的，從這開始
|||-onResume() 在最上層顯示的時候
|||-onPause() 被蓋在下層的時候
||-onStop() 被閒置很久的時候
|-onDestroy() 被關掉的時候
```

網路上有很多生命週期圖可以看，這裡就不附了。

`onCreate()` 慣例是請你準備好/初始化的東西，像是 `setContentView(@Layout int layoutId)` ，沒做還會爆炸，這點在後來才提供的 Fragment 類別有簡單的防呆 `@NonNull View onCreateView();`


