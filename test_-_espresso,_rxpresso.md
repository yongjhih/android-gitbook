# Test - UI

UI 測試框架：

* Robotium
* uiautomator
* espresso
* Appium
* Calabash

跨 iOS 使用 Appium 或 Calabash

## RxPresso

```java
rxPresso.given(mockedRepo.getUser("id"))
           .withEventsFrom(Observable.just(new User("some name")))
           .expect(any(User.class))
           .thenOnView(withText("some name"))
           .perform(click());
```

## See Also

* https://github.com/novoda/rxpresso
