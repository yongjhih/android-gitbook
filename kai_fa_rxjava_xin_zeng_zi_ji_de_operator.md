# 開發 RxJava 新增自己的 Operator

## 倒序 Observable : OperatorToReversedList

Before:

```java
Observable.range(1, 10).toList().doOnNext(list -> Collections.reverse(list))
    .subscribe(System.out::println);
// [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]
```

After:

```java
Observable.range(1, 10).lift(new OperatorToReversedList())
    .subscribe(System.out::println);
```

OperatorToReversedList.java

```java
public final class OperatorToReversedList<T> implements Operator<List<T>, T> { // 進貨 <T> 操作員處理後出貨 <R extends List<T>>
    @Override
    public Subscriber<? super T> call(final Subscriber<? super List<T>> o) {
        return new Subscriber<T>(o) { // 回傳承辦窗口
 
            final List<T> list = new ArrayList<T>();
 
            @Override
            public void onStart() { // 開始運作
                request(Long.MAX_VALUE);
            }
 
            @Override
            public void onCompleted() { // 上一站結束
                try {
                    Collections.reverse(list);
 
                    o.onNext(list);
                    o.onCompleted();
                } catch (Throwable e) {
                    onError(e);
                }
            }
 
            @Override
            public void onError(Throwable e) { // 上一站出狀況
                o.onError(e);
            }
 
            @Override
            public void onNext(T value) { // 傳遞給下一站
                list.add(value);
            }
 
        };
    }
}
```

https://gist.github.com/yongjhih/20ccfab5007ea6bc9f0d

實現一個 Operator (操作員) 主要需要回傳一個 Subscriber (承辦窗口) 讓其他人可以塞資料給你，然後等你處理完吐資料出去。

```java
// T 是進來的型別
// R 是出去的型別
public class OperatorFrequency<T> implements Operator<R, T> {
    @Override
    public Subscriber<? super T> call(Subscriber<? super R> child) { ... } // child 下一站的窗口
```

```java
// Operator 是一個簡單的介面，只要求回傳承辦口以及告知下一站窗口型別
public interface Operator<R, T> extends Func1<Subscriber<? super R>, Subscriber<? super T>> { ... }
```

```java
// Subscriber
public abstract class Subscriber<T> implements Observer<T>, Subscription {
    public abstract void onStart();

    // public interface Observer<T> {
    public abstract void onCompleted();
    public abstract void onError(Throwable e);
    public abstract void onNext(T t);
    // }
}
```
    


## OperatorFrequency 每隔一段時間

```java
Observable.range(1, 10).lift(new OperatorFrequency(1, TimeUnit.SECONDS))
    .subscribe(i -> System.out.println(i + ": " + System.currentTimeMillis()).subscribe());

// 1: 1428053481338
// 2: 1428053482339
// 3: 1428053483338
// 4: 1428053474339
// 5: 1428053475338
// 6: 1428053476338
// 7: 1428053477338
// 8: 1428053478338
// 9: 1428053479338
// 10: 1428053480338
```

OperatorFrequency.java

```java
public class OperatorFrequency<T> implements Operator<T, T> {
    private long interval;
    private TimeUnit unit;
 
    public OperatorFrequency(long interval, TimeUnit unit) {
        this.interval = interval;
        this.unit = unit;
    }
 
    @Override
    public Subscriber<? super T> call(final Subscriber<? super T> child) {
        return new FrequencySubscriber<>(interval, unit, child);
    }
 
    static class FrequencySubscriber<T> extends Subscriber<T> {
        private long interval;
        private TimeUnit unit;
        private final Subscriber<? super T> child;
        private final Observable<Long> tick;
        private PublishSubject stop = PublishSubject.create();
        private Subject<T, T> subject;
        private Observable<T> zip;
        private Subscription subscription;
        private long zipCount = 0;
 
        public FrequencySubscriber(long interval, TimeUnit unit, final Subscriber<? super T> child) {
            super();
 
            this.interval = interval;
            this.unit = unit;
            this.child = child;
 
            tick = Observable.interval(interval, unit).map(l -> zipCount).distinct().onBackpressureBuffer(1);
            subject = PublishSubject.create();
            zip = Observable.zip(subject.asObservable().onBackpressureBuffer(1024), tick,
                    (emit, t) -> {
                zipCount++;
                return emit;
            });
        }
 
        @Override
        public void onStart() {
            if (subscription == null) {
                subscription = zip.subscribe(child);
            }
        }
 
        @Override
        public void onError(Throwable e) {
            try {
                child.onError(e);
            } finally {
                unsubscribe();
            }
        }
 
        @Override
        public void onCompleted() {
            subject.onCompleted();
        }
 
        @Override
        public void onNext(T t) {
            subject.onNext(t);
        }
    }
}
```

https://gist.github.com/yongjhih/ba24c9b3d025333ede17

# See Also

* https://github.com/yongjhih/RxJava-Operators/blob/master/src/main/java/com/github/yongjhih/Main.java
* https://github.com/yongjhih/RxJava-GroupByTest/blob/master/src/main/java/rx/internal/operators/OperatorGroupByGroup.java