# Clean architecture

先將程式碼擺放包裝劃分清楚再來針對動線做規劃。

Clean architecture 主要探討的是系統分層以及互動模型

常見的互動模型：

* MVC
* MVP (P, Presenter)
* MVVM (VM, ViewModel)
* Flux

<!-- ![MVP](http://upload.wikimedia.org/wikipedia/commons/a/a0/MVC-Process.svg)

![MVVM](http://upload.wikimedia.org/wikipedia/commons/8/87/MVVMPattern.png)
-->

在 Android 開發上，常見的是 MVP 與 MVVM 一般開發上，並不一定都只有一套互動模型在運行。

MVP 其實大家應該十分熟悉，只是沒有意識到而已。那就是 ListView/RecyclerView 就有用到了。

我們知道 MVP 三層，所以只要介紹中間那層 P 會比較直接，這邊稍微簡化舉例：

```java
class UserCardPresenter extends Presenter<User> {
    public ViewHolder onCreateViewHolder(ViewGroup parent) {
        return new UserCardViewHolder(parent, R.layout.item_icon);
    }

    public void onBindViewHolder(/* View */UserCardViewHolder userCardView, /* Model */ User user) {
        // userCardView.textView1.setText(item.name);
    }
}
```

MVVM 基本上，可當作是 MVP 的子集合，注重雙向連動，以利個別測試，你可以虛擬化其中一方做測試。


## ogaclejapan/RxBinding MVVM

利用 RxJava 實現了 MVVM 的 two-way binding (雙向連動)

## Data-Binding support v22

讓連動更簡便撰寫 VM 應該就更薄了。

## Flux

![](https://facebook.github.io/flux/img/flux-simple-f8-diagram-with-client-action-1300w.png)

不直接雙向回 model/store ，透過一個 dispatcher 來做管理。

## See Also

* http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/
* http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/
* http://manuel.kiessling.net/2012/09/28/applying-the-clean-architecture-to-go-applications/
* https://www.groupbuddies.com/posts/20-clean-architecture
* http://konmik.github.io/introduction-to-model-view-presenter-on-android.html
* https://medium.com/mobiwise-blog/android-basic-project-architecture-for-mvp-72f4b33252d0
* http://armueller.github.io/android/2015/03/29/flux-and-android.html
* http://lgvalle.xyz/2015/08/04/flux-architecture/

* https://github.com/skimarxall/RxFlux

Sample:

* https://github.com/android10/Android-CleanArchitecture
* https://github.com/sockeqwe/mosby
* https://github.com/JorgeCastilloPrz/Dagger2Scopes
* https://github.com/PaNaVTEC/Clean-Contacts
* https://github.com/techery/presenta
* https://github.com/antoniolg/androidmvp
* https://github.com/wongcain/MVP-Simple-Demo
* https://github.com/pedrovgs/EffectiveAndroidUI
* https://github.com/glomadrian/MvpCleanArchitecture
* https://github.com/spengilley/ActivityFragmentMVP
* https://github.com/jlmd/UpcomingMoviesMVP
* https://github.com/JorgeCastilloPrz/EasyMVP
* https://github.com/richardradics/MVPAndroidBootstrap
* https://github.com/richardradics/RxAndroidBootstrap
* https://github.com/inloop/AndroidViewModel
* https://github.com/lgvalle/android-flux-todo-app
