# 開發 RxJava 新增自己的 Operator

## 倒序 Observable : OperatorToReversedList

```java
Observable.range(1, 10).lift(new OperatorToReversedList())
    .subscribe(System.out::println);

// [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]
```

OperatorToReversedList.java

```java
public final class OperatorToReversedList<T> implements Operator<List<T>, T> {
    @SuppressWarnings("unchecked")
    public OperatorToReversedList() {
    }
 
    @Override
    public Subscriber<? super T> call(final Subscriber<? super List<T>> o) {
        return new Subscriber<T>(o) {
 
            final List<T> list = new ArrayList<T>();
 
            @Override
            public void onStart() {
                request(Long.MAX_VALUE);
            }
 
            @Override
            public void onCompleted() {
                try {
                    Collections.reverse(list);
 
                    o.onNext(list);
                    o.onCompleted();
                } catch (Throwable e) {
                    onError(e);
                }
            }
 
            @Override
            public void onError(Throwable e) {
                o.onError(e);
            }
 
            @Override
            public void onNext(T value) {
                list.add(value);
            }
 
        };
    }
}
```

https://gist.github.com/yongjhih/20ccfab5007ea6bc9f0d


## 每隔一段時間 frequency

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

* https://github.com/yongjhih/RxJava-GroupByTest/blob/master/src/main/java/rx/internal/operators/OperatorGroupByGroup.java