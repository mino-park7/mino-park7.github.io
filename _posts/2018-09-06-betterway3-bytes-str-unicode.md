---
layout: "post"
title: "이펙티브 파이썬 - 3. bytes, str, unicode의 차이점을 알자 "
date: "2018-09-06 21:07"
category: effective python Study
tag: [python, effective,]
---

# 3. bytes, str, unicode의 차이점을 알자

- 파이썬 3 의 문자타입
  - bytes : raw 8 bit
  - str : unicode 문자
- 파이썬 2의 문자타입
  - str : raw 8 bit
  - unicode : unicode 문자

---

- 유니코드 문자를 바이너리 데이터 (raw 8 bit)로 표현하는 방법은 많음 (ex: UTF-8)
- 중요한 것은 파이썬 3의 `str` 인스턴스와 파이썬 2의 `unicode` 인스턴스는 **연관된 바이너리 인코딩이 없음**
  - unicode -> binary data : `encode` method
  - binary data -> unicode : `decode` method

---

- 파이썬 프로그래밍을 할 때 외부에 제공할 인터페이스에서는 유니코드를 인코드하고 디코드 해야함
- **프로그램의 핵심 부분**에서는 **유니코드 문자 타입** 사용
  - 문자 인코딩에 대해서는 어떤 가정도 하지 말아야 함
- 출력 텐스트 인코딩 (이상적으로는 UTF-8)을 엄격하게 유지하면서 다른 텍스트 인코딩 수용 가능

---

- 문자 타입이 분리되어 있는 탓에 파이썬 코드에서 일반적으로 다음 두 가지 상황에 부딪힘
  - UTF-8(or 다른 인코딩)으로 인코드된 문자인 raw 8 bit 값을 처리하려는 상황
  - 인코딩이 없는 유니코드 문자를 처리하려는 상황
- 이 두경우 사이에서 변환하고 코드에서 원하는 타입과 입력값의 타입이 정확히 일치하게 하려면 헬퍼 함수 두 개가 필요
  - 파이썬 3
    - 먼저 `str`이나 `bytes`를 입력으로 받고 `str`을 반환하는 메서드 필요
    ```python
    def to_str(bytes_or_str):
        if isinstance(bytes_or_str, bytes):
            value = bytes_or_str.decode('utf-8')
        else:
            value = bytes_or_str
        return value # str 인스턴스
    ```
    - `str` or `bytes` 를 받아 `bytes` 반환하는 메서드 필요
    ```python
    def to_bytes(bytes_or_str):
        if isinstance(bytes_or_str, str):
            value = bytes_or_str.encode('utf-8')
        else:
            value = bytes_or_str
        return value # bytes 인스턴스
    ```

  - 파이썬 2
    - 먼저 `str`이나 `unicode`를 입력으로 받고 `unicode`를 반환하는 메서드가 필요
    ```python
    def to_unicode(unicode_or_str):
        if isinstance(unicode_or_str, str):
            value = unicode_or_str.decode('utf-8')
        else:
            value = unicode_or_str
        return value # unicode 인스턴스
    ```
    - `str` or `unicode`를 입력으로 받아 `unicode`를 반환하는 메서드 필요
    ```python
    def to_str(unicode_or_str):
        if isinstance(unicode_or_str, unicode):
            value = unicode_or_str.encode('utf-8')
        else:
            value = unicode_or_str
        return value # str 인스턴스
    ```

---

- 파이썬에서 raw 8bit 와 unicode 문자를 처리할 때는 **중대한 이슈 2개**가 있다.
  1. 파이썬 2에서 `str`이 7bit ascii 문자만 포함하고 있다면, `unicode`와 `str` 인스턴스가 같은 타입으로 보임
    - 이런 `str`과 `unicode` 를 `+` 연산자로 묶을 수 있음
    - `=` or `!=` 로 비교 가능
    - `%s` 같은 포맷 문자열에 `unicode` 인스턴스 사용 가능
    - 파이썬 3에서는 `bytes`와 `str`은 빈 문자열인 경우에도 절대 같이 않으므로, 함수에 넘기는 문자열의 타입을 신중하게 처리해야함

  2. 파이썬 3에서 내장 함수 `open()`이 반환하는 파일 핸들을 사용하는 연산 : `utf-8`
    - 파이썬 2에서는 기본으로 바이너리 인코딩 사용
    - 아래 코드는 python 2에서는 작동하지만, python 3에서는 작동하지 않는다
    ```python
    with open('/tmp/random.bin', 'w') as f:
        f.write(os.urandom(10))

    >>>
    TypeError: must be str, not bytes
    ```
    - 문제가 일어난 이유는 파이썬3의 `open`에 `encoding` 인수가 추가되었고 기본값은 `utf-8`
    - 따라서 파일 핸들을 사용하는 `read`나 `write` 연산은 바이너리 데이터를 담은 `bytes` 인스턴스가 아니라, 유니코드 문자를 담은 `str` 인스턴스를 기대
    - 해결법
    ```python
    with open('/tmp/random.bin', 'wb') as f:
        f.write(os.urandom(10))
    ```

    - 파일 오픈 시 `rb` 를 사용하여 바이너리 모드임을 알리면 된다.

---

## 핵심정리

- 파이썬 3에서 `bytes` : raw 8bit, `str` : unicode
  - > or + 같은 연산자에 `bytes`와 `str` 인스턴스를 함께 사용할 수 없다.
- 파이썬 2에서 `str` : raw 8 bit, `unicode` : unicode
  - 만약 7 bit ascii 사용시 연산자에 `str`과 `unicode` 인스턴스 동시 사용 가능
- 헬퍼 함수를 사용해서 처리할 입력값 원하는 문자 시퀀스 타입 (8 bit, utf-8 encoding, unicode)으로 되어있게 한다.
- 바이너리 데이터를 파일에서 읽거나 쓸 때는 파일을 바이너리 모드 (`rb` or `wb`)로 오픈
