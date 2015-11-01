# adb

安裝 apk

```bash
adb install -r ${apk}
```

清除資料

```bash
adb shell pm clear `adb shell pm list packages ${部分名稱}`
```

移除 app

```bash
adb shell pm uninstall `adb shell pm list packages ${部分名稱}`
```

列出 apps

```bash
adb shell pm list packages ${部分名稱}
```

強制停止 app

```bash
adb shell am kill
```