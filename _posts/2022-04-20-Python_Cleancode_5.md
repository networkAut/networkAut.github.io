---
layout: post
toc: true
title: Python Cleancode - 5
categories: Python
tags: [Python, OOP, Cleancode]
author:
  - Heeseong
---


# 좋은 코드의 일반적인 특징

> 클린 코드에 대해서 말하면 디자인 보다는 세부 구현의 모범 사례에 대해서만 생각하기 쉽다.
하지만 잘못된 생각. 코드가 디자인이고, 디자인이 코드다.


* 궁극적인 목표는 코드를 가능한 견고하고, 결함을 최소화하고 완전히 자명하도록 하는 것
* 더 높은 수준의 추상화 디자인 원칙에 중점을 둔다.


## 목표
* 견고한 소프트웨어의 개념을 이해
* 작업 중 잘못된 데이터를 다루는 방법
* 새로운 요구 사항을 쉽게 받아들이고 확장할 수 있는 유지보수가 쉬운 소프트웨어 설계
* 재사용 가능한 소프트웨어 설계
* 개발팀의 생산성을 높이는 효율적인 코드 작성


## 계약에 의한 디자인
* 애플리케이션의 책임을 나누어 레이어, 컴포넌트로 분리한 경우 서로 간의 교류에 대해 고민해보자

컴포넌트는 기능을 숨겨 캡슐화하고, 함수를 사용할 고객에게는 API를 노출해야한다.
컴포넌트의 함수, 클래스, 메서드는 특별한 유의사항에 따라 동작해야하며, 그렇지 않을 경우 코드가 깨짐,
클라이언트의 경우 호출 실패하고 결함이 발생


* 예를 들어 정수 파라미터를 받는 함수에 문자열을 전달하면서 호출한 경우
  * 사용자가 실수를 하여 잘못 호출했기 때문에 절대로 실행이 되어서는 안된다.



## 사전조건 - Precondition
> 함수나 메서드가 제대로 동작하기 위해 보장해야하는 모든 것.
한마디로 적절한 데이터를 전달하는 것이다. 

* 포인트는 함수가 데이터의 유효성을 어디서 할지이다!!
  * 실제로 개발할 때 많이 했던 고민...

* 관용적인 방법 - 클라이언트가 함수를 호출하기 전에 모든 유효성 검사
  * 깨진 데이터 조차 수용하기 때문에 관용적

* 까다로운 방법 - 함수가 자체적으로 로직을 실행하기 전에 검사

> 까다로운(demanding) 방법을 쓰자. 가장 안전하고 견고하다. 이러한 방버에도 중복 제거 원칙을 지키자. 검증을 양쪽에서 하지 않도록


## 사후조건 - Postcondition

> 메서드 또는 함수가 반환된 후의 상태를 강제하는 계약의 일부

* 사전 조건에 맞게 함수가 호출 되었다면, 사후조건 검증에 통과하고 클라이언트는 반환 객체를 아무 문제 없이 사용할 수 있어야 한다.

## 방어적 프로그래밍
방어적 프로그램의 주요 주제는 아래 두 가지에 대한 이야기
1. 에러 핸들링 프로시저
  - 예상할 수 있는 시나리오의 오류를 처리하는 방법
2. 어썰션(assertion)
  - 발생하지 않아야하는 오류를 처리하는 방법

## 에러 핸들링
일반적으로 데이터 입력 시 자주 사용됨

> 예상되는 에러에 대해서 실행을 계속할 수 있을지, 아니면 극복할 수 없는 오류라 프로그램을 중단할지 결정하는 것

* 에러 처리 방법
  - 값 대체
  - 에러 로깅
  - 예외 처리

### 값 대체
* 일부 시나리오에서 잘못된 값 생성이나, 전체 프로그램 종료될 위험이 있을 경우 값을 대체할 수 있음.
* 기본 값, 잘 알려진 상수 혹은 초기 값으로 바꾸어 정합설을 깨지 않는 것
  * 실제로 API 개발 과정에서 필수로 입력 받아야하는 항목에 대해서 클라이언트가 필수값 누락 상태로 호출하였을 때 어떻게 처리해야하는지 고민한 기억이 난다..

