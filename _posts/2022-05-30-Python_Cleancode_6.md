---
layout: post
toc: true
title: Python Cleancode - 6
categories: Python
tags: [Python, OOP, Cleancode]
author:
  - Heeseong
---


# SOLID 원칙

* SOLID 원칙을 파이썬 스러운 방식으로 구현하는 방법을 알아보자.
  * S: 단일 책임 원칙
  * O: 개방/폐쇄의 원칙
  * L: 리스코프 치환 원칙
  * I: 인터페이스 분리 원칙
  * D: 의존성 역전 원칙

## 단일 책임 원칙 - Single Responsibility Principle
> 소프트웨어 컴포넌트(클래스)가 단 하나의 책임을 가져야한다는 원칙
* 즉 하나의 책임 = 하나의 구체적인 일을 담당하는 것

* 어떤 경우에도 여러 책임을 가진 객체를 만들어서는 안 된다.
* 여기서 추구하는 것은 클래스에 있는 프로퍼티와 속성이 항상 메서드를 통해서 사용되도록 하는 것

* 이 원칙을 다른 방법으로도 생각해볼 수 있다. 

> 클래스의 메서드는 상호 배타적이며, 서로 관련이 없어야 하고 따라서 서로 다른 책임을 가지고 있으므로 더 작은 클래스로 분해할 수 있어야 한다.


### 너무 많은 책임을 가진 클래스
* 로그 파일, DB 읽어 이벤트 정보 읽어 로그별로 필요한 액션을 분류하는 애플리케이션

* SRP를 준수하지 않은 디자인.

|SystemMonitor|
|--|
| +load_activity() |
| +indentify_events() |
| +stream_events() |


``` python
class SystemMonitor:
    def load_activity(self):
        """소스에서 처리할 이벤트를 가져오기"""

    def indentify_events(self):
        """가져온 데이터를 파싱하여 도메인 객체 이벤트로 변환"""

    def stream_events(self):
        """파싱한 이벤트를 외부 에이전트로 전송"""
```

* 문제점
  * 독립적인 동작을 하는 메서드를 하나의 인터페이스에 정의 - 유지보수 어렵고 에러 발생하기 쉽다.
  * 데이터의 표현이 변경되었다고해서 시스템 모니터링하는 객체를 변경해서는 안된다.


### 책임 분산
* 보다 관리하기 쉽게 모든 메서드를 다른 클래스로 분리하여 각 클래스마다 단일 책임을 갖게 하자.


|SystemMonitor| AlertSystem | ActivityReader | SystemMonitor | Output |
|--|--|--|--|--|
| +load_activity() | +run() | +load() | |  |
| +indentify_events() | - | | +identify_events() |  |
| +stream_events() | - | | | +stream() |


* 각자의 책임을 가진 여러 객체를 만들고, 이들 객체와 협력하여 동일한 기능을 수행하는 객체를 만들 수 있다.

* 이떄 각 객체들은 특정한 기능을 캡슐화하고 나머지 객체에 영향을 미치지 않고 구체적 의미를 가지게 됨
* 데이터 소스에서 이벤트를 로드하는 방법을 변경해도 AlertSystem은 이러한 변경 사항과는 관련이 없으므로 SystemMonitor는 아무런 수정을 하지 않아도 됨

> 그러나 각 클래스가 딱 하나의 메서드를 가져야 한다는 것을 뜻하는 것은 아님에 주의하자.

## 개방/페쇄 원칙
* Open/Close Priciple
* 모듈이 개방되어 있으면서도 폐쇄되어야 한다는 원칙이다.

* 클래스를 디자인할 때는 유지보수가 쉽도록 캡슐화하여 확장에는 ````개방```` 수정에는 ````페쇄````

* 새로운 요구사항이나 도메인 변화에 잘 적응하는 코드를 작성해야한다는 뜻이다.

> 새로운 문제가 발생할 경우 새로운 것을 추가만 할 뿐 기존 코드는 그대로 유지해야 한다는 뜻이다.

### 개방/페쇄 원칙을 따르지 않을 경우 유지보수의 어려움

* 다른 시스템에서 발생하는 이벤트를 분류하는 기능
  * 데이터는 사전형태로 저장, 로그, 쿼리로 이미 데이터 수집했다고 가정

``` python
class Event:
    def __init__(self, raw_data):
        self.raw_data = raw_data

class UnknownEvent(Event):
    """데이터만으로 식별할 수 없는 이벤트"""

class LoginEvent(Event):
    """로그인 사용자에 의한 이벤트"""


class LogoutEvent(Event):
    """로그아웃 사용자에 의한 이벤트"""

class SystemMonitor:
    """시스템에서 발생한 이벤트 분류"""

    def __init__(self, event_data):
        self.event_data = event_data

    def identify_event(self):
        if(self.event_data["before"]["session"] == 0 and self.event_data["after"]["session"] == 1):
            return LoginEvent(self.event_data)
        elif(self.event_data["before"]["session"] == 1 and self.event_data["after"]["session"] == 0):
            return LogoutEvent(self.event_data)

        return UnknownEvent(self.event_data)


if __name__ == "__main__":
    l1 = SystemMonitor({"before": {"session": 0}, "after": {"session": 1}})
    print(l1.identify_event().__class__.__name__) # LoginEvent

    l1 = SystemMonitor({"before": {"session": 1}, "after": {"session": 0}})
    print(l1.identify_event().__class__.__name__) # LogoutEvent
    
    l1 = SystemMonitor({"before": {"session": 1}, "after": {"session": 1}})
    print(l1.identify_event().__class__.__name__) # UnknownEvent

```

