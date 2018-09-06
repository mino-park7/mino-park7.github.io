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
    -
  - 파이썬 2에서는 먼저 `str`이나 `unicode`를 입력으로 받고 `unicode`를 반환하는 메서가 필요
  ```python
  def to_unicode(unicode_or_str):
      if isinstance(unicode_or_str, str):
          value = unicode_or_str.decode('utf-8')
      else:
          value = unicode_or_str
      return value # unicode 인스턴스
  ```