> 약간 다른 방법으로 안전한 선택은 제공되지 않은 데이터에 기본 값을 사용하는 것

* 설정되지 않은 환경 변수의 기본 값 등

* 2번째 라인에서 configuration에 정의 되지않은 key에 대해서 localhost라는 기본 값을 주어 값을 대체 한 것임. 기본 값 지정하지 않을 경우 None을 반환

``` python
configuration = { "dbport": 3306 }
print(configuration.get("dbhost", "localhost")) # localhost
print(configuration.get("dbport")) # 3306
print(configuration) # { "dbport": 3306 }
```

* 함수 파라미터로 기본 값 전달 가능함. 그런데 이런 기본 값 설정에는 일부 오류를 숨길 수 있는 부분을 고려해야함

``` python
def connect_database(host="localhost", port=3306):
    logger.info(f"다음 정보로 데이터베이스에 접속: {host}, {port}")
```




### 예외 처리
* 사전조건 검증 실패 등 호출자에게 실패했음을 알리는 것이 좋은 선택인 경우가 있음

* 함수는 심각한 오류에 대해 명확하고 분명하게 알려줘서 적절하게 해결할 수 있도록 해야 한다.

> 예외적인 상황을 명확하게 알려주고 원래의 비즈니스 로직에 따라 흐름을 유지하는 것이 중요함

* 예외를 사용하여 시나리오나, 비즈니스 로직을 처리하려고 하면 프로그램의 흐름을 읽기 어려워진다. go-to문 같은 것을 사용하느 경우가 예시


* 예외는 대개 호출자에게 잘못을 알려주는 것, 또한 캡슐화를 약화시키기 때문에 신중하게 사용해야한다.
  * 함수에 예외가 많을수록 호출자가 함수에 대해 많은 것을 알아야만 한다. 너무 많은 예외를 발생시키면 문맥에서 자유롭지 않다는 것을 의미한다.

> 이것은 함수가 응집력이 약하고, 너무 많은 책임을 가지고 있다는 것을 알 수 있음. 여러개의 작은 것으로 나누어야 한다는 신호일 수 있다.


### 올바른 수즌의 추상화 단계에서 예외 처리





``` python 
class DataTransport:
    """다른 레벨에서 예외를 처리하는 객체의 예"""

    retry_threshold: int = 5
    retry_n_times: int = 3

    def __init__(self, connector):
        self._connector = connector
        self.connection = None

    def deliver_event(self, event):
        try:
            self.connect()
            data = event.decode()
            self.send(data)
        except ConnectionError as e:
            print(f"연결 실패: {e}")
            raise
        except ValueError as e:
            print(f"{event} 잘못된 데이터 포함: {e}")
            raise

    def connect(self):
        for _ in range(self.retry_n_times):
            try:
                self.connection = self._connector.connect()
            except ConnectionError as e:
                print(f"{e}: 새로운 연결 시도 {self.retry_threshold}")
                time.sleep(self.retry_threshold)
            else:
                return self.connection

        raise ConnectionError(f"{self.retry_n_times} 번째 재시도 연결 실패")

    def send(self, data):
        return self.connection.send(data)
```

* deliver_event()에서 connection error와 value error는 무슨 관계일까? 관련이 없다. 책임을 분산해보자
* Connection Error는 connect() 함수 내에서 처리 되어야한다.
  * 이 메서드가 재시도를 지원하는 경우 이 안에서 처리를 할 수 있다.

* Value Error 는 evnet의 decode()에 속한 에러이다. 이러면 deliver_event()에서 에러를 catch할 필요가 없음

> 따라서 deliver_event() 함수는 다른 함수로 분리되어야만 한다. 연결 관리는 작은 함수로 충분하다.



