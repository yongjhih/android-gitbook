# 安裝精靈

最近剛好有人問到，setup wizard 安裝精靈 沒有被呼叫起來的問題。主要以前是配置出廠預設配置，device/product/ ，
印象比較深刻的是 device_provisioned ，這個在 Android 系統隱私資料庫內，也有記載，主要得知是否已經配置過 provisioned 。

一種是正向追 code ：

```sh
sgrep -i device_provisioned
```

另一種，逆向追 code 是誰負責呼叫你的？安插在系統的 broadcaster 身上，印出 setup wizard caller 是誰。這個比較黑手一點，當作最終手段吧。

再來另一個線索是，setup wizard 的 manifest ，因為系統還是依據 manifest 某個特殊條件才會呼叫的：

線索一：

```xml
<meta-data android:name="android.SETUP_VERSION" android:value="eclair_1" />
```

線索二：

```xml
<action android:name="android.intent.action.DEVICE_INITIALIZATION_WIZARD" />
```

所以我們還可以 `sgrep -i SETUP_VERSION` 以及 `sgrep -i DEVICE_INITIALIZATION_WIZARD`，最終，我們可以找到 ActivityManagerService.java

```java
    void startSetupActivityLocked() {
        // ...

        // We will show this screen if the current one is a different
        // version than the last one shown, and we are not running in
        // low-level factory test mode.
        final ContentResolver resolver = mContext.getContentResolver();
        if (mFactoryTest != SystemServer.FACTORY_TEST_LOW_LEVEL &&
                Settings.Secure.getInt(resolver,
                        Settings.Secure.DEVICE_PROVISIONED, 0) != 0) {

        // ...

            ResolveInfo ri = null;
            for (int i=0; ris != null && i<ris.size(); i++) {
                if ((ris.get(i).activityInfo.applicationInfo.flags
                        & ApplicationInfo.FLAG_SYSTEM) != 0) {
                    ri = ris.get(i);
                    break;
                }
            }

        // ..

                String vers = ri.activityInfo.metaData != null 
                        ? ri.activityInfo.metaData.getString(Intent.METADATA_SETUP_VERSION) 
                        : null; 
                if (vers == null && ri.activityInfo.applicationInfo.metaData != null) { 
                    vers = ri.activityInfo.applicationInfo.metaData.getString( 
                            Intent.METADATA_SETUP_VERSION); 
                } 
                String lastVers = Settings.Secure.getString( 
                        resolver, Settings.Secure.LAST_SETUP_SHOWN); 
```

不是工廠低階測試以及還沒跑過安裝精靈了話，而且是系統權限的應用程式，而且 SETUP_VERSION 的版本還沒跑過，就顯示安裝精靈。
對了，如果很多應用程式符合，看得出來它這裡挑第一個符合的應用程式，所以我們可以試著把你的目標安裝精靈的優先度調高 priority 吧。
