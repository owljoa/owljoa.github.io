---
title: "자바 primitive method"
excerpt: "primitive method 개념 정리"
categories:
  - java
tags:
  - primitive method
  - 함수형 인터페이스
toc: true
---

# 개요

함수형 인터페이스에 대해 공부하던 중에 프리미티브 메소드(primitive method)라는 표현이 등장했고, 이에 대해 알아본 내용을 간단히 정리한다.

<br><br>

# 정의

프리미티브 메소드는 다음의 조건을 만족하는 메소드이다.

- 객체의 필드를 직접 참조한다
- 다른 메소드에 의존하지 않는다
- 더 이상 나눌 수 없는 단위작업이다

<br><br>

# 예시

LocalTime 클래스의 `getHour()`, `getMinute()` 메소드는 객체의 필드를 직접 참조하며 다른 메소드에 의존하지 않고, 더 이상 나눌 수 없는 작업이다.

```java
// LocalTime.java
...
private final byte hour;
private final byte minute;
...
public int getHour() {
    return hour;
}

public int getMinute() {
    return minute;
}
...
```

<br>

다른 메소드를 보면, LocalTime 클래스의 `LocalTime withSecond(int second)` 메소드는 내부에서 `checkValidValue(...)`, `create(...)` 메소드와 같은 하위작업들을 순차적으로 수행해서 완료되는 작업으로 프리미티브 메소드라고 할 수 없다.
이런 메소드를 합성 메소드(comoposed method) 라고 한다.

```java
// LocalTime.java
...
public LocalTime withSecond(int second) {
    if (this.second == second) {
        return this;
    }
    SECOND_OF_MINUTE.checkValidValue(second);
    return create(hour, minute, second, nano);
}
...
```

<br><br>

# References

https://stackoverflow.com/questions/56278814/what-is-primitive-interface-method-in-java

- 포함 링크: https://riehle.org/computer-science/industry/2000/jr-2000-method-properties.pdf