---
layout: "post"
title: "파이썬 디자인 패턴 - 구조디자인패턴 - 데코레이터 패턴 2"
date: "2018-09-02 16:30"
category: python Study
tag: [python, designpattern, 구조디자인패턴]
---



### 2-4-2. 클래스 데코레이터

* 읽기 및 쓰기 프로퍼티가 아주 많은 클래스를 사용할 일이 자주 있다.
  - 그러한 클래스에는 비슷비슷한 getter와 setter 코드가 많이 포함되어 있기 마련
  - ex) 어떤 책에 대해 title, ISBN, price, quantity 값을 저장하는 Book 클래스가 있다고 하자
    - 이러한 경우 `@property` 데코레이터를 4개 써야할 것이다.
    - 그런데 이것들은 모두 기본적으로 동일한 코드 (ex: `@property def title(self): return title`)
    - 또한 값 검증 기능이 들어간 세터 메서드도 4개 필요
    - 이때 price와 quantity를 검증하는 코드는 실제로 허용되는 최댓값과 최솟값만 다를 뿐 코드는 완전히 동일
  - 이런 클래스가 많다면 중복에 가까운 코드가 엄청나게 늘어날 수밖에 없음
* 다행히 파이썬의 클래스 데코레이터를 사용하면 이런 중복을 없앨 수 있다.
  - 2.2에서 클래스 데코레이터를 사용해 매번 열 줄 정도 되는 코드를 반복할 필요 없이 인터페이스 검사용 클래스를 만들 수 있었음
  - 여기서는 다른 예로 네 개의 프로퍼티(완전한 검증 코드가 포함된)가 들어있는 Book클래스를 보겠다

```python
@ensure("title", is_non_empty_str)
@ensure("isbn", is_valid_isbn)
@ensure("price", is_in_range(1, 10000))
@ensure("quantity", is_in_range(0, 1000000))
class Book:

    def __init__(self, title, isbn, price, quantity):
        self.title = title
        self.isbn = isbn
        self.price = price
        self.quantity = quantity

    @property
    def value(self):
        return self.price * self.quantity
```

- `self.title`, `self.isbn`등은 모두 프로퍼티이다. 따라서 `__init()__`에서 이러한 프로퍼티를 초기화하면 프로퍼티 세터에 지정된 대로 검증이 이뤄진다.
  - 하지만 직접 이러한 프로퍼티의 getter와 setter를 만드는 코드를 작성하는 대신 클래스 데코레이터를 4번 사용해서 모든 기능을 제공
- `ensure()` 함수는 인자를 두 개(프로퍼티 이름과 검증 함수) 받아 클래스 데코레이터를 만든다. 이렇게 만들어진 클래스 데코레이터는 바로 뒤에 오는 클래스에 적용
- 실제 실행 순서는 `class Book...` > `@ensure('property',...)` > `@ensure('price',...)` > `@ensure("isbn", ...)` > `@ensure("title",...)` 임

```python
def is_non_empty_str(name, value):
    if not isinstance(value, str):
        raise ValueError("{} must be of type srt".format(name))
    if not bool(value):
        raise ValueError("{} may not be empty".format(name))
```

- 이 검증함수는 Book의 title 프로퍼티 검증에 사용
  - 제목이 비어있는지 검사
  - `ValueError`에서 볼 수 있듯이, 프로퍼티의 이름은 오류 메시지를 표시할 때 유용하게 쓸 수 있음

```python
def is_in_range(minimum=None, maximum=None):
    assert minimum in not None or maximum is not None
    def is_in_range(name, value):
        if not isinstance(value, numbers.Number):
            raise ValueError("{} must be a number".format(name))
        if minimum is not None and value < minimum:
            raise ValueError("{} {} is too small".format(name, value))
        if maximum is not None and value > maximum:
            raise ValueError("{} {} is too big".format(name, value))
    return is_in_range
```

- 이 함수는 입력 값이 수(이를 위해 numbers.Number를 사용) 인지 여부와 값이 제한 범위 내에 들어있는지 검사하는 검증 함수를 만들고 그 함수를 반환

```python
def ensure(name, validate, doc=None):
    def decorator(Class):
        privateName = "__" + name
        def getter(self):
            return getattr(self, privateName)
        def setter(self, value):
            validate(name, value)
            setattr(self, privateName, value)
        setattr(Class, name, property(getter, setter, doc=doc))
        return Class
    return decorator
```

