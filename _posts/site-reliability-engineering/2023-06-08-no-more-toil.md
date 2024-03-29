---
layout: "post"
title: "5. 삽질은 이제 그만!"
date: "2023-06-08 00:10+0900"
category: Site Reliability Engineering
tag: [Site Reliability Engineering]
---

# 5. 삽질은 이제 그만

## 삽질의 정의

- 프로덕션 서비스를 운영하는 것과 직접적으로 연관이 있지만 수작업을 동반하고, 반복적이며, 자동화가 가능하고, 사후 대처가 필요하며, 지속적인 가치가 결여되어 있으면서도 서비스의 성장에 따라 지속적으로 늘어나는 업무

#### 수작업을 필요로 한다

- 자동화된 작업을 실행하기 위해 수작업으로 스크립트를 실행하는 경우
- 스크립트를 실행하기 위해 소비하는 시간은 분명히 삽질에 소비된 시간

#### 반복적이다

- 삽질은 계속해서 반복되는 작업
- 새로운 문제를 해결 중이거나 새로운 솔루션을 개발하는 작업은 삽질이라고 보지 않는다

#### 자동화가 가능하다

- 작업이 기본적으로 인간의 판단에 의해 실행되어야 한다면 삽질로 분류되지 않을 수도 있다.

#### 사후 대처가 필요하다

- 업무를 방해하며, 사후에 처리하게 되는 일들

#### 가치가 지속되지 않는다

- 어떤 작업을 끝냈는데도 서비스가 계속 같은 상태로 남아있다면 그건 삽질이다

#### 서비스의 성장에 따라 `O(n)`으로 증가한다

- 작업에 필요한 업무량이 서비스 크기나 트래픽 양, 혹은 사용자의 수에 따라 선형적으로 증가한다면 이건 삽질이다

## 삽질이 줄어들면 좋은 이유

- 구글의 SRE조직은 전체 작업 시간 중 삽질을 50% 이내로 유지한다는 목표를 공공연하게 가짐
- SRE들은 최소 50%의 시간을 엔지니어링 프로젝트 업무에 투입해서 향후에 발생 가능한 삽질을 줄이거나 혹은 서비스에 새로운 기능을 추가해야 함
- 기능의 개발은 주로 안정성이나 성능 혹은 활용도를 개선하고 결과적으로 삽질의 발생 가능성을 줄이는 것에 중점을 둔다

## 엔지니어링에 해당하는 업무는?

#### 소프트웨어 엔지니어링

- 코드를 작성하거나 수정하고, 관련된 디자인이나 문서화 작업을 수행

#### 시스템 엔지니어링

- 한 번의 노력으로 지속적인 개선을 이루어내기 위해 프로덕션 시스템의 설정을 조정하거나 문서화를 수행

#### 삽질

#### 부하

- 서비스 운영과 직접 관련되지 않은 관리 업무

## 삽질은 무조건 나쁜 것일까?

- 삽질의 양이 얼마 되지 않는다면, 예측이 가능하고 반복되는 작업들은 큰 무리 없이 처리할 수 있다. 이런 업무들을 처리하면 성취감이 있으며, 빠른 시간 내에 처리할 수 있다. 그다지 위험하지도 않고 스트레스도 적은 편이다. 몇몇 사람들을 이 업무를 선호한다.
- 하지만 삽질의 양이 엄청나지기 시작하면 이것은 문제가 된다.

#### 경력 개발이 침체된다

#### 의욕이 저하된다

#### 혼란이 가중된다

#### 성장이 저하된다

#### 좋지 않은 선례를 남기게 된다

- 만일 삽질을 너무 많이 하려고 하면, 함께 일하는 개발팀은 더 많은 삽질을 떠넘기려 하고, 심지어는 개발티이 수행해야 할 운영 업무까지도 떠안게 된다.

#### 인력 유출이 발생한다

#### 신뢰에 문제가 생긴다

## 결론

- 모두가 매주 조금씩 삽질을 걷어낼 수 있다면 서비스를 지속적으로 깔끔하게 유지할 수 있다
- 삽질은 줄이고 창의적인 일에 더 집중하자