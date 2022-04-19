---
layout: post
toc: true
title: Python Cleancode - 2
categories: Python
tags: [Python, OOP, Cleancode]
author:
  - Heeseong
---

Docstring과 어노테이션

> 좋은 코드 레이아웃에서 가장 필요한 특성은 ``````일관성``````이다.

## Docstring과 어노테이션

* 코드를 문서화 하는 것과 주석은 다르다. 주석은 가급적 피해야만 한다.

### Docstring

* Docstring를 통해 데이터 타입이 무엇인지 설명하고 예제 제공이 가능함.
* 로직의 일부를 문서화하기 위해 코드 어딘가 배치
* `__doc__` 속성 생김

### 어노테이션

* 함수, 메서드를 거치면서 변수나 객체의 값이 무엇인지 명시
* Mypy 같은 도구를 사용해 탕비 힌트 등의 자동화된 검증을 실행할 수 있음
* `__annotations__` 속성 생김

``` python
class Point:
    def ___init__(self, lat, long):
        self.lat = lat
        self.long = long

def locate(latitude: fload, longitude: float) -> Point:
    """맵에서 좌표에 해당하는 객체를 검색"""
```

### Docstringr과 어노테이션을 이용

* 파라미터와 함수 반환 값의 예상 형태를 anstjghk

``` python
def data_from_response(response: dict) -> dict:
    """response에 문제가 없다면 response의 payload를 반환

    - response 사전의 예제::
    {
        "status": 200, # <int>
        "timestamp": ".....", # 현재 시간의 ISO 포맷 문자열
        "payload": { ... } # 반환하려는 dict data
    }

    - return dict example::
    { "data" : { .. } }

    - 발생 가능한 예외:
    - HTTP status가 200이 아닌 경우 ValueError 발생
    """

    if response["status"] != 200:
        raise ValueError
    return { "data": response["payload"] }
```