---
layout: "post"
title: "이펙티브 파이썬 - 6. 한 슬라이스에 start, end, stride를 함께 쓰지 말자"
date: "2018-09-09 18:08"
category: effective python Study
tag: [python, effective,]
---



# 한 슬라이스에 start, end, stried를 함께 쓰지 말자

- `somelist[start:end:stride]` 처럼 슬라이스의 스트라이드(간격)를 설정하는 문법이 있다.
  - 시퀀스를 슬라이스 할 때, 매 n번째 아이템을 가져올 수 있음

```python
a = ['red', 'orange', 'yellow', 'green', 'blue', 'purple']
odds = a[::2]
evens = a[1::2]
print(odds)
print(evens)

>>>
['red', 'yellow', 'blue']
['orange', 'green', 'purple']
```

- 문제는 stride문법이 종종 예상치 못한 동작을 해서 버그를 만들어내기도 하는 것
  - ex: 파이썬에서 바이트 문자열을 역순으로 만드는 일반적인 방법은 스트라이드 -1로 문자열을 슬라이스하는 것
  ```python
  x = b'mongoose'
  y = x[::-1]
  print(y)

  >>>
  b'esoognom'
  ```

  - 위의 코드는 바이트 문자열이나 아스키 문자에는 잘 동작하지만, **utf-8 바이트 문자열로 인코드된 유니코드 문자에는 원하는 대로 동작하지 않음**

  ```python
  w = '正正'
  x = w.encode('utf-8')
  y = x[::-1]
  z = y.decode('utf-8')

  >>>
  UnicodeDecodeError: 'utf-8' codec can`t decode byte 0x9d in
  position 0: invalid start byte
  ```

---

- stride를 사용하면 리스트 내의 인수를 다양하게 뽑아낼 수 있지만 문제가 있음
- 슬라이싱 문법의 `stride` 부분이 매우 혼란스러울 수 있음
  - 대괄호 안에 숫자가 세 개나 있으면 빽빽해서 읽기 어려움, 그래서 start와 end 인덱스가 stride와 연계되어 어떤 작용을하는지 불분명
  - 특히 `stride`가 음수인 경우는 더 심함
- `stride`를 사용해야 한다면 양수 값을 사용하고, start와 end 인덱스는 생략하자
- `stride`를 꼭 start나 end 인덱스와 함께 사용해야 한다면
  - `stride`를 적용한 결과를 변수에 할당하고, 이 변수를 슬라이스한 결과를 다른 변수에 할당해서 사용
  ```python
  b = a[::2]
  c = b[1:-1]
  ```

- 슬라이싱부터 하고 스트라이딩을 하면 데이터의 얕은 복사본이 추가로 생김
  - 첫 번째 연산은 결과로 나오는 슬라이스의 크기를 최대한 줄여야함 (보통 stride가 1/n 크기로 리스트를 줄이므로 먼저 쓰자)
  - 시간과 메모리가 충부하지 않다면 내장 모듈 `itertools`의 `islice` 메서드를 사용
  - `islice` 메서드는 start, end, stride에 음수값을 허용하지 않음

---

## 핵심정리
- 한 슬라이스에 start, end, stride를 지정하면 매우 혼란스러울 수 있다.
- 슬라이스에 start와 end 인덱스 없이 양수 stride값을 사용하자. 음수 stride 값은 가능하면 피하는게 좋다.
- 한 슬라이스에 start, end, stride를 함께 사용하는 상황은 피하자. 파라미터 세 개를 사용해야 한다면 할당 두개 (하나는 슬라이스, 다른 하나는 스트라이드)를 사용하거나 내장모듈 `itertools`의 `islice`를 사용하자
