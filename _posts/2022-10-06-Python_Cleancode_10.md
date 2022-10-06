---
layout: post
toc: true
title: Python Cleancode - 10
categories: Python
tags: [Python, OOP, Cleancode]
author:
  - Heeseong
---

# 디스크립터로 더 멋진 객체 만들기

* 디스크립터가 무엇인지, 어떻게 동작하는지, 어떻게 효율적으로 구현하는지 알아보자.
* 두 가지 유형의 디스크립터 데이터, 비데이터 디스크립터의 개념적 차이와 세부구현의 차이를 분석해보자.
* 디스크립터 활용법
* 좋은 사용 예를 알아보자




## 디스크립터 개요

### 디스크립터 메커니즘
* 디스크립터 동작방식은 복잡하지 않으나, 세부 구현 시의 주의 사항이 많다는 점에 유의하자

* 디스크립터 구현 시 최소 2개의 클래스가 필요하다.
  * 클라이언트 클래스 - 디스크립터 구현 기능을 활용할 도메인 모델(객체)
  * 디스크립터 클래스 - 디스크립터 프로토콜을 구현한 클래스의 인스턴스

* 디스크립터 클래스는 다음 매직 메서드 중에 최소 1개 이상을 포함해야한다.
  * `__get__`
  * `__set__`
  * `__delete__`
  * `__set_name__`


* 아래와 같은 네이밍 컨벤션을 사용한다.

| 이름 | 의미 |
|-|-|
| ClientClass | 디스크립터 구현체의 기능을 활용할 도메인 추상화 객체, 디스크립터의 클라이언트이다. 클래스 속성으로 디스크립터를 갖는다.(DescriptorClass의 인스턴스) |
| DescriptorClass | 디스크립터 클래스. 이 글래스는 앞으로 언급할 디스크립터 프로토콜을 따르는 매직 메서드를 구현해야만 한다. |
| client | ClientClass의 인스턴스 client = ClientClass() |
| descriptor | DescriptorClass의 인스턴스 descriptor = DescriptorClass, 이 객체는 클래스 속성으로서 ClientClass에 위치한다. |




> 명심해야 할 중요한 사실은 이 프로토콜이 동작하려면 디스크립터 객체가 클래스 속성으로 정의되어야 한다는 것

* 인스턴스 속성으로 생성하면 동작하지 않으므로, init 메서드가 아니라 `클래스 본문`에 있어야 한다.

* 디스크립터 프로토콜의 일부만 구현해도 된다는 것에도 유의하자. 꼭 모든 것을 구현할 필요가 없다.

* 런타임 중에 어떻게 동작하는 것일까?

* 일반적인 클래스의 속성 또는 프로퍼티

``` python
class Attribute:
    vlaue = 42

class Client:
    attribute = Attribute()


Client().attribute
>>> <__main__.Attribute object at 0x7ff37ea90940>

Client().attribute.value
>>> 42
```


* 디스크립터의 경우 약간 다르게 동작한다.
* 클래스 속성을 객체로 선언하면 디스크립터로 인식되고, 클라이언트에서 해당 속성을 호출하면 객체 자체를 반환하는 것이 아니라 `__get__` 매직 메서드의 결과를 반환한다.

``` python
class DescriptorClass:
    def __get__(self, instance, owner):
        if instance is None:
            return self

        print(f"Call: {self.__class__.__name__}.__get__({instance}, {owner})")
        return instance

class ClientClass:
    descriptor = DescriptorClass()

if __name__ == "__main__":

    client = ClientClass()
    print(client.descriptor)
    print(client.descriptor is client)
```

```
Call: DescriptorClass.__get__(<__main__.ClientClass object at 0x109783160>, <class '__main__.ClientClass'>)
<__main__.ClientClass object at 0x109783160>
Call: DescriptorClass.__get__(<__main__.ClientClass object at 0x109783160>, <class '__main__.ClientClass'>)
True
```

* 이 예제에서는 클라이언트 자체를 그대로 반환했으므로 마지막 비교 문장은 True가 된다.
* 이걸 시작으로 더 복잡한 추상화와 더 나은 데코레이터를 만들어 볼 것이다.
* 디스크립터를 통해 완전히 새롭게 프로그렘의 제어 흐름을 변경할 수 있다.

* 이 도구를 사용해 `__get__` 메서드 뒤쪽으로 모든 종류의 논리를 추상화할 수 있으며 클라이언트에게 내용을 숨긴 채로 모든 유형의 변환을 투명하게 실행할 수 있다.






