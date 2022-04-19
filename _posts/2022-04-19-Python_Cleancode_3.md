---
layout: post
toc: true
title: Python Cleancode - 3
categories: Python
tags: [Python, OOP, Cleancode]
author: 
    - Heeseong
---


# Pythonic 코드
* 인덱싱 가능한 객체를 올바른 방식으로 구현하기
* 시퀀스와 이터러블 구현
* 컨텍스트 관리자를 만드는 모범 사례
* 매직 메서드를 사용해 보다 관용적인 코드 구현
* 파이썬에서 부작용을 유발하는 흔한 실수 피하기

## 인덱스와 슬라이스

* 튜플, 문자열, 리스트의 특정 요소를 가져오려고 한다면 루프돌지 말고 인덱스, 슬라이스하는게 좋음

## 자체 시퀀스 생성

* 객체가 리스트를 어떻게 래핑하는지
* `__getitem__` 클래스의 인덱스에 접근할 때 자동으로 호출되는 메서드이다.

``` python
class Items:
    def __init__(self, *values):
        self._values = list(values)

    def __len__(self):
        return len(self._values)

    def __getitem__(self, item):
        return self._valuse.__getitem__(item)
```

* 범위로 인덱싱하는 결과는 해당 클래스와 같은 타입의 인스턴스여야 한다.
* slice에 의해 제공된 범위는 파이썬이 하는 것 처럼 마지막 요소는 제외

## 컨텍스트 관리자

* 주요 동작 전후에 작업을 실행하려고 할 때 유용하다.
    * 파일 열 때 디스크립터 누수 막기 위해 작업이 끝나면 적절히 닫히길 기대
    * 소켓 연결 등 할당된 리소스를 해제해야함
* 일반적인 방법은 finally 블록에 정리코드를 넣는 것
* 간단한 예)

``` python
fd = open(filename)
try:
    process_file(td)
finally:
    fd.close()
```

> 똑같은 기능을 매우 우아하고 파이썬스러운 방법으로 구현할 수도 있다 .

``` python
with open(filname) as fd:
    process_file(fd)
```

* with 문은 컨텍스트 관리자로 진입하게 한다. 이 경우에 open 함수는 컨텍스트 관리자 프로토콜을 구현해서,
예외가 발생해도 블록이 완료되면 파일이 자동으로 닫힌다.
* `__enter__` 와 `__error__` 두 개의 매직 메서드로 구성됨
* as 뒤 변수에 메서드가 반환하는 걸 저장함
* 결론적으로 컨텍스트 관리자는 관심사를 분리하고 독립적으로 유지되어야하는 코드를 분리하기 좋은 방법이다.

### 스크립트로 DB 백업을 하려는 경우

* 백업은 db가 실행되고 있지 않은 동안에만 진행, 백업 끝나면 성공 여부에 관계 없이 db 시작

``` python
def stop_database():
    run("systemctl stop mysqld.service")

def start_database():
    run("systemctl start mysqld.service")

class DBHandler:
    def __enter__(self):
        stop_database()
        return self

    def __exit__(self, exc_type, ex_value, ex_traceback):
        start_database()

def db_backup():
    run("pg_dump database")

def main():
    with DBHandler():
        db_backup()
```

## 컨텍스트 관리자 구현

* 앞선 방법 외에 contextlib 모듈 사용하여 구현
* 함수에 contextlib.contextmanager 데이코레이터를 적용하면 해당 함수의 코드를 컨텍스트 관리자로 변환한다.
* 함수는 제너레이터 함수의 형태여야함

``` python
import contextlib

@contextlib.contextmanager
def db_handler():
    stop_database()
    yield
    start_database()

with db_handler():
    db_backup()
```

* 먼저 제너레이터 함수를 정의하고 데코레이터 적용
* yield 문을 사용했으므로 제너레이터 함수가 됨.
* yield 문 앞의 모든 것은 `__enter__` 메서드의 일부처럼 취급됨
* yield 문 뒤의 모든 것은 `__exit__` 로직 일부처럼 취급됨

* 또 다른 방법으로 contextlib.ContextDecorator를 사용하는 방법
    * 컨텍스트 관리자 안에서 실행될 함수에 데코레이터를 적용하기 위한 로직을 제공하는 믹스인 클래스
