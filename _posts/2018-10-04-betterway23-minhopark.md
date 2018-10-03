
# BetterWay 23. 인터페이스가 간단하면 클래스 대신 함수를 받자

- 파이썬 내장 API의 상당수에는 함수를 넘겨서 동작을 사용자화하는 기능이 있음
- API는 이런 hook를 이용해서 여러분이 작성한 코드를 실행 중에 호출한다
- ex) `list` 타입의 `sort` 메서드는 정렬에 필요한 각 인덱스의 값을 결정하는 선택적인 `key`인수를 받는다.
   - 다음 코드에서는 `lambda` 표현식을 `key` 후크로 넘겨서 이름 리스트를 길이로 정렬한다


```python
names = ['Socrates', 'Archimedes', 'Plato', 'Aristotle']
names.sort(key=lambda x: len(x))
print(names)
```

    ['Plato', 'Socrates', 'Aristotle', 'Archimedes']


- 다른 언어에서라면 후크를 추상 클래스로 정의할 것이라고 예상할 수도 있다.
- 하지만 파이썬의 후크 중 상당수는 인수와 반환 값을 잘 정의해놓은 단순히 상태가 없는 함수다.
- 함수는 클래스보다 설명하기 쉽고 정의하기도 간단해서 후크로 쓰기에 이상적임
- 함수가 후크로 동작하는 이유는 파이썬이 일급 함수(first-class function)를 갖췄지 때문
    - 언어에서 함수와 메서드를 다른 값처럼 전달하고 참조할 수 있음

---

- 예를들어 `defaultdict` 클래스의 동작을 사용자화한다고 해보자
- 이 자료 구조는 찾을 수 없는 키에 접근할 때마다 호출될 함수를 받는다.
- `defaultdict`에 넘길 함수는 딕셔너리에서 찾을 수 없는 키에 대응할 기본값을 반환해야 한다.
- 다음은 키를 찾을 수 없을 때마다 로그를 남기고 기본값으로 0을 반환하는 후크를 정의한 코드다


```python
from collections import defaultdict
def log_missing():
    print('Key added')
    return 0
```


```python
# 초깃값을 담은 딕셔너리와 원하는 증가 값 리스트로 `log_missing` 함수를 두번(각각 'red'와 'orange'일 때) 실행하여 로그를 출력하게 해보자.
current = {'green': 12, 'blue': 3}
increments = [
    ('red', 5),
    ('blue', 17),
    ('orange', 9),
]
result = defaultdict(log_missing, current)
print('Before:', dict(result))
for key, amount in increments:
    result[key] += amount
print('After: ', dict(result))

```

    Before: {'blue': 3, 'green': 12}
    Key added
    Key added
    After:  {'blue': 20, 'orange': 9, 'red': 5, 'green': 12}


- `log_missing`같은 함수를 넘기면 결정 동작과 부작용을 분리하므로 API를 쉽게 구축하고 테스트 할 수 있음
- 예를 들어 기본값 후크를 `defualtdict`에 넘겨서 찾을 수 없는 키의 총 개수를 센다고 해보자.
    - 이렇게 만드는 한 가지 방법은 상태 보존 클로저를 사용
    - 다음은 상태 보존 클로저를 기본값 후크로 사용하는 헬퍼 함수다


```python
def increment_with_report(current, increments):
    added_count = 0
    
    def missing():
        nonlocal added_count  # 상태 보존 클로저
        added_count += 1
        return 0
    
    result = defaultdict(missing, current)
    for key, amount in increments:
        result[key] += amount
        
    return result, added_count
```

- `defaultdict` 는 `missing` 후크가 상태를 유지한다는 사실을 모르지만, `increment_with_report` 함수를 실행하면 튜플의 요소로 기대한 개수인 2를 얻는다
- 이는 간단한 함수를 인터페이스용으로 사용할 때 얻을 수 있는 또 다른 이점
- 클로저 안에 상태를 숨기면 나중에 기능을 추가하기도 쉬움


```python
result, count = increment_with_report(current, increments)
assert count == 2
```

