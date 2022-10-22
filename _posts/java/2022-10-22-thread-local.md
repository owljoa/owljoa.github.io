---
title: "자바 ThreadLocal"
excerpt: "ThreadLocal 개념 정리"
categories:
  - java
tags:
  - 동시성
toc: true
---


# 1. 정의 및 목적

- 쓰레드의 로컬 변수 사용을 위한 클래스
- 특정 쓰레드에 의해서만 접근 가능한 데이터를 저장할 수 있게 해주는 구조
- 쓰레드 안전성(thread-safe)을 보장하기 위해 사용

<br>

# 2. 특징

- ThreadLocal에 저장되어있으면 하나의 요청 흐름 내에서 모든 메소드 호출 시 파라미터로 사용하지 않아도 쓰레드에서 꺼내서 사용이 가능
    - ex) 웹 어플리케이션의 경우 현재 요청과 세션을 쓰레드 로컬에 담아두면, 어플리케이션 내에서 쉽게 접근이 가능
- 모든 쓰레드는 각자만의 ThreadLocal 변수를 할당받음
- 조회와 수정을 위한 get, set 메소드가 제공됨

<br>

## 2.1. 동기화 vs ThreadLocal

- 동기화는 변수를 하나로 한정시켜서 해당 변수를 사용중인 쓰레드 외에 다른 쓰레드는 대기하도록 하는 것(”time for space” 접근방법)
- ThreadLocal은 개별 쓰레드에게 변수를 할당하고 대기시간 없이 접근하도록 하는 것(”space for time” 접근방법)

<br>

# 2. 구현

## 2.1. Thread 클래스에 포함된 ThreadLocalMap

- ThreadLocalMap은 쓰레드 로컬 값을 위해 해시맵을 커스터마이징해서 구현됨
- 쓰레드 내에 ThreadLocalMap으로 쓰레드의 로컬 변수를 관리
    
    → 모든 쓰레드는 각자만의 ThreadLocal 변수를 할당받음
    

```java
// java.lang.Thread
...
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
...
```

<br>

## 2.2. 쓰레드의 로컬 변수 수정 및 조회

- ThreadLocal 객체를 ThreadLocalMap의 키로 사용함

```java
// java.lang.ThreadLocal
...
public T get() {
    // 현재 실행중인 쓰레드 객체 가져옴
    Thread t = Thread.currentThread();

    // 쓰레드의 로컬변수 관리용 커스텀 해시맵 구조 
    ThreadLocalMap map = getMap(t);
    if (map != null) {
				// 해당 ThreadLocal 객체를 키로 저장해둔 값을 불러옴
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    // 해당 쓰레드 키에 대한 값이 없으면 null을 저장하고, null을 반환
    return setInitialValue();
}
...

public void set(T value) {
  // 현재 실행중인 쓰레드 객체 가져옴
  Thread t = Thread.currentThread();

  // 쓰레드의 로컬변수 관리용 커스텀 해시맵 구조
  ThreadLocalMap map = getMap(t);

  // 맵이 없으면 맵을 초기화하면서 입력받은 값과 현재 쓰레드로 키-값 저장
  // 맵이 있으면, ThreadLocal 객체를 키로 입력받은 값 저장
  if (map != null) {
      map.set(this, value);
  } else {
      createMap(t, value);
  }
}
```

<br>

# 3. 사용법

- ThreadLocal 인스턴스는 상태(멤버)를 쓰레드와 연결하기위해 사용하기 때문에 보통 private static 변수로 선언함
    - ex) 트랜잭션이나 사용자의 아이디

- 생성/조회/수정/제거(CRUD)
    
    ```java
    ...
    
    // 선언 및 생성
    private static final ThreadLocal<String> userNameInThreadLocal = ThreadLocal.withInitial(() -> "");
    
    // 조회
    userIdInThreadLocal.get();
    
    // 수정
    userNameInThreadLocal.set(name);
    
    // 제거
    userNameInThreadLocal.remove();
    
    ...
    ```
    

<br>

# 4. 쓰레드풀 환경에서 주의사항

쓰레드풀(Thread Pool)은 쓰레드를 재활용하기 때문에 ThreadLocal 사용 후 저장한 값을 제거하지 않으면 다른 쓰레드에서 해당 값을 조회할 가능성이 있음