* 이전 예제와의 다른점은 `````with````` 문이 없다는 것
    * 그저 함수를 호출하기만 하면 offline\_backup 함수가 컨텍스트 관리자 안에서 자동으로 실행됨 - 원본 함수를 래핑하는 데코레이터가 하는 일
* 하지만 컨텍스트 관리자 내부에서 사용하고자 하는 객체를 얻을 수 없다는 단점이 있음
    * `with offline_backup() as bp:` 처럼 사용할 수 없음.
    * `__enter__` 메서드가 반환한 객체를 사용해야 하는 경우는 이전의 접근 방법 선택해야함

``` python
class dbhandler_decorator(contextlib.ContextDecorator):
    def __enter__(self):
        stop_database()

    def __exit__(self, ext_type, ex_value, ex_traceback):
        start_database()


@dbhandler_decorator()
def offline_backup():
    run(pg_dump database)
```

## 프로퍼티, 속성과 객체 메서드의 다른 타입들

* 파이썬은 private 속성 사용을 위해 `_` 로 시작하는 변수를 사용함 -> 실제로는 private으로 만드는게 아님

### 파이썬에서의 밑줄

* 기본적으로 객체의 모든 속성은 public이다.
* source, timeout 속성은 전자는 Public, 후자는 private 이지만,
* 실제로는 모두 접근 된다.
* `_timeout` connector 자체에서만 사용되고 호출자는 이 속성에 접근하지 않아야 한다.
* 즉 timeout 속성은 내부에서만 사요되고 바깥에서는 호출되지 않아서 동일한 인터페이스를 유지하므로, 언제든 필요한 경우에 안전하게 리팩토링할 수 있어야 하다.
* 객체는 외부 호출 객체와 관련된 속성과 메서드만을 노출해야 한다.
* 객체의 인터페이스로 노출하는 용도가 아니라면 모든 멤버는 접두사로 하나의 밑줄을 사용하는 것이 좋다.

``` python
class Connector:
    def __init__(self, source):
        self.source = source
        self._timeout = 60


conn = Connector("postgresql://localhost")
conn.source
>> 'postgresql://localhost'
conn._timeout
>> 60
conn.__dict__
>> { 'source' : 'postgresql://localhost', '_timeout': 60 }
```

* 이중 밑줄은 이름 맹글링이라하며, 이름의 속성을 만드는 것임
* conn.\_\_timeout으로 접근이 불가능하나, 이것이 private으로 만드는게 아닌, 이름 맹글링으로 아래와 같은 속성이 생긴 것임
* `_Connector__timeout`
* 여러 번 확장되는 클래스의 메서드를 이름 충돌 없이 오버라이드하기 위해 만들어진 것임
* ``````외부에 노출하지 않은 멤버는 _으로 private 을 정의하는 파이썬스러운 관습을 지키도록 하자``````

``` python
class Connector:
    def __init__(self, source):
        self.source = source
        self.__timeout = 60

    def connect(self):
        print("connecting with {0}s".format(self.__timeout))
```

### 프로퍼티

* 프로퍼티는 객체의 어떤 속성에 대한 접근을 제어하려는 경우 사용한다.
* 자바에서는 게터, 세터 사용하지만, 파이썬에서는 프로퍼티 사용 -> pythonic한 방법
* 사용자가 등록한 정보에 잘못된 정보가 입력되지 않게 보호하려고 한다고 해보자.

``` python
import re

EMAIL_FORMAT = re.compile(r"[^@]+@[^@]+[^@]+")

def is_valid_email(potentially_valid_email: str):
    return re.match(EMAIL_FORMAT, potentailly_valid_email) is not None

class User:
    def __init__(self, username):
        self.username = username
        self._email = None

@property
def email(self):
    return self._email

@email.setter
def email(self, new_email):
    if not is_valid_email(new_email):
        raise ValueError(f"유효한 이메일이 아니므로 {new_email} 값을 사용할 수 없음")
    self._email = new_email
```

* 첫번째 `@property` 메서드는 private 속성인 email 값을 반환한다. (getter)
* `@email.setter` 메서드는 setter
* 프로퍼티는 명령-쿼리 분리 원칙을 지키기 좋은 방법임
* 명명-쿼리 분리 원칙이란, 객체의 매서드는 상태 변환 혹은 상태 반환 둘 중 하나만 해야한다.


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