# Bolts-Android

如果你已經在用 RxJava ，那這應該僅供參考。

Bolts 是一款 promise 的實現。由 Parse.com 主持。Facebook 在收購 Parse.com 後，也積極整合 bolts 。

`Bolts.Task<T>` 相當於`Observable<T>`

`Bolts.Task.continueWith()` 相當於 `Observable.map()`

```java
Task<ParseObject> saveAsync;
...
// .map(o -> null);
saveAsync(obj).continueWith(new Continuation<ParseObject, Void>() {
  public Void then(Task<ParseObject> task) throws Exception {
    if (task.isCancelled()) {
      // 取消
    } else if (task.isFaulted()) {
      // 失敗
      Exception error = task.getError();
    } else {
      // 成功
      ParseObject object = task.getResult();
    }
    return null;
  }
});
```

`Bolts.Task.continueWithTask()` 相當於 `Observable.flatMap()`

```java
// .flatMap(o -> saveObs(o)).map(o -> null);
query.findInBackground().continueWithTask(new Continuation<List<ParseObject>, Task<ParseObject>>() {
  public Task<ParseObject> then(Task<List<ParseObject>> task) throws Exception {
    if (task.isFaulted()) {
      return null;
    }

    List<ParseObject> students = task.getResult();
    students.get(1).put("salutatorian", true);
    return saveAsync(students.get(1));
  }
}).onSuccess(new Continuation<ParseObject, Void>() {
  public Void then(Task<ParseObject> task) throws Exception {
    return null;
  }
});
```

`Task.forResult()` 相當於 `Observable.just()`

```java
Task<String> successful = Task.forResult("The good result.");
```

`Task.create()` 相當於 `Observable.create()`

```java
public Task<ParseObject> fetchAsync(ParseObject obj) {
  final Task<ParseObject>.TaskCompletionSource tcs = Task.create();
  obj.fetchInBackground(new GetCallback() {
    public void done(ParseObject object, ParseException e) {
     if (e == null) {
       tcs.setResult(object);
     } else {
       tcs.setError(e);
     }
   }
  });
  return tcs.getTask();
}
```

`Task.callInBackground()` 相當於 `Observable.defer()`:

```java
 Task<Void> Task.callInBackground(new Callable<Void>() {
  public Void call() {
    // Do a bunch of stuff.
  }
});
```

`Task.waitForCompletion()` 相當於 `Observable.toBlocking()`:

Tasks.java:

```java
public static <T> T wait(Task<T> task) {
  try {
    task.waitForCompletion();
    if (task.isFaulted()) {
      Exception error = task.getError();
      if (error instanceof RuntimeException) {
        throw (RuntimeException) error;
      }
      throw new RuntimeException(error);
    } else if (task.isCancelled()) {
      throw new RuntimeException(new CancellationException());
    }
    return task.getResult();
  } catch (InterruptedException e) {
    throw new RuntimeException(e);
  }
}
```

## RxBolts

ref. [yongjhih/RxBolts/.../TaskObservable.java](https://github.com/yongjhih/RxBolts/blob/master/rxbolts/src/main/java/rx/bolts/TaskObservable.java#L36)

```java
    public static <R> Observable<R> just(Task<R> task) {
        return Observable.create(sub -> {
            task.continueWith(t -> {
                if (t.isCancelled()) {
                    sub.unsubscribe(); //sub.onCompleted();?
                } else if (t.isFaulted()) {
                    sub.onError(t.getError());
                } else {
                    R r = t.getResult();
                    if (r != null) sub.onNext(r);
                    sub.onCompleted();
                }
                return null;
            });
        });
    }
```

## See Also

* https://github.com/BoltsFramework/Bolts-Android
