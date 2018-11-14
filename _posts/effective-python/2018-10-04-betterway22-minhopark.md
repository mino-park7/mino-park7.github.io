---
layout: "post"
title: "이펙티브 파이썬 - 22. 딕셔너리와 튜플보다는 헬퍼 클래스로 관리하자"
date: "2018-10-04 01:30"
category: effective python Study
tag: [python, effective,]
---

# BetterWay 22. 딕셔너리와 튜플보다는 헬퍼 클래스로 관리하자

- 딕셔너리 타입은 객체의 수명이 지속되는 동안 동적인 내부 상태를 관리하는 용도로 아주 좋다
   - '동적'이란? : 예상하지 못한 식별자들을 관리해야 하는 상황
- ex) 이름을 모르는 학생 집단의 성적을 기록하고 싶다, 학생별로 미리 정의된 속성을 사용하지 않고 딕셔너리에 이름을 저장하는 클래스를 정의 가능


```python
class SimpleGradebook(object):
    def __init__(self):
        self._grades = {}

    def add_student(self, name):
        self._grades[name] = []

    def report_grade(self, name, score):
        self._grades[name].append(score)

    def average_grade(self, name):
        grades = self._grades[name]
        return sum(grades) / len(grades)
```


```python
# 클래스 사용
book = SimpleGradebook()
book.add_student('Isaac Newton')
book.report_grade('Isaac Newton', 90)

print(book._grades)
print(book.average_grade('Isaac Newton'))
```

    {'Isaac Newton': [90]}
    90.0


- 딕셔너리는 정말 사용하기 쉬워서 과도하게 쓰다가 코드를 취약하게 작성할 위험이 있음
- 예를들어 `SimpleGradebook` 클래스를 확장해서 모든 성적을 한 곳에 저장하지 않고 과목별로 저장한다고 하자
- 이런 경우 `_grades` 딕셔너리를 변경해서 학생이름(키)를 또 다른 딕셔너리(값)에 매핑하면 된다.
- 가장 안쪽에 있는 딕셔너리는 과목(키)를 성적(값)에 매핑한다.


```python
class BySubjectGradebook(object):
    def __init__(self):
        self._grades = {}

    def add_student(self, name):
        self._grades[name] = {}

# report_grade와 average_grade 메서드는 여러단계의 딕셔너리를 처리하느라 약간 복잡해지지만 아직은 다룰만

    def report_grade(self, name, subject, grade):
        by_subject = self._grades[name]
        grade_list = by_subject.setdefault(subject, [])
        grade_list.append(grade)

    def average_grade(self, name):
        by_subject = self._grades[name]
        total, count = 0, 0
        for grades in by_subject.values():
            total += sum(grades)
            count += len(grades)
        return total / count
```


```python
book = BySubjectGradebook()
book.add_student('Albert Einstein')
book.report_grade('Albert Einstein', 'Math', 75)
print(book._grades)
```

    {'Albert Einstein': {'Math': [75]}}



```python
book.report_grade("Albert Einstein", 'Math', 65)
book.report_grade('Albert Einstein', 'Gym', 90)
book.report_grade('Albert Einstein', 'Gym', 95)
print(book._grades)
```

    {'Albert Einstein': {'Gym': [90, 95], 'Math': [75, 65]}}


- 아직 까지는 괜찮았는데.... 요구사항이 더 생겼다고 해보자
- 수업의 최종 성적에서 weight 가 추가가 되서, 중간고사와 기말고사를 쪽지시험보다 중요하게 하려고 한다고 해보자
- 이 기능을 구현하는 방법은 가장 안쪽 딕셔너리를 변경해서 과목(키)을 성적(값)에 매핑하지 않고, 성적과 비중을 담은 튜플 (score, weight)에 매핑하는것이다


