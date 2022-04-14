---
layout: post
toc: true
title: "읽기 좋은 코드"
categories: cleancode
tags: [code, readable, cleancode]
author:
  - Heeseong
---


## 시작하기 전에
* 절대적인 기준은 없음
* 일관성이 가장 중요

## 코드는 이해하기 쉬워야 한다
### 간결하게

```
for (Node* node = list->head; node!=NULL; node=node->next)
  Print(node->data);

```

```
Node* node = list->head;
if(node == NULL) return;

while(node->next != NULL) {
  Print(node->data);
  node = node->next;
}
if(node != NULL) Print(node->data);

```

### 간결 != 적은줄

```
return exponent >= 0 ? mantissa * (1 << exponent): mantissa / (1 << -exponent);
```

```
if(exponent >= 0) {
  return mantissa * (1 << exponent);
} else {
  return mantissa / (1 << -exponent);
}
```

핵심 아이디어는 코드란 다른 사람이 그것을 이해하는데 들이는 시간을 최소화하는 방식으로 작성해야한다.

* 이해
  * 코드를 자유롭게 수정하고
  * 버그를 짎어내고
  * 수정한 내용이 여러분이 작성한 다른 부분의 코드와 어떻게 상호작용하는지 알 수 있어야함

> 지금의 나와 6개월 뒤의 나는 다른 사람 ^^


## 이름에 정보 담기
이름에 정보를 담아라

### 특정한 단어 고르기

#### Get

``` python
def GetPage(url):
  ...
```
* get은 지나치게 보편적이다
* 로컬 캐시에서? 인터넷에서?
  * FetchPage(), DownloadPage()

#### Size

``` c++
class BinaryTree{
  int Size();
  ...
}
```

* 트리의 높이? Height()
* 노드의 개수? NumNodes()
* 트리의 메모리? MemoryBytes()

### Stop

``` python
class Thread {
  void Stop();
}
```
* 더 정확한 이름을 고를 수 있음
* 되돌릴 수 없는 최종 동작이라면 Kill()
* Resume() 등으로 다시 돌이킬 수 있다면 Pause()

보다 다양한 단어를 활용

| 단어 | 유의어 |
| ---- | ---- |
| send | deliver, dispatch, announce, distribute, route|
| find | search, extract, locate, recover|
| start | launch, create, begin, open |
| make | create, setup, build, generate, compose, add, new |





### tmp, retval 같은 보편적 이름 피하기
#### retval
* 리턴값이라고 무조건 retval을 쓰지 않기
* 그 변수가 의미하는 바를 이름으로

#### tmp
* 적당한 예

``` c
if(right < left) {
  tmp = right;
  right = left;
  left = temp;
}
```

* 적당하지 않은 예: tmp --> userInfo

``` java
String tmp = user.name();
tmp += "" + user.phoneNumber();
tmp ++ "" + user.email();
...
template.set("user_info", tmp);
```

* 이름의 일부에 tmp를 사용

``` java
tmp_file = tempFile.NamedTemporaryFiles();
...
SaveData(tmp_file, ...);
```
* tmp가 파일? 파일 이름? 파일 데이터?
``` java
SaveData(tmp, ...);
```

> tmp라는 이름은 대상이 짧게 임시적으로만 존재하고, 임시적 존재 자체가 변수의 가장 중요한 용도일 때 한해서 사용해야함.


### 추가적인 정보를 이름에 추가
* stringid;//Example:"AF84EF845CD8" ◦--> stringhexId;
* 단위:sec?ms?: Start(intdelay_secs)
* plaintext_password , unescaped_comment , html_utf8



### 변수 길이
* 충분히 설명
* BEManager --> BackEndManager
* 스코프가 좁을 때는 짧아도 OK

``` cpp
if(debug) {
  map m;
  LookUpNamesNumbers(&m);
  Print(m);
}
```

## 오해할 수 없는 이름들

> 본인이 지은 이름을 다른 사람들이 다른 의미로 해석할 수 있을까? 라는 질문을 던져보며 철저하게 확인해야함.


### Filter
``` js
results = Database.all_objects.filter("year <= 2011")
```

* 제외? 해당?
* 고르는 경우 select()
* 제외하는 경우 exclude()

### min, max
* 경계를 포함하는 한계를 다룰 때

``` cpp
CART_TOO_BIG_LIMIT = 10
if shopping_cart.num_items() >= CART_TOO_BIG_LIMIT:
  Error("Toomanyitemsincart!")
```


### first, last
* 경계를 포함하는 범위를 다룰 때

### begin, end
* 경계를 포함하고/배제하는 범위를 다룰 때

### Boolean
* is, has, can, should 등의 접두어 활용
* 변수는 긍정 문구로

``` cpp
bool disableSsl = false;
```

