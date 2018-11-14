---
layout: "post"
title: "이펙티브 파이썬 - 45. 지역 시간은 time이 아닌 datetime으로 표현하자"
date: "2018-10-31 10:47"
category: effective python Study
tag: [python, effective,]
---


# BetterWay 45. 지역 시간은 time이 아닌 datetime으로 표현하자
- 협정 세계시(UTC, Coordinated Universal Time)는 시간대에 의존하지 않는 표준시간 표현
- UTC는 유닉스 기원 이후로 지나간 초로 시간을 표현하는 컴퓨터에서 잘 작동 (사람한테는 안맞음)
- 사람이 사용하는 시간은 현재 있는 위치는 기준
    - 사람들 : 정오 or 오전 8시
    - 컴퓨터 : UTC 15:00 - 7시
- UTC와 지역 시간 사이의 변환이 필요

---

- 파이썬은 두가지 시간대 변환 방법을 제공 (time, datetime)
- time은 치명적인 오류가 일어날 가능성이 큼

## time 모듈
- 내장 모듈 time의 localtime함수는 유닉스 타임스탬프(유닉스 기원 이후 지난 초 UTC기준)를 호스트 컴퓨터의 시간대와 일치하는 지역 시간으로 변환


```python
from time import localtime, strftime

now = 1407694710
local_tuple = localtime(now)
time_format = "%Y-%m-%d %H:%M:%S"
time_str = strftime(time_format, local_tuple)
print(time_str)
```

- 때로는 지역시간을 받아서 유닉스 타임스탬프로 바꿔야하는 경우도 있음
- strptime함수로 시간 문자열을 파싱 > mktime으로 지역 시간을 유닉스 타임스탬프로 변환


```python
from time import mktime, strptime

time_tuple = strptime(time_str, time_format)
utc_now = mktime(time_tuple)
print(utc_now)
```

    1407694710.0


- 한 시간대의 지역 시간을 다른 시간대의 지역 시간으로 변환하려면? (샌프란시스코 -> 뉴욕)
- time, localtime, strptime 함수의 반환 값을 직접 조작해서 시간대 반환은 ㄴㄴ 너무 복잡함
- 많은 운영체제에서 시간대 변경을 자동으로 관리하는 설정 파일을 갖추고 있음, 파이썬에서는 `time` 모듈을 이용해 이러한 시간대를 사용 가능


