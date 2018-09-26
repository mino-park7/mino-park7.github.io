---
layout: "post"
title: "이펙티브 파이썬 - 19. 키워드 인수로 선택적인 동작을 제공하자"
date: "2018-09-24 17:48"
category: effective python Study
tag: [python, effective,]
---


# BETTER WAY 19. 키워드 인수로 선택적인 동작을 제공하자

- 대부분의 다른 프로그래밍 언어와 마찬가지로 파이썬에서 함수를 호출할 때 인수를 위치로 전달할 수 있다.


```python
def remainder(number, divisor):
    return number % divisor

assert remainder(20, 7) == 6
```

- 파이썬 함수의 위치 인수를 모두 키워드로 전달할 수도 있다.
  - 이때 인수의 이름을 함수 호출의 괄호 안에 있는 할당문에서 사용
  - 필요한 위치 인수를 모두 지정한다면 키워드 인수로도 전달할 수 있다.
  - 키워드와 위치 인수를 섞어서 사용할 수 있다.


```python
# 다음 호출은 모두 동일
remainder(20, 7)
remainder(20, divisor=7)
remainder(number=20, divisor=7)
remainder(divisor=7, number=20)
```




    6




```python
# 위치 인수는 키워드 인수 앞에 지정해야 한다.
remainder(number=20, 7)
```


      File "<ipython-input-3-bc33add96835>", line 2
        remainder(number=20, 7)
                            ^
    SyntaxError: positional argument follows keyword argument




```python
# 각 인수는 한 번만 지정할 수 있다.
remainder(20, number=7)
```

- 키워드 인수의 유연성은 세 가지 중요한 이점이 있다.

---

  - 키워드 인수의 첫 번째 이점
    - 코드를 처음 보는 사람이 함수 호출을 더 명확하게 이해할 수 있다는 점
      - remainder 메서드의 구현을 보지 않고는 `remainder(20, 7)`호출에서 어떤 인수가 숫자이고 어떤 인수가 나눗수인지 분명하지 않음
      - 키워드 인수를 사용한 호출에서는 `number=20, divisor=7`을 보면 각각의 목적으로 어떤 파라미터를 사용했는지 명확하게 이해 가능

---

- 키워드 인수의 두번째 이점
    - 함수를 정의할 때 기본값 설정 가능
      - 덕분에 함수에서 대부분은 기본값을 사용하지만 필요할 때 부가 기능을 제공할 수 있다.
      - 반복코드가 줄어들고 코드가 깔끔해짐

- 예를 들어 큰 통에 들어가는 액체의 유속을 계산하고 싶다고해보자
- 큰 통의 무게를 잴 수 있다면, 각기 다른 시각에 측정한 두 무게의 차이를 이용해 유속을 알 수 있음


```python
def flow_rate(weight_diff, time_diff):
    return weight_diff / time_diff

weight_diff = 0.5
time_diff = 3
flow = flow_rate(weight_diff, time_diff)
print('%.3f kg per second' % flow)
```

- 보통은 초당 킬로그램 단위로 유속을 아는 게 좋다. 하지만 센서의 최근 측정값을 이용해 시간이나 날짜처럼 더 큰 시간 단위로 계산하는게 좋을 때도 있다.
- 함수의 인수에 기간 환산 계수를 추가하면 이런 동작 제공 가능


```python
def flow_rate(weight_diff, time_diff, period):
    return (weight_diff/ time_diff) * period
```

- 문제는 함수를 호출할 때마다, 심지어 초당 유속을 사용하는 일반적인 경우(period가 1일때)에도 period를 설정해야함


```python
flow_per_second = flow_rate(weight_diff, time_diff, 1)
```

- period 인수에 기본값을 설정하면 위의 코드를 좀 더 깔끔하게 만들 수 있다.


```python
def flow_rate(weight_diff, time_diff, period=1):
    return (weight_diff / time_diff) * period
# period는 선택적임 인수
```


```python
flow_per_second = flow_rate(weight_diff, time_diff)
flow_per_second = flow_rate(weight_diff, time_diff, period=3600)
```

- 위의 코드는 간단한 기본값에는 잘 동작 (기본값이 복잡할 때는 다루기 까다롭다. better way 20에서 배울예정)

---

  - 키워드 인수의 세 번째 이점
    - 기존의 호출 코드와 호환성을 유지하면서도 함수의 파라미터를 확장할 수 있는 강력한 수단
      - 코드를 많이 수정하지 않고서도 추가적인 기능을 제공 가능
      - 버그가 생길 가능성을 줄일 수 있다.

- 예를 들어 킬로그램 단위는 물론 다른 무게 단위로도 유속을 계산하려고 앞의 `flow_rate` 함수를 확장한다고 하자.
  - 원하는 측정 단위의 변환 비율을 새 선택 파라미터로 추가하여 확장



```python
def flow_rate(weight_diff, time_diff, period=1, units_per_kg=1):
    return ((weight_diff / units_per_kg) / time_diff) * period
```

- `units_per_kg` 인수의 기본값은 1로, 반환되는 무게 단위는 킬로그램이 된다. 즉, 기존 코드의 호출 코드의 동작에는 변화가 없음을 의미
- `flow_rate`를 새로 호출하는 코드에서는 새 키워드 인수로 새로운 동작을 지정할 수 있음


```python
pounds_per_hour = flow_rate(weight_diff, time_diff, period=3600, units_per_kg=2.2)
print("%.3f pounds per hour" % pounds_per_hour)
```

- 위 방법의 유일한 문제는 period와 units_per_kg 같은 선택적인 키워드 인수를 여전히 위치 인수로도 넘길 수 있다는 점


```python
pounds_per_hour = flow_rate(weight_diff, time_diff, 3600, 2.2)
```

- 선택적인 인수를 위치로 넘기면 3600과 2.2 값에 대응하는 인수가 무엇인지 명확하지 않아 혼종을 일으킬 수 있다.
  - 가장 좋은 방법은 항상 키워드 이름으로 선택적인 인수를 지정하고 위치 인수로는 아예 넘기지 않는 것

## Note

- 이런 선택적인 키워드 인수를 사용하면 `*args`를 인수로 받는 함수에서 하위 호환성을 지키기 어렵다. (better way 18)
- 더 좋은 방법은 키워드 전용인수를 사용하는 것 (better way 21)

## 핵심 정리

- 함수의 인수를 키워드로 지정할 수 있다.
- 위치 인수만으로는 이해하기 어려울 때 키워드 인수를 쓰면 각 인수를 사용하는 목적이 명확해짐
- 키워드 인수에 기본값을 지정하면 함수에 새 동작을 쉽게 추가할 수 있다. 특히, 함수를 호출하는 기존 코드가 있을 때 사용하면 좋다.
- 선택적인 키워드 인수를 항상 위치가 아닌 키워드로 넘겨야 한다.