- `ensure()` 함수는 프로퍼티 이름, 검증 함수, 그리고 선택적으로 docstring(인라인 도움말 문자열)을 받는다.
  - 따라서 특정 클래스를 대상으로 `ensure()`가 반환하는 클래스 데코레이터를 사용하면, 대상 클래스에는 새로운 프로퍼티가 추가됨
  - `decorator()`함수의 유일한 인자는 클래스다.
    - `decorator()` 함수는 먼저 클래스 외부에서 접근할 수 없는 이름을 만듦
    - 프로퍼티의 값은 이 이름으로 된 애트리 뷰트에 저장한다(따라서 Book 예제에서 `self.title` 프로퍼티의 값은 전용 `self.__title` 애트리뷰트에 저장)
    - 해당 애트리뷰트에 저장된 값을 반환하는 함수 getter함수를 만든다.
      - 내장 함수 `getattr()`은 객체와 애트리뷰트 이름을 인자로 받아 애트리뷰트의 값을 반환
      - 값이 없으면 `AttributeError`를 반환
    - setter함수는 저장된 `validate()` 함수를 호출하고 애트리뷰트를 새로운 값으로 변경
      - 내장 함수 `setattr()`은 객체와 애트리뷰트 이름, 그리고 값을 받아 객체에 해당 애트리뷰트를 설정
      - 만약 이름에 해당하는 애트리뷰트가 객체에 없다면 새로 하나 만듦
    - 이렇게 `setter()`와 `getter()` 를 만든 다음, 프로퍼티 이름으로 대상 클래스에 새 애트리뷰트를 만든다.
      - 이때 프로퍼티 이름(외부로 노출된)으로 내장 `setattr()` 함수를 사용
        - 내장 `property()` 함수는 getter, setter(optional), docstring을 받아 프로퍼티를 반환
        - 이 프로퍼티는 앞에서 본 것처럼 메서드 데코레이터로 사용할 수 있다.
    - 이렇게 변경된 클래스는 `decorator()` 함수가 반환
  - `decorator()` 함수 자체는 `ensure()` 클래스 데코레이터 팩터리 함수가 반환

#### 2-4-2-1. 데코레이터로 프로퍼티 추가하기

- 앞의 예에서는 `@ensure` 클래스 데코레이터를 사용해 검증이 필요한 애트리뷰트를 만들어야 했다.
  - 이런 식으로 데코레이터를 여러 개 겹쳐서 사용하는 것을 좋아하지 않는 사람도 있다.
  - 그런 사람들은 데코레이터는 하나만 사용하고, 클래스의 몸체안에 애트리뷰트를 넣어 가독성을 높이는 쪽을 선호

```python
@do_ensure
class Book:

    title = Ensure(is_non_empty_str)
    isbn = Ensure(is_valid_isbn)
    price = Ensure(is_in_range(1, 10000))
    quantity = Ensure(is_in_range(0, 1000000))

    def __init__(self, title, isbn, price, quantity):
        self.title = title
        self,isbn = isbn
        self.price = price
        self.quantity = quantity

    @property
    def value(self):
        return self.price * self.quantity
```

- 위 코드는 `@do_ensure` 클래스 데코레이터와 `Ensure` 인스턴스를 함께 사용하는 새로운 Book 클래스다.
  - 각 `Ensure`는 검증함수를 저장
  - `@do_ensure` 클래스 데토레이터는 각 `Ensure` 인스턴스를 같은 이름의 프로퍼티로 바꿔치기
  - 물론 여기서 검증함수(`is_non_empty_str()` 등)는 앞에서 본 것과 동일

```python
class Ensure:

    def __init__(self, validate, doc=None):
        self.validate = validate
        self.doc = doc
```

- 이 클래스는 프로퍼티의 세터에서 사용될 검증 함수와 문서화 문자열을 담아두는데 사용
  - ex) Book 클래스의 title 애트리뷰트는 Ensure 인스턴스로 되어있음
  - 하지만 일단 Book 클래스가 만들어지고 `@do_ensure` 데코레이터가 모든 Ensure을 프로퍼티로 바꿔치기 함
  - 결국 title 애트리뷰트는 결국 마지막에는 title 프로퍼티가 된다.

