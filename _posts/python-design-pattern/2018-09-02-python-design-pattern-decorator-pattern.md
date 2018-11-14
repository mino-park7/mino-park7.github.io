---
layout: "post"
title: "파이썬 디자인 패턴 - 구조디자인패턴 - 데코레이터 패턴 1"
date: "2018-09-02 13:32"
category: python Study
tag: [python, designpattern, 구조디자인패턴]
---




## 2-4. 데코레이터 패턴

* 일반적으로 데코레이터(decorator)는 어떤 함수를 인자로 받아 원래 함수와 이름은 같지만 기능은 더 향상된 함수를 반환하는 함수를 말함
* 데코레이터는 프레임워크에서 프레임 워크과 개발자가 필요로 하는 기능을 **편하게 통합** 하는데 사용
* 파이썬에는 데코레이터 기능을 내장
  - 함수와 메서드에 모두 적용 가능
  - 클래스 데코레이터도 만들 수 있다.
    - 클래스를 유일한 인자로 받아 새로운 기능이 추가된 이름이 같은 새 클래스를 반환
  - 상속 대진 클래스 데코레이터 사용 가능

---

- 파이썬의 내장 `property()` 함수는 데코레이터로 사용 가능
  - 2-3 컴포지트 패턴에서 composite와 price 프로퍼티에서 사용
- 파이썬의 표준 라이브러리에도 몇몇 데코레이터가 들어있음
  - ex) `@functools.total_ordering` 클래스 데코레이터는 `__eq()__` 과 `__lt()__` 특수메서드를 구현한 클래스에 적용가능
    - `total_ordering` 데코레이터를 적용하면 클래스의 이름은 같지만 해당 클래스에서 모든 다른 비교 연산자를 활용 가능

---

- 데코레이터는 한 함수, 메서드, 클래스만을 인자로 받을 수 있다.
- 이론적으로는 데코레이터를 매개변수화 할 수 없다.
- 하지만 데코레이터 함수를 반환하는 데코레이터 팩터리를 만들 수 있고, 이 팩터리는 매개변수를 방을 수 있다.
- 이 데코레이터 팩터리로 만든 데코레이터를 다시 함수, 메서드, 클래스 등에 적용할 수 있다.

---

### 2-4-1. 함수 및 메서드 데코레이터

- 모든 함수(메서드) 데코레이터는 구조가 같다.
  - 우선 래퍼 함수를 만든다.
  - 래퍼 함수 안에서는 반드시 원래의 함수를 호출
  - 래퍼 안에서는 함수를 호출하기 전에 원하는 전처리를 수행 할 수 있고, 호출 결과를 받아 후처리를 할 수도 있다.
  - 원래 함수가 반환한 값을 그대로 반환하거나 변경된 값을 반환하거나, 원한다면 어떤 값이든 반환할 수 있다.
  - 이렇게 변화된 함수는 원래 함수와 똑같은 이름으로 원래 함수를 대체
- 데코레이터를 함수, 메서드, 클래스에 적용하려면 `@` 을 `def`나 `class`와 같은 들여쓰기 수준으로 입력한 다음, 데코레이터의 이름을 적으면 된다.
- 데코레이터를 계속 둘러싸는 것도 가능
  - 데코레이터가 적용된 함수에 다시 데코레이터를 적용하고, 그런 과정을 반복 가능

---

```python
@float_args_and_return
def mean(fisrt, second, *rest):
    numbers = (first, second) + rest
    return sum(numbers) / len(numbers)
```

- 위 코드에서 `@float_args_and_return` 데코레이터를 사용해 `mean()` 함수를 변경했다.
- 데코레이션 되기 전의 `mean()`함수는 둘 이상의 수를 받아 평균을 float로 돌려준다.
- 데코레이션 된 `mena()`에서는 어떤 종류의 인자든 두개 이상 받아 float로 변환해 평균을 계산
  - 데코레이션 하기 전이라면 `mean(5, '6', '7.5')`는 `TypeError`를 반환 할 것이다
  - 하지만 데코레이션된 버전에서는 `float('6')` ,`float('7.5')` 로 처리해주어서 잘 작동
- 근데 데코레이션 문법은 단지 편의 문법(syntactic sugar)에 불과하다. 위 코드를 아래와 같이 작성해도 동일하게 동작한다.

```python
def mean(first, second, *rest):
    numbers = (fisrt, second) + rest
    return sum(numbers) / len(numbers)
mean = float_args_and_return(mean)
```

