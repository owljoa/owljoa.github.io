---
title: "자바 CompletableFuture - 개념 정리"
excerpt: "CompletableFuture 개념 정리"
categories:
  - java
tags:
  - 비동기
toc: true
---

# 1. 개요

API 응답시간을 줄이기 위한 방법으로 API 로직 내 I/O 작업을 수행하는 부분을 비동기 방식으로의 전환이 있다. 먼저 자바에서 제공하는 비동기 관련 API부터 이해하기위해 앞서 Future를 살펴봤고, 본 포스트에서는 CompletableFuture에 대해 정리한다.

CompetableFuture는 Java 8 버전에서 도입됐고, Java 9에서 좀 더 향상된 비동기 작업을 다루기 위해 자바에서 제공하는 API다.

<br><br>

# 2. 특징

2.1. Future 인터페이스의 다음과 같은 한계점을 보완했다.
- 예외처리 기능 부재
- 병렬 수행 및 병합 기능 부재
- 연산의 조합 기능 부재
- 값과 상태 직접 세팅 불가능

2.2. CompletableFuture는 Java 5(1.5)버전의 Future 인터페이스를 구현했다.

2.3. CompletableFuture는 CompletionStage 인터페이스를 구현했다.
  - CompletionStage: 다른 CompletionStage가 완료됐을때 어떤 행위나 연산을 수행하는 비동기 작업의 단계를 나타낸 인터페이스

2.4. complete, completeExceptionally 메소드를 이용하여 명시적으로 완료(값과 상태 설정)할 수 있다.

2.5. 여러개의 쓰레드가 하나의 CompletableFuture 인스턴스에 대해 complete, completeExceptionally, cancel 메소드를 수행하면 하나의 쓰레드만 성공한다.

<br><br>

# 3. 주요 메소드 목록

CompletableFuture 클래스에서 제공하는 주요 메소드들로 전체 메소드를 소개하지는 않습니다.

<br>

## 3.1. 비동기 작업 실행

CompletableFuture 생성 시 사용 <br>
인자로 입력한 작업을 비동기로 수행

3.1.1. runAsync
  - 작업의 반환 값이 없는 케이스(void)

3.1.2. supplyAsync
  - 작업의 반환 값이 있는 케이스

<br>

## 3.2. 결과값 조회

비동기 작업의 결과값을 조회 <br>
메소드 호출 당시 비동기 작업이 완료되지 않았다면 완료될 때까지 대기

3.2.1. get
  - 비동기 작업의 결과값을 반환
  - 작업이 정상적으로 완료되지 않았다면 checked exception 발생

3.2.2. join
  - 비동기 작업의 결과값을 반환
  - 작업이 정상적으로 완료되지 않았다면 unchecked exception 발생
    - unchecked exception: RuntimeException 상속한 exception 으로 명시적인 예외처리가 필요하지 않다.

<br>

## 3.3. **비동기 작업 후처리**

비동기 작업이 완료된 이후 이어져야하는 작업 수행

3.3.1. thenApply
  - 반환 값을 받아서 다른 값을 반환
  - 함수형 인터페이스 Function을 파라미터로 받음

3.3.2. thenAccept
  - 반환 값을 받아 처리하고 값을 반환하지 않음
  - 함수형 인터페이스 Consumer를 파라미터로 받음

3.3.3. thenRun
  - 반환 값을 받지 않고 다른 작업을 실행함
  - 함수형 인터페이스 Runnable을 파라미터로 받음

<br>

## 3.4. 비동기 작업 조합

비동기 작업의 조합

3.4.1. thenCompose
  - 두 작업이 이어서(의존) 실행하도록 조합하며, 선행 작업의 결과를 받아서 사용 가능
  - 함수형 인터페이스 Function을 파라미터로 받음

3.4.2. thenCombine
  - 두 작업을 독립적으로 실행하고, 둘 다 완료되었을 때 파라미터로 받은 작업을 수행

3.4.3. allOf
  - 여러 작업들을 동시에 실행해서 파라미터로 받은 CompletableFuture들이 모두 완료되어야 완료됨
  - 결과값을 반환하지 않음
  - 이후에 then~ 메소드와 함께 이용하면 모든 작업이 완료된 이후에 수행해야할 작업을 정의할 수 있을 것으로 보임

3.4.4. anyOf
  - 여러 작업들 중 가장 빨리 끝난 하나의 결과값을 새로운 CompletableFuture 인스턴스에 포함하여 반환
  - then~ 메소드를 이용해서 여러 작업들 중 가장 먼저 끝난 작업에 이어질 작업을 정의할 수 있을 것으로 보임

<br>

## 3.5. 예외 처리

3.5.1. exceptionally
  - 발생한 에러를 받아서 예외를 처리함
  - 함수형 인터페이스 Function을 파라미터로 받음

