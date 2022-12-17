---
title: "자바 functional interface"
excerpt: "functional interface 개념 정리"
categories:
  - java
tags:
  - functional interface
  - 함수형 인터페이스
toc: true
---

# 1. 개요

Supplier, Runnable 등의 인터페이스는 내부 코드를 보면 모두 `@FunctionalInterface` 라는 어노테이션이 붙어있다. 그래서 해당 인터페이스들을 알아보기 이전에 Java 8에서 도입된 함수형 인터페이스(Functional Interface)가 무엇인지 알아보려고 한다.

<br><br>

# 2. 정의

함수형 인터페이스는 하나의 추상메소드만 가지고 있는 인터페이스이다.

- 하나의 기능만 표현
- 디폴트 메소드와 정적 메소드는 여러개 포함 가능

<br><br>

# 3. 등장 배경 및 개념

## 3.1. Java 8 이전

8버전 이전의 자바는 객체지향 언어이기 때문에 몇몇 프리미티브 타입과 메소드를 제외하면 모든 것이 객체를 통해서 동작했고, 어떤 기능을 메소드 단위로 표현했기 때문에 단일 기능을 만들더라도 캡슐화를 위한 (익명)객체가 필요했다.

> 프리미티브 메소드: [프리미티브 메소드](../primitive-method)<br>
> 프리미티브 타입: int, long 등

<br>

[예시]

Thread 클래스 생성 시 필요한 Runnable 인터페이스의 구현 객체 생성

```java
class Main {
	public static void main(String args[]) {
		new Thread(new Runnable() {
			@Override
			public void run() {
				System.out.println("Thread Created");
			}
		}).start();
	}
}
```

<br>

## 3.2. Java 8 이후

Java 8 버전에서 단일 기능을 위해 작성되는 코드들을 간결하게 하기위해 새로운 기초 프로그래밍 방식을 제공하는 함수형 인터페이스와 그 구현 객체로 사용될 수 있는 람다 표현식, 메소드 참조를 도입했다.

이 때 자바에 여러 인터페이스가 함수형 인터페이스로 전환됐는데 Runnable, Callable 등이 있다.

<br>

[예시]

람다표현식으로 Runnable 인터페이스 구현 객체를 대체

```java
class Main {
	public static void main(String[] args) {
		new Thread(() -> {
			System.out.println("Thread Created");
		}).start();
	}
}
```

<br>

[용어 설명]

> 람다표현식은 그 자체로 파라미터로 전달되고 메소드로부터 반환될 수 있는 [일급객체(first-class citizen)](../../functional%20programming/programming%20language/first-class-function)이며 익명함수이다.


> 메소드 참조는 말그대로 이미 존재하는 메소드에 대한 참조를 말한다.
>
> 사용하고 싶은 익명함수의 동작이 이미 다른 메소드에서 제공하는 경우, 람다표현식 대신 해당 메소드의 참조를 사용할 수 있다.
>
> 이중 콜론(::)을 사용하여 클래스명과 메소드명을 구분한다.
> - ex) Interger::sum
> 
> 람다 표현식과 달리 인자를 전달할 필요가 없다.
>
> 인자는 메소드 참조의 타입에 따라 처리한다.
>
> 람다표현식과 마찬가지로 일급객체로 취급된다.


<br><br>

# 4. 어노테이션을 이용한 명시

정의에 명시된 조건을 만족한다면 함수형 인터페이스라고 할 수 있지만, 컴파일러와 다른 개발자들이 함수형 인터페이스임을 인지할 수 있도록 하기 위한 방법으로 `@FunctionalInterface` 어노테이션으로 해당 인터페이스가 함수형 인터페이스라는 것을 표현할 수 있다.

- `@FunctionalInterface` 어노테이션이 적용된 인터페이스에 정의된 추상메소드가 두개 이상이면, 아래와 같은 메시지와 함께 컴파일 에러 발생
    
    ```java
    {인터페이스 파일 경로} error: Unexpected @FunctionalInterface annotation
    @FunctionalInterface
    ^
      TestInterface is not a functional interface
        multiple non-overriding abstract methods found in interface {인터페이스 이름}
    ```

<br><br>

# 5. 구현 예시

한가지 연산만 할 수 있는 계산기를 표현하는 함수형 인터페이스 Calculator

```java
// Calculator.java
@FunctionalInterface
public interface Calculator {

  int calculate(int x, int y);
}
```

Calculator 인터페이스를 덧셈 연산만을 하는 계산기로 구현한 구현

```java
// Main.java
public class Main {
	public static void main(String[] args) {
		// 람다 표현식
		Calculator plusCalculator = (x, y) -> x + y;
		// 메소드 참조
		Calculator plusCalculatorMethodReference = Integer::sum;

		// output: 3
		System.out.println(plusCalculator.calculate(1, 2));
		// output: 3
		System.out.println(plusCalculatorMethodReference.calculate(1, 2));
	}
}
```

<br><br>

# 6. References

[함수형 인터페이스]

- [https://docs.oracle.com/javase/8/docs/api/index.html?java/lang/FunctionalInterface.html](https://docs.oracle.com/javase/8/docs/api/index.html?java/lang/FunctionalInterface.html)


- [https://www.baeldung.com/java-8-functional-interfaces](https://www.baeldung.com/java-8-functional-interfaces)

- [https://www.geeksforgeeks.org/functional-interfaces-java/](https://www.geeksforgeeks.org/functional-interfaces-java/)

- [https://m.blog.naver.com/zzang9ha/222087024412](https://m.blog.naver.com/zzang9ha/222087024412)

<br>

[메소드 레퍼런스]

- [https://developer-talk.tistory.com/462](https://developer-talk.tistory.com/462)

- [https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)