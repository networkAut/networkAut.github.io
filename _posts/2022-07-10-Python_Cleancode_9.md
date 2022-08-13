---
layout: post
toc: true
title: Python Cleancode - 9
categories: Python
tags: [Python, OOP, Cleancode]
author:
  - Heeseong
---

# 데코레이터를 사용한 코드 개선 - 2

## 데코레이터 활용 우수 사례

일반적으로 데코레이터가 좋은 선택이 될 수 있는 경우

* 파라미터 변환
* 코드 추적
* 파라미터 유효성 검사
* 재시도 로직 구현
* 일부 반복 작업을 데코레이터로 이동하여 클래스 단순화



### 파라미터 변환
* DbC 원칙에 따라 사전조건 또는 사후조건을 강제하며 유효성을 검사할 수  있다.
* 유사한 객체를 반복적으로 생성하거나, 추상화를 위해유사한 변형을 반보갛는 경우 단순 데코레이터를 만들어 사용하면 이 작업을 쉽게 처리할 수 있다.

### 코드 추적
* 실제 함수의 실행 경로 추적
* 함수 지표 모니터링(cpu, mem 사용량 등)
* 함수 실행 시간 측정
* 언제 함수가 실행되고 전달된 파라미터는 무엇인지 로깅


## 데코레이터의 활용 - 흔한 실수 피하기
* 효과적인 데코레이털르 만들기 위해 피해야 할 몇 가지 공동된 사항을 살펴보자!

### 래핑된 원본 객체의 데이터 보존

``` python

def trace_decorator(function):
    def wrapped(*args, **kwargs):
        print(f"{function.__qualname__} 실행")
        return function(*args, **kwargs)

    return wrapped

@trace_decorator
def process_account(account_id):
    """id 별 계정 처리"""
    print(f"{account_id} 게정 처리")


if __name__ == "__main__":
    help(process_account)
    print(process_account.__qualname__)
```



```
Help on function wrapped in module __main__:

wrapped(*args, **kwargs)

trace_decorator.<locals>.wrapped
```

* 데코레이터는 원래 함수의 어떤 것도 변경하지 않아야 하지만, 이름이나 docstring을 변경하는 경우가 있다.
* 위의 경우 원본함수를 wrapped로 변경했기 때문에 원본 함수의 이름이 아닌 새로운 함수의 이름을 출력하게 된다.
* wrapped 함수에 @wraps 데코레이터를 사용하여 실제로는 function 파라미터 함수를 래핑한 것이라고 알려주면 문제는 해결 된다.



```python
from functools import wraps


def trace_decorator(function):
    @wraps(function)
    def wrapped(*args, **kwargs):
        print(f"{function.__qualname__} 실행")
        return function(*args, **kwargs)

    return wrapped

@trace_decorator
def process_account(account_id):
    """id 별 계정 처리"""
    print(f"{account_id} 게정 처리")


if __name__ == "__main__":
    # help(process_account)
    print(process_account.__qualname__)
```

```
process_account
```
* `__qualname__` 이 올바르게 찍히는 것 확인
* 가장 중요한 것은 docstring에 포함된 단위테스트 기능이 복구 되었다는 것이다

* 일반적인 데코레이터의 경우 아래와 같은 구조에 따라 functools.wraps를 추가하면 된다


```python
def decorator(original_function):
  @wraps(original_function):
  def decorated_function(*args, **kwargs):
    # 데코레이터에 의한 수정 작업
      return original_function(*args, **kwargs)
  return decorated_function
```



### 데코레이터 부작용 처리


``` python
import time
from functools import wraps


def traced_function_wroing(function):
    print(f"{function} 함수 실행")

    start_time = time.time()

    @wraps(function)
    def wrapped(*args, **kwargs):
        result = function(*args, **kwargs)
        print(f"함수 {function}의 실행시간: {time.time() - start_time}")
        return result
    return wrapped


@traced_function_wroing
def process_with_delay(callback, delay=0):
    time.sleep(delay)
    return callback()
```

```
from decorator_function_wrong import process_with_delay
<function process_with_delay at 0x10cf02ee0> 함수 실행
```

* 인터프리터에서 import 했을 뿐인데 함수가 실행되었다.
* 무언가 이상하다. 실행한 적이 없는데..?
* 그리고 같은 함수를 여러번 호출 하면 비슷한 결과가 나올 것으로 예상되지만, 시간이 점점 길어진다.
* @traced_function_wrong은 실제로 다음을 의미한다.
  * `process_with_delay = traced_function_wrong(process_with_delay)`
* 이 모듈은 문장을 import할 때 실행된다. 따라서 함수에 설정된 start_time이 모듈을 처음 import하는 시간에 할당됨
* staart_time을 wrapped 함수 안으로 이동시키면 해결


``` python
def traced_function_wrong(function):

    @wraps(function)
    def wrapped(*args, **kwargs):
        print(f"{function} 함수 실행")
        start_time = time.time()
        result = function(*args, **kwargs)
        print(f"함수 {function}의 실행시간: {time.time() - start_time}")
        return result
    return wrapped
```