### 사용자의 기대에 부응하기
* getXXX()
  * 가볍게 값을 읽을 거라는 기대
  * getMean() vs computeMean()

* list::size()
  * C++ 표준라이브러리 O(n)
  * size는 O(1)이어야 한다

## 주석
* 코드에서빠르게 유추할 수있는내용으로주석을 달지 말라
``` js
name = '*'.join(line.split('*')[:2]) //두번째'*'뒤에오는내용을모두제거한다.
```

* 나쁜 이름에 주석을 달지 말라 이름을 고쳐라

``` cpp
// 해당 키를 위한 핸들을 놓아 준다. 이 함수는 실제 레지스트리를 수정하지는 않는다. 
void DeleteRegistry(RegistryKey *key);
```

``` cpp
void ReleaseRegistryHandle(RegistryKey *key);
```


* 중요한 통찰을 기록
```
// 놀랍게도, 이 데이터에서 이진트리는 해시테이블보다 40% 정도 빠르다. 
// 해시를 계산하는 비용이 좌/우를 비교하는 비용을 능가한다.
```

* 상수에 대한 설명
``` java
NUM_THREADS = 8; // 이 상수 값이 2 * num_processors보다 크거나 같으면 안된다.
const int MAX_RSS_SUBSCRIPTIONS = 1000; // 합리적인 한계를 설정하라 - 이렇게 많이 읽을 수 있는 사람 은없음
image_quality = 0.72; // 사용자들은 0.72가 크기/해상도 대비 최선이라고 생각한다.
```

### 코드를 읽는 사람의 입장이 될 것

* 사람들이 쉽게 빠질 것 같은 함정을 경고

``` java
// 리턴값이 `true`라고 메일이 전달된 것은 아님!
// SMTP 서버에 전달 성공을 뜻함
boolean SendEmail(String to, String subject, String body);
```


## 읽기 쉽게 제어 흐름 만들기
### 조건문에서 인수의 순서

``` python
if(length>=10)
if (10<=length)
```

```python
while(bytes_received < bytes_expected)
while(bytes_expected > bytes_received)
```

* 읽기 쉬운 규칙

| 왼쪽 | 오른쪽 |
| --- | --- |
| 값이 더 유동적인 질문을 받는 표현 | 더 고정적인 값으로 비교 대상으로 사용하는 표현 |


### if/else 블록의 순서
* 부정이 아닌 긍정을 다룰 것
* 두 불록 중 간단한 불록을 먼저
* 더 흥미롭고, 확실한 것을 먼저


### 삼항 연산자?
> 줄 수를 최소화하는 일보다 다른 사람이 코드를 읽고 이해하는데 걸리는 시간을 최소화하는 일이 더 중요하다..

``` js
time_str += (hour >= 12) ? "pm" : "am";
```


### do/while 루플를 피하라

``` python
do {
  continue;
} while(false);
```


### 함수 중간에 반환하기
* 중간에 반환하는 것을 막는 이유는 클린업 코드 때문
* 하지만 현대 언어는 더 정교한 방법을 제공
  * try/catch/finally, with, using 등

### 코드 중첩 최소화
#### 중첩된 코드
* N번 중첩된 코드라면, 변수 N개의 상태를 기억하면 읽어야 함.

``` java
if (user_result == SUCCESS) {
  if (permission_result != SUCCESS) {
    reply.WriteErrors("error reading permissions");
    reply.Done();
    return;
  }
  reply.WriteErrors(""); 
} else {
  reply.WriteErrors(user_result);
  reply.Done();
}
```

* 중첩 제거

``` java
if (user_result != SUCCESS) { 
  reply.WriteErrors(user_result); 
  reply.Done();
  return;
}
if (permission_result != SUCCESS) { 
  reply.WriteErrors("error reading permissions"); 
  reply.Done();
  return;
}
reply.WriteErrors(""); 
reply.Done();
```


## 거대한 표현을 잘게 쪼개기

### 설명 변수

``` java
if line.split(':')[0].strip() == "root":
```

``` java
username = line.split(':')[0].strip() 
if username == "root"
...
```

### 요약 뱐수


``` java
if (request.user.id == document.owner_id) { 
  // 사용자가 이 문서를 수정할 수 있다.
}
...
if (request.user.id != document.owner_id) {
  // 문서는 읽기 전용이다.
}
```

``` java
final boolean user_owns_document = (request.user.id == document.owner_id);
if (user_owns_document) {
  // 사용자가 이 문서를 수정할 수 있다.
}
...
if (!user_owns_document) {
  // 문서는 읽기 전용이다.
}
```

### 드모르간 법칙 사용하기

``` java
if (!(file_exists && !is_protected)) Error("...");
```

``` java
if(!file_exists || is_protected) Error("...");
```