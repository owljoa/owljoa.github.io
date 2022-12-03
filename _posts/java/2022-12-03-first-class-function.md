---
title: "first-class function(일급 함수) 개념 정리"
excerpt: "first-class function(일급 함수) 개념 정리"
categories:
  - functional programming
  - programming language
tags:
  - 함수형 프로그래밍
  - 함수형 인터페이스
toc: true
---

# 개요

자바의 함수형 인터페이스에 대해 공부하던 중에 first-class citizen 이라는 표현이 등장했고, 그 중 람다(익명함수)에 대한 내용을 이해하기 위해 알아본 내용을 간단히 정리한다.

<br><br>

# 정의

## 일급 객체(first-class citizen)

일급객체는 일반적으로 다른 객체들이 이용할 수 있는 모든 연산을 제공하는 객체를 말한다.<br>
여기서 "일반적으로 제공해야할 모든 연산"의 예시로는 아래 3가지가 대표적인 것으로 보인다.

- 인자로 전달될 수 있을 것
- 함수로부터 반환될 수 있을 것
- 변수에 할당될 수 있을 것

<br>

## 일급 함수(first-class function)

일급함수는 프로그래밍 언어에서 함수를 일급객체로 취급하는 것으로, 함수가 일급 객체의 조건을 만족하도록 설계된 언어에서의 함수를 일급함수라고 한다.


<br><br>

# 예시

일급함수를 제공하는 파이썬 언어를 예시로 사용한다.
아래 코드로 파이썬이 일급함수를 제공함을 알 수 있다.

1. `function = plus` 구문으로 함수(plus)가 변수(function)에 할당됨을 확인할 수 있다.

2. `print(plus)` 구문으로 함수(print)에 함수(plus)를 인자로 전달됨을 확인할 수 있다.

3. `minus_function = find_function_by_operation('minus')` 구문으로 함수(minus)가 함수(find_function_by_operation)로부터 반환됨을 확인할 수 있다.

<br>

```python
def find_function_by_operation(operation):
  if operation == 'plus':
    return plus
  if operation == 'minus':
    return minus
  raise Exception('operation must be "plus" or "minus"')

def plus(x, y):
  return x + y

def minus(x, y):
  return x - y

print(plus(1, 2))

# 변수에 함수 할당 가능
function = plus

# 함수를 다른 함수의 파라미터로 전달 가능
print(plus)
print(function)

# 함수에서 함수 반환 가능
minus_function = find_function_by_operation('minus')
print(minus_function(2, 1))
```

<br><br>

# References

[https://en.wikipedia.org/wiki/First-class_citizen](https://en.wikipedia.org/wiki/First-class_citizen)

[https://en.wikipedia.org/wiki/First-class_function](https://en.wikipedia.org/wiki/First-class_function)