* 이벤트 유형의 계층 구조와 이를 구성하는 일부 비즈니스 로직을 명확하게 알 수 있다.
* 이벤트를 식별할 수 없는 경우는 UnknownEvent를 반환한다. 이것은 None을 반환하는 대신 기본 로직을 가진 null 객체를 반환하는 패턴으로 다형성을 보장하기 위한 것임.

* 문제점
  1. 이벤트 유형을 결정하는 논리가 일체형으로 중앙 집중화 됨 - 이벤트 늘어날 수록 메서드도 커질 것
  2. 같은 방법으로 이 메서드가 수정을 위해 닫히지 않았다는 것을 알 수 있음 - 이벤트에 따른 메서드 수정 필요


### 확장성을 가진 이벤트 시스템으로 리팩토링

* 이전 예제의 문제점은 SystemMonitor 클래스가 분류하려는 구체 클래스와 직접 상호 작용한다는 점
* 추상화가 필요하다.

* SystemMonitor 클래스를 추상적인 이벤트와 협력하도록 변경하고, 이벤트에 대응하는 개별 로직은 각 이벤트 클래스에 위임하는 것
* 그런 다음 각각의 이벤트에 다형성을 가진 새로운 메서드를 추가해야함


* @staticmethod 의미
  * 데코레이터를 이용한 정적메소드
  * self 인자를 받지 않음
  * 유틸리티성 메소드를 생성할 때 사용
  * 클래스 내의 다른 속성이나 함수에 의존하지 않기 때문에 유틸리티성 함수로 static하게 정의하는 것이 맞다고 판단
  * self를 이용해서 클래스의 속성이나 함수를 사용하지 않는 경우... 즉, 의존성이 없는 경우 모두 staticmethod로 정의


``` python
class Event:
    def __int__(self, raw_data):
        self.raw_data = raw_data

    @staticmethod
    def meets_condition(event_data: dict):
        return False

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

class SystemMonitor:
    """시스템에서 발생한 이벤트 분류"""

    def __init__(self, event_data):
        self.event_data = event_data

    def identify_event(self):
        for event_cls in Event.__subclasses__():
            try:
                if event_cls.meets_condition(self.event_data):
                    return event_cls(self.event_data)
            except KeyError:
                continue
        return UnknownEvent(self.event_data)

if __name__ == "__main__":
    l1 = SystemMonitor({"before": {"session": 0}, "after": {"session": 1}}) # LoginEvent
    l2 = SystemMonitor({"before": {"session": 1}, "after": {"session": 0}}) # LogoutEvent
    l3 = SystemMonitor({"before": {"session": 1}, "after": {"session": 1}}) # UnknownEvent
```

* Event 클래스를 상속하는 분류를 위한 구체 클래스들에서 다형성을 구현할 수 있도록 @staticmethod로 meets_condition을 추상화
* SystemMonitor의 indetify_event에서 Event 클래스를 상속한 서브 클래스들에 대해 `__subclasses__()`를 통해 loop를 돌며 각 분류를 체크

> 분류 메서드는 이제 특정 이벤트 타입 대신 일반적인 인터페이스를 따르는 제네릭 이벤트와 동작한다. 이 인터페이스를 따르는 제네릭 들은 모두 meets_conditiond 메서드를 구현하여 다형성을 보장한다.

* __subclasses__() 메서드를 사용해 이벤트 유형을 찾는 것이 주목하자. 이제 새로운 유형의 이벤트를 지원하려면 단지 Event 클래스를 상속 받아 비즈니스 로직에 따라 meets_condition() 메서드를 구현하기만 하면 된다.




### 이벤트 시스템 확장
* 새로운 요구사항이 생겨서 모니터링 중인 시스템의 사용자 트랜잭션에 대응하는 이벤트를 지원해야 한다고 가정하자
* TransactionEvent라는 새로운 클래스를 추가하는 것만으로 기존 코드 잘 동작한다.


``` python
class Event:
    def __int__(self, raw_data):
        self.raw_data = raw_data

    @staticmethod
    def meets_condition(event_data: dict):
        return False

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
        for event_cls in Event.__subclasses__():
            try:
                if event_cls.meets_condition(self.event_data):
                    return event_cls()
            except KeyError:
                continue
        return UnknownEvent()


if __name__ == "__main__":
    l1 = SystemMonitor({"before": {"session": 0}, "after": {"session": 1}}) # LoginEvent
    l2 = SystemMonitor({"before": {"session": 1}, "after": {"session": 0}}) # LogoutEvent
    l3 = SystemMonitor({"before": {"session": 1}, "after": {"session": 1}}) # UnknownEvent
    l4 = SystemMonitor({"after": {"transaction": "Tx001"}}) # TransactionEvent
```
* 새 이벤트 추가 했지만 SystemMonitor.identify_event() 메서드는 전혀 수정하지 않은 것에 주목하자 !!
* 새로운 유형의 이벤트에 대해서 폐쇄되어 있다.
* 반대로 Event 클래스 필요할 때마다 새로운 유형의 이벤트를 추가할 수 있게 해준다. - 이벤트는 새로운 타입 확장에 대해 개방되어 있다고 말할 수 있다.

> 이것이 바로 이 원칙의 진정한 본질이다. 도메인에 새로운 문제가 나타나도 기존 코드를 수정하지 않고 새 코드를 추가하기만 하면 된다.


### OCP 최종 정리
* 이 원칙은 다형성의 효과적인 사용과 밀접하게 관련되어 있다. 다형성을 따르는 형태의 계약을 만들고 모델을 쉽게 확장할 수 있는 일반적인 구조로 디자인 하는 것이다.

* 코드를 변경하지 않고 기능을 확장하기 위해서는 보호하려는 추상화에 대해서 (여기서는 새로운 이벤트) 적절한 폐쇄를 해야 함.