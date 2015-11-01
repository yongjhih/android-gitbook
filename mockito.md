# Mockito

虛擬物件

用來驗證是否如預期的互動行為，例如:

```java
void add(List<String> list, String item) {
   list.add(item);
}

void testAddList() {
    List<String> mockedList = mock(List.class);
    add(mockedList, "one");
    verify(mockedList).add("one");
}
```

也可以偽裝行為，例如：

```java
LinkedList mockedList = mock(LinkedList.class);

// 如果有人呼叫 get(0) ，就回傳 "first"
when(mockedList.get(0)).thenReturn("first");

assertEquals("first", mockedList.get(0));
```


## RxParse 測項範例

測試 `ParseObservable.all(ParseQuery).subscribe(onNext, onError, onCompleted);` ，不應該先呼叫 onCompleted 才呼叫 onNext。

```java
  /**
   * <code>ParseObservable.all(ParseQuery).subscribe(onNext, onError, onCompleted);</code> should've onNext after onCompleted.
   */
  @Test
  public void testParseObservableAllNextAfterCompleted() {
    // 建立三個偽裝 ParseUser
    ParseUser user = mock(ParseUser.class);
    ParseUser user2 = mock(ParseUser.class);
    ParseUser user3 = mock(ParseUser.class);
    List<ParseUser> users = new ArrayList<>();
    when(user.getObjectId()).thenReturn("" + user.hashCode()); // 偽裝呼叫 user.getObjectId() 時，回傳 hashCode 字串.
    users.add(user);
    when(user2.getObjectId()).thenReturn("" + user2.hashCode());
    users.add(user2);
    when(user3.getObjectId()).thenReturn("" + user3.hashCode());
    users.add(user3);
    // 偽裝 ParseQueryController ，讓 ParseQuery.findInBackground(), ParseQuery.countInBackground 使用到偽裝的 ParseQueryController
    ParseQueryController queryController = mock(ParseQueryController.class);
    ParseCorePlugins.getInstance().registerQueryController(queryController);

    Task<List<ParseUser>> task = Task.forResult(users);
    // 偽裝呼叫 findAsync(ParseQuery.State, ParseUser, Task) 時，回傳已經裝好三個使用者的 task
    when(queryController.findAsync(
            any(ParseQuery.State.class),
            any(ParseUser.class),
            any(Task.class))
    ).thenReturn(task);

    // 偽裝呼叫 countAsync(ParseQuery.State, ParseUser, Task) 時，回傳使用者個數: 3
    when(queryController.countAsync(
      any(ParseQuery.State.class),
      any(ParseUser.class),
      any(Task.class))).thenReturn(Task.<Integer>forResult(users.size()));

    ParseQuery<ParseUser> query = ParseQuery.getQuery(ParseUser.class);
    query.setUser(new ParseUser());

    final AtomicBoolean completed = new AtomicBoolean(false);
    rx.parse.ParseObservable.all(query)
        //.observeOn(Schedulers.newThread())
        //.subscribeOn(AndroidSchedulers.mainThread())
        .subscribe(new Action1<ParseObject>() {
        @Override public void call(ParseObject it) {
            System.out.println("onNext: " + it.getObjectId());
            if (completed.get()) { // 竟然發現 onCompleted 已經被呼叫了
                fail("Should've onNext after onCompleted.");
            }
        }
    }, new Action1<Throwable>() {
        @Override public void call(Throwable e) {
            System.out.println("onError: " + e);
        }
    }, new Action0() {
        @Override public void call() {
            System.out.println("onCompleted");
            completed.set(true);
        }
    });

    try {
        ParseTaskUtils.wait(task); // 因為是非同步執行，需要等待 task 把結果送出，確保 callbacks 跑完。
    } catch (Exception e) {
        // do nothing
    }
  }
```

## 安裝

```gradle
dependencies { testCompile "org.mockito:mockito-core:1.+" }
```

## See Also

* Mockito - https://github.com/mockito/mockito http://mockito.org/
* EasyMock - https://github.com/easymock/easymock http://easymock.org/
* PowerMock - https://github.com/jayway/powermock
