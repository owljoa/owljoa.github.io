---
title: "자바 CompletableFuture - 내부 코드 구경"
excerpt: "CompletableFuture 내부 코드 구경"
categories:
  - java
tags:
  - 비동기
toc: true
---


# 1. 개요

오라클 문서에 있는 CompletableFuture 관련 내용들을 직접 확인해보기 위해 내부 구현을 보면서 정리한 내용입니다.

“예제코드”라고 지칭하는 것은 이전 포스트([CompetableFuture 개념 정리](../completable-future))에서 사용한 예제코드입니다. ([예제코드 링크](https://github.com/owljoa/java-study/tree/main/completable-future))

<br><br>

# 2. 내부 구현 관찰

## 2.1. CompletionStage 인터페이스 구현

CompletableFuture에서 CompletionStage 인터페이스 구현 시 준수한 정책

<br>

### 2.1.1. 비동기 작업 중 메소드간 의존이 있는 케이스 처리

다른 메소드에 의존하는 메소드의 완료를 위해 제공된 기능들은 현재의 CompetableFuture를 완료하는 쓰레드나 완료 메소드의 다른 호출자에 의해 수행될 수 있다.

예제코드 실행 로그를 보면 `commonPool-worker-5` 쓰레드가 진행중인 CompletableFuture가 끝나야 수행할 수 있는 결제취소 후 주문의 결제상태 변경 메소드가 `commonPool-worker-5` 쓰레드에 의해 수행됨을 확인할 수 있다.
    
```log
[2022-11-19T15:17:24.913426][ForkJoinPool.commonPool-worker-5] 결제취소 요청 발송
...
[2022-11-19T15:17:26.732630][ForkJoinPool.commonPool-worker-5] 결제취소 응답 수신
[2022-11-19T15:17:26.733277][ForkJoinPool.commonPool-worker-5] 결제취소 후 주문의 결제상태 변경 시작
...
[2022-11-19T15:17:27.035966][ForkJoinPool.commonPool-worker-5] 결제취소 후 주문의 결제상태 변경 완료
```

<br>

### 2.1.2. 디폴트로 사용하는 쓰레드풀

명시된 Executor 인자가 없는 모든 비동기 메소드들은 ForkJoinPool을 이용해서 수행된다.

2 이상의 병렬 처리 수준(parallelism level)을 지원하지 않으면, 각각의 작업을 수행하기위한 새로운 Thread가 생성된다.

> 병렬 처리 수준(parallelism level): ForkJoinPool이 사용할 작업 쓰레드 갯수
    >> CompletableFuture의 디폴트 쓰레드풀은 `ForkJoinPool.commonPool` (디폴트 병렬화 수준으로 설정됨)<br>디폴트값: PC의 CPU 코어 수

내부 코드를 보면 디폴트 쓰레드풀로 `ForkJoinPool.commonPool`을 사용한다는 것을 확인할 수 있다.
    
```java
// CompletableFuture.java
...
/**
    * Default executor -- ForkJoinPool.commonPool() unless it cannot
    * support parallelism.
    */
private static final Executor ASYNC_POOL = USE_COMMON_POOL ?
        ForkJoinPool.commonPool() : new ThreadPerTaskExecutor();
...
```

예제코드 실행 로그에서도 디폴트 쓰레드풀(ForkJoinPool.commonPool)을 확인할 수 있다.
    
```log
...
[2022-11-19T15:42:04.100389][ForkJoinPool.commonPool-worker-5] 결제취소 요청 발송
...
[2022-11-19T15:42:04.100557][ForkJoinPool.commonPool-worker-19] 배송취소 요청 발송
...
```
    

<br>

### 2.1.3. 비동기 작업 타입 통일

모든 비동기 작업들은 모니터링, 디버깅, 트래킹을 단순화하기 위해 마커 인터페이스(`CompletableFuture.AsynchronousCompletionTask`)의 인스턴스로 생성된다.

> 마커 인터페이스
    >> 아무 메소드도 선언하지 않은 타입 체크 목적의 인터페이스
        >>> ex) Serializable, Clonable, EventListener

<br>

반환값 있는 CompletableFuture 생성을 위해 사용하는 `supplyAsync` 메소드 `AsyncSupply`의 인스턴스를 생성해서 작업을 수행하는데, `AsyncSupply` 클래스를 찾아보면 `AsynchronousCompletionTask` 를 구현함을 알 수 있다.
    
```java
// CompletableFuture.java
...
public static interface AsynchronousCompletionTask {
}
...

public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
    return asyncSupplyStage(ASYNC_POOL, supplier);
}

...
static <U> CompletableFuture<U> asyncSupplyStage(Executor e,
                                                    Supplier<U> f) {
    if (f == null) throw new NullPointerException();
    CompletableFuture<U> d = new CompletableFuture<U>();
    e.execute(new AsyncSupply<U>(d, f)); // <-- AsyncSupply 인스턴스 생성
    return d;
}

...

static final class AsyncSupply<T> extends ForkJoinTask<Void>
        implements Runnable, AsynchronousCompletionTask { // <-- AsynchronousCompletionTask를 구현함
    ...
}
```
    
<br>

### 2.1.4. 독립적인 내부 구현

모든 `CompletionStage` 메소드들은 한 메소드의 행위가 오버라이드된 메소드에 의해 영향받지 않도록 다른 public 메소드들과는 독립적으로 구현된다.

구현된 내부 메소드는 다시 private 메소드를 호출하는 방식으로, 구현된 메소드가 public 메소드를 호출하면 하위 클래스에서 해당 메소드 재정의 시 영향받을 수 있음

[예시]
- public A 메소드 내에서 public B 메소드를 호출하는 상위클래스
- 하위클래스에서 A, B 메소드 재정의
- 하위클래스에서 재정의된 A 메소드 호출 시 재정의된 B 메소드가 호출됨
<br>→ 상위클래스에 정의된 A 메소드 기능과 다르게 동작할 수 있음

<br>

thenApply 메소드를 살펴보면 내부 로직을 담고 있는 uniApplyStage 메소드가 private 접근 제어자와 함께 정의되어있음을을 확인할 수 있다.

```java
// CompletableFuture.java
...
public <U> CompletableFuture<U> thenApply(
    Function<? super T,? extends U> fn) {
    return uniApplyStage(null, fn);
}
...

// 실제 로직은 private 메소드로 구현되어있음
private <V> CompletableFuture<V> uniApplyStage(
    ...
}
...
```
    

<br>

## 2.2. Future 인터페이스 구현

CompletableFuture에서 Future 인터페이스 구현 시 준수한 정책

<br>

### 2.2.1. cancel 메소드

FutureTask와는 다르게 CompletableFuture에는 직접 작업을 완료(completion)시키는 기능은 없기 때문에, 취소(cancellation)는 예외 상황에서의 완료(exceptional completion)로 취급된다.
> `cancel` 메소드 호출은 `completeExceptionally(new CancellationException())` 형태로 호출하는 것과 같다.

isCompletedExceptionally() 메소드는 CompletedFuture가 예외 상황에서 완료되었는지 체크한다.

내부 구현을 보면, `cancel` 메소드의 로직이 `CancellationException`을 인자로 넣어 `completableExceptionally` 메소드를 호출하는 것과 같고, `AltResult` 구조에 예외를 포함하여 결과값으로 설정하는 것을 알 수 있다.
> AltResult: 작업 결과값으로 null이나 excpetion을 담기 위한 구조

```java
// CompletableFuture.java
...
public boolean completeExceptionally(Throwable ex) {
    if (ex == null) throw new NullPointerException();
    boolean triggered = internalComplete(new AltResult(ex)); // <-- CancellationException을 파라미터인 ex로 입력하면 cancel 메소드와 동일
    postComplete();
    return triggered;
}
...
public boolean cancel(boolean mayInterruptIfRunning) {
    boolean cancelled = (result == null) &&
        internalComplete(new AltResult(new CancellationException()));
    postComplete();
    return cancelled || isCancelled();
}
...
final boolean internalComplete(Object r) { // CAS from null to r
    return RESULT.compareAndSet(this, null, r);
}

```

### 2.2.2. 예외상황에서 완료된 경우 결과값 조회 시 처리

비동기 작업 중 예외가 발생하여 완료 처리된 경우, `get()`, `get(long, TimeUnit)` 메소드는 `ExecutionException`을 발생시킨다.
- 예외상황에서의 완료 시 해당 작업의 결과값(result)으로 `CompletionException`이 저장되고 get 메소드 수행 시 저장된 값이 예외임을 감지해서 cause필드에 CompletionException의 cause를 넣고 `ExecutionException`을 발생시킨다.
- 내부 구현을 보면, `get` 메소드에서 호출하는 `reportGet` 메소드를 보면, 작업 결과값이 AltResult 인스턴스이면 AltResult가 포함한 값이 null이면 그대로 반환하고, CancellationException은 그대로 예외를 발생시키고, `CompletionException`이면 `ExecutionException`을 발생시킴을 알 수 있다.

```java
...
public T get() throws InterruptedException, ExecutionException {
    Object r;
    if ((r = result) == null)
        r = waitingGet(true);
    return (T) reportGet(r);
}
...
private static Object reportGet(Object r)
    throws InterruptedException, ExecutionException {
    if (r == null) // by convention below, null means interrupted
        throw new InterruptedException();
    if (r instanceof AltResult) {
        Throwable x, cause;
        if ((x = ((AltResult)r).ex) == null)
            return null;
        if (x instanceof CancellationException)
            throw (CancellationException)x;
        if ((x instanceof CompletionException) &&
            (cause = x.getCause()) != null)
            x = cause;
        throw new ExecutionException(x); // CompletionException 인 경우 cause를 복사하고 ExecutionException을 발생시킴
    }
    return r;
}
...
```

<br>

대부분 상황에서의 사용 편의를 위해, `join()`, `getNow(T)` 메소드는 비동기 로직 결과값이 `CompletionException`인 경우 `ExecutionException`이 아닌  `CompletetionException`을 발생시키도록 정의되어있다.
- `join` 메소드의 내부 로직인 `reportJoin` 메소드를 보면, 작업 결과값이 AltResult 인스턴스인 경우 null을 반환하거나 CancellationException이나 `CompletionException` 예외를 발생시키는 것을 알 수 있다.

```java
...
public T join() {
    Object r;
    if ((r = result) == null)
        r = waitingGet(false);
    return (T) reportJoin(r);
}
...
private static Object reportJoin(Object r) {
    if (r instanceof AltResult) {
        Throwable x;
        if ((x = ((AltResult)r).ex) == null)
            return null;
        if (x instanceof CancellationException)
            throw (CancellationException)x;
        if (x instanceof CompletionException)
            throw (CompletionException)x;
        throw new CompletionException(x); // 예외발생 케이스에서 Cancellation Exception이 아니면 CompletionException을 발생시킴
    }
    return r;
}
...
```