```python
class WeightedGradebook(object):
    def __init__(self):
        self._grades = {}

    def add_student(self, name):
        self._grades[name] = {}

    def report_grade(self, name, subject, score, weight):
        by_subject = self._grades[name]
        grade_list = by_subject.setdefault(subject, [])
        grade_list.append((score, weight)) # append에 들어가는 값을 tuple로 변경

    #report_grade 수정은 간단하지만, average_grade 메서드는 2중 루프분이 생긴다
    def average_grade(self, name):
        by_subject = self._grades[name]
        score_sum, score_count = 0, 0
        for subject, scores in by_subject.items():
            subject_avg, total_weight = 0, 0
            for score, weight in scores:
                subject_avg += score * weight
                total_weight += weight
            score_sum += subject_avg
            score_count += total_weight

        return score_sum / score_count
```


```python
# 클래스를 사용하는 방법도 더 어려워짐, 숫자들이 무엇을 의미하는지도 명확하지 않음
book = WeightedGradebook()
book.add_student('Albert Einstein')
book.report_grade('Albert Einstein', 'Math', 80, 0.10)
book.report_grade('Albert Einstein', 'Math', 90, 0.40)
book.report_grade('Albert Einstein', 'Math', 85, 0.50)
book.report_grade('Albert Einstein', 'Gym', 80, 0.10)
book.report_grade('Albert Einstein', 'Gym', 80, 0.40)
book.report_grade('Albert Einstein', 'Gym', 100, 0.50)
print(book._grades)
```

    {'Albert Einstein': {'Gym': [(80, 0.1), (80, 0.4), (100, 0.5)], 'Math': [(80, 0.1), (90, 0.4), (85, 0.5)]}}



```python
print(book.average_grade('Albert Einstein'))
```

    88.25


- 이렇게 복잡해지면 딕셔너리와 튜플 대신 클래스의 계층 구조를 사용할 때가 된 것이다.

---

- 파이썬 내장 딕셔너리와 튜플을 사용하면 계층이 한 단계가 넘는 중첩은 피해야 한다
   - 딕셔너리를 담은 딕셔너리는 피해야 한다.
- 여러 계층으로 중첩하면 다른 프로그래머들이 코드를 이해하기 어려워지고 유지보수의 악몽에 빠지게 된다.
- 관리하기가 복잡하다고 느끼는 즉시 클래스로 옮겨가야한다.
- 데이터를 더 잘 캡슐화한 잘 정의된 인터페이스 제공 가능
- 인터페이스와 실제 구현 사이에 추상화 계층을 만들 수 있음

## 클래스 리팩토링

- 의존 관계에서 가장 아래에 있는 성적부터 클래스로 옮겨보자
- 이렇게 간단한 정보를 담기에 클래스는 너무 무거워 보인다. 성적은 변하지 않으니 튜플을 사용하는 게 적절해 보임
- 다음 코드에서는 리스트 안에 성적을 기록하려고 (score, weight) 튜플을 사용


```python
grades = []
grades.append((95, 0.45))
#...
total = sum(score * weight for score, weight in grades)
total_weight = sum(weight for _, weight in grades)
average_grade = total /total_weight
```

- 문제는 일반 튜플은 위치에 의존한다는 점
- 성적에 선생님의 의견 같은 더 많은 정보를 연관지으려면 이제 튜플을 사용하는 곳을 모두 찾아서 아이템 두 개가 아니라 세 개를 쓰도록 수정해야 한다.
- 다음 코드에서는 세번째 값을 `_`로 받아 그냥 무시하도록 함


```python
grades = []
grades.append((95, 0.45, 'Great job'))
#...
total = sum(score * weight for score, weight, _ in grades)    # _ 사용
total_weight = sum(weight for _, weight, _ in grades)    # _ 사용
average_grade = total /total_weight
```

- 튜플을 점점 더 길게 확장하는 패턴은 딕셔너리의 계층을 깊게 두는 방식과 비슷
- 튜플의 아이템이 두 개를 넘어가면 다른 방법을 고려해야 한다

