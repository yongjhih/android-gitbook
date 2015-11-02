# 佈景 Theme

常用的佈景架構：

res/values/theme.xml:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- 主色調 -->
    <color name="colorPrimary">@color/md_teal_400</color>
    <color name="colorPrimaryDark">@color/md_teal_500</color>

    <!-- 應用程式主要佈景決定使用明亮地暗底白字 Theme.AppCompat.Light.DarkActionBar -->
    <style name="Theme.App" parent="{Theme.AppCompat.Light.DarkActionBar}">
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    </style>

    <!-- 常用分頁佈景 -->
    <style name="Theme.App.ToolBar" parent="Theme.App.NoActionBar">
    </style>

    <style name="Theme.App.ToolBar.ActionBarOverlay" parent="Theme.App.NoActionBar">
        <item name="windowActionBarOverlay">true</item>
        <item name="android:windowActionBarOverlay">true</item>
        <item name="android:windowContentOverlay">@null</item>
        <item name="windowActionModeOverlay">true</item>
        <item name="actionModeBackground">@color/colorPrimary</item>
    </style>

    <style name="Widget.App.ToolBar" parent="Theme.App">
        <item name="background">@color/colorPrimary</item>
        <item name="android:background">@color/colorPrimary</item>
        <item name="theme">@style/ThemeOverlay.AppCompat.Dark.ActionBar</item>
    </style>

    <style name="Theme.App.ToolBar.ActionBarOverlay.NoDisplayOptions" parent="Theme.App.ToolBar.ActionBarOverlay">
        <item name="displayOptions"></item>
        <item name="android:displayOptions"></item>
    </style>

    <!-- 如果 actionbar 需要壓過內容 -->
    <style name="Theme.App.ActionBarOverlay" parent="Theme.App">
        <item name="windowActionBarOverlay">true</item>
        <item name="android:windowActionBarOverlay">true</item>
        <item name="android:windowContentOverlay">@null</item>
    </style>

    <!-- 如果 actionbar 需要壓過內容且自訂 navigation bar -->
    <style name="Theme.App.ActionBarOverlay.FullScreen" parent="Theme.App.ActionBarOverlay">
        <item name="android:windowFullscreen">true</item>
        <item name="android:windowContentOverlay">@null</item>
    </style>

    <!-- 全螢幕無系統列 -->
    <style name="Theme.App.NoActionBar.FullScreen" parent="Theme.App.NoActionBar">
        <item name="android:windowFullscreen">true</item>
        <item name="android:windowContentOverlay">@null</item>

        <item name="android:windowBackground">@color/colorPrimary</item>
        <item name="android:colorBackground">@color/colorPrimary</item>
    </style>

    <style name="Theme.App.NoWindowContentOverlay" parent="Theme.App">
        <item name="android:windowContentOverlay">@null</item>
    </style>

    <style name="Theme.App.NoActionBar" parent="Theme.App"> <!-- Copy from Theme.AppCompat.NoActionBar -->
        <item name="android:windowNoTitle">true</item>
        <item name="windowActionBar">false</item>
    </style>
</resources>
```
