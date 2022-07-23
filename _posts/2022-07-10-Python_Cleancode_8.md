---
layout: post
toc: true
title: Python Cleancode - 8
categories: Python
tags: [Python, OOP, Cleancode]
author:
  - Heeseong
---

# 데코레이터를 사용한 코드 개선

데코레이터가 무엇인지, 어떻게 동작하는지, 어떻게 구현되는지 살펴본다.

* 파이썬에서 데코레이터가 동작하는 방식 이해
* 함수와 클래스에 적용되는 데이코레이터 구현 방법
* 일반적인 실수를 피하여 효과적으로 구현하는 방법
* 데코레이터를 활영한 코드 중복 회피 - DRY 원칙 준수
* 데코레이터를 이용한 관심사의 분리
* 좋은 데코레이터 사례
* 데코레이터가 좋은 선택이 될 수 있는 일반적인 상황, 관용구, 패턴



## 파이썬의 데코레이터

* 원래는 함수에 변형을 할 때마다 modifier 함수를 사용해서 함수 호출한 뒤에 처음 정의한 것과 같은 이름으로 재할당해야 했다.

``` python
def original(...):
  ...

original = modifier(original)
```

* 함수를 어떻게 동일하는 이름으로 다시 할당하는지 확인해보자.(번거롭고, 혼란스럽다)

``` python
@modifier
def original(...):
  ...

```
* 데코레이터는 데코레이터 이후에 나오는 것을 데코레이터의 첫 번째 파라미터로 하고 데코레이터의 결과 값을 반환하게 하는 synstactic sugar이다.
* modifier를 데코레이터, original을 래핑(wrapped) 객체라 한다.

* 데코레이터라는 이름은 래핑된 함수의 기능을 수정하고 확장하기 떄문에 정확한 이름이지만, 데코레이터 디자인 패턴과 혼동하며 안 된다.

### 함수 데코레이터

* 함수에 데코레이터를 사용하면 어떤 종류의 로직이라도 적용할 수 있다.
  * 파라미터 유효성 검사, 사전조건 검사, 기능 전체 새로 정의

* 도메인의 특정 예외에 대해서 특정 횟수만큼 재시도하는 데코레이터를 만들어 보자


``` python
from functools import wraps


class ControllException(Exception):
    """도메인에서 발생하는 일반적인 예외"""

def retry(operation):
    @wraps(operation)
    def wrapped(*args, **kwargs):
        last_raised = None
        RETRIES_LIMIT = 3
        for _ in range(RETRIES_LIMIT):
            try:
                return operation(*args, **kwargs)
            except ControllException as e:
                print(f"retrying {operation.__qualname__}")
                last_raised = e
        return last_raised

    return wrapped

@retry
def run_operation(task):
    """실행 중 예외가 발생할 것으로 예상되는 특정 작업을 실행"""
    return task.run()
```
* @wrap 은 지금 당장은 무시해도 된다. 자세한 설명은 잠시 후
* retry 데코레이터는 파라미터가 필요없으므로 어떤 함수에도 쉽게 적용할 수 있다.
* run_operation 위에있는 @retry는 실제로 파이썬에서 `run_operation = retry(run_operation)` 을 실행하게 해주는 문법적 설탕일 뿐
* `__qualname__` 은 순수함수명과 클래스 이름을 구분짓기 위해 한 번 걸러진 값을 제공
  * A클래스의 B 함수의 `__name__`은 B, `__qualname__`은 A.B

* timeout 같은 예외 발생할 경우 여러 번 호출을 반복하는 retry 로직을 어떻게 데코레이터로 만들 수 있는지 살펴보았다.



## 클래스 데코레이터

* 함수에 적용한 것처럼 클래스에도 사용 가능, 차이점은 함수의 파라미터로 클래스를 받는다는 점
* 클래스 데코레이터가 복잡하고 가독성을 떨어뜨릴 수 있다고 말하는 사람도 있다. (남용하면)

* 클래스 데코레이터 장점
  * 코드 재사용과 DRY 원칙의 이점을 공유, 여러 클래스가 특정 인터페이스나 기준을 따르도록 강제할 수 있다.
  * 당장은 작고 간단한 클래스를 생성하고, 데코레이터로 기능을 보강할 수 있다.


* 데코레이터가 유용하게 쓸일 수 있는 예제를 살펴보자
* 이벤트 시스템은 각 이벤트 데이터를 변환하여 외부 시스템으로 보내야한다. 각 이벤트 유형은 데이터 전송 방법에 특별한 점이 있을 수 있다.
* 특히 로그인 이벤트는 자격 증명 정보를 숨겨야하고, timestamp 같은 형식의 포맷 변경이 필요할 수 있음
* 이러한 요구 사항을 준수하기 위한 가장 간단한 방법은 각 이벤트마다 직렬화 방법을 정의한 클래스를 만드는 것이다.


``` python
class LoginEventSerializer:
    def __init__(self, event):
        self.event = event

    def serialize(self) -> dict:
        return {
            "username": self.event.username,
            "password": "**민감한 정보 삭제**",
            "ip": self.event.ip,
            "tiemstamp": self.event.timestamp.strftime("%Y-%m-%d %H:%M")
        }

class LoginEvent:
    SERIALIZER = LoginEventSerializer

    def __init__(self, username, password, ip, timestamp):
        self.username = username
        self.password = password
        self.ip = ip
        self.timestamp = timestamp

    def serialize(self) -> dict:
        return self.SERIALIZER(self).serialize()
```