3.5.2. handle, handleAsync
  - (결과값, 에러)를 반환받아 에러가 발생한 경우와 아닌 경우 모두를 처리할 수 있음
  - 함수형 인터페이스 BiFunction을 파라미터로 받음

<br><br>

# 4. 사용

## 4.1. 시나리오

1. 특정 주문의 취소를 위해 주문취소 메소드 호출
2. 배송취소 요청 <br>
  \- 배송취소 정상응답 수신하면 주문의 결제상태 “취소”로 변경
3. 결제취소 요청 <br>
  \- 결제취소 정상응답 수신하면 주문의 배송상태 “취소”로 변경
4. 상품 조회 및 재고수량 증가 처리
5. 주문상태 “취소”로 변경

<br>

## 4.2. 예제 코드

- 전체 코드: [https://github.com/owljoa/java-study/tree/main/completable-future](https://github.com/owljoa/java-study/tree/main/completable-future)
- (참고) 편의상 트랜잭션 처리는 주석으로 대체했습니다.

<br>

### 4.2.1. 주문 취소 메소드

- 주문 조회 이후 배송취소, 결제취소, 배송취소와 결제취소 결과 조합은 모두 비동기로 별도의 쓰레드에서 이루어집니다.
- 메인 쓰레드는 배송취소, 결제취소 요청에 대한 응답을 기다리지 않고 상품 조회 후 재고수량 증가 처리와 주문상태를 취소로 변경하는 작업을 수행합니다.

```java
// 주문취소 트랜잭션 메소드
public void cancel(long orderId) {
	// 주문 조회
  final Order order = findById(orderId);

  // 배송취소
  CompletableFuture<Long> canceledDeliveryId = cancelDeliveryAsync(order);
  // 결제취소
  CompletableFuture<Long> canceledPaymentId = cancelPaymentAsync(order);
  // 배송취소, 결제취소 결과 조합
  CompletableFuture<Boolean> preprocessToCancelOrder = canceledDeliveryId.thenCombine(
      canceledPaymentId,
      (cancelDeliverResult, cancelPaymentResult) -> {
        if (cancelDeliverResult == null || cancelPaymentResult == null) {
          return false;
        }
        return cancelDeliverResult == order.getDeliveryId() && cancelPaymentResult == order.getPaymentId();
      }
  );

  // 상품 조회
  final Product product = productService.findById(order.getProductId());
	// 상품 재고수량 증가 처리
  product.increaseAmount(order.getAmount());

  // 주문상태 취소로 변경
  order.cancelSuccess();

  // 배송취소, 결제취소 결과 대기
  if (!preprocessToCancelOrder.join()) {
    // 배송취소 혹은 결제취소 실패 시 트랜잭션 롤백
    throw new RuntimeException("주문취소 전처리 실패");
  }
  LogUtil.printWithThreadName("주문취소 처리 완료");
}
```

<br>

### 4.2.2. 배송취소

1. 배송취소 요청<br>
  \- supplyAsync 메소드로 별도 쓰레드에서 수행할 "배송취소 요청” 작업 등록
2. 배송취소 요청에 대한 에러처리<br>
  \- exceptionally 메소드로 배송취소 요청중 발생할 수 있는 예외에 대해 로깅 후 null 반환 처리
3. 결과에 대한 후처리<br>
  1\. 정상 케이스<br>
    \- 결과값이 요청 시 입력했던 배송 아이디(deliveryId)라면 주문의 배송상태 “취소”로 변경<br>
  2\. 예외발생 케이스<br>
    \- 결과값이 요청 시 입력했던 배송 아이디가 아니면 다른 작업 없이 해당 결과값 반환

```java
private CompletableFuture<Long> cancelDeliveryAsync(Order order) {
  return CompletableFuture.supplyAsync(() -> DeliveryApiAdapter.cancelDelivery(order.getDeliveryId()))
      .exceptionally(throwable -> {
        LogUtil.printWithThreadName("배송취소 실패 - " + throwable.getMessage());
        return null;
      }).thenApply(result -> {
        if (result != null && result == order.getDeliveryId()) {
          // 주문의 배소상태 "취소"로 변경
          order.cancelDeliverySuccess();
        }
        return result;
      });
}
```

<br>

### 4.2.3. 결제취소

1. 결제취소 요청<br>
  \- supplyAsync 메소드로 별도 쓰레드에서 수행할 "결제취소 요청” 작업 등록
2. 결제취소 요청에 대한 에러처리<br>
  \- exceptionally 메소드로 결제취소 요청중 발생할 수 있는 예외에 대해 로깅 후 null 반환 처리
3. 결과에 대한 후처리<br>
    1\. 정상 케이스<br>
      \- 결과값이 요청 시 입력했던 결제 아이디(paymentId)라면 주문의 결제상태 “취소”로 변경<br>
    2\. 예외발생 케이스<br>
        \- 결과값이 요청 시 입력했던 결제 아이디가 아니면 다른 작업 없이 해당 결과값 반환<br>
        \- 기존 성공 케이스 구문을 주석처리하고 결제취소 처리 실패 케이스 주석 아래의 구문을 주석해제하면 확인하실 수 있습니다.

```java
private CompletableFuture<Long> cancelPaymentAsync(Order order) {
  return CompletableFuture.supplyAsync(() ->
          // 결제취소 처리 성공 케이스
          PaymentApiAdapter.cancelPaymentSuccess(order.getPaymentId())
      // 결제취소 처리 실패 케이스
//        PaymentApiAdapter.cancelPaymentFail(order.getPaymentId())
  ).exceptionally(throwable -> {
    LogUtil.printWithThreadName("결제취소 실패 - " + throwable.getCause());
    return null;
  }).thenApply(result -> {
    if (result != null && result == order.getPaymentId()) {
      // 주문의 결제상태 "취소"로 변경
      order.cancelPaymentSuccess();
    }
    return result;
  });
}
```

<br>

### 4.2.4. 실행 결과

로그 패턴은 “[날짜 및 시간][작업 쓰레드 이름] 내용” 입니다.

[정상 수행 케이스]

- 결제취소, 배송취소 작업이 서로 다른 쓰레드에서 수행되는 것을 확인할 수 있습니다.
- 메인 쓰레드는 결제취소, 배송취소와는 별도로 상품조회, 재고수량 증가, 주문상태 변경 작업을 수행하는 것을 확인할 수 있습니다.

```log
[2022-11-12T17:18:01.262306][ForkJoinPool.commonPool-worker-5] 결제취소 요청 발송
[2022-11-12T17:18:01.262558][main] 상품 조회 시작
[2022-11-12T17:18:01.262520][ForkJoinPool.commonPool-worker-19] 배송취소 요청 발송
[2022-11-12T17:18:01.573998][main] 상품 조회 완료
[2022-11-12T17:18:01.574992][main] 재고수량 +1 시작
[2022-11-12T17:18:01.680180][main] 재고수량 +1 완료
[2022-11-12T17:18:01.680398][main] 주문상태 취소로 변경 시작
[2022-11-12T17:18:01.886272][main] 주문상태 취소로 변경 완료
[2022-11-12T17:18:02.775852][ForkJoinPool.commonPool-worker-19] 배송취소 응답 수신
[2022-11-12T17:18:02.778172][ForkJoinPool.commonPool-worker-19] 배송취소 후 주문의 배송상태 변경 시작
[2022-11-12T17:18:03.079039][ForkJoinPool.commonPool-worker-5] 결제취소 응답 수신
[2022-11-12T17:18:03.079522][ForkJoinPool.commonPool-worker-5] 결제취소 후 주문의 결제상태 변경 시작
[2022-11-12T17:18:03.080171][ForkJoinPool.commonPool-worker-19] 배송취소 후 주문의 배송상태 변경 완료
[2022-11-12T17:18:03.380870][ForkJoinPool.commonPool-worker-5] 결제취소 후 주문의 결제상태 변경 완료
[2022-11-12T17:18:03.384166][main] 주문취소 처리 완료
```

<br>

[결제취소 실패 케이스]

- 결제취소나 배송취소 중 하나라도 실패하면 `preprocessToCancelOrder` 의 결과값이 false 로 반환되고, false가 반환되면 RuntimeException을 발생시키도록 했으므로 예외가 발생함을 확인할 수 있다.

```log
[2022-11-19T14:51:14.007736][ForkJoinPool.commonPool-worker-5] 결제취소 요청 발송
[2022-11-19T14:51:14.007891][ForkJoinPool.commonPool-worker-19] 배송취소 요청 발송
[2022-11-19T14:51:14.007875][main] 상품 조회 시작
[2022-11-19T14:51:14.321377][main] 상품 조회 완료
[2022-11-19T14:51:14.324627][main] 재고수량 +1 시작
[2022-11-19T14:51:14.429928][main] 재고수량 +1 완료
[2022-11-19T14:51:14.430092][main] 주문상태 취소로 변경 시작
[2022-11-19T14:51:14.634833][main] 주문상태 취소로 변경 완료
[2022-11-19T14:51:15.043757][ForkJoinPool.commonPool-worker-5] 결제취소 실패 - java.lang.IllegalArgumentException: 결제 오류 발생 - 사유: 블라블라
[2022-11-19T14:51:15.523170][ForkJoinPool.commonPool-worker-19] 배송취소 응답 수신
[2022-11-19T14:51:15.526267][ForkJoinPool.commonPool-worker-19] 배송취소 후 주문의 배송상태 변경 시작
[2022-11-19T14:51:15.828988][ForkJoinPool.commonPool-worker-19] 배송취소 후 주문의 배송상태 변경 완료
...
Exception in thread "main" java.lang.RuntimeException: 주문취소 전처리 실패
...
```