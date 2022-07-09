---
layout: post
toc: true
title: Python Cleancode - 7
categories: Python
tags: [Python, OOP, Cleancode]
author:
  - Heeseong
---


# SOLID 원칙 - 2

* SOLID 원칙을 파이썬 스러운 방식으로 구현하는 방법을 알아보자.
  * S: 단일 책임 원칙
  * O: 개방/폐쇄의 원칙
  * `L: 리스코프 치환 원칙`
  * `I: 인터페이스 분리 원칙`
  * `D: 의존성 역전 원칙`

## 리스포크 치환 원칙 - Liskov substitution principle
* 설계 시 안정성을 유지하기 위해 객체 타입이 유지해야하는 일련의 특성을 말함
* 어떤 클래스에서든 클라이언트는 특별한 주의 없이 하위 타입을 사용할 수 있어야 한다.
* 즉, 클라이언트는 클래스의 변경 사항과는 독립되어야 한다.

## 도구를 사용해 LSP 문제 검사하기
* Mypy 혹은 Pylint 같은 도구를 사용해 쉽게 검출할 수 있다.

```
(venv)  ~/PycharmProjects/cleancode  mypy iterable_unpacking.py 
Success: no issues found in 1 source file
```

### 메서드 서명의 잘못된 데이터타입 검사

* 코드 전체에 타입 어노테이션을 사용했고, Mypy를 설정했다면 초기에 기본 오류 여부와 LSP 준수 여부를 빠르게 확인할 수 있다.
* Event 클래스 하위 클래스 중 하나가 호환되지 않느 방식으로 메서드를 재정의하면 Mypy는 어노테이션을 검사하여 이를 확인한다.

``` python
class Event:
  . . . 
  def meets_condition(self, event_data: dict) -> bool:
    return False

class LoginEvent(Event):
  def meets_condition(self, event_data: list) -> bool:
    return bool(event_data)
```

```
lsp_bad_type.py:6: error: Argument 1 of "meets_condition" is incompatible with supertype "Event"; supertype defines the argument type as "Dict[Any, Any]"
lsp_bad_type.py:6: note: This violates the Liskov substitution principle
```

* 위 코드에서는 LSP 위반하였음
* 파생 클래스가 부모 클래스에서 정의한 파라미터와 다른 타입을 사용했기 때문에 다르게 동작한다.
* 반환 타입이 달라져도 동일한 오류가 발생한다.

### Pylint로 호환되지 않는 서명 검사
* 메서드의 서명 자체가 완전히 다른 경우가 자주 발생하는 LSP 위반 중 하나임



```python
class LogoutEvent(Event):
    def meets_condition(self, event_data: dict, override: bool) -> bool:
        if override:
            return True
```
* 이 경우 파라미터 추가하여 계층 구조의 호환성을 꺠는 클래스



### 애매한 LSP 위반 사례
* LSP를 위반한 것이 명확하지 않아서 자동화된 도구로 검사하기 애매할 수 있다.
* 이런 경우 코드 리뷰를 하면서 자세히 코드를 살펴볼 수 밖에 없음

* 클라이언트는 제공자가 유효성을 검사할 수 있도록 사전조건을 제공하고, 제공자는 클라이언트가 사후조건으로 검사할 값을 반환한다.

* 부모 클래스는 클라이언트와의 계약을 정의. 이 하위 클래스는 그러한 계약을 따라야 한다.
  * 하위 클래스는 부모 클래스에 정의된 것보다 사전조건을 엄격하게 만들면 안 된다.
  * 하위 클래스는 부모 클래스에 정의된 것보다 약한 사후조건을 만들면 안 된다.

* 이전 섹션에서 정의한 이벤트 계층구조를 LSP와 DbC 간의 관계를 보여주기 위해 변경해보자