- 상태 보존 후크용으로 클로저를 정의할 때 생기는 문제는 상태가 없는 함수의 예제보다 이해하기 어렵다는 점
- 또 다른 방법은 보존할 상태를 캡슐화하는 작은 클래스를 정의 하는 것


```python
class CountMissing(object):
    def __init__(self):
        self.added = 0
        
    def missing(self):
        self.added += 1
        return 0
```

- 다른 언어에서라면 이제 `CountMissing` 인터페이스를 수용하도록 `defaultdict`를 수정해야 한다고 생각할 것이다.
- 하지만 파이썬에서는 일급 함수 덕분에 객체로 `CountMissing.missing` 메서드를 직접 참조해서 `defaultdict`의 기본값 후크로 넘길 수 있다.
- 메서드가 함수 인터페이스를 충족하는 건 자명하다


```python
counter = CountMissing()
result = defaultdict(counter.missing, current)   # 메서드 참조

for key, amount in increments:
    result[key] += amount
assert counter.added == 2
```

- 헬퍼 클래스로 상태 보존 클로저의 동작을 제공하는 방법이 앞에서 `increment_with_report` 함수를 사용한 방법보다 명확
- 그러나 `CounterMissing` 클래스 자체만으로는 용도가 무엇인지 바로 이해하기 어렵다. 누가 `CountMissing`객체를 생성하는가? 누가 `missing` 메서드를 호출하는가? 나중에 다른 공개 메서드를 클래스에 추가할 일이 있을까? `defaultdict`와 연계해서 사용한 예를 보기 전까지는 이 클래스가 수수께끼로 남는다.

---

- 파이썬에서는 클래스에 `__call__`이라는 특별한 메서드를 정의해서 이런상황을 명확하게 할 수 있다.
- `__call__` 메서드는 객체를 함수처럼 호출할 수 있게 해준다.
- 또한 내장 함수 `callable`이 이런 인스턴스에 대해서는 `True`를 반환하게 만든다.


```python
class BetterCountMissing(object):
    def __init__(self):
        self.added = 0
        
    def __call__(self):
        self.added += 1
        return 0
    
counter = BetterCountMissing()
counter()
assert callable(counter)
```


```python
# 다음은 BetterCountMissing 인스턴스를 defaultdict의 기본값 후크로 사용하여 딕셔너리에 없어서 새로 추가된 키의 개수를 알아내는 코드
counter = BetterCountMissing()
result = defaultdict(counter, current)   # __call__이 필요함
for key, amount in increments:
    result[key] += amount
assert counter.added == 2
```

- 이 예제가 `CountMissing.missing` 예제보다 명확하다.
- `__call__` 메서드는 (API 후크처럼) 함수 인수를 사용하기 적합한 위치에 클래스의 인스턴스를 사용할 수 있다는 사실을 드러낸다.
- 이 코드를 처음 보는 사람을 클래스의 주요 동작을 책임지는 진입점(entry point)으로 안내하는 역할도 한다.
- 클래스의 목적이 상태 보존 클로저로 동작하는 것이라는 강력한 힌트를 제공

---

- 무엇보다도 `__call__` 을 사용할 때 `defaultdict`는 여전히 무슨 일이 일어나는지 모른다
- `defaultdict`에 필요한 건 기본값 후크용 함수뿐이다. 파이썬은 하고자 하는 작업에 따라 간단한 함수 인터페이스를 충족하는 다양한 방법을 제공

## 핵심 정리

- 파이썬에서 컴포넌트 사이의 간단한 인터페이스용으로 클래스를 저으이하고 인스턴스를 생성하는 대신에 함수만 써도 종종 충분
- 파이썬에서 함수와 메서드에 대한 참조는 일급이다. 즉, 다른 타입처럼 표현식에서 사용할 수 있다.
- `__call__`이라는 특별한 메서드는 클래스의 인스턴스를 일반 파이썬 함수처럼 호출할 수 있게 해준다.
- 상태를 보존하는 함수가 필요할 때 상태 보존 클로저를 정의하는 대신 `__call__` 메서드를 제공하는 클래스를 정의하는 방안을 고려하자.
