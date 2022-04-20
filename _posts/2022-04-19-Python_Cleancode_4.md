---
layout: post
toc: true
title: Python Cleancode - 4
categories: Python
tags: [Python, OOP, Cleancode]
author:
  - Heeseong
---


# 파이썬에서 유의해야할 점
> 언어의 주요 기능을 이해하는 것 외에도 흔히 발생하는 잠재적 문제 피할 수 있는 관용적 코드를 작성하는 것도 중요하다.

방어코드 미작성 시 발생할 수 있는 이슈들에 대해 알아보자 !

## 변경 가능한 파라미터의 기본 값
* 쉽게 말해 변경 가능한 객체를 함수의 기본 인자로 사용하면 안된다.
  * 기대와 다른 결과를 얻게 됨

``` python
def wrong_user_display(user_metadata: dict = { "name": "Jojn", "age": 30 }):
  name = user_metadata.pop("name")
  age = user_metadata.pop("age")
  return f"{name} ({age})"
```

* 위 코드에는 두 가지 문제가 있다.
1. 변경 가능한 인자를 사용한 것
2. 함수의 본문에서 가변 객체를 수정

아래와 같은 경우에 에러가 KeyError 발생함.

```python

wrong_user_display()
# Jojn (30)
wrong_user_display({ "name": "Jojn", "age": 50 })
# Jojn (50)
wrong_user_display()
# KeyError!
```

* 처음 호출에 기본 데이터 dict를 한 번만 생성하고, 두 번째 호출에서 새로운 dict 생성하며, 기존 생성한 dict key 버림
* 이후 기본 호출 시 KeyError

* 아래와 같이 변경되는게 맞음
``` python
def wrong_user_display(user_metadata = None):
  name = user_metadata.pop("name")
  age = user_metadata.pop("age")
  return f"{name} ({age})"
```

## 내장(built-in) 타입 확장
* 리스트, 문자열, dict와 같은 내장 타입을 확장하는 올바른 방법은 collections 모듈 사용하는 것

* 예를 들어 dict를 직ㅈ버 확장하는 클래스를 만들면 예상하지 못한 결과 얻을 수 있음
  * CPython에서는 클래스의 메서드를 서로 호출하지 않기 때문에, 메서드 중에 하나를 오버라이드하면 나머지는 반영하지 않음

* 예제를 보고 이해해보자
* 입력 받은 숫자를 접두어가 있는 문자열로 변환하는 리스트를 만드는 예제
  * super() - 자식 클래스에서 부모클래스의 내용을 사용하고 싶을경우 사용

```python
class BadList(list):
    def __getitem__(self, index):
        value = super().__getitem__(index)
        if index % 2 == 0:
            prefix = "Even"
        else:
            prefix = "Odd"

        return print(f"[{prefix}] {value}")

b1 = BadList((0,1,2,3,4,5))
print(b1[0]) # [Even] 0
print(b1[1]) # [Odd] 1
"".join(b1) # Error
```

* 얼핏 보면 정답 같지만, 원하는 결과랑 다른 결과가 나옴
* join은 문자열 리스트를 반복하는 함수. 하지만 __getitem__ 호출 되지 않음


> 어떠한 상황에서도 이식/호환 가능한 코드를 작성해야 하므로 리스트가 아니라 UserList에서 확장하여 수정해야함
  dict -> UserDict, string -> UserString

> collections의 UserList, UserDict, UserString은 각 타입의 Wrapper 역할을 수행하는 클래스

``` python
from collections import UserList

class GoodList(UserList):
  def __getitem__(self, index):
      value = super().__getitem__(index)
      if index % 2 == 0:
          prefix = "Even"
      else:
          prefix = "Odd"

      return f"[{prefix}] {value}"


g1 = GoodList((0,1,2))
print(g1[0]) # [Even] 0
print(g1[1]) # [Odd] 1
print(";".join(g1)) # [Even] 0;[Odd] 1;[Even] 2
```






> 코드 작성 규칙이 좋은 코드에 필수적이긴 하지만 충분한 조건은 아니다. 파이썬스러운 관용적인 코드를 작성하는 가장 좋은 방법은
파이썬이 제공하는 모든 기능을 최대한 활용하는 것이다. 떄로는 매직메서드를 구현하고, 컨텍스트 관리자를 구현해야하는 것