``` python
def connect_with_retry(connector, retry_n_times, retry_threshold=5):
    """connector의 연결을 맺는다. <retry_n_times> 회 재시도,

    연결에 성공하면 connection 객체 반환
    재시도까지 모두 실패하면 ConnectionError 발생
    """

    for _ in range(retry_n_times):
        try:
            return connector.connect()
        except ConnectionError as e:
            print(f"{e} 새로운 연결 시도 {retry_threshold}")
            time.sleep(retry_threshold)

    exec = ConnectionError(f"{retry_n_times} 번째 재시도 연결 실패")
    raise exec


class DataTransport:
    """다른 레벨에서 예외를 처리하는 객체의 예"""

    retry_threshold: int = 5
    retry_n_times: int = 3

    def __init__(self, connector):
        self._connector = connector
        self.connection = None

    def deliver_event(self, event):
        self.connection = connect_with_retry(self._connector, self.retry_n_times, self.retry_threshold)
        self.send(event)

    def send(self, event):
        try:
            return self.connection.send(event.decode())
        except ValueError as e:
            print(f"{event} 잘못된 데이터 포함: {e}")
            raise
```

* connect 함수를 retry 가능하도록 수정하여, 함수로 분리하고 호출한 모습. 또한 예외 처리를 각 함수로 분리

### Traceback 노출 금지
* traceback 정보는 로깅하여 문제를 해결할 수 있는 정보를 남기는 것은 좋으나 이것은 절대 사용자에게 노출되어서는 안된다.
  * 보안적으로 위험

* 예외 전파 시 중요한 정보를 제공하지 않도록 주의해야한다.

### 비어있는 except 블록 지양

> 파이썬의 안티패턴 중에서도 가장 악마 같은 패턴이다. 일부 오류에 대비하여 프로그램을 방어하는 것은 좋지만, 너무 방어적인 것은 더 심각한 문제로 이어질 수 있다.

* 다음과 같은 경우 오류를 발생 시키지 않는다.

```python
try:
  process_data()
except:
  pass
```

* 이것은 가장 큰 문제는 절대 실패하지 않는다는 것이다. 심지어 실패해야만 할 때 조차도..


* 두 가지 대안
  1. 보다 구체적인 예외를 사용
  2. except 블록에서 실제 오류 처리를 한다.

> 제일 좋은 방법은 두 항목을 동시에 적용하는 것 

### 원본 예외 포함
* 오류 처리 과정에서 다른 오류를 발생시키고 메시지를 변경할 수도 있다. 이 경우 원래 예외를 포함하는 것이 좋음
* raise <e> from <original_exception>






## 관심사의 분리
* 책임이 다르면 컴포넌트, 계층 또는 모듈로 분리되어야 한다. 각 부분은 기능의 일부(관심사)에 대해서만 책임을 지며, 나머지 부분에 대해서는 알 필요가 없다.

* 관심사 분리의 목표는 파급 효과를 최소화하여 유지보수성을 향상시키는 것이다.
  * 일반적으로 파이썬 모듈, 패키지 그리고 모든 소프트웨어 컴포넌트에 대해서 적용된다.

### 응집력(cohesion)과 결합력(coupling)
* 응집력: 객체가 작고 잘 정의된 목적을 가져야하며 가능하면 작아야 한다는 것을 의미
* 결합력: 두 개 이상의 객체가 서로 어떻게 의존하는지를 나타낸다.
  * 객체 또는 메서드가 서로 너무 의존적이라면 바람직하지 못하다.
  1. 낮은 재사용성
  2. 파급 효과
  3. 낮은 수준의 추상화

> 높은 응집력과 낮은 결합력을 갖도록 노력하자

## 개발 지침 약어

### DRY / OAOO
* DRY - Do not Repeat Yourself
* OAOO - Once and Only Once

* 코드에 있는 지식은 단 한 곳에 정의 되어야 한다. 코드 변경하려고 할 때 수정이 필요한 곳은 단 한 곳 !



