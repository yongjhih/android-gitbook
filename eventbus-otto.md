# EventBus: greenrobot/EventBus, Square/otto

通常用來處理跨 Activity, View, Thread 等溝通問題。

1. 通常遇到這種問題，一種是建立溝通管道，例如傳遞 callback 的方式。
2. 利用廣播方式，intent broadcast
3. 懶得傳遞 callback 就依靠全域變數註冊 callback.

greenrobot/EventBus 就是走第三種方案。

*我們都知道全域東西都有毒*

## greenrobot/EventBus vs. Otto

網路上有很多比較，可以參考。

這裡先給個簡單的參考： EventBus 比較快、 Otto 比較小。

Otto 由知名開發商 Square 所維護，可靠度較高

*如果已經引進 RxJava 應該使用 RxJava 的 Subject 取代, RxRelay*

## 補充 - 回復性、耦合性

儲存問題, 可回復性. 如果透過 saveInstanceState 就可以交由系統負責儲存。一種是儲存在 disk 如: sharedPreferences. 再來是 DB 。

也就是 http://endlesswhileloop.com/blog/2015/06/11/stop-using-event-buses/ 提及的：

> * Requires the data as a dependency.
> * Handle the state where the data is not available.

不過 http://endlesswhileloop.com/blog/2015/06/11/stop-using-event-buses/ 標題過激，事實上，只要不要濫用，能夠正常建立 callback 溝通管道的就建立，不得已我們再來考慮 EventBus 。

# See Also

有 memory leak 的問題:

* [#57 Weak reference to the subscriber](https://github.com/greenrobot/EventBus/issues/57), 解法: [yongjhih/EventBus/commit/3d3c1ca6](https://github.com/yongjhih/EventBus/commit/3d3c1ca6676112bd9dd6bb78245b03b31e5c25fc)

*Otto 名字的由來自辛普森家庭卡通中，校車的司機(bus driver) 就叫做 Otto*

* https://github.com/greenrobot/EventBus
* https://github.com/square/otto
* http://endlesswhileloop.com/blog/2015/06/11/stop-using-event-buses/
