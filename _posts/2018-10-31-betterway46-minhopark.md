---
layout: "post"
title: "이펙티브 파이썬 - 46. 내장 알고리즘과 자료 구조를 사용하자"
date: "2018-10-31 14:00"
category: effective python Study
tag: [python, effective,]
---



# BetterWay 46. 내장 알고리즘과 자료 구조를 사용하자

- 알고리즘을 니들 맘대로 짜면 느려지는데, 그건 파이썬 문제가 아님
- 이런 문제는 최적의 알고리즘과 **자료 구조**를 사용하지 않아서 일어났을 가능성이 큼
- 파이썬에 이런 최적 알고리즘과 자료구조를 갖추고 있으니 그걸 사용하자 다시 구현하면 머리 터짐

## 더블 엔디드 큐
- `collection` 모듈의 `deque`클래스는 더블 엔디드 큐 (double ended queue)이다.
- `deque`는 큐의 처음과 끝에서 아이템을 삽입하거나 삭제할 때 항상 일정한 시간이 걸리느 연산을 제공
- FIFO 큐를 만들 때 이상적


```python
from collections import deque
fifo = deque()
fifo.append(1)
x = fifo.popleft()
print (x)
```

    1


- 이렇게만 보면 `list`와 같지 않느냐? 할 수 있는데!
- `list`의 경우 리스트의 시작 부분에서 아이템을 삽입하거나 삭제하는 연산에서 **linear time이 걸린다** `deque`보다 훨씬 느림

## 정렬된 딕셔너리
- 표준 딕셔너리는 정렬되어 있지 않음
- 즉, 같은 키와 값을 담은 `dict`를 순회해도 다른 순서가 나올 수 있다는 것
- 이런 동작은 딕셔너리의 빠른 해시 테이블을 구현하는 방식이 만들어낸 뜻밖의 부작용임


```python
from random import randint
a={}
a['foo'] = 1
a['bar'] = 2

# 무작위로 'b'에 데이터를 추가해서 해시 충돌을 일으킴
while True:
    z = randint(99, 1013)
    print(z)
    b = {}
    for i in range(z):
        b[i] = i
    b['foo'] = 1
    b['bar'] = 2
    for i in range(z):
        del b[i]
    if str(b) != str(a):
        break

print(a)
print(b)
print('Equal?', a == b)

```

    152
    {'foo': 1, 'bar': 2}
    {'bar': 2, 'foo': 1}
    Equal? True


- `collection`모듈의 `OrderedDict`클래스는 키가 삽입된 순서를 유지하는 특별한 딕셔너리 타입
- `OrderedDict`의 키를 순회하는 것은 예상 가능한 동작이므로 테스팅과 디버깅이 쉬워짐



```python
from collections import OrderedDict

a = OrderedDict()
a['foo'] = 1
a['bar'] = 2

b = OrderedDict()
b['foo'] = 'red'
b['bar'] = 'blue'

print(a)
print(b)
for value1, value2 in zip(a.values(), b.values()):
    print(value1, value2)
```

    OrderedDict([('foo', 1), ('bar', 2)])
    OrderedDict([('foo', 'red'), ('bar', 'blue')])
    1 red
    2 blue


## 기본 딕셔너리
- 딕셔너리는 통계를 관리하고 추적하는 작업에 유용
- 딕셔너리를 사용할 때 한 가지 문제는 어떤 키가 이미 존재한다고 가정할 수 없다는 것
- 이 문제 때문에 딕셔너리에 저장된 카운터를 증가시키는 것처럼 간단한 작업도 까다로움


```python
stats = {}
key = 'my_counter'
if key not in stats:
    stats[key] = 0
stats[key] += 1
```

- `collections` 모듈의 `defaultdict`클래스는 키가 존재하지 않으면 자동으로 기본값을 저장하도록 하여 이런 작업을 간소화
- 할 일은 그저 키가 없을 때마다 기본값을 반환할 함수를 제공하는 것 뿐
- 다음 예제에서 내장 함수 `int`는 0을 반환 (카운터를 증가시키는것은 간단데스)


```python
from collections import defaultdict
stats = defaultdict(int)
stats['my_counter'] += 1
```