``` python
def process_students_list(students):
    # 중간 생략 ...

    students_ranking = sorted(
        students, key=lamda s: s.passed * 11 - s.failed * 5 - s.year * 2
    )

    # 학생별 순위 출력
    for student in students_ranking:
        print(
            "이름 {0}, 점수: {1}".format(
                student.name,
                (student.passed * 11 - student.failed * 5 - student.year * 2)
            )
        )
```
* sorted key로 사용되는 lamda가 특별한 도메인 지식을 나타내지만 아무런 정의가 없다는 것에 유의하자
* 순위 출력하는 동안 중복이 발생 -> 도메인 문제에 대한 지식이 사용된 경우 의미를 부여해야 한다.
* 중복으로 부터 덜 고통받고, 이해하기 쉬운 코드가 된다~



``` python
def score_for_student(student):
    return student.passed * 11 - student.failed * 5 - student.year * 2

def process_students_list(students):
    # 중간 생략 ...

    students_ranking = sorted(students, key=score_for_student)

    # 학생별 순위 출력
    for student in students_ranking:
        print(
            "이름 {0}, 점수: {1}".format(
                student.name, score_for_student(student)
            )
        )

```

* 중복 피하는 가장 쉬운 방법은 함수로 빼내는 것이지만, 경우에 따라 다를 수 있다.
* 전체적으로 추상화를 하지 않은 경우 완전히 새로운 객체를 만드는 것이 좋다.
* 컨텍스트 관리자도 방법이 될 수 있음
* 이터레이터, 제너레이터가 코드의 반복을 피하는데 도움이 될 수 있으며 데코레이터 또한 마찬가지다






### YAGNI - You Ain't Gonna Need it
* 과잉 엔지니어링을 하지 않기 위해 솔루션 작성 시 계속 염두에 두어야하는 원칙이다.
* 많은 개발자들이 미래의 모든 요구사항을 고려하여 매우 복잡한 솔루션을 만들고, 추상화를 하여 읽기 어렵고, 유지보수가 어렵고, 이해하기 어려운 코드를 만든다.

> 현재의 요구사항도 제대로 처리하지 못한 채 미래의 요구 사항을 예측하는 것은 유지보수가 가능한 소프트웨어를 만드는 것이 아니다.


### KIS - Keep It Simple
* 문제를 올바르게 해결하는 최소한의 기능을 구현하고 필요한 것 이상으로 솔루션을 복잡하게 만들지 않도록 해야 한다.

> 디자인이 단순할 수록 유지 관리가 쉽다 !

* 코드 측면의 단순함이란 문제에 맞는 가장 작은 데이터 구조를 사용하는 것을 의미한다. -> 표준 라이브러리에 대부분 있다
* 때로는 코드를 지나치게 복잡하고, 필요한 것보다 많은 함수를 만들 수 있다.




``` python
class ComplicatedNamespace:
    """프로퍼티를 가진 객체를 초기화하는 복잡한 예제"""

    ACCEPTED_VALUES = ("id_", "user", "location")

    @classmethod
    def init_with_data(cls, **data):
        instance = cls()
        for key, value in data.items():
            if key in cls.ACCEPTED_VALUES:
                setattr(instance, key, value)
        return instance
```
* 위의 코드는 제공된 키워드 파라미터 세트에서 네임 스페이스를 작성하지만 다소 복잡한 코드 인터페이스를 가지고 있음
* classmethod 통해서 함수 직접 호출
* 객체를 초기화 하기 위해서 클래스 호출 불필요하며, 반복을 통해 setattr 호출하는 것은 더 이상함. 사용자에게 노출된 인터페이스 또한 분명하지 않다.

``` python
cn = ComplicatedNamespace.init_with_data(id_=42, user="root", location="127.0.0.1", extra="excludede")
cn.id_, cn.user, cn.location
# (42, 'root', '127.0.0.1')

hasattr(cn, "extra")
# False 
```
* 또한 사용자는 초기화를 위해 init_with_data라는 일반적이지 않은 메서드 이름을 알아야함.
  * 다른 객체 초기화할 때는 __init__ 메서드를 사용하는 것이 훨씬 간편할 것이다.

