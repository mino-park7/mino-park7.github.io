---
layout: "post"
title: "이펙티브 파이썬 - 21. 키워드 전용 인수로 명료성을 강요하자"
date: "2018-09-27 01:30"
category: effective python Study
tag: [python, effective,]
---


# BetterWay21. 키워드 전용 인수로 명료성을 강요하자

- 키워드로 인수를 넘기는 방법은 파이썬 함수의 강력한 기능임
- 키워드 인수의 유연성 덕분에 쓰임새가 분명하게 코드 작성 가능
- ex) 어떤 숫자를 다른 숫자로 나눈다고 해보자. 하지만 특별한 경우를 매우 주의해야함
    - 때로는 `ZeroDivisionError` 예외를 무시하고 무한대 값을 반환하고 싶을 수 있음
    - `OverflowError` 예외를 무시하고 0을 반환하고 싶을 수도 있음


```python
def safe_division(number, divisor, ignore_overflow, ignore_zero_division):
    try:
        return number / divisor
    except OverflowError:
        if ignore_overflow:
            return 0
        else:
            raise
    except ZeroDivisionError:
        if ignore_zero_division:
            return float('inf')
        else:
            raise
```

- 이 함수를 사용하는 방법은 간단데스
    - 나눗셈에서 일어나는 float 오버플로우를 무시하고 0을 반환


```python
result = safe_division(1, 10**500, True, False)
print(result)
```

    0.0


- 다음 함수 호출은 0으로 나누면서 일어나는 오류를 무시하고 무한대 값을 반환


```python
result = safe_division(1, 0, False, True)
print(result)
```

    inf


- 문제는 예외 무시 동작을 제어하는 두 bool 인수의 위치를 혼동하기 쉽다는 점이다.
- 이것 때문에 찾기 어려운 버그가 쉽게 발생할 수 있다.
- 이런 코드의 가독성을 높이는 한 가지 방법은 키워드 인수를 사용하는 것
- 다음과 같이 함수가 기본적으로 매우 주의 깊고 항상 예외를 다시 일으키게 만들 수 있다.


```python
def safe_division_b(number, divisor, ignore_overflow=False, ignore_zero_division=False):
    try:
        return number / divisor
    except OverflowError:
        if ignore_overflow:
            return 0
        else:
            raise
    except ZeroDivisionError:
        if ignore_zero_division:
            return float('inf')
        else:
            raise
```

- 이렇게 하면 호출하는 쪽에서 키워드 인수로 특정 연산에는 기본 동작을 덮어쓰고 무시할 플래그를 지정할 수 있다.


```python
print(safe_division_b(1, 10**500, ignore_overflow=True))
print(safe_division_b(1, 0, ignore_zero_division=True))
```

    0.0
    inf


- 문제는 이런 키워드 인수가 선택적인 동작이라서 함수를 호출하는 쪽에 키워드 인수로 의도를 명확하게 드러내라고 **강요**할 방법이 없다는 점이다.
- `safe_division_b`라는 새 함수를 정의해도 여전히 위치 인수를 사용하는 이전 방식으로 호출 할 수 있다.


```python
safe_division_b(1, 10**500, True, False)
```




    0.0



- 이처럼 복잡한 함수를 작성할 때는 호출하는 쪽에서 의도를 명확히 드러내도록 요구하는 게 낫다.
- 파이썬3에서는 키워드 전용 인수(keyword-only argument)로 함수를 정의해서 의도를 명확히 드러내도록 요구할 수 있다.
- 키워드 전용 인수는 키워드로만 넘길 뿐, 위치로는 절대 넘길 수 없다.

---

- 다음은 키워드 전용 인수로 `safe_division` 함수를 다시 정의한 버전이다.
- 인수 리스트에 있는 * 기호는 위치 인수의 끝과 전용 인수의 시작을 가리킨다.


```python
def safe_division_c(number, divisor, *, ignore_overflow=False, ignore_zero_division=True):
    try:
        return number / divisor
    except OverflowError:
        if ignore_overflow:
            return 0
        else:
            raise
    except ZeroDivisionError:
        if ignore_zero_division:
            return float('inf')
        else:
            raise
```


```python
# 이제 키워드 인수가 아닌 위치 인수를 사용하는 함수 호출은 동작하지 않는다.
safe_division_c(1, 10**500, True, False)
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-8-f48f06e28880> in <module>()
          1 # 이제 키워드 인수가 아닌 위치 인수를 사용하는 함수 호출은 동작하지 않는다.
    ----> 2 safe_division_c(1, 10**500, True, False)


    TypeError: safe_division_c() takes 2 positional arguments but 4 were given



```python
# 키워드 인수과 그 기본값은 기대한 대로 동작한다.
safe_division_c(1, 0, ignore_zero_division=True)  # 문제 없음

try:
    safe_division_c(1, 0)
except ZeroDivisionError:
    pass
```

## 파이썬 2의 키워드 전용 인수

- 불행하게도 파이썬 2에서는 * 같은 명시적 문법이 없음
- 하지만 인수 리스트에 `**` 연산자를 사용해 올바르지 않은 함수 호출을 할 때 `TypeError`를 일으키는 방법으로 같은 동작을 만들 수 있다.
- 가변 개수의 위치 인수 대신에 키워드 인수를 몇 개든(심지어 정의하지 않았을 때 조차도) 받을 수 있다는 점만 빼면 `**` 연산자는 * 연산자와 비슷하다.


```python
# 파이썬2
def print_args(*args, **kwargs):
    print 'Positional:', args
    print 'Keyword: ', kwargs

print_args(1, 2, foo='bar', stuff='meep')
```

- 파이썬 2에서는 `safe_division`이 `**kwargs`를 받게 만들어서 키워드 전용 인수를 받게 한다.
- 그런 다음 `pop` 메서드로 `kwargs` 딕셔너리에서 원하는 키워드 인수를 꺼낸다.
- 키가 없을 때의 기본값은 `pop` 메서드의 두번째 인수로 지정한다.
- 마지마기으로 `kwargs`에 더는 남아 있는 키워드가 없음을 확인하여 호출하는 쪽에서 올바르지 않은 인수를 넘기지 않게 한다.


```python
# 파이썬 2
def safe_division_d(number, divisor, **kwargs):
    ignore_overflow = kwargs.pop('ignore_overflow', False)
    ignore_zero_div = kwargs.pop('ignore_zero_division:', False)
    if kwargs:
        raise TypeError('Unexpected **kwargs: %r' % kwargs)
    # ...
```

## 핵심 정리
- 키워드 인수는 함수 호출의 의도를 더 명확하게 해준다
- 특히 불 플래그를 여러 개 받는 함수처럼 헷갈리기 쉬운 함수를 호출할 때 키워드 인수를 넘기게 하려면 키워드 전용 인수를 사용하자.
- 파이썬 3는 함수의 키워드 전용 인수 문법을 명시적으로 지원한다.
- 파이썬 2에서는 `**kwargs`를 사용하고 `TypeError` 예외를 직접 일으키는 방법으로 함수의 키워드 전용 인수를 흉내낼 수 있다.