시나리오

1. 어플리케이션이 요청 처리를 위해 쓰레드풀에서 쓰레드 하나를 대여함
2. ThreadLocal에 어떤 값을 저장
3. 어플리케이션에서 해당 쓰레드로 필요한 로직을 수행한 후 쓰레드를 쓰레드풀로 반납
4. 이후 어플리케이션에서 또 다른 요청 처리를 위해 쓰레드풀에서 이전에 사용한 쓰레드를 대여함
5. 어플리케이션이 이전 요청처리 완료 후 쓰레드 내 로컬데이터를 정리하지 않았기 때문에 새로운 요청처리에도 같은 ThreadLocal 데이터를 사용하게 될 수 있음

→ 쓰레드를 사용하고 수동으로 ThreadLocal 데이터를 제거하는 방법도 있지만, 매번 사용 로직에 넣어야해서 코드량이 늘어나고 실수의 가능성이 있음

→ ThreadPoolExecutor를 상속받으면, beforeExecute(), afterExecute() 메소드를 이용해서 쓰레드풀의 쓰레드 작업 수행 전/후의 작업을 일괄로 추가할 수 있음

```java
public class ThreadPoolKnowingThreadLocal extends ThreadPoolExecutor {
	
	@Override
	protected void afterExecute(Runnable r, Throwable t) {
		// ThreadLocal 데이터 제거하는 메소드 호출
		SomeStore.cleanThreadLocals();
	}
}
```

<br>

# 5. 예제

예제코드 참고: [ThreadLocal 예제 코드](https://github.com/owljoa/java-study/tree/main/java-thread-local)

<br>

## 5.1. 테스트 시나리오

1. 현재 요청자의 userId를 특정 저장소(인스턴스 변수 or ThreadLocal)에 담아두고 
2. 요청을 받을 때 userId를 요청자의 userId로 변경하고 (변경된)현재 요청자의 userId로 유저이름을 조회하는 로직 A
    - userId 변경 소요시간 100ms
    - 이름 조회 소요시간 80ms
4. 사전 세팅
    |쓰레드|userId|유저 이름|
    |---|---|---|
    |1|1|A|
    |2|2|B|
    |3|3|C|

    
4. 3개의 쓰레드에 3명의 요청자 할당해서 동시에 가깝게 로직 A 수행

    1. userA, userB, userC 순서로 동시에 가까운 요청
        - 00:00:00.000 userA 요청
        - 00:00:00.040 userB 요청
        - 00:00:00.080 userC 요청
    2. userId 수정 및 조회 시간 
        - userA userId 수정 완료 00.100
        - userB userId 수정 완료 00.140
        - userA 이름 조회 완료 00.180
        - userC userId 수정 완료 00.180
        - userB 이름 조회 완료 00.220
        - userC 이름 조회 완료 00.260

5. 결과
- 인스턴스 변수 사용의 경우 쓰레드들이 인스턴스 변수를 공유하므로 아래와 같은 상황 발생
    - userA 이름 조회 결과 “B”<br>
    - userB, userC 이름 조회 결과 “C”<br>
- ThreadLocal은 의도한대로 각 userId별로 알맞은 유저 이름이 조회됨


<br>

# 참고

[https://docs.oracle.com/javase/8/docs/api/java/lang/ThreadLocal.html](https://docs.oracle.com/javase/8/docs/api/java/lang/ThreadLocal.html)

[https://www.baeldung.com/java-threadlocal](https://www.baeldung.com/java-threadlocal)

[https://ko.n4zc.com/article/qz4ebxkk.html](https://ko.n4zc.com/article/qz4ebxkk.html)

[https://www.digitalocean.com/community/tutorials/java-threadlocal-example](https://www.digitalocean.com/community/tutorials/java-threadlocal-example)

[https://gong-story.tistory.com/17](https://gong-story.tistory.com/17)

[https://blog.actorsfit.com/a?ID=01450-6eeb46e3-2500-4d9a-827b-ec466a71b60f](https://blog.actorsfit.com/a?ID=01450-6eeb46e3-2500-4d9a-827b-ec466a71b60f)