---
layout: "post"
title: "이펙티브 파이썬 - 8. 리스트 컴프리헨션에서 표현식을 두 개 넘게 쓰지 말자"
date: "2018-09-09 19:21"
category: effective python Study
tag: [python, effective,]
---



# 8. 리스트 컴프리헨션에서 표현식을 두 개 넘게 쓰지 말자

- 리스트 컴프리헨션은 기본 사용법뿐만 아니라 다중 루프로 지원함
  - ex: 행렬을 모든 셀이 포함된 평평한 리스트 하나로 간략화, for를 두개 사용한 리스트 컴프리헨션 (왼쪽 > 오른쪽 순서)
```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [x for row in matrix for x in row]
print(flat)

>>>
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

  - 입력 리스트의 레이아웃을 두 레벨로 중복해서 구성(매트릭스 제곱 구하기)
```python
squared = [[x**2 for x in row] for row in matrix]
print(squared)

>>>
[[1, 4, 9], [16, 25, 36], [49, 64, 81]]
```
  - 위 표현식을 다른 루프에 넣는다면, 리스트 컴프리헨션이 여러 줄로 구분해야 할 정도로 길어진다.
```python
my_list = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [x for sublist1 in my_list
        for sublist2 in sublist1
        for x in sublist2]
```
- 이번엔 일반 루프문으로 같은 결과를 만들어보자. 이 비전은 들여쓰기를 사용해서 리스트 컴프리헨션보다 이해하기 쉽다.

```python
flat = []
for sublist1 in my_lists:
    for sublist2 in sublist1:
        flat.extend(sublist2)
```

- 리스트 컴프리헨션도 다중 if 조건을 지원한다.
  - 같은 루프 레벨에 **여러조건이 있으면 암시적인 `and` 표현식**이 된다
  - ex: 숫자로 구성된 리스트에서 4보다 큰 짝수 값만 가지고 오는 것
```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
b = [x for x in a if x > 4 if x % 2 == 0]
c = [x for x in a if x > 4 and x % 2 == 0]
```

- 조건은 루프의 각 레벨에서 for 표현식 뒤에 설정 가능
  - ex: 행렬에서 row의 합이 10 이상이고 3으로 나누어 떨어지는 셀을 구하자
```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
filtered = [[x for x in raw if x % 3 ==0]
            for raw in matrix if sum(row) >= 10]
print(filtered)

>>>
[[6], [9]]
```

  - 리스트 컴프리헨션으로 표현하면 간단하지만 이해하기가 매우 어렵다.


- 이런 리스트 컴프리헨션은 피하라고 강력히 말하고 싶음
- 이런 코드는 다른 사람들이 이해하기가 매우 어렵다
- 몇 줄을 절약한 장점이 나중에 겪을 어려움보다 크지않음
- **리스트 컴프리헨션을 사용할 때는 표현식이 두 개를 넘어가면 피하는 게 좋다**.
  - 조건 두개, 루프 두개, 혹은 조건 한 개와 루프 한 개 정도면 됨
  - 이것보다 복잡해지면 일반적인 if 문과 for 문을 사용하고 헬퍼 함수를 작성

---

## 핵심정리
- 리스트 컴프리헨션은 다중 루프와 루프 레벨별 다중 조건을 지원
- 표현식이 두 개가 넘게 들어 있는 리스트 컴프리헨션은 이해하기 매우 어려우므로 피해야 한다.