```python
class Namespace:
    """create an object from keyword arguments"""

    ACCEPTED_VALUES = ("id_", "user", "location")

    def __init__(self, **data):
        accepted_data = {
            k: v for k, v in data.items() if k in self.ACCEPTED_VALUES
        }
        self.__dict__.update(accepted_data)


ns = Namespace(id_=42, user="root", location="127.0.0.1")
ns.id_, ns.user, ns.location
# (42, 'root', '127.0.0.1')
```

> 파이썬의 철학을 기억하자: 단순한 것이 복잡한 것보다 낫다.



### EAFP/LBYL

* EAFP - Easier to Ask Forgiveness than Permission
  * 일단 코드를 실행하고 실제 동작하지 않을 경우에 대응한다.

* LBYL - Look Before You Leap
  * 도약하기 전에 먼저 무엇을 사용하려고 하는지 확인하라.
  * ex) 파일을 사용하기 전에 파일을 사용할 수 있는지 확인하는 것


``` python
if os.path.exists(filename):
  with open(filename) as f:
    ...
```

* 위는 다른 언어는 모르겠으나, 파이썬스러운 방식은 아니다.
* 파이썬은 EAFP 방식으로 만들어졌으며 그렇게 하는 것을 권장한다 !!

``` python
try:
  with open(filename) as f:
    ...
except FileNotFoundError as e:
  logger.error(e)
```


## 컴포지션과 상속

* 개발자는 종종 필요한 클래스들의 계층 구조를 만들고 각 클래스가 구현해야하는 메서드를 결정하는 것으로 개발을 시작한다.
* 상속은 강력한 개념이지만, 단순히 코드 재사용을 위해 부모 클래스에 있는 메서드를 공짜로 얻을 수 있기 때문이라면 좋지 않다.

> 코드를 재사용하는 올바른 방법은 여러 상황에서 동작 가능하고, 쉽게 조합할 수 있는 응집력 높은 객체를 사용하는 것! 


### 상속이 좋은 선택인 경우

* 파생 클래스를 만드는 것은 양날의 검.
* 새로운 하위 클래스를 만들 떄 클래스가 올바르게 정의되었는지 확인하기 위해서는 상속된 모든 메서드를 실제로 사용할 것인지 생각해보는 것이 좋다.
* 만약 대부분의 메서드를 필요로하지 않고 재정의해야하면 아래와 같은 이유의 설계 오류
  - 상위 클래스는 잘 정의된 인터페이스 대신 막연한 정의와 너무 많은 책임을 가졌다.
  - 하위 클래스는 확장하려고하는 상위 클래스의 적절한 세분화가 아니다.



* 상속을 잘 활용한 예
  * public 메서드와 속성 인터페이스를 정의한 컴포넌트.
  * 클래스의 기능을 그대로 물려 받으면서 추가 기능을 더하려는 경우 또는 특정 기능을 수정하려는 경우
  * http.server 패키지의 BaseHTTPRequestHandler 기본 클래스와 이 기본 인터페이스의 일부를 추가하거나 변경하여 확장하는 SimpleHTTPRequestHandler 하위 클래스가 잘 상속한 예시
  * 인터페이스 정의는 상속의 또 다른 좋은 예, 어떤 객체에 인터페이스 방식을 강제하고자 할 때 구현하지 않은 기본 추상 클래스를 만들고, 상속하는 하위 클래스에서 적절한 구현하도록 하는 것




### 상속 안티패턴
> 상속 안티패턴 == 전문화
* 부모 클래스는 파생 클래스의 공통 정의의 일부가 된다. 클래스의 public 메서드는 부모 클래스가 정의하는 것과 일치해야한다.

* 예를 들어 BaseHTTPRequestHandler에서 파생된 클래스가 handle()이라는 메서드를 구현했다면, 부모 클래스의 일부를 오버라이딩한 것이다.
* HTTP 요청과 관련되어 보이는 이름의 메서드가 있다면 올바른 위치 라고 볼 수 있다. 그러나 process_purchase()와 같은 메서드의 경우에는 상속된 메서드라고 생각하지 않을 것이다.