``` python

class Event:
    def __init__(self, raw_data):
        self.raw_data = raw_data

    @staticmethod
    def meets_condition(event_data: dict):
        return False

    @staticmethod
    def meets_condition_pre(event_data: dict):
        """
        인터페이스 계약의 사전조건
        ''event_data'' 파라미터가 적절한 형태인지 유효성 검사
        """

        assert isinstance(event_data, dict), f"{event_data!r} is not a dict"
        for moment in ("before", "after"):
            assert moment in event_data, f"{moment} not in {event_data}"
            assert isinstance(event_data[moment], dict)

class UnknownEvent(Event):
    """데이터만으로 식별할 수 없는 이벤트"""

class LoginEvent(Event):
    @staticmethod
    def meets_condition(event_data: dict):
        return (
            event_data["before"]["session"] == 0
            and event_data["after"]["session"] == 1
        )

class LogoutEvent(Event):
    @staticmethod
    def meets_condition(event_data: dict):
        return (
            event_data["before"]["session"] == 1
            and event_data["after"]["session"] == 0
        )

class TransactionEvent(Event):
    """시스템에서 발생한 트랜잭션 이벤트"""
    @staticmethod
    def meets_condition(event_data: dict):
        return event_data["after"].get("transaction") is not None

class SystemMonitor:
    """시스템에서 발생한 이벤트 분류"""

    def __init__(self, event_data):
        self.event_data = event_data

    def identify_event(self):
        Event.meets_condition_pre(self.event_data)

        event_cls = next(
            (event_cls for event_cls in Event.__subclasses__() if event_cls.meets_condition(self.event_data)),
            UnknownEvent
        )

        return event_cls(self.event_data)
```

``` python
l1 = SystemMonitor({"before": {"session": 0}, "after": {"session": 1}})
print(l1.identify_event().__class__.__name__) # LoginEvent

l1 = SystemMonitor({"before": {"session": 1}, "after": {"session": 0}})
print(l1.identify_event().__class__.__name__) # LogoutEvent

l1 = SystemMonitor({"before": {"session": 1}, "after": {"session": 1}})
print(l1.identify_event().__class__.__name__) # UnknownEvent

l1 = SystemMonitor({"before": {}, "after": {"transaction": "Tx001"}})
print(l1.identify_event().__class__.__name__) # KeyError: 'session'
```

* 계약은 오직 최상위 레벨의 키 'before', 'after' 가 필수이고, 그 값 또한 dict 타입이어야 한다고만 명시 되어 있다.
* 하위 클래스에서 보다 제한적인 파라미터를 요구하는 경우 검사에 통과하지 못한다.
* 트랜잭션 이벤트는 올바르게 설계되어있다. 'transaction' 이라는 키에 제한을 두지 안혹 사용하고 있다.
  * 그 값이 있을 경우에만 사용하고 필수로 꼭 필요한 것은 아니다.

* 그러나 이전에 사용하던 LoginEvent, LogoutEvent 클래스는 'before', 'after' 의 'session이라는 키를 사용하기 때문에 그대로 사용할 수 없다. 이렇게 되면 계약이 깨지고 KeyError가 발생

* 이 문제는 TransactionEvent 클래스와 마찬가지로 대괄호 대신 .get() 메서드로 수정하여 해결할 수 있다.


``` python
class LoginEvent(Event):
    @staticmethod
    def meets_condition(event_data: dict):
        return (
            event_data["before"].get("session") == 0
            and event_data["after"].get("session") == 1
        )

class LogoutEvent(Event):
    @staticmethod
    def meets_condition(event_data: dict):
        return (
            event_data["before"].get("session") == 1
            and event_data["after"].get("session") == 0
        )
```

``` python
LoginEvent
LogoutEvent
UnknownEvent
TransactionEvent
```

* 두 클래스에서 대괄호를 .get()으로 수정 후 TransactionEvent 가 정상적으로 출력
* dict에 없는 key의 값을 가져올 떄 차이
  * [] - KeyError 발생, NoneType을 return 함
  * .get() - 아무것도 반환하지 않음. .get('key', default value) 처럼 기본 값을 설정할 수 있음

* 클래스 디자인을 할 떄는 실수로 메서드의 입력과 출력을 변경해서 원래 기대한 것과 달라지지 않도록 주의해야 한다.

### LSP 최종 정리
* LSP는 객체지향 소프트웨어 설계의 핵심이 되는 다형성을 강조하기 떄문에 좋은 디장니의 기초가 된다.
* 인터페이스의 메서드가 올바른 계층구조를 갖도록 하여 상속된 클래스가 부모 클래스와 다형성을 유지하도록 하는 것이다.
* 새로운 클래스가 원래의 계약과 호환되지 않는 확장을 하려고 하면 클라이언트와의 계약이 깨져서 결과적으로 그러한 확장이 가능하지 않을 것이다.
* LSP에서 제안하는 방식으로 신중하게 클래스를 디자인하면 계층을 올바르게 확장하는데 도움이 된다.
* 즉, LSP가 OCP에 기여한다고 말할 수 있다.

