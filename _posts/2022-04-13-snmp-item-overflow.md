---
layout: post
toc: true
title: "Zabbix discards item value"
categories: zabbix
tags: [zabbix, snmp, overflow]
author:
  - Heeseong
---

Zabbix를 이용한 네트워크 장비 모니터링 중, Interface Traffic이 자주 수집이 안되는 증상이 나타남

## 문제 인식
Extreme 무선 컨트롤러를 새롭게 도입하기 위해, 테스트 진행 과정에서 Zabbix로 모니터링 등록을 진행하였음.

이후 모니터링 간 특이사항이 없어. 정상으로 판단하였으나, 실제 서비스 투입 이후 트래픽 모니터링 그래프에서
자주 이빨이 빠진 증상을 확인함.


## 해결 과정
우선, item 수집 interval은 60초로 설정한 것으로 확인하고 수집 누락이 발생하는 item의 raw data를 확인하였음.
결과는 2분, 4분 등 1분 간격으로 수집되어야할 데이터가 누락된 것을 확인 ㅠㅠ...

```
2022-04-13 15:48:19	349331416
2022-04-13 15:44:18	317890456
```

4분동안.. 수집된 데이터는 어디로..?

우선 Zabbix 데몬 서버 상태를 확인해보기로 하였음. 
(간혹 housekeeper 프로세스가 돌아가는 중에, history syncer가 수집한 데이터를 못 밀어넣는 경우가 있을 수 있음.)
결과는 ~~~~ 정상...

![image](https://user-images.githubusercontent.com/103275329/163206167-d4c561c4-9593-47e1-9757-d26003b1c645.png)


---


다음 의심은 Zabbix 서버가 네트워크 장비로 설정된 interval에 snmp query를 못날리거니, 장비에서 응답을 주지 않는 경우를 의심

패킷덤프를 통해 정상적으로 주고 받는지 확인해보기로함

```
18:04:18.406363 IP zabbix.40710 > controller.snmp:  C="###" GetRequest(374)  interfaces.ifTable.ifEntry.ifOutOctets.2013
18:04:18.703776 IP controller.snmp > zabbix.40710:  C="###" GetResponse(418)  interfaces.ifTable.ifEntry.ifOutOctets.2013=29749613
```

수집이 누락된 시간에 snmp 질의 및 응답도 정상으로 온 것으로 확인

그럼 도대체 왜 누락된거야?!

수집 item에 설정된 snmp oid는 ifInOctets, ifOutOctets 
얘네는 32bit counter 사용... unsigned int 4bytes는 0 ~ 4,294,967,295
그래프 보면 짧은 시간에 overflow 발생한 것 확인됨(4GBytes~) !!!

SNMP 수집 결과 Overflow 발생
![image](https://user-images.githubusercontent.com/103275329/163207275-0adb44ce-b6ea-450d-b3e1-556cd438623a.png)


트래픽 속도 측정 item은 change per second 옵션 걸려 있어서 값을 아래와 같이 계산함


> Calculate the value change (difference between the current and previous value) speed per second.
Evaluated as (value-prev_value)/(time-prev_time)

overflow 발생 시 수집한 트래픽 계산 결과가 음수가 되서, zabbix에서 수집한 데이터를 버려버림...

> If the current value is smaller than the previous value, Zabbix discards that difference (stores nothing) and waits for another value.


결과적으로 수집 item의 oid를 64bit counter로 변경하는게 맞음
ifHcInOctets, ifHcOutOctets

32bit counter ifInOctets이면 interface 대역폭이 1G만 되어도 full로 사용할 경우 34초면 overflow가 발생한다 ^^

모니터링 수집에 이빨 빠지는 경우에 SNMP Counter overflow를 의심해보자~