* 위 설명은 당연해 보일 수 있지만 개발자가 코드 재사용만을 목적으로 상속을 사용하려고할 때 매우 자주 발생한다.


``` python
import collections
from datetime import datetime


class TransactionalPolicy(collections.UserDict):
    """잘못된 상속의 예"""

    def change_in_policy(self, customer_id, **new_policy_data):
        self[customer_id].update(**new_policy_data)

policy = TransactionalPolicy(
    {
        "client001": {
            "fee": 1000.0,
            "expiration_date": datetime(2020, 1, 3)
        }
    }
)

policy["client001"] # {'client001': {'fee': 1000.0, 'expiration_date': datetime.datetime(2020, 1, 3, 0, 0)}}
policy.change_in_policy("client001", expiration_date=datetime(2022, 4, 3))
policy["client001"] # {'client001': {'fee': 1000.0, 'expiration_date': datetime.datetime(2022, 4, 3, 0, 0)}}

dir(policy)
#['_MutableMapping__marker', '__abstractmethods__', '__class__', '__contains__', '__copy__', '__delattr__', '__delitem__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__setattr__', '__setitem__', '__sizeof__', '__slots__', '__str__', '__subclasshook__', '__weakref__', '_abc_impl', 'change_in_policy', 'clear', 'copy', 'data', 'fromkeys', 'get', 'items', 'keys', 'pop', 'popitem', 'setdefault', 'update', 'values']

```


* 문제점
  1. 계층 구조가 잘못됨
    - 파생 클래스는 부모 클래스를 개념적으로 확장하는 것임
    - TransactionPolicy라는 이름만 보고 dict 타입 이라는 것을 알 수 없음.
  2. 결합력에 대한 문제 
    - TransactionPolicy는 이제 dict의 모든 메서드를 포함한다. 필요하지 않는 것들도

> 구현 객체를 도메인 객체와 혼합할 때 발생하는 문제이다.

* 단지 첨차 기능을 얻기 위해 dict를 확장하는 것은 충분한 근거가 되지 않는다.


``` python
class TransactionalPolicy():
    """컴포지션을 활용한 리팩토링"""
    
    def __init__(self, policy_data, **extra_data):
        self._data = {**policy_data, **extra_data}
        
    def change_in_policy(self, customer_id, **new_policy_data):
        self[customer_id].update(**new_policy_data)
        
    def __getitem__(self, customer_id):
        return self._data[customer_id]
    
    def __len__(self):
        return len(self._data)
    
```


* 개념적으로 정확할 뿐 아니라 확장성도 뛰어나다
* dict 구조를 나중에 변경하려고 해도 인터페이스만 유지하면 사용자는 영향을 받지 않음


## 함수와 메서드의 인자
* 파이썬에서 인자가 함수에 전달되는 방법의 특수성을 알아보자


### 인자는 함수에 어떻게 전달되는가
* 모든 인자가 값에 의해 전달 된다.
* string 객체는 불변형이라 새로운 객체를 만들어서 arg에 다시 할당
* 변경 객체인 리스트는 전달하면 list의 extend()를 호출하는 것과 같다.
  * 이 연산자는 리스트 객체에 대한 참조를 보유하고 있는 변수를 통해 값을 수정하므로 함수 외부에서도 실제 값을 수정할 수 있음
  * string을 iterable하게 뒤에다 다 추가함 - extend()


``` python
def function(arg):
    arg += " in function"
    print(arg)

immutable = "hello"
function(immutable)
mutable = list("hello")
immutable
# hello in function
function(mutable)
mutable
# ['h', 'e', 'l', 'l', 'o', ' ', 'i', 'n', ' ', 'f', 'u', 'n', 'c', 't', 'i', 'o', 'n']
```


> 함수 인자를 변경하지 않아야 한다. 최대한 함수에서 발생할 수 있는 부작용을 회피하라.