- 여기서는 데코레이터 없이 함수를 만든 다음, 데코레이터를 직접 호출하는 방식으로 데코레이션 된 버전
- 데코레이터를 사용하는 것은 아주 편리하지만 때때로 직접 호출해야 하는 경우도 있음

```python
def float_args_and_return(function):
    def wrapper(*args, **kwargs):
        args = [float(arg) for arg in args]
        return float(function(*args, **kargs))
    return wrapper
```
- 함수 데코레이터인 `float_args_and_return` 함수는 어떤 함수를 유일한 인자로 받음
- 래퍼함수는 `*args`와 `*kargs`를 받는 것이 일반적임
- 인자에 대한 제약사항은 원래의(래핑이전의) 함수가 처리한다.
- 이 예제의 경우
  - 해퍼 함수는 위치에 따른 인자를 부동소수점 수 리스트로 만든다.
  - 그 다음 원래의 함수를 변경된 `*args`로 호출하고, 결과를 받아 float로 변환해 반환
  - 데코레이터 함수 전레의 결과값으로는 바로 이 래퍼 함수를 돌려주면 된다.
- 하지만 아쉽게도 반환된 함수의 `__name__` 애트리뷰트는 원래 함수의 이름이 아니고 `wrapper`가 된다.
- 문서화 문자열(docstring)도 없어진다.
- 이러한 문제를 해결하기 위해 파이썬 표준 라이브러리에는 `@functools.wrap` 데코레이터가 들어있다.
  - 이는 데코레이터 안에 있는 래퍼 함수에게 사용 가능
  - 래핑된 함수의 `__name__`과 `__doc__`이 원래 함수의 내용을 유지할 수 있게 해줌

```python
def float_args_and_return(function):
    @functools.wraps(function)
    def wrapper(*args, **kwargs):
        args = [float(arg) for arg in args]
        return float(function(*args, **kargs))
    return wrapper
```

- 위와 같이 코드를 작성하면 `__name__`과 `__doc__`이 유지된 채로 데코레이터 사용가능 하다.
- 항상 `@functools.wraps`를 사용하는 것이 좋다.
  - 오류를 추적할 때 래핑된 함수의 이름이 제대로 표시
  - 원본 함수의 문서화 문자열도 볼수 있음

---

```python
@statically_typed(str, str, return_type=str)
def make_tagged(text, tag):
    return "<{0}>{1}</{0}>".format(tag, escape(text))

@statically_typed(str, int, str) # 어떤 반환 타입이든 받아들일 수 있음
def repeat(what, count, separator):
    return((what + separator) * count)[:-len(separator)]
```

- `statically_typed()` 함수는 `make_tagged()`와 `repeat()` 함수를 데코레이션 할 때 사용함
- 데코레이터 팩토리, 즉 데코레이터를 만드는 함수다.
- `statically_typed()`는 함수나 메서드 또는 클레스를 유일한 인자로 받지 않기 때문에 데코레이터는 아님
  - 하지만 여기서는 데코레이터를 매개변수화 할 필요가 있음
  - 데코레이션 되는 함수가 받아들일 수 있는 위치 기반 인자의 자료형과 개수를 지정하고 싶고, 그러한 특성을 함수마다 다르게 적용하고 싶기 때문
  - 따라서 `statically_typed()` 함수를 만들어 원하는 매개변수를 받고, 데코레이터를 반환
- 파이썬이 코드상에서 `@statically_typed(...)`를 만나면 인자를 `statically_typed()` 함수에 전달
  - 그런 다음 반환받은 데코레이터를 다시 다음에 오는 함수 (여기서는 `make_tagged()`와 `repeat()`)에 적용
- 데코레이터 팩터리를 만드는 것에도 패턴이 있다.
  - 먼저 데코레이터 함수 생성
  - 그 함수의 내부에 래퍼 함수 생성
  - 보통 래퍼의 마지막 부분에서는 원래 함수의 결과값(필요 시 변경하거나 새 값으로 대체된)이 반환
  - 데코레이터 함수의 끝에서는 래퍼가 반환
  - 마지막으로 데코레이터 팩터리 함수 끝에서 데코레이터 함수를 반환

```python
def statically_typed(*types, return_type=None): #데코레이터 팩토리
    def decorator(function):                    #팩토리가 반환할 테코레이터 함수
        @functools.wraps(function)
        def wrapper(*args, **kwargs):
            if len(args) > len(types):
                raise ValueError("too many arguments")
            elif len(args) < len(types):
                raise ValueError("too few arguments")

            for i, (arg, type_) in enumerate(zip(args, types)):
                if not isinstance(arg, type_):
                    raise ValueError("argument {} must be of type{}"
                          .format(i, type_.__name__))
            result = funtion(*args, **kwargs)
            if (return_type is not None and
                not isinstance(result, return_type)):
                raise ValueError("return value must be of type {}"
                    .format(return_type.__name__))

            return result
        return wrapper
    return decorator
```

