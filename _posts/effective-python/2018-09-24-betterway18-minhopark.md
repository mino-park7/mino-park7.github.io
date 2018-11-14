---
layout: "post"
title: "이펙티브 파이썬 - 18. 가변 위치 인수로 깔끔하게 보이게 하자"
date: "2018-09-24 16:48"
category: effective python Study
tag: [python, effective,]
---


# BETTER WAY 18. 가변 위치 인수로 깔끔하게 보이게 하자

- 선택적인 위치 인수를 받게 만들면 함수 호출을 더 명확하게 할 수 있고 보기에 방해되는 요소를 없앨 수 있음
  - 이런 파라미터의 이름을 관례적으로 `*args`라고 해서 종종 'star args'라고도 한다

- 예를 들어 디버그 정보 몇 개를 로그로 남긴다로 해보자.
  - 인수의 개수가 고정되어 있다면 메시지와 값 리스트를 받는 함수가 필요


```python
def log(message, values):
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print('%s: %s' % (message, values_str))

log('My numbers are', [1, 2])
log('Hi there', [])
```

    My numbers are: 1, 2
    Hi there


- 로그로 남길 값이 없을때 빈 리스트 []를 남겨야하는거 귀찮다
  - 두번째 인수를 아예 남겨둔다면 더 좋을 것
  - 파이썬에서는 * 기호를 바지막 위치 파라미터 이름 앖에 붙이면 된다.
- 로그 메시지를 의미하는 첫번째 파라미터는 필수지만, 다음에 나오는 위치 인수는 몇 개든 선택적이다.
- 함수 본문은 수정할 필요가 없고 호출하는 쪽만 수정해주면 된다.


```python
def log(message, *values): # 유일하게 다른 부분
    if not values:
        print(message)
    else:
        values_str = ','.join(str(x) for x in values)
        print('%s: %s' % (message, values_str))

log('My numbers are', 1, 2)
log('Hi there')
```

    My numbers are: 1,2
    Hi there


- 리스트를 `log`같은 가변 인수 함수를 호출하는데 사용하고 싶다면 * 연산자를 쓰면 된다.
  - 파이썬은 시퀀스에 들어 있는 아이템들을 위치 인수로 전달


```python
favorites = [7, 33, 99]
log("Favorite colors", *favorites)

```

    Favorite colors: 7,33,99


---

- 가변 개수의 위치 인수를 받는 방법에는 두가지 문제가 있다.
  - 1) 가변 인수가 함수에 전달되기에 앞어 항상 튜플로 변환되는 점
    - 함수를 호출하는 쪽에서 제너레이터에 * 연산자를 쓰면 제너레이터가 모두 소진 될 때까지 순회됨을 의미
    - 결과로 만들어지는 튜플은 제너레이터로부터 생성된 모든 값을 담으므로 메모리를 많이 차지해 결국 프로그램이 망가지게 할 수도 있다.


```python
def my_gernerator():
    for i in range(10):
        yield i

def my_func(*args):
    print(args)

it = my_gernerator()
my_func(*it)
```

    (0, 1, 2, 3, 4, 5, 6, 7, 8, 9)


- `*args`를 받는 함수를 인수 리스트에 있는 입력의 수가 적당히 적다는 사실을 아는 상황에서 가장 좋은 방법
  - 이런 함수는 많은 리터럴이나 변수 이름을 한꺼번에 넘기는 함수 호출에 이상적, 주로 개발자들을 편하게 하고 코드의 가독성을 높이려고 사용

---

- 2) 추후에 호풀 코드를 모두 변경하지 않고서는 새 위치 인수를 추가할 수 없다는 점
  - 인수 리스트의 앞쪽에 위치 인수를 추가하면 기존의 호출 코드가 수정없이는 이상하게 동작함


```python
def log(sequence, message, *values):
    if not values:
        print('%s: %s' % (sequence, message))
    else:
        values_str = ', '.join(str(x) for x in values)
        print('%s: %s: %s' % (sequence, message, values_str))


log(1, "Favorite", 7, 33) # 새로운 용법은 OK
log('Favorite number', 7, 33) # 오래된 용법은 제대로 동작하지 않음
```

    1: Favorite: 7, 33
    Favorite number: 7: 33


- 이 코드의 문제점은 두 번째 호출이 `sequence`인수를 받지 못했기 때문에 7을 `message` 파라미터로 사용한다는 점
  - 이런 버그는 코드에서 예외를 일으키지 않고 계속 실행되므로 발견하기가 극히 어렵다.
  - 이런 문제가 생길 가능성을 완전히 없애려면 `*args`를 받는 함수를 확장할 때 키워드 전용(keyword-only)인수를 사용해야 한다.

## 핵심정리

- `def`문에서 `*args`를 사용하면 함수에서 가변 개수의 위치 인수를 받을 수 있다.
- * 연산자를 쓰면 시퀀스에 들어있는 아이템을 함수의 위치 인수로 사용할 수 있다.
- 제너레이터와 * 연산자를 함께 사용하면 프로그램이 메모리 부족으로 망가질 수도 있다.
- `*args`를 받는 함수에 새 위치 파라미터를 추가하면 정말 찾기 어려운 버그가 생길 수도 있다.
