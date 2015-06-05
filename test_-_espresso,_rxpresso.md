# Test - espresso, rxpresso

Ui 操作測試函式庫

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