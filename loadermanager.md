# LoaderManager

* Cursor Leak: 關於 cursor 關閉問題，LoaderManader 就是在管理 lifeCycle 的，cursorLoader 就會被 loaderManager 通知來關閉 cursor。
* ContentProvider: 包裝成 contentProvider 的好處在於有通用的 ContentUri 存取點(endpoint) 可用，跨 app 存取能力。所以可衡量實現的必要性。
* LoaderManager + Cursor: 如果資料來源本來用就是 SQLiteDatabase ，就算沒有包裝成 ContentProvider 也勢必也有 Cursor 生命管理業務，這時可利用 LoaderManager 來管理。
* Cursor without contentUri: CursorLoader 需要 ContentUri endpoint，所以在沒有實現 ContentProvider 的狀況下，可以利用 AsyncTaskLoader 來包裝 Cursor，網路上很多人寫了 SimpleCursorLoader 就是基於 AsyncTaskLoader 實現的。

BTW, rxer 如果通用 loaderManager 有個 rxloader 可用： https://github.com/evant/rxloader