- 여기서 먼저 데코레이터 함수를 생성
  - 함수의 이름을 `decorator()`로 지정하긴 했지만 이름은 중요 ㄴㄴ
  - 데코레이터 함수 내부에는 래퍼 생성
    - 래퍼가 조금 복잡 : 위치 기반 인자의 자료형과 개수를 원래의 함수를 호출하기 전에 검사, 특정 반환 자료형이 명시돼 있다면 반환 자료명 검사, 그 다음 결과 반환
  - 래퍼가 만들어지고 나면 데코레이터는 해당 래퍼를 반환
  - 그 다음 마지막에 데코레이터 자체를 반환
- ex)
  - 소스 코드에 `@statically_typed(str, int, str)`라는 구문이 있다면 파이썬은 `statically_typed()`함수를 호출
    - 그러면 만들어진 `decorator()` 함수가 반환
      - 이 함수에는 `statically_typed()`에 전달된 인자의 정보가 저장
    - 이제 다시 @로 돌아와서 파이썬은 `@statically_typed(str, int, srt)` 구문 다음에 오는 함수 (def로 정의했거나 다른 데코레이터에서 반환하는 데코레이션된 함수)를 유일한 인자로 `decorator()`에 유일한 인자로 전달
    - 여기서는 다음에 오는 함수가 `repeat()`였다.
      - 따라서 `repeat`가 `decorator()`에 유일한 인자로 전달
    - 이제 `decorator()`는 저장된 자료형 정보에 따른 새로운 `wrapper()` 함수를 만들고 반환
  - 이 래퍼 함수는 원래의 `repeat()` 함수를 대체

---

```python
@application.post("/mailinglists/add")
@Web.ensure_logged_in
def person_add_submit(username):
    name = bottle.request.forms.get("name")
    try:
        id = Data.MailingLists.add(name)
        bottle.redirect("/mailinglists/view")
    except Data.Sql.Error as err:
        return bottle.mako_template("error", url="/mailinglists/add",
                text="Add Mailinglist", message=str(err))
```

- 이 코드는 메일 목록을 관리하는 웹 애플리케이션에서 가져왔으며, 경량 웹 프레임워크인 bottle(bottlepy.org)을 활용
- 프레임워크에서 지원하는 `@application.post` 데코레이터는 함수와 URL을 연관시키는데 사용
  - 위 예제에서는 사용자가 로그인해 있을 때에만 mailinglist/add 페이지에 접근할 수 있고, 로그인한 상태가 아니라면 login 페이지로 이동하게 하고싶다.
  - 웹 페이지를 생성하는 모든 함수마다 사용자가 로그인했는지 검사하는 똑같은 코드를 넣는 대신, 이러한 일을 처리하는 `@Web.ensure_logged_in` 데코레이터를 생성해서 이러한 기능이 필요한 어떤 함수든 로그인 관련 코드에 신경쓰지 않게 했다.

```python
def ensure_logged_in(function):
    @functools.wraps(function)
    def wrapper(*args, **kwargs):
        username = bottle.request.get_cookie(COOKE,
                      secret=secret(bottle.request))

        if username is not None:
            kwargs["username"] = username
            return function(*args, **kwargs)
        bottle.redirect("/login")
    return wrapper
```

- 사용자가 웹 사이트에 로그인할 때, login 페이지에 있는 코드는 사용자 계정명과 암호를 검증하고, 유효하다면 사용자 브로우저에 해당 세션동안 유효한 쿠키를 생성
- 사용자가 접근하려는 페이지와 연게된 함수 (예: mailinglists/add 페이지의 person_add_submit())에 `@ensure_logged_in` 데코레이터가 설정돼 있다면
  - 위의 코드에 있는 `wrapper()`함수가 호출된다.
  - 래퍼는 먼저 쿠키로부터 사용자 계정명을 조회
    - 조회에 실패하면 사용자가 로그인하지 않은 상태이므로 웹어플리케이션의 login 페이지로 이동시킴
    - 사용자가 로그인한 상태라면 키워드 인자에 사용자 계정명을 추가하고 호출 경과를 원래 함수에 반환
  - 즉 원래 함수를 호출한 시점에 사용자가 유효하게 로그인해 있을을 안전하게 가정가능
  - 사용자 계정명에 대한 권한을 얻을 수 있음