## 힙 큐
- 힙(heap)은 우선순위 큐(priority queue)를 유지하는 유용한 자료구조다
- `heapq`모듈은 표준 `list`타입으로 힙을 생성하는 `heappush`, `heappop`, `nsmallest`같은 함수를 제공한다
- 임의우 우선순위를 가지는 아이템을 어떤 순서로도 힙에 삽입할 수 있다.



```python
from heapq import *
a = []
heappush(a, 5)
heappush(a, 3)
heappush(a, 7)
heappush(a, 4)

# 우선 순위가 높은 것 (가장 낮은 수)부터 제거됨
print(heappop(a), heappop(a), heappop(a), heappop(a))
```

    3 4 5 7


- 결과로 만들어지는 `list`를 `heapq`외부에서도 쉽게 사용할 수 있다.
- 힙의 0 인덱스에 접근하면 항상 가장 작은 아이템이 반환됨



```python
a=[]
heappush(a, 5)
heappush(a, 3)
heappush(a, 7)
heappush(a, 4)
assert a[0] == nsmallest(1, a)[0] == 3
```


```python
print(nsmallest(2, a))
```

    [3, 4]


- `list`의 `sort`메서드를 호출하더라도 힙의 불변성이 유지



```python
print('Before', a)
a.sort()
print('After', a)
```

    Before [3, 4, 7, 5]
    After [3, 4, 5, 7]


- 이러한 각 `heapq`연산에 걸리는 시간은 리스트의 길이에 비례하여 **로그 형태로 증가**
- 표준 파이썬 리스트로 같은 동작을 수행하면 시간이 **선형적으로 증가**

## 바이섹션
- `list`에서 아이템을 검색하는 작업은 `index`메서드를 호출할 때 리스트의 길이에 비례한 **선형적 시간**이 걸린다


```python
import time
x = list(range(10**8))
```


```python
start_time = time.time()
i = x.index(991234)
print(time.time() - start_time)
```

    0.01512289047241211



```python
from bisect import *
start_time = time.time()
i = bisect_left(x, 991234)
print(time.time() - start_time)
```

    9.894371032714844e-05


- 바이너리 검색의 복잡도는 **로그 형태로 증가**한다.
- 아이템 백만 개를 담은 리스트를 `bisect`로 검색할 때 걸리는 시간은 아이템 14개를 담은 리스트를 `index`로 순차 검색할 때 걸리는 시간과 거의 같다

## 이터레이터 도구
- 내장 모듈 `itertools`는 이터레이터를 구성하거나 이터레이터와 상호작용하는데 유용한 함수를 다수 포함(이미 bw16, 17에서 본 바 있음)
- `itertools`에 있는 함수는 세가지 주요 카테고리로 나뉜다.

---

- 이터레이터 연결
    - `chain`: 여러 이터레이터를 순차적인 이터레이터 하나로 결합
    - `cycle`: 이터레이터의 아이템을 영원히 반복한다.
    - `tee` : 이터레이터 하나를 병렬 이터레이터 여러개로 나눈다.
    - `zip_longest` : 길이가 서로 다른 이터레이터들에도 잘 동작하는 내장 함수 `zip`의 변형

---

- 이터레이터에서 아이템 필터링
    - `islice` : 복사없이 이터레이터를 숫자로된 인덱스로 슬라이스한다.
    - `takewhile` : 서술 함수(predicate function)가 `True`를 반환하는 동안 이터레이터의 아이템을 반환
    - `dropwhile` : 서술함수가 처음으로 `False`를 반환하고 나면 이터레이터의 아이템을 반환
    - `filterfalse` : 서술함수가 `False`를 반환하는 이터레이터의 모든 아이템을 반환. 내장함수 `filter`의 반대 기능

----

- 이터레이터에 있는 아이템들의 조함
    - `product`: 이터레이터에 있는 아이템들의 카테시안 곱을 반환한다. 깊게 중첩된 리스트 컴프리헨션에 대한 훌륭한 대안
    - `permutations` : 이터레이터에 있는 아이템을 담은 길이 N의 순서 있는 순열(permutations)을 반환
    - `combinations` : 이터레이터에 있는 아이쳄이 중복되지 않게 담은 길이 N의 순서없는 조합(combinations)을 반환

## 핵심정리
- 알고리즘과 자료 구조를 표현하는 데는 파이썬의 내장 모듈을 사용하자
- 이 기능들을 직접 재구현하지는 말자. 올바르게 만들기가 어렵기 때문이다.
