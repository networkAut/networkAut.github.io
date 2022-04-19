---
layout: post
toc: true
title: Python Cleancode - 2
categories: Python
tags: [Python, OOP, Cleancode]
author:
  - Heeseong
---


# [파이썬 클린코드]

* 좋은 코드 레이아웃에서 가장 필요한 특성은 ``````일관성``````이다.

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

## 이터러블 객체

* 파이썬은 기본적으로 반복 가능한 객체가 있음
    * 리스트, 튜플, 세트 및 dictionary
* 반복을 위해 정의한 로직을 사용해 자체 이터러블을 만들 수도 있다.
* 어터러블은 `__iter__` 매직 메서드를 구현한 객체
* 이터레이터는 `__next__` 매직 메서드를 구현한 객체
* 파이썬은 이터러블 프로토콜이라는 자체 프로토콜을 사용해 반복을 동작함
* `for e in myobject:` 형태로 반복이 가능한지를 확인
    * 객체가 `__next__` 나 `__iter__` 이터레이터 메서드 중 하나를 포함하는지 여부
    * 객체가 시퀀스이고, `__len__`과 `getitem__`를 모두 가졌는지 여부

### 참고

* What is the purpose of iterator?

The primary purpose of an iterator is to allow a user to process every element of a container while <b>isolating the user from the internal structure of the container.</b>

* Is iterator faster than for loop?

Iterator and for-each loop are faster than simple for loop for collections with no random access, while in collections which allows random access there is no performance change with for-each loop/for loop/iterator.

### 이터러블 객체 만들기

* 객체를 반복하려고하면 파이썬은 해당 객체의 ````iter()```` 함수를 호출한다.
* 함수가 있으면 호출

[일정 기간의 날짜를 하루 간격으로 반복하는 객체의 코드]

* for 루프는 앞서 만든 객체를 사용해 새로운 반복을 시작
* iter() 함수 호출하고, `__iter__ ` 매직 매서드 호출, 해당 매서드는 self를 반환하고 있으므로 객체 자신이 이터러블임을 나타내고 있음
* 따라서, 루프의 각 단계에서마다 자신의 next() 함수를 호출 -> `__next__` 매서드 호출
* ``````하지만, 두 개 이상의 for 루프에서 사용할 수 없음. 처음 루프가 StopIteration 을 발생시키면 두 번째 호출은 계속 에러 발생함. 반복 프로토콜이 작동하는 방식 때문``````

``` python
from datetime import timedelta

class DateRangeIterable:
    """자체 이터레이터 메서드를 가지고 있는 이터러블"""


    def __init__(self, start_date, end_date):
        self.start_date = start_date
        self.end_date = end_date
        self._present_dat = start_date

    def __iter__(self):
        return self

    def __next__(self):
        if self._present_day >= self.end_date:
            raise StopIteration
        today = self._present_day
        self._present_day += timedelta(days=1)
        return today
```

``` python
for day in DateRangeIterable(date(2019, 1,1), date(2019, 1, 5)):
    print(day)


2019-01-01
2019-01-02
2019-01-03
2019-01-04
```

- - -

* `__iter__` 가 self를 반환해서 발생한 문제를 호출 시 마다 새로운 이터레이터를 만드는 방법
* 매번 새로운 DateRangeIterable 인스턴스를 만들거나, `__iter__` 에서 제너레이터(이터레이터 객체)를 사용할 수도 있다.
* 제너레이터는 뒷단에서 자세히 보자\~

``` python
class DateRangeContainerIterable:
    def __init__(self, start_date, end_date):
        self.start_date = start_date
        self.end_date = end_date

    def __iter__(self):
        current_day = self.start_date
        while current_day < self.end_date:
            yield current_day
            current_day += timedelta(days=1)
```

## 시퀀스 만들기

* 객체에 `__iter()__` 매서드를 정의하지 않았지만, 반복하기를 원하는 경우가 있다.
    * iter() 함수는 객체에 iter 매직 매서드가 정의되어 있지 않으면 `__getitem__` 을 찾고 없으면 TypeError를 발생시킴
* 시퀀스는 `__len__` 과 `__getitem__` 을 구현하고 첫 번째 인덱스가 0부터 시작하여 포함된 요소를 한 번에 하나씩 차례로 가져올 수 있어야 한다.
* 이터러블은 한 번에 하나씩 보관, 생성 -> 메모리 적게 쓰지만 n번째 요소 얻기 위한 시간 복잡도 O(n)
* 시퀀스는 메모리는 많이 사용해도, n번째 요소 얻기 위한 시간 복잡도는 O(1)

``` python
class DateRangeSequence:
    def __init__(self, start_date, end_date):
        self.start_date = start_date
        self.end_date = end_date
        self._range = self._create_range()

    def _create_range(self):
        days = []
        current_day = self.start_date
        while current_day > self.end_date:
            days.append(current_day)
            current_day += timedelta(days=1)
        return days

    def __getitem__(self, day_no):
        retrun self._range[day_no]

    def __len__(self):
        return len(self._range)
```

## 컨테이너 객체

* 컨테이너는 `__contains__` 메서드를 구현한 객체로 일반적으로 Boolean 값을 반환한다.
* 이 메서드는 in 키워드가 발견될 때 호출된다.
* 코드의 가독성 높일 수 있음.

``` python
아래와 같이 해석됨
element in container
->
container.__contains__(element)
```

* 2차원 게임 지도에서 특정 우치에 표시를 해야한다고 생각해보자
* if 문 뭔 소린지 바로 알아 들을 수 없음
* 매번 경계선을 검사하기 위해 if문을 중복해서 호출함

``` python
def mark_coordinate(grid, coord):
    if 0 <= coord.x < grid.width and 0 <= coord.y < grid.height:
        grid[coord] = MARKED
```

* 지도에서 자체적으로 grid라 부르는 영역을 판단해주면? 그리고 더 작은 개체로 일을 위임하면?
    * 지도에게 특정 좌표가 포함되어 있는지만 물어보면 된다.

``` python
class Boundaries:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def __contains__(self, coord):
        x, y = coord
        return 0 <= x < self.width and 0 <= y < self.height


class Grid:
    def __init__(self, width, height):
        self.width = width
        self.height = height
        self.limits = Boundaries(width, height)


    def __contains__(self, coord):
        return coord in self.limits
```

* 구성이 간단하고, 위임을 통해 문제를 해결. 두 객체 모두 최소한의 논리를 사용했고, 메서드는 으집력이 있음

``` python
def mark_coordinate(grid, coord):
    if coord in grid:
        grid[coord] = MARKED
```