```python
def do_ensure(Class):
    def make_property(name, attribute):
        privateName = "__" + name
        def getter(self):
            return getattr(self, privateName)
        def setter(self, value):
            attribute.validate(name, value)
            setattr(self, privateName, value)
        return property(getter, setter, doc=attribute.doc)
    for name, attribute in Class.__dict__.items():
        if isinstance(attribute, Ensure):
            setattr(Class, name, make_property(name, attribute))
    return Class
```

- 이 클래스 데코레이터는 세 부분으로 구성
  - 첫 부분에서는 중첩된 함수 (`make_property()`)를 정의
    - 이 함수는 이름(ex:title) 과 Ensure 타입의 애트리뷰트를 받아 전용 애트리뷰트 (ex:__title)에 값을 저장하는 새로운 프로퍼티를 만들고 반환
    - 아울러 해당 프로퍼티의 세터에 접근하는 경우 검증함수를 호출
  - 두 번째 부분에서는 클래스의 모든 애트리뷰트를 순회하면서 모든 Ensure를 새로운 프로퍼티로 바꿔치기
  - 세 번째 부분에서는 이렇게 변경된 클래스를 반환
- 데코레이터가 호출되고 나면 데코레이션된 클래스의 모든 Ensure 애트리뷰트는 같은 이름의 프로퍼티로 바뀌너다.
  - 또한 각 프로퍼티에는 유효성 검증이 추가
- 이론적으로는 함수가 중첩되지 않게 하고 대신 그 코드를 `if isinstance()`다음에 넣을수도 있음
  - 하지만 그렇게 구현하면 바인딩 시점의 문제로 제대로 동작하지 않는다.
  - 따라서 여기서는 중첩 함수를 꼭 사용해야 함
  - 데코레이터나 데코레이터 팩터리를 만드는 경우 보통 이런 식으로 별도의 함수(경우에 따라 중첩된)를 사용

#### 2-4-2-2. 클래스 테코레이터를 상속대신 활용하기

- 메서드나 데이터만 담긴 기반 클래스를 만들고 이 클래스를 상속해서 재사용할 때가 있다.
  - 물론 이렇게 하면 메서드나 데이터를 여러번 정의하지 않아도 되고 하위 클래스를 더 추가해도 잘 동작한다
  - 하지만 하위 클래스에서 상속받은 데이터나 메서드를 결코 변경(재정의)하지 않는 경우, 같은 목적을 달성하기 위해 클래스 데코레이터를 사용할 수도 있다.

---

- 여기서는 `self.mediator` 데이터 애트리뷰트와 `on_change()` 메서드를 제공하는 조정자(mediator) 클래스를 활용할 텐데, 이 클래스는 Button과 Text에 상속되며, 이 둘은 데이터와 메서드를 사용하기는 하되 값을 변경하지는 않는다.

```python
class Mediated:

    def __init__(self):
        self.mediator = None

    def on_change(self):
        if self.mediator is not None:
            self.mediator.on_change(self)
```

- 이 클래스는 일반적인 클래스와 마찬가지로 `class Button(Mediated): ...`과 `class Text(Mediated): ...`를 사용해 상속된다.
  - 하지만 하위 클래스 중 어느 것도 상속받은 `on_change()` 메서드를 변경하지 않는다.
  - 따라서 이러한 경우 상속 대신 클래스 데코레이터를 쓸 수 있다.

```python
def mediated(Class):
    setattr(Class, "mediator", None)
    def on_change(self):
        if self.mediator is not None:
            self.mediator.on_change(self)
    setattr(Class, "on_change", on_change)
    return Class
```

- 클래스 데코레이터는 여느 때와 같이 사용할 수 있다.
  - `@mediated class Button: ...` 이나 `@mediated class Text: ...` 같이 하면 된다.
  - 데코레이션된 클래스는 상속을 사용해 만든 클래스와 동일한 동작 방식을 보여준다.

---

- 함수나 클래스 데코레이터는 아주 강력하지만 사용하기 쉬운 파이썬 기능 중 하나
- 클래스 데코레이터는 경우에 따라 상속을 대신할 수도이 ㅣㅆ음
- 데코레이터를 만드는 것은 메타 프로그래밍의 일종
- 메타 클래스처럼 더 복잡한 메타프로그래밍대신 클래스 데코레이터를 활용할 만한 경우가 많을 것임