```python
# 태평양 연안 표준시의 샌프란시스코 시간대에서의 출발 시각을 파싱하는 코드
parse_format = '%Y-%m-%d %H:%M:%S %Z'
depart_sfo = '2014-05-01 15:45:16 PDT'
time_tuple = strptime(depart_sfo, parse_format)
time_str = strftime(time_format, time_tuple)
```


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    <ipython-input-3-44d00c3f5106> in <module>()
          2 parse_format = '%Y-%m-%d %H:%M:%S %Z'
          3 depart_sfo = '2014-05-01 15:45:16 PDT'
    ----> 4 time_tuple = strptime(depart_sfo, parse_format)
          5 time_str = strftime(time_format, time_tuple)


    /Library/Frameworks/Python.framework/Versions/3.5/lib/python3.5/_strptime.py in _strptime_time(data_string, format)
        502     """Return a time struct based on the input string and the
        503     format string."""
    --> 504     tt = _strptime(data_string, format)[0]
        505     return time.struct_time(tt[:time._STRUCT_TM_ITEMS])
        506


    /Library/Frameworks/Python.framework/Versions/3.5/lib/python3.5/_strptime.py in _strptime(data_string, format)
        341     if not found:
        342         raise ValueError("time data %r does not match format %r" %
    --> 343                          (data_string, format))
        344     if len(data_string) != found.end():
        345         raise ValueError("unconverted data remains: %s" %


    ValueError: time data '2014-05-01 15:45:16 PDT' does not match format '%Y-%m-%d %H:%M:%S %Z'


- 에러가 뜨는데.... 플랫폼에 의존적인 time 모듈의 특성 (본 코드가 돌아간 플랫폼은 OS X)
- 실제 동작은 내부의 C함수가 호스트 운영체제와 어떻게 동작하느냐에 따라 결정됨
- 이와 같은 동작 때문에 파이썬 `time`은 신뢰하기 어려움

## datetime 모듈
- 파이썬에서 시간을 표현하는 두 번째 방법은 내장 모듈 `datetime`의 `datetime` 클래스를 사용하는 것
- `time`모듈과 마찬가지로 `datetime`은 UTC에서의 현재 시각을 지역 시간으로 변경하는 데 사용할 수 있다.


```python
# 현재 시각을 UTC로 얻어와서 한국 로컬 시간으로 변경하는 코드
from datetime import datetime, timezone

now = datetime(2014, 8, 10, 18, 18, 30)
now_utc = now.replace(tzinfo=timezone.utc)
now_local = now_utc.astimezone()
print(now_local)
print(now)
print(now_utc)
```

    2014-08-11 03:18:30+09:00
    2014-08-10 18:18:30
    2014-08-10 18:18:30+00:00


- `datetime` 모듈로도 지역 시간을 다시 UTC의 유닉스 타임스탬프로 쉽게 변경 가능


```python
time_str = '2014-08-11 03:18:30'
now = datetime.strptime(time_str, time_format)
time_tuple = now.timetuple()
utc_now = mktime(time_tuple)
print(utc_now)
```

    1407694710.0


- `datetime`모듈은 `time`모듈과 달리 한 지역 시간을 다른 지역 시간으로 신뢰성 있게 변경
- 하지만 `tzinfo` 클래스와 관련 메서드를 이용한 시간대 변환 기능만 제공....
- `pytz`를 사용하면 모든 시간대 사용가능, 모든 시간대에 대한 정의를 담은 전체 데이터베이스를 포함함
- `pytz`를 효과적으로 사용하려면 항상 지역 시간을 UTC로 먼저 변경해야함.
- 이후 UTC값에 필요한 `datatime`연산 (오프셋 지정 등)을 수행
- 마지막으로 지역 시간으로 변환


```python
# NYC 도착시간은 UTC datetime으로 변환하는 코드
import pytz
arrival_nyc = '2014-05-01 23:33:24'
nyc_dt_naive = datetime.strptime(arrival_nyc, time_format)
eastern = pytz.timezone('US/Eastern')
nyc_dt = eastern.localize(nyc_dt_naive)
utc_dt = pytz.utc.normalize(nyc_dt.astimezone(pytz.utc))
print(utc_dt)
```

    2014-05-02 03:33:24+00:00



```python
# UTC datetime을 얻었으니 샌프란시스코 지역 시간으로 변환해보자
pacific = pytz.timezone('US/Pacific')
sf_dt = pacific.normalize(utc_dt.astimezone(pacific))
print(sf_dt)
```

    2014-05-01 20:33:24-07:00



```python
# 마찬가지로 한국의 지역시간으로 변환 가능
seoul = pytz.timezone('Asia/Seoul')
seoul_dt = seoul.normalize(utc_dt.astimezone(seoul))
print(seoul_dt)
```

    2014-05-02 12:33:24+09:00


- `datetime`과 `pytz`를 사용하면 이런 변환이 호스트 컴퓨터에서 구동하는 운영체제와 상관없이 모든 환경에서 동일하게 동작

## 핵심 정리
- 서로 다른 시간대를 변환하는 데는 `time`모듈을 사용하지 말자
- `pytz`모듈과 내장 모듈 `datetime`으로 서로 다른 시간대 사이에서 시간을 신뢰성 있게 변환하자.
- 항상 UTC로 시간을 표현하고, 시간을 표시하기 전에 마지막 단게로 UTC 시간을 지역 시간으로 변환하자
