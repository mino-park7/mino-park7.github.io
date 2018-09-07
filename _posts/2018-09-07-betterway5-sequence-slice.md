---
layout: "post"
title: "이펙티브 파이썬 - 5. 시퀀스를 슬라이스하는 방법을 알자"
date: "2018-09-07 21:48"
category: effective python Study
tag: [python, effective,]
---

# 시퀀스를 슬라이스하는 방법을 알자

- 파이썬은 시퀀스를 슬라이스해서 조각으로 만드는 문법을 제공
  - 슬라이싱 대상 : list, str, bytes ...
  - `__getitem__`, `__setitem__` 매직메서드를 구현하는 클래스에서도 슬라이싱 적용 가능
- 기본 형태 : `somelist[start:end]`
  - start 인덱스는 포함, end 인덱스는 제외
- 
