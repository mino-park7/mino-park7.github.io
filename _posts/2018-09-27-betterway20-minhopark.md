---
layout: "post"
title: "이펙티브 파이썬 - 20. 동적 기본 인수를 지정하려면 None과 docstring을 사용하자"
date: "2018-09-27 00:48"
category: effective python Study
tag: [python, effective,]
---


# BetterWay20. 동적 기본 인수를 지정하려면 None과 docstring을 사용하자

- 키워드 인수의 기본값으로 비정적(non-static)타입을 사용해야 할 때도 있음
  - ex) 이벤트 발생 시각까지 포함해 로깅 메시지를 출력한다고 하자.
  - 함수를 호출한 시각을 메시지에 포함
  - 함수가 호출될 때마다 기본 인수를 평가한다고 가정하고 다음과 같이 처리하려 할 것이다


```python
from datetime import datetime
from time import sleep

def log(message, when=datetime.now()):
    print('%s: %s' % (when, message))

log('Hi there!')
sleep(0.1)
log('Hi again!')
```

    2018-09-26 23:23:56.775506: Hi there!
    2018-09-26 23:23:56.775506: Hi again!


   - `datetime.now`는 함수를 정의할 때 딱 한번만 실행되므로 타임스탬프가 동일하게 출력됨
   - 기본 인수의 값은 모듈이 로드될 때 한 번만 평가되며 보통 프로그램이 시작할 때 일어남
   - 이 코드를 담고 있는 모듈이 로드된 후에는 기본 인수인 `datetime.now`를 다시 평가하지 않음

---

- 파이썬에서 결과가 기대한 대로 나오게 하려면 기본값은 `None`으로 설정하고 **docstring(문서화 문자열)** 으로 실제 동작을 문서화 하는게 관례
- 코드에서 인수 값으로 `None`이 나타나면 알맞은 기본값을 할당하면 됨


```python
def log(message, when=None):
    """
    Log a message with a timestamp

    Args:
        message: Message to print.
        when : datetime of when the message occurred.
            Defaults to the present time.
    """
    when = datetime.now() if when is None else when
    print('%s: %s' % (when, message))
```


```python
log('Hi there')
sleep(0.1)
log('Hi again!')
```

    2018-09-26 23:23:56.901623: Hi there
    2018-09-26 23:23:57.005307: Hi again!


---

- 기본 인수 값으로 `None`을 사용하는 바법은 인수가 수정 가능(mutable)할 때 특히 중요
- ex) JSON 데이터로 인코드된 값을 로드한다고 해보자.
- 데이터 디코딩이 실패하면 기본값으로 빈 딕셔너리를 반환하려고 한다. 다음을 보자


```python
import json
def decode(data, default={}):
    try:
        return json.loads(data)
    except ValueError:
        return default
```

- 위 코드에는 `datetime.now` 예제와 같은 문제가 있음
- 기본 인수 값은(모듈이 로드될 때) 딱 한 번만 평가되므로, 기본값으로 설정한 딕셔너리를 모든 `decode` 호출에서 공유, 이 문제는 예상치 못한 동작을 야기한다.


```python
foo = decode('bad data')
foo['stuff'] = 5
bar = decode('also bad')
bar['meep'] = 1
print('Foo:',foo)
print('Bar:',bar)
```

    Foo: {'stuff': 5, 'meep': 1}
    Bar: {'stuff': 5, 'meep': 1}


- foo와 bar가 같은 객체로 처리됨
- 하나를 수정하면 다른 하나도 수정되고 있음
- foo와 bar 둘 다 기본 파라미터가 같아서 생기는 문제


```python
assert foo is bar
```

- 키워드 인수의 기본값을 `None`으로 설정하고 함수의 docstring에 동작을 문서화해서 문제를 해결하자


```python
def decode(data, default=None):
    """
    Load JSON data form a string.

    Args:
        data: JSON data to decode.
        defualt: Value to return if decoding fails.
            Defualts to an empty dictionary.
    """
    if default is None:
        default = {}
    try:
        return json.loads(data)
    except ValueError:
        return default
```


```python
foo = decode('bad data')
foo['stuff'] = 5
bar = decode('also bad')
bar['meep'] = 1
print('Foo:', foo)
print('Bar:', bar)
```

    Foo: {'stuff': 5}
    Bar: {'meep': 1}


## 핵심 정리

- 기본 인수는 모듈 로드 시점에 함수 정의 과정에서 딱 한 번만 평가된다. 그래서 {}나 [] 같은 동적 값에는 이상하게 동작하는 원인이 되기도 한다.
- 값이 동적인 키워드 인수에는 기본값으로 `None`을 사용하자. 그러고 나서 함수의 docstring에 실제 기본 동작을 문서화하자.