### 가변인자
* 가변 인자 함수를 위한 몇 가지 권장사항과 기본 원칙
* 가번 인자를 사용하려면 해당 인자를 패킹할 변수의 이름 앞에 * 를 사용한다.

``` python
def f(first, second, third):
  print(first)
  print(second)
  print(third)

l = [1, 2, 3]
f(*l)
#1
#2
#3
```

* 패킹 기법은 다른 방향으로도 동작한다. 리스트의 값을 각 위취별로 변수에 언패킹하는 법

``` python
a, b, c = [1, 2, 3]
a #1
b #2
c #3
```


* 부분적인 언패킹도 가능하다. 제너레이터와 험깨 언패킹이 어떻게 동작하는지 알아보자


```python
def show(e, rest):
    print(f"element: {e} - rest: {rest}")

first, *rest = [1, 2, 3, 4, 5]
show(first, rest)
# 1
# [2, 3, 4, 5]

*rest, last = range(6)
show(last, rest)
# 5
# [0, 1, 2, 3, 4]

first, *middle, last = range(6)
first # 0
middle # 1, 2, 3, 4
last # 5


first, last, *empty = (1,2)
first # 1
last # 2
empty # []
```


* 변수 언패킹의 가장 좋은 예는 반복이다.
* 일련의 요소를 반복해야 하고 각 요소가 차례로 있다면 각 요소를 반복할 때 언패킹하는 것이 좋다.

* db 결과를 리스트로 받는 함수를 가정해보자. 데이터 받아서 사용자를 생성한다.


``` python
USERS = [ (i, f"first_name_{i}", f" last_name_{i}") for i in range(1_000)]
# python 3.6 부터 숫자 리터럴값 자리수 구분 위해서 구분자로 사용 가능하다.


class User:
    def __init__(self, user_id, first_name, last_name):
        self.user_id = user_id
        self.first_name = first_name
        self.last_name = last_name

def bad_users_from_rows(dbrows) -> list:
    """DB 레코드에서 사용자를 생성하는 파이썬스럽지 않은 잘못된 사용 예"""
    return [User(row[0], row[1], row[2]) for row in dbrows]

def users_from_rows(dbrows) -> list:
    """db 레코드에서 사용자 생성"""
    return [User(user_id, first_name, last_name) for (user_id, first_name, last_name) in dbrows]

bad_users_from_rows(USERS)
users_from_rows(USERS)
```
> 핵심은 dbrows에서 받은 인자를 읽기 쉬운 변수에 바로 언패킹하고, 언패킹한 변수를 객체 생성에 이용함.
row[0]와 같이 받은 변수보다 훨씬 읽기 쉽다.





* 이중 별표 ** 키워드를 인자에 사용할 수 있다.
  * 사전에 ** 를 사용하여 함수에 전달하면, 파라미터의 이름으로 key, 값으로 value 사전 사용한다.
  * ** 파라미터를 함수에 사용하면, 키워드 제공 인자들이 사전으로 패킹된다.

```python
def function(**kwargs):
    for key, value in kwargs.items():
        print(f"{key} and {value}")

function(**{"name": "heeseong"}) # name and heeseong
function(name="heeseong") # name and heeseong

```

## 함수 인자의 개수
* 너무 많은 인자를 가진 함수가 왜 나쁜 디자인인지, 해결 방법은 어떤 것인지 알아보자

* 첫번째 방법, 구체화 - reflication 하는 것
  * 전달하는 모든 인자를 포함하는 새로운 객체를 만드는 것(이는 추상화를 빼먹었기 때문일 것임)
* 두번째 방법, 가변 인자나 키워드 인자를 사용하여 동적 서명을 가진 함수를 만든다.
  * 파이썬 스러운 방법이지만 남용하지 않도록 주의해야함
  * 매우 동적이기 때문에 유지보수가 어려움

> 파라미터가 올바르게 사용되었다고해도, 너무 많은 것들을 함수에서 처리하고 있다면 작은 함수로 분리하라는 사인임.
함수는 오직 한 가지 일만 해야 한다는 것을 기억하자.