---

- `collection` 모듈의 `namedtuple` 타입이 정확히 이런 요구에 부합한다.
- `namedtuple`을 이용하면 작은 불면 데이터 클래스(immutable data class)를 쉽게 정의할 수 있다.


```python
import collections
Grade = collections.namedtuple('Grade', ('score', 'weight'))
```

- 불변 데이터 클래스는 위치 인수나 키워드 인수로 생성할 수 있다.
- 필드는 이름이 붙은 속성으로 접근할 수 있다. 이름이 붙은 속성이 있으면 나중에 요구 사항이 또 변해서 단순 데이터 컨테이너에 동작을 추가해야 할 때, `namedtuple`에서 직접 작성한 클래스로 쉽게 바꿀 수 있다

### namedtuple의 제약

`namedtuple`이 여러 상황에서 유용하긴 하지만 장점보다 단점을 만들어낼 수 있는 상황도 이해해야 함

---

- `namedtuple`로 만들 클래스에 기본 인수 값을 설정할 수 없다.
   - 그래서 데이터에 선택적인 속성이 많으면 다루기 힘들어진다.
   - 속성을 사용할 때는 클래스를 직접 정의하는게 나을 수 있다.

- `namedtuple` 인스턴스의 속성 값을 여전히 숫자로 된 인덱스와 순회 방법으로 접근할 수 있다.
   - 특히 외부 API로 노출한 경우에는 의도와 다르게 사용되어 나중에 실제 클래스로 바꾸기 더 어려울 수도 있다.
   - `namedtuple` 인스턴스를 사용하는 방식을 모두 제어할 수 없다면 클래스를 직접 정의하는게 낫다


```python
# 성적들을 담은 단일 과목을 표현하는 클래스를 작성해보자
class Subject(object):
    def __init__(self):
        self._grades = []

    def report_grade(self, score, weight):
        self._grades.append(Grade(score, weight))

    def average_grade(self):
        total, total_weight = 0, 0
        for grade in self._grades:
            total += grade.score * grade.weight
            total_weight += grade.weight
        return total / total_weight

```


```python
# 이제 한 학생이 공부한 과목들을 표현하는 클래스를 작성해보자
class Student(object):
    def __init__(self):
        self._subjects = {}

    def subject(self, name):
        if name not in self._subjects:
            self._subjects[name] = Subject()
        return self._subjects[name]

    def average_grade(self):
        total, count = 0, 0
        for subject in self._subjects.values():
            total += subject.average_grade()
            count += 1
        return total / count
```


```python
# 마지막으로 학생의 이름을 키로 사용해 동적으로 모든 학생을 담을 컨테이너를 작성
class Gradebook(object):
    def __init__(self):
        self._students = {}

    def student(self, name):
        if name not in self._students:
            self._students[name] = Student()
        return self._students[name]
```


```python
# 이 세 클래스의 코드 줄 수는 이전에 구현한 코드의 두 배에 가깝다.
# 하지만 이해하기 훨씬 쉬움
# 사용 예제도 명확하고 확장하시 쉽다.
book = Gradebook()
albert = book.student('Albert Einstein')
math = albert.subject('Math')
math.report_grade(80, 0.10)
# ...
print(albert.average_grade())
```

    80.0


- 필요하면 이전 형태의 API 스타일로 작성한 코드를 새로 만든 객체 계층 스타일로 바꿔주는 하위 호환용 메서드를 작성해도 됨

## 핵심 정리

- 다른 딕셔너리나 긴 튜플을 담은 딕셔너리를 생성하지 말자.
- 정식 클래스의 유연성이 필요 없다면 가벼운 불변 데이터 컨테이너에는 `namedtuple`을 사용하자.
- 내부 상태를 관리하는 딕셔너리가 복잡해지면 여러 헬퍼 클래스를 사용하는 방식으로 관리 코드를 바꾸자
