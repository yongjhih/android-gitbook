# Aspect

\#AspectJ \#AOP

AOP 面向導向設計。筆者認為也是一種 Meta Programming 的範疇。

我們還是以範例來作說明。在 Android 比較知名的代表作是 [JackWharton/hugo](https://github.com/JakeWharton/hugo) 。

```java
@DebugLog
public String getName(String first, String last) {
  SystemClock.sleep(15);
  return first + " " + last;
}
```

```java
getName("Andrew", "Chen");
```

```
V/Example: ⇢ getName(first="Andrew", last="Chen")
V/Example: ⇠ getName [16ms] = "Andrew Chen"
```

```java
@Aspect
public class Hugo {
  @Pointcut("within(@hugo.weaving.DebugLog *)") // 產生一個叫 withinAnnotatedClass() 的攔截點，專門攔截有掛 @hugo.weaving.DebugLog 的標記
  public void withinAnnotatedClass() {}

  @Pointcut("execution(* *(..)) && withinAnnotatedClass()")
  public void methodInsideAnnotatedType() {} // 攔方法參數型別

  @Pointcut("execution(*.new(..)) && withinAnnotatedClass()")
  public void constructorInsideAnnotatedType() {} // 攔建構子參數型別

  @Pointcut("execution(@hugo.weaving.DebugLog * *(..)) || methodInsideAnnotatedType()")
  public void method() {} // 攔方法

  @Pointcut("execution(@hugo.weaving.DebugLog *.new(..)) || constructorInsideAnnotatedType()")
  public void constructor() {} // 攔建構子

  @Around("method() || constructor()") // 如果欄到就
  public Object logAndExecute(ProceedingJoinPoint joinPoint) throws Throwable {
    long startNanos = System.nanoTime();
    Object result = joinPoint.proceed(); // 代為執行攔截區間
    long stopNanos = System.nanoTime();
    // 算一下時間、印一下。
  }
}
```

## See Also

* https://github.com/JakeWharton/hugo