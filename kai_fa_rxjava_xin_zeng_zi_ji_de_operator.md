# 開發 RxJava 新增自己的 Operator


## 倒序 Observable : OperatorToReversedList

```java
Observable.range(1, 10).lift(new OperatorToReversedList()).subscribe(System.out::println);

// [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]
```


## 每隔一段時間 frequency

```
Observable.range(1, 10).lift(new OperatorFrequency(1, TimeUnit.SECONDS)).subscribe(i, System.out.println(i + ": " + System.currentTimemillis()));

// 
```

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

# See Also

* https://github.com/yongjhih/RxJava-GroupByTest/blob/master/src/main/java/rx/internal/operators/OperatorGroupByGroup.java