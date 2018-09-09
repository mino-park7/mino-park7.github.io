---
layout: "post"
title: "이펙티브 파이썬 - 7. map과 filter 대신 리스트 컴프리헨션을 사용하자"
date: "2018-09-09 18:39"
category: effective python Study
tag: [python, effective,]
---




# 7. map과 filter 대신 리스트 컴프리헨션을 사용하자

- 파이썬에는 한 리스트에서 다른 리스트를 만들어내는 간결한 문법이 있다.
- 이 문법을 사용한 표션식을 리스트 컴프리헨션(list comprehension) 이라고 함
  - ex: 리스트에 있는 각 숫자의 제곱을 계산 한다고 하자 몇가지 방법이 존재함
    - 첫번째, 리스트 컴프리헨션 사용


```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
squares = [x**2 for x in a]
print(squares)

>>>
[1, 4, 9. 16, ...]
```
    - 두번째, map 사용 (lambda 사용해야 해서 깔끔해 보이지 않음)


```python
squares = map(lambda x: x ** 2, a)
```

- map과 달리 리스트 컴프리헨션을 사용하면 입력 리스트에 있는 아이템을 간편하게 걸러내서 그에 대응하는 출력을 결과에서 삭제할 수 있음
  - ex: 2로 나누어 떨어지는 숫자의 제곱만 계산
    - 첫번쨰, 리스트 컴프리헨션 사용


```python
even_squares = [x**2 for x in a if x % 2==0]
print(even_squares)

>>>
[4, 16, 36,...]
```

- 두번째, filter와 map 사용 (훨씬 읽기 어려움)
  
```python
alt = map(lambda x: x ** 2, filter(lambda x: x % 2 == 0, a))
assert even_squares == list(alt)
```

---

- 딕셔너리와 세트에도 리스트 컴프리헨션에 해당하는 문법이 존재
- 컴프리헨션 문법을 쓰면 알고리즘을 작성할 때 파생되는 자료 구조를 쉽게 생성할 수 있음

```python
chile_ranks = {'ghost': 1, 'habanero': 2, 'cayenne': 3}
rank_dict = {rank: name for name, rank in chile_ranks.items()}
chile_len_set = {len(name) for name in rank_dict.values()}
print(rank_dict)
print(chile_len_set)

>>>
{1: 'ghost', 2: 'habanero', 3: 'cayenne'}
{8, 5, 7} # 알파벳 역순인가...? 아님 그냥 python set 자체가 unordered collection 임....
```

---

## 핵심정리

- 리스트 컴프리헨션은 추가적임 `lambda` 표현식이 필요 없어서 내장 함수인 map 이나 filter를 사용하는 것보다 명확하다.
- 리스트 컴프리헨션을 사용하면 입력 리스트에서 아이템을 간단히 건너뛸 수 있다. map으로는 filter를 사용하지 않고서는 이런 작업을 못한다.
- 딕셔너리와 세트도 컴프리헨션 표현식을 지원한다.
