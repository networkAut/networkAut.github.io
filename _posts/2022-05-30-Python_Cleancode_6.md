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