* 로그인 이벤트에 직접 매핑할 클래스를 선언, password 숨기고, timestamp 포맷팅
* 시스템 확장시 문제점 발생
  * 클래스가 너무 많아진다. 
    - 이벤트 클래스 <> 직렬화 클래스가 1대1이라 점점 많아짐
  * 이러한 방법은 유연하지 않다. 
    - 만약 password를 가진 다른 클래스에서도 이 필드르 숨기려면 함수로 분리한 다음 여러 클래스에서 호출 해야한다
    - 코드 재사용 충분하지 않음
  * 표준화
    - serialize() 메서드가 모든 이벤트 클래스에 있어야만 한다.



* 또 다른 방법은 이벤트 인스턴스와 변형 함수를 필터로 받아서 동적으로 객체를 만드는 것
* 필터를 이벤트 인스턴스의 필드들에 적용해 직렬화하는 것이다.

```python
from datetime import datetime

def hide_field(field) -> str:
    return "**민감한 정보 삭제**"

def format_time(field_timestamp: datetime) -> str:
    return field_timestamp.strftime("%Y-%m-%d %H:%M")

def show_original(event_field):
    return event_field

class EventSerializer:
    def __init__(self, serialization_fields: dict) -> None:
        self.serialization_fields = serialization_fields

    def serialize(self, event) -> dict:
        return {
            field: transformation(getattr(event, field)) for field, transformation in self.serialization_fields.items()
        }

class Serialization:
    def __init__(self, **transformations):
        self.serializer = EventSerializer(transformations)

    def __call__(self, event_class):
        def serialize_method(event_instance):
            return self.serializer.serialize(event_instance)
        event_class.serialize = serialize_method
        return event_class

@Serialization(username=show_original, password=hide_field, ip=show_original, timestamp=format_time)
class LoginEvent:
    def __init__(self, username, password, ip, timestamp):
        self.username = username
        self.password = password
        self.ip = ip
        self.timestamp = timestamp

```


## 데코레이터에 인자 전달
* 파라미터를 갖는 데코레이터를 구현하는 방법은 여러가지이지만 예를 들어 두 가지의 방법이있다.
1. 간접 참조를 통해 새로운 레벨의 중첩 함수를 만들어 데코레이터의 모든 것을 한 단계 더 깊게 만든는 것
2. 데코레이터를 위한 클래스를 만드는 것

* 일반적으로 2번이 가독성이 더 좋다.

### 중첩 함수의 데코레이터 

* 데코레이터로 함수를 사용할 때는 데코레이터 함수에 원본함수를 파라미터로 넘김!

``` python
from functools import wraps

RETRIES_LIMIT = 3

def with_retry(retries_limit=RETRIES_LIMIT, allowed_exceptions=None):
    """중첩 함수의 데코레이터로 아래 retry 함수를 반환하는 것에 집중"""

    def retry(operation):

        @wraps(operation)
        def wrapped(*args, **kwargs):
            last_raised = None

            for _ in range(retries_limit):
                try:
                    return operation(*args, **kwargs)
                except allowed_exceptions as e:
                    print(f"retrying {operation} due to {e}")
                    last_raised = e

            raise last_raised

        return wrapped

    return retry

@with_retry(retries_limit=5, allowed_exceptions=ZeroDivisionError)
def run_with_custom_parameter(task):
    return task.run()

class Task:
    def __init__(self):
        self._test = None

    @staticmethod
    def run():
        return 10/0


if __name__ == "__main__":
    task = Task()
    run_with_custom_parameter(task)

```

```
retrying <function run_with_custom_parameter at 0x10d484040> due to division by zero
retrying <function run_with_custom_parameter at 0x10d484040> due to division by zero
retrying <function run_with_custom_parameter at 0x10d484040> due to division by zero
retrying <function run_with_custom_parameter at 0x10d484040> due to division by zero
retrying <function run_with_custom_parameter at 0x10d484040> due to division by zero
```

### 데코레이터 객체
* 클래스를 사용하여 데코레이터 정의하여 사용
* `__init__` 메서드에 파라미터를 전달한 다음 `__call__` 이라는 매직 메서드에서 데코레이터의 로직을 구현하면 된다.

``` python
from functools import wraps

RETRIES_LIMIT = 3

class WithRetry:

    def __init__(self, retries_limit=RETRIES_LIMIT, allowed_exceptions=None):
        self.retries_limit = retries_limit
        self.allowed_exceptions = allowed_exceptions

    def __call__(self, operation):

        @wraps(operation)
        def wrapped(*args, **kwargs):
            last_raised = None

            for _ in range(self.retries_limit):
                try:
                    return operation(*args, **kwargs)
                except self.allowed_exceptions as e:
                    print(f"retrying {operation} due to {e}")
                    last_raised = e
            raise last_raised
        return wrapped

@WithRetry(retries_limit=5, allowed_exceptions=ZeroDivisionError)
def run_with_custom_parameter(task):
    return task.run()

class Task:
    def __init__(self):
        self._test = None

    @staticmethod
    def run():
        return 10/0


if __name__ == "__main__":
    task = Task()

    run_with_custom_parameter(task)
```

``` python
retrying <function run_with_custom_parameter at 0x10aff8dc0> due to division by zero
retrying <function run_with_custom_parameter at 0x10aff8dc0> due to division by zero
retrying <function run_with_custom_parameter at 0x10aff8dc0> due to division by zero
retrying <function run_with_custom_parameter at 0x10aff8dc0> due to division by zero
retrying <function run_with_custom_parameter at 0x10aff8dc0> due to division by zero
```



* 여기서 제일 중요한 부분은 클래스로 구현한 데코레이터가 어떻게 동작하는지이다.
1. @연산전에 전달된 파라미터를 사용해 데코레이터로 객체를 생성
2. 데코레이터 객체는  `__init__` 로직에 따른 객체 초기화
3. 마지막으로 @ 연산이 호출됨.
4. 호출된 함수를 래핑하여 `__call__` 매직 메서드를 호출























