### 함수 인자와 결합력
* 인자가 많을 수록 호출자 함수와 밀접하게 결합될 가능성이 높다.

* f1, f2 함수가 있다. f2가 더 많은 파라미터 사용할수록 호출자는 모든 정보 수집이 어려워짐 
  * f2는 파라미터 5개
* f1이 f2 호출을 위한 모든 정보 가지고 있다고 해보자.
  * f2는 추상화가 부족했을 것이다
    * f1은 f2가 필요로 하는 모든 것을 알고 있기 때문에, f2 내부적 로직을 알아낼 수 있다.(자체적으로 수행할 수 있다. == 추상화 부족)
  * f2는 다른 환경에서 사용하기가 어려워 f1에서만 유용하여 재사용성이 떨어진다.


### 너무 많은 파라미터를 취하는 작은 함수
* 아래 함수를 보자.
* 모든 파라미터가 request와 관련이 있다. -> 그냥 request를 전달하는 것은 어떨까?
  * 간단한 변경이지만 코드를 크게 향상 시킨다.
```python
track_request(request.headers, request.ip_addr, request.request_id)

track_request(request)
```

* 변경 가능한 객체를 전달할 때에는 부작용을 주의하자.
> 함수는 전달 받은 객체를 변경해서는 안된다.
객체의 무언가를 바꾸고 싶다면 전달된 값을 복사해서 새로운 수정본을 반환하는 것이 나은 대안이다.



## 소프트웨어 디자인 우수 사례 결론
* 소프트웨어 엔지니어링의 우수 사례를 따르고 언어의 기능이 제공하는 대부분의 장점을 활용하는 디자인.

### 소프트웨어의 독립성 - orthogonality
* 직교 - 수학에서 두 요소가 독립적이라는 것을 의미

* 모듈, 클래스 또는 함수를 변경하면 수정한 컴포넌트가 외부 세계에 영향을 미치지 않아야한다.
  * 어떤 객체의 메서드 호출하는 것이 다른 관련 없는 객체의, 내부 상태를 변경해서는 안된다는 것을 의미

* 독립성에 관한 간단한 예시
* 위쪽 두개의 함수는 독립성을 갖는다.
* 마지막 함수는 fmt_function으로 아무것도 전달 안하면 default로 문자열 변환, 함수 전달하면 문자열 포맷
* show_price의 변경은 calculate_price에 어떤 영향도 미치지 않는다.

``` python
def calculate_price(base_price: float, tax: float, discount: float) -> float:
    return (base_price * ( 1 + tax ) * ( 1 - discount ))

def show_price(price: float) -> str:
    return f"$ {price:,.2f}"

def str_final_price(base_price: float, tax: float, discount: float, fmt_function=str) -> str:
    print(fmt_function(calculate_price(base_price, tax, discount)))



str_final_price(10, 0.2, 0.5)
str_final_price(1000, 0.2, 0)
str_final_price(1000, 0.2, 0.1, fmt_function=show_price)

# 6.0
# 1200.0
# $ 1,080.00
```


### 코드 구조
* 코드 구조화는 팀의 작업 효율성과 유지보수성에 영향을 미친다.

* 클래스, 함수, 상수가 들어있는 큰 파일을 만드는 것은 좋지 않다.
* 유사한 컴포넌트끼리 정리하여 구조화해야한다.
* __init__.py 파일을 가진 새 디렉토리를 만들면 파이썬 패키지가 만들어진다.
  * 이 파일과 함께 특정 정의를 포함하는 여러 파일을 생성한다.
  * __init__.py 파일에 다른 파일에 있던 모든 정의를 가져옴으로써 호환성도 보장할 수 있다.





* 프로젝트에서 모든 파일에 상수를 정의하는 대신, 사용할 상수 값을 저장할 특정한 파일을 만들고 다음과 같이 임포트하면 된다.
  * from myproject.constants import CONNECTION_TIMEOUT

                            













