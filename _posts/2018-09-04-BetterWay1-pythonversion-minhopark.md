---
layout: "post"
title: "이펙티브 파이썬 - 1. 사용 중인 파이썬의 버전을 알자"
date: "2018-09-04 20:20"
category: effective python Study
tag: [python, effective,]
---





# 1. 사용중인 파이썬의 버전을 알자

* python 명령어는 보통 python2.7을 가리키지만 때때로 2.6, 2.5를 가르킬 수도 있기 때문에, --version 플래그를 사용하여 버전 확인하자

```bash
$ python --version
Python 2.7.8
```

* 파이썬3은 보통 python3으로 실행

```bash
$ python3 --version
Python 3.5.2
```

* 파이썬에 내장된 sys 모듈을 이용하여 파이썬의 버전을 알아낼 수도 있음

```python
import sys
print(sys.version_info)
print(sys.version)

>>>
sys.version_info(major=3, minor=4, micro=2, releaselevel='final', serial=0)
3.4.2 (default, ....
```

* 파이썬 2와 3 모두 커뮤니티에서 유지보수 되는중
  * 파이썬 2의 개발은 버그 수정, 보안 강화, 2를 3으로 쉽게 포팅하는 기능 이외에는 중지됨
  * 2to3, six 와 같은 module을 사용하면 파이썬 3과 호환이 쉬워짐
* 파이썬 3은 새 기능과 향상이 지속적으로 이루어짐.
* 새로 프로젝트를 실행한다면 파이썬 3 사용을 적극 추천

---
* **핵심정리**
  * 파이썬의 주요 버전인 파이썬2, 3 모두 활발히 사용됨
  * 파이썬에는 CPython, Jython, IronPython, PyPy 같은 다양한 런타임이 존재함
  * 시스템에서 파이썬을 실행하는 명령이 사용하고자 하는 파이썬 버전인지 확이 필요
  * 파이썬 3 추천
---

## 추가 정리 - python runtime

1. CPython
    - C로 짜여진 파이썬
    - 우리가 가장 흔히 쓰는 파이썬 인터프리터 - 파이썬은 C로 짜여져 있다!
    - Global Interpreter Lock로 인한 멀티쓰레딩 이슈가 있음

2. CPython
    - 싸이썬은 파이썬의 superset
      - 파이썬이 동적으로 결정되는 부분이 있어서 인터프리팅을 해야함, 그래서 느리다면?
        - 그것을 정적으로 결정하고 컴파일하자! -
    - 인터프리팅이 느리다 > 컴파일이 필요하다 > 모두 다 할 필요가...? > 속도에 치명적인 영향을 주는 부분만 컴파일하자!

3. PyPy
    - 파이썬으로 작성된 파이썬 인터프리터
    - **JIT** 기술을 활용해 **Cpython보다 빠름!**
      - JIT?: Just In Time 컴파일러
        - 파이썬 바이트코드를 인터프리팅 하다가, 자주 사용되는 부분은 **컴파일**함. 즉 처음에는 느리게 돌아가지만, 점점 빨라 짐
    - 현재 파이썬 구현체중 가장 각광받는 중

4. Stackless python
    - CPython 구현 시, 파이썬의 함수 호출 스택을 C의 스택에 그대로 얹음
      - 파이썬에서 메모리를 얼마나 쓸 수 있는지와 상관없이 **C 스택을 꽉 채우면 스택 오버플로 에러가 뜬다**
      - 파이썬 프로그램의 호출 스택(프로그램의 실행흐름)을 CPython 스스로 제어 할 수 없어서 **코루틴** 등 실행 흐름을 제어하는 언어 기능을 쓸 수 없음
        - 코루틴?: caller가 함수를 call하고, 함수가 caller에게 값을 return하면서 종료하는 것에 더해 return 하는 대신 suspend(or yeild)하면 caller가 나중에 resume하여 중단된 지점부터 실행을 이어나갈 수 있음
    - 이러한 단점들 때문에, Cpython 소스를 수정하여 C스택 쓰는 부분을 모두 교체한 것이 Stackless python
      - PyPy에 밀려서 사양길...

5. Jython
    - java 위에서 돌라가는 파이썬
    - GIL 이슈가 없어서 쓰레드 쓰기 편리함

6. IPython, IPython Notebook
    - IPython은 추가 기능을 제공하는 interactive shell
      - 파이썬 기본 인터프리터의 업그레이드 버전
    - IPython Notebook은 웹 기반 쉘 환경을 제공
      - 파이썬 코드, 텍스트, 수학식, 그래프, 다양한 미디어들을 하나의 도큐먼트로 만들 수 있음

[출처](http://khanrc.tistory.com/entry/다양한-Python들#fnref-f1)
