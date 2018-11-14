---
layout: "post"
title: "이펙티브 파이썬 - 4. 복잡한 표현식 대신 헬퍼 함수를 작성하자"
date: "2018-09-07 20:34"
category: effective python Study
tag: [python, effective,]
---

# 4. 복잡한 표현식 대신 헬퍼 함수를 작성하자
## 헬퍼함수를 사용하지 않는 경우
- 파이썬의 간결한 문법을 이용하면 많은 로직을 표현식 한 줄로 쉽게 작성 가능

```python
from urllib.parse import parse_qs
my_values = parse_qs('red=5&blue=0&green=', keep_blank_values=True)
print(repr(my_values))

$$$
{'red': ['5'], 'green': [''], 'blue': ['0']}
```

- 쿼리 문자열 파라미터에 따라 값이 여러개, 한개일 수 있고, 파라미터는 존재하지만 값x 혹은 파라미터 x
- 결과 딕셔너리에 get 메서드를 사용하면 각 상황에 따라 다른 값을 반환 할 것이다.

```python
print('Red:       ', my_values.get('red'))
print('Green:     ', my_values.get('green'))
print('Opacity:   ', my_values.get('opacity'))


$$$
Red:      ['5']
Green:    ['']
Opacity:  None
```

- 이때 파라미터가 없거나 비어 있으면 기본값으로 0을 할당하게 하면 좋을 것 같다
- 사용하는 트릭은 **빈 문자열, 빈 리스트, 0이 모두 암시적으로 `False`로** 평가되는 것

```python
red = my_values.get('red', [''])[0] or 0
green = my_values.get('green', [''])[0] or 0
opacity = my_values.get('opacity', [''])[0] or 0
print('Red:       %r' % red)
print('Green:     %r' % green)
print('Opacitry:  %r' % opacity)


$$$
Red:    '5'
Green:   0
Opacity: 0
```

- 위 코드의 `red`, `green`, `opacity` 표현식에서 첫 번째 서브 표현식이 `False`일 때 `or` 연산자 뒤에 오는 서브 표현식을 평가한 값이 결과가 됨
  - `red` : 키값이 my_values 딕셔너리에 존재 -> 값은 멤버 하나만 있는 list -> 이 문자열은 `True` -> `red`는 `or` 의 첫 번째 부분을 할당받음
  - `green` : 키값이 my_values 딕셔너리에 존재 -> 빈 문자열인 list -> 암시적으로 'False' -> `or` 의 결과는 0
  - `opacity` : 키 값이 없음 -> `get` 메서드는 키가 딕셔너리에 없으면 두 번째 인수를 반환(기본값은 빈 문자열) -> `green`과 똑같이 동작

- 하지만 이 표현식의 문제점은
  - 읽기 어렵다
  - 필요한 작업을 다 수행하지도 않음 : 모든 파라미터 값이 정수가 되도록 해야함, `int` 사용 필요

```python
red = int(my_values.get('red', [''])[0] or 0)
```

    - 이것도 읽기 너무 어려움....
    - if/else 조건 삼항 표현식 사용해보자 !


```python
red = my_values.get('red', [''])
red = int(red[0]) if red[0] else 0
```

    - 훨씬 낫지만 if/else 삼항 표현식 자체가 어렵다....
    - 여러 줄 if/else 문으로 표현한다면?

```python
green = my_values.get('green', [''])
  if green[0]:
      green = int(green[0])
  else:
      green = 0
```
## 헬퍼함수를 사용할 때!

- 위와 같은 로직을 반복해서 써야한다면 **헬퍼 함수를 만들자!**

```python
def get_first_int(values, ket, default=0):
    found = values.get(key, [''])
    if found[0]:
        found = int(found[0])
    else:
        found = default
    return found

green = get_first_int(my_values, 'green')
```
  
- 표현식이 복잡해지기 시작하면
  - 최대한 빨리 해당 표현식을 작은 조각으로 분할
  - 로직을 헬퍼 함수로 옮기는 방안 고려
- 무조건 짧은 코드를 만들기 보다는 가독성을 선택하는 편이 나음


---

## 핵심 정리
- 파이썬의 문법을 이용하면 **한 줄짜리 표현식을 쉽게 작성할 수 있지만 코드가 복잡해지고 읽기 어려워진다**
- 복**잡한 표현식은 헬퍼 함수로 옮기는** 게 좋다. 특히, **같은 로직을 반복해서 사용**해야 한다면 헬퍼 함수를 사용하자.
- **if/else 표현식을 이용**하면 or나 and 같은 bool 연산자를 사용할 때보다 읽기 수월한 코드 작성 가능
