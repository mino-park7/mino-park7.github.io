---
layout: "post"
title: "9. 코드 리뷰"
date: "2023-02-20 22:22"
category: Software Engineering
tag: [Software Engineering]
---

# 9. 코드 리뷰

- 구글은 자체 리뷰 도구 (Critique)을 활용
- 버그가 코드베이스로 침투하기 전에 잡아낸다
- 미묘한 이점도 많음

## 9.1 코드 리뷰 흐름

- 구글에서는 프리커밋 리뷰를 활용
- 절차는 다음과 같음
  - 작성자가 자신의 작업 공간에서 코드베이스에 적용할 변경사항을 작성. 그 다음 젼경의 스냅샷을 만듬. (스냅샷 : 코드 리뷰 도구에 업로드할 코드 패치와 관련된 설명) 이 변경으로 현재 코드베이스와의 diff를 떠서 코드가 어떻게 달라졌는지를 평가
  - 작성자는 초기 패치를 사용하여 자동 리뷰 의견을 받거나 스스로 리뷰를 해봄. 변경 내용이 만족스럽다면 한 명 이상의 리뷰어에게 메일을 보내 리뷰를 요청.
  - 리뷰어들은 코드 리뷰 도구에서 변경을 열고 해당  diff에 댓글로 의견을 남김. 작성자에게 해결해야 할 문제를 던져주거나, 단순히 유용한 정보를 제공해주기도 함
  - 작성자는 피드백을 기초로 변경을 수정하고 새로운 스냅샷을 업로드한 후 리뷰어들에게 회신 (3~4차례 반복 가능)
  - 리뷰어들이 변경 내용에 모두 만족하면 LGTM 태그를 달아줌
  - 커밋 가능!
- 코드는 부채다! 필요 없는 코드라면 없는게 있는것보다 나음

## 9.2 코드 리뷰 @ 구글

- 구글의 방식이 시간과 규모 측면에서 확장하기 용이한 이유를 알아보자
- Approve를 얻기 위해서 3가지 측면의 리뷰를 통과해야 함
  - 정확성과 이해 용이성 평가
  - 변경되는 코드 영역을 관리하는 코드 소유자로부터 변경 코드가 적정하다는 승인을 방아야 함
  - 가독성 승인을 받아야 함
- 뭔가 통제가 많고 과해보이긴하지만 구글에서는 한다!
- 이는 모두 확장성을 위한 것임
- 구글에서는 소유권을 관리 함 (repository에 OWNERS 파일이 있음)

## 9.3 코드 리뷰의 이점

- 코드가 정확한지 확인해줍니다.
- 변경된 코드를 다른 에니지니어도 잘 이해합니다.
- 코드베이스가 일관되게 관리됩니다.
- 팀이 소유권(주인의식)을 더 강하게 느낍니다.
- 지식이 공유됩니다.
- 코드 리뷰 자체의 기록이 남습니다.

### 9.3.1 코드 정확성

- 결함을 프로세스 초반에 잡아낼수록 시간이 덜 듬
- 코드 리뷰에 들이는 시간은 테스트, 디버그, 회귀 테스트에 투입 되는 시간을 줄여줌

### 9.3.2 코드 이해 용이성

- 코드 리뷰는 주어진 변경이 수많은 다름 사람에게도 쉽게 이해되는지를 평가하는 첫 번째 시험대
- 코드의 정확성과 이해 용이성 검토는 다른 엔지니어가 LGTM을 주기 위해 평가하는 주요 기준 -> 해당 코드가 의도한 일을 정확히 수행하면서 이해하기도 쉽다는 뜻

### 9.3.3 코드 일관성

- 가독성 승인
  - 해당 프로그래밍 언어의 모범 사례들을 잘 따라야 합니다.
  - 구글 코드 리포지터리에서 같은 언어로 작성된 다른 코드들과 일관되어야 합니다.
  - 필요 이상으로 복잡하지 않아야 합니다.

### 9.3.4 심리적, 문화적 이점

- 코드는 '조직의 공동 소유물'
- 작업 결과를 검증하고 인정해주는 심리적 이점
- 자신의 코드를 한번 더 들여다보게 하는 효과

### 9.3.5 지식 공유

- FYI 주석
- 구글 엔지니어들이 코드 리뷰에 쏟아부은 시간으로 유추해보면 축적된 지식의 양이 엄청남을 알 수 있음
- 코드베이스에 변경이력을 남기기에도 유용

## 9.4 코드 리뷰 모범 사례

- 공손하고 전문가답게
- 작게 변경하기
- 변경 설명 잘쓰기
- 리뷰어는 최소한으로
- 가능한 한 자동화 하기

## 9.5 코드 리뷰 유형

- 그린필드 리뷰와 새로운 기능 개발
- 동작 변경, 개선, 최적화
- 버그 수정과 롤백
- 리팩터링과 대규모 변경

## 핵심 정리

- 코드베이스 전반의 정확성, 이해 용이성, 일관성 보장 등 코드 리뷰가 주는 이점이 많습니다.
- 여러분의 가정에 대해 항시 다른 사람의 검토를 받도록 하고, 코드를 읽는 사람에게 최적화 합니다.
- 전문가다움을 지키면서 중요한 피드백을 받을 기회를 제공하세요.
- 코드 리뷰는 지식을 조직 전체에 공유하는 데도 중요한 역할을 합니다.
- 코드 리뷰 프로세스를 확장하려면 자동화가 아주 중요합니다.
- 코드 리뷰 자체가 변경 이력이 되어줍니다.

