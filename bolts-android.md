# Bolts-Android

`Task<T>` 相當於`Observable<T>`

`Bolts.Task.continueWith()` 相當於 `Observable.map()`

```java
Task<ParseObject> saveAsync;
...
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

`Task.callInBackground()` 相當於 `Observable.defer()`

```
Task.callInBackground(new Callable<Void>() {
  public Void call() {
    // Do a bunch of stuff.
  }
}).continueWith(...);
```

## See Also

* https://github.com/BoltsFramework/Bolts-Android