### 어느 곳에서나 동작하는 데코레이터 만들기
* 데코래이터는 여러 시나리오에 사용될 수 있다.
  * 데코레이터를 함수나 클래스, 메서드 또는 정적 메서드 등 여러곳에 재사용하려는 경우 ! (실제로 고민을 했었다..)

* 함수에 적용될 데코레이터를 클래스의 메서드에 적용하려는 경우
  * *args, **kwargs를 파라미터로 받으면 모든 경우에 사용할 수 있다.

* 그러나, 원래 함수의 서명과 비슷하게 데코레이터를 정의하는 것이 좋을 떄 가 있다.
  1. 원래의 함수와 모양이 비슷하기 때문에 읽기가 쉽다.
  2. 파라미터를 받아서 뭔가를 하려면 *args, **kwargs가 불편하다.

* 파라미터를 받아서 특정 객체를 생성하는 경우가 많다고 생각해보자.
* DBDriver 객체는 연결 문자열을 받아서 데이터베이스에 연결하고 DB 연산을 수행하는 객체이다.
* 메서드는 DB 정보 문자열을 받아서 DBDriver 인스턴스를 생성
* 데코레이터는 이러한 변환을 자동화하여 문자열을 받아 DBDriver를 생성하고 함수에 전달
* 따라서 마치 객체를 직접 받은 것처럼 가정할 수 있다.


``` python
from functools import wraps

class DBDriver:
    def __init__(self, dbstring):
        self.dbstring = dbstring

    def execute(self, query):
        return print(f"{self.dbstring} 에서 쿼리 {query} 실행")

def inject_db_driver(function):
    """데이터베이스 dns 문자열을 받아서 DBDriver 인스턴스를 생성하는 데코레이터"""

    @wraps(function)
    def wrapped(dbstring):
        return function(DBDriver(dbstring))

    return wrapped

@inject_db_driver
def run_query(driver):
    return driver.execute("test_function")


if __name__ == "__main__":
    run_query("test ok")
```

```
test ok 에서 쿼리 test_function 실행

Process finished with exit code 0
```
* run_query 함수에 문자열을 전달하면 DBDriver 인스턴스를 반환하므로 예상한 것처럼 동작한다.
* 하지만 이제 같은 기능의 데코레이터를 클래스 메서드에서 재사용하고 싶다면?

```python
from functools import wraps

class DBDriver:
    def __init__(self, dbstring):
        self.dbstring = dbstring

    def execute(self, query):
        return print(f"{self.dbstring} 에서 쿼리 {query} 실행")

def inject_db_driver(function):
    """데이터베이스 dns 문자열을 받아서 DBDriver 인스턴스를 생성하는 데코레이터"""

    @wraps(function)
    def wrapped(dbstring):
        return function(DBDriver(dbstring))

    return wrapped

class DataHandler:
    @inject_db_driver
    def run_query(self, driver):
        return driver.execute(self.__class__.__name__)


if __name__ == "__main__":
    DataHandler().run_query("test_fails")a
```

```
DataHandler().run_query("test_fails")
TypeError: wrapped() takes 1 positional argument but 2 were given
```


* 실행 결과 동작하지 않는다..
* 무엇이 문제일까?
  * 클래스의 메서드에는 self라는 추가 변수가 있다.
  * 메서드는 자신이 정의된 객체를 나타내는 self라는 특수한 변수를 항상 첫 번째 파라미터로 받도록 되어 있다.
  * 따라서 하나의 파라미터만 받도록 설계된 이 데코레이터는 연결 문자열 자리에 self를 전달하고, 두 번째 파라미터로 아무것도 전달하지 않아서 에러가 발생한다.

* 이 문제를 해겨랗라면 메서드와 함수에 대해서 동일하게 동작하는 데코레이터를 만들어야한다.
* `디스크립터 프로토콜을 구현한 데코레이터 객체를 만든다.`

* 해결책은 데코레이터를 클래스 객체로 구현하고 `__get__` 메서드를 구현한 디스크립터 객체를 만드는 것이다.


``` python
from functools import wraps
from types import MethodType

class DBDriver:
    def __init__(self, dbstring):
        self.dbstring = dbstring

    def execute(self, query):
        return print(f"{self.dbstring} 에서 쿼리 {query} 실행")

class inject_db_driver:
    """문자열을 DBDriver 인스턴스로 변환하여 래핑된 함수에 전달"""

    def __init__(self, function):
        self.function = function
        wraps(self.function)(self)

    def __call__(self, dbstring):
        return self.function(DBDriver(dbstring))

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self.__class__(MethodType(self.function, instance))

class DataHandler:
    @inject_db_driver
    def run_query(self, driver):
        return driver.execute(self.__class__.__name__)

@inject_db_driver
def run_query_1(driver):
    return driver.execute("test_function")

if __name__ == "__main__":
    DataHandler().run_query("test_fails")
    run_query_1("test_fails")

```

```
test_fails 에서 쿼리 DataHandler 실행
test_fails 에서 쿼리 test_function 실행

Process finished with exit code 0
```


* 디스크립터에 대한 자세한 내용은 뒤에서 설명하고, 지금은 호출할 수 있는 객체를 메서드에 다시 바인딩한다는 정도만 알면 된다.
* 즉, 함수를 객체에 바인딩하고 데코레이터를 새로운 호출 가능 객체로 다시 생성한다.





















































































































































































