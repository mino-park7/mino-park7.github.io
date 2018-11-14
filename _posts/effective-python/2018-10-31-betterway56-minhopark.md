---
layout: "post"
title: "이펙티브 파이썬 - 56. unittest로 모든 것을 테스트 하자"
date: "2018-10-31 14:02"
category: effective python Study
tag: [python, effective,]
---




# BetterWay 56. unittest로 모든 것을 테스트 하자

- 파이썬에는 정적 타입 검사 기능이 없음.... 컴파일러는 프로그램이 제대로 동작함을 보장하지 않음
    - c, java 같은 경우는 syntax error같은거를 binary 만들어주는 compiler 동작때 모두 잡아줌
- 파이썬으로는 프로그램에서 호출하는 함수가 런타임에 정의되어 있을지 알 수 없으며, 심지어 해당 함수가 소스코드에 있다고 해도 알 수 없다.
- 일례로, 동적 임포트의 부작용으로 `SyntaxError`가 **제품**에서 발생함

---

- 이러한 문제점을 잡아내기 위해서는 **테스트**가 필수적! 어떤 언어로 프로그램을 작성하든!
- 다행히 파이썬에서는 동적 기능이 파이썬의 정적 타입 검사는 방해하지만, 동시에 코드 테스트를 아주 쉽게 작성할 수 있게 해줌
- 파이선의 동적 특성과 쉽게 오버라이드할 수 있는 동작으로 테스트를 구현하여 프로그램이 예상대로 잘 동작함을 보장 할 수 있음

---

- 테스트는 코드에 대한 보장 정책
- 좋은 테스트는 코드가 올바르게 동작한다는 신회를 줌
- 코드를 리팩토링하거나 확장하는 용도로 테스트를 사용하면, 동작이 어떻게 바뀌는지 쉽게 구별할 수 있음
- 테스트를 작성하는 가장 간단한 방법은 내장 모듈`unittest`를 사용하는 것

- 예를 들어 `utils.py`에 다음과 같은 유틸리티 함수를 정의했다고 하자


```python
# utils.py
def to_str(data):
    if isinstance(data, str):
        return data
    elif isinstance(data, bytes):
        return data.decode('utf-8')
    else:
        raise TypeError('Must supply str or bytes, ' 'found: %r' % data)
```

- 기대하는 각 동작에 대한 테스트를 담은 `test_utils.py`라는 두번째 파일을 생성 (`utils_test.py`)


```python
# utils_test.py
from unittest import TestCase, main
# from utils import to_str

class UtilsTestCase(TestCase):
    def test_to_str_bytes(self):
        self.assertEqual('hello', to_str(b'hello'))

    def test_to_str_str(self):
        self.assertEqual('hello', to_str('hello'))

    def test_to_str_bad(self):
        self.assertRaises(TypeError, to_str, object())

#if __name__ == '__main__':
#    main()
```

    E
    ======================================================================
    ERROR: /Users/minhopark/Library/Jupyter/runtime/kernel-0e8bea65-c31a-4cea-928e-764744e126c2 (unittest.loader._FailedTest)
    ----------------------------------------------------------------------
    AttributeError: module '__main__' has no attribute '/Users/minhopark/Library/Jupyter/runtime/kernel-0e8bea65-c31a-4cea-928e-764744e126c2'

    ----------------------------------------------------------------------
    Ran 1 test in 0.004s

    FAILED (errors=1)



    An exception has occurred, use %tb to see the full traceback.


    SystemExit: True



    /Users/minhopark/tensorflow/lib/python3.5/site-packages/IPython/core/interactiveshell.py:2918: UserWarning: To exit: use 'exit', 'quit', or Ctrl-D.
      warn("To exit: use 'exit', 'quit', or Ctrl-D.", stacklevel=1)


- `TestCase` 클래스는 테스트에서 `assertion`을 하는 데 필요한 헬퍼 메서드를 제공
    - 동등성 검사 `assertEqual`
    - bool 표현식 검사 `assertTrue`
    - 적절할 때 예외가 발생하는지 검사하는 `assertRaises`
    - ... 자세한거는 `help(TestCase)`로 확인 요망
- `TestCase` 서브 클래스에 자신만의 헬퍼 메서드를 정의하여 테스트를 더 이해하기 쉽게 만들 수 있다.
    - 헬퍼 메서드의 이름이 `test`로 시작하지 않게만 하면 됨

### note
- 테스트를 작성할 때 다른 일반적인 관례는 mock함수로 클래스의 특정 동작을 제거하는 것
- 파이썬3는 이런 목적으로 내장 모듈 `unittest.mock`을 제공 (참고 [python mock](https://blog.leop0ld.org/posts/about-mocking/))

- 때때로 `TestCase`클래스에서 테스트 메서드를 실행하기 전에 테스트 환경을 설정해야 함
    - `setUp`, `tearDown` Override 필요
- 위 메서드 들은 각 테스트 메서드를 싱행하기 전후에 각각 호출
- 각 테스트라 분리되어 실행됨을 보장 (적절한 테스트의 모범 사례)
- ex) 각 테스트 전에 임시 디렉터리를 생성하고, 각 테스트가 종료한 후에 그 내용을 삭제하는 `TestCase` 정의


```python
class MyTest(TestCase):
    def setUP(self):
        self.test_dir = TemporaryDirectory()
    def tearDown(self):
        self.test_dir.cleanup()
    # 테스트 메서드
    # ...
```

- 보통 관련 테스트의 각 집합별로 `TestCase`하나 정의
- 때때로 예외 상황이 많은 각 함수별로 `TestCase`만듦
- 또는 때때로 `TestCase`하나로 한 모듈에 속한 모든 함수를 처리하기도 함
- 또한 단일 클래스와 그 클래스의 메서드를 테스트할 용도로 `TestCase`를 생성하기도 함

---

- 프로그램이 복잡해지면 코드를 별도로 테스트하는 대신에 모듈 간의 상호 작용을 검증할 테스트를 추가함 - Integration test (통합 테스트)
- 파이썬도 또한 단위 테스트와 통합 테스트를 모두 진행해야 함

### Note
- 프로젝트에 따라 데이터 중심 테스트를 정의하거나 관련 기능을 다르게 묶어서 테스트를 구성하는 편이 유용 할 수 있음
- 이런 목적과 코드 커버리지 보고서, 다른 고급 활용 사례에는 [`nose`](http://nose.readthedocs.org/)와 [`pytest`](http://pytest.org) 가 도움 된다.

## 핵심 정리
- 파이썬 프로그램을 신뢰할 수 있는 유일한 방법은 테스트 작성!
- 내장 모듈 `unittest`는 좋은 테스트를 작성하는 데 필요한 대부분의 기능을 제공
- `TestCase`를 상속하고 테스트 하려는 동작별로 메서드 하나를 저으이해서 테스트를 정의 할 수 있음
- `TestCase`클래스에 있는 테스트 메서드는 `test`라는 단어로 시작해야 함
- 단위 테스트(고립된 기능용)와 통합 테스트(상호 작용하는 모듈용) 모두를 작성해야 함
