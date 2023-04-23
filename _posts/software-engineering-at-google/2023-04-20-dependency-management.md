---
layout: "post"
title: "21. 의존성 관리"
date: "2023-04-20 00:22"
category: Software Engineering
tag: [Software Engineering]
---

# 21. 의존성 관리

- 의존성 관리란?
  - 우리가 통제하지 못하는 라이브러리, 패키지, 그 외 의존성들의 네트워크를 관리하는 일
  - SWE에서 제일 어렵다
- 의존성 관리에서 답하려는 질문은
  - 외부 의존성의 버전을 업데이트하는 방법은 무엇일까요?
  - 버전을 기술하는 방법은 무엇일까요?
  - 의존성에서 어떤 유형의 변경이 허용되거나 예상되나요?
  - 다른 조직에서 만든 코드에 의존하는 게 직접 만드는 것보다 낫다고 판단하는 기준은 무엇일까요?
- 소스코드와 비슷하지만 훨씬 어렵다
- 의존성 관리는 시간과 확장 양 축 모두에서 복잡도를 키운담
- 업스트림 의존성들은 내가 작성한 코드와 손발을 맞출 수 없으므로 내 빌드나 테스트를 실패하게 만들 가능성이 크다
- 다른 조건이 모두 같다면 의존성 관리 문제보다는 소스 관리 문제를 선택하라 -> 직접 구현해라?
- OSS가 커지면서 의존성 그래프도 점점 커짐
- 더 이상 의존성 관리를 회피할 수는 없음

## 21.1 의존성 관리가 어려운 이유

- 정의 부터가 쉽지않다
- 수많은 의존성들로 구성된 네트워크와 그 네트워크에 미래에 일어날 변화까지 고려해 관리하는 방법을 강구해야 함
- 시간이 흐르면서 의존성 네트워크의 여러 노드가 새 버전을 출시하고 그중 몇개는 중대한 업데이트일 것
- 전이 의존성들에서 이루어지는 이러한 업그레이드 도미노를 어떻게 관리해야 할까요...?

### 21.1.1 요구사항 충돌과 다이아몬드 의존성

- 의존성 네트워크상의 두 의존성 사이에서 요구사항이 충돌할 때 어떻게 해결하는가?
  - 다양한 이유로 문제가 발생함
  - 운영체제, 언어버전, 컴파일러
  - 업그레이한 라이브러리가 기존 버전과 호환되지 않아서
- 대표 문제는 **다이아몬드 의존성 문제 (diamond dependency problem)**

[그림 21-1]

- 위 그림에서 libbase는 liba과 libb 모두에서 쓰이며, liba와 libb는 상위 노드인 libuser가 이용함
- 이 때 libbase가 하위호환 되지 않는 업데이트를 했는데, liba만 업데이트를 따라 갔다면?
- 언어에 따라 문제가 생길수도, 안생길 수도있음
  - 자바, js등은 그냥 여러 버전을 설치해서 해결해버림 -> 구글에서 극혐하는 해결법
  - C++은 **'하나의 정의 규칙 One Definition Rule (ODR)'**을 거슬렀기 때문에 `undefined symbol` 에러가 발생함
  - Function 정도는 shading할 수 있지만 타입은 불가능
- 위 문제를 쉽게 해결하는 유일한 방법은 모두와 호환되는 더 상위 혹은 하위 버전의 라이브러리를 찾는 것 뿐
- 이게 불가능하다면 문제가 되는 의존성을 로컬에서 따로 패치해야 함 -> 매우매우 어려운 일

## 21.2 의존성 임포트하기

- 프로그래밍 측면에서 보면, 내가 직접 구현하는 것 보다 만들어진거 쓰는게 낫다

### 21.2.1 호환성 약속

- 시간을 고려하기 시작하면 몇가지 복잡한 트레이트 오프가 생겨남
  - "개발" 비용을 줄이는 반면에 지속적인 유지보수 비용을 감안해야 함
  - 업그레이드 할 계획이 없던 의존성이라 해도 보안 취약점이 발견될 수 있음
- 따라서 다음을 고려해야 함
  - 호환성이 얼마나 잘 지켜지나요?
  - 진화가 얼마나 빠르게(크게) 이루어지나요?
  - 변경 처리 방법은 무엇일까요?
  - 각 버전의 지원 기간은 어떻게 되나요?

#### C++

- C++ 표준 라이브러리는 거의 무한대의 하위 호환성을 제공 -> ABI ㅎ호환성

#### Go

- Go 언어는 대부분의 릴리스에서 소스코드가 호환되지만, 바이너리 호환은 되지 않음

#### Abseil

- 완전한 하위 호환성을 보장하지 않음
- 대신 자동 리팩터링 도구를 함께 제공 함

#### Boost

- 하위 호환성을 보장해주지 않음 (실험적인 프로젝트이기 때문)

### 21.2.2 임포트 시 고려사항

- 구글에서는 의존성을 임포트하려는 엔지니어들에게 몇 가지 질문들을 던져보라고 권한다
  - 여러분이 실행해볼 수 있는 테스트가 딸려 있는 프로젝트인가요?
  - 테스트는 모두 통과하나요?
  - 의존성 제공자는 누구인가요? 똑같이 호환성을 보장하지 않는 오픈 소스 프로젝트더라도 경험과 기술력 차이는 어마어마할 수 있음
    - C++ 표준 라이브러리, 자바의 Guava 라이브러리 vs 깃허브나 npm의 임의의 프로젝트
  - 지향하는 호환성 정책은?
  - 앞으로 어떤 분야나 용도를 지원해나갈 것인지 자세히 설명하고 있는지?
  - 얼마나 인기 있는 프로젝트인지?
  - 언제까지 이용할 것인지?
  - 파괴적인 변경이 얼마나 빈번하게 행해지고 있는지?
- 조직 관점에서의 질문
  - 구글이 같은 기능을 새로 구현하려면 얼마나 복잡한지?
  - 그 의존성을 최신 상태로 유지하면 어떤 이점이 있는지?
  - 업그레이드는 누가 할 건지?
  - 업그레이드하는 난이도는 어느 정도일 거라 예상하는지?
- 참고자료 : [Russ Cox : Our Sofware Dependency Problem](https://research.swtch.com/deps)

### 21.2.3 의존성 임포트하기@구글

- 구글은 거의 다 직접 개발해버려서 의존성 관리가 아닌 소스코드 관리를 해버림
- 그럼에도 외부 프로젝트를 쓰는 경우, 'third_party'라는 디렉터리에 추가한다
  - 그런데 여기에도 하이럼의 법칙이 적용되서 망할 수 있음

### 21.3 (이론상의) 의존성 관리

- 변경 불가
- 유의적 버전
- 하나로 묶어 배포하기
- 헤드에서 지내기

#### 21.3.1 변경 불가(정적 의존성 모델)

- 애초부터 변경 자체를 허용하지 안흠
- API 병경, 기능 (동작) 변경, 아무것도 안됨
- 사용자 코드 동작에 영향을 주지 않는 선에서의 버그 수정만 유일하게 허용
- 변경 불가는 사실 대부분의 신생 조직에게는 적합한 모델 일 수 있다
- 하지만 프로젝트가 오래될 수록 의존성을 업그레이드해야함 하는 보안 버그나 기타 사항이 생길 수 있음
- 이런 사항이 생기면 대처하기가 어렵다

#### 21.3.2 유의적 버전(SemVer)

- SemVer는 오늘날 의존성 네트워크를 관리하는 가장 대표적인 방법 (ex: 4.28.1)
  - 메이저 : API가 변경되어 의존성을 이용하던 기존 코드를 깨뜨릴 수 있음
  - 마이너 : 순수하게 기능 추가만 있음 (기존 코드를 깨뜨리지 않음)
  - 패치 : API에 여향을 주지 않는 내부 구현 개선과 버그 수정
- 버전 표기에 이렇게 의미를 담으면 API가 호환되면서 더 최신 버전 같은 표현이 가능
- 메이저 버전이 달라지면 일반적으로 호환성이 크게 떨어짐
- 이렇게 요구사항 표현법을 정형화하면 의존성 네트워크를 소프트웨어 컴포넌트 (노드)와 그 사이의 요구사항(에지)의 집합으로 표현 가능
- 충족 가능성 솔버 (SAT solver) 사용하여 요구사항을 모두 충족하는 버전의 집할을 찾아볼 수 있음
- 하지만 이것이 모든것을 해결해주진 않음, SAT solver로도 답을 못찾을 수 있음 -> 의존성 지옥 dependency hell

#### 21.3.3 하나로 묶어 배포하기

- 애플리케이션 구동에 필요한 의존성들을 모두 찾아서 애플리케이션과 함께 배포한다 -> 리눅스 배포판
- 전문 배포자가 필요함

#### 21.3.4 헤드에서 지내기

- 구글 C++ 커뮤니티 일원들이 일고있는 모델
- 트런크 기반 개발을 의존성 관리 영역까지 확장한 것
- SemVer를 버리고 의존성 제공자가 변경사항을 커밋하기 전에 생태계 전체를 대상으로 검증하게 함
- 모든 컴포넌트가 항상 최신 버전에 의존
- 유닛 테스트 및 CI가 필수적으로 필요함
- API 제공자 다운 스트림 의존성들이 깨지는 지 확인 가능 해야함
- API 소비자가 테스트들이 계속 통과되고 지원 가능한 방식으로 의존성을 이용 중

### 21.4 유의적 버전의 한계

- 사실 사람들이 SemVer의 메이저, 마이너, 패치를 잘 지키는가? 안지킬 수도 있다
- SAT solver의 믿음도 떨어짐

#### 21.4.1 지나치게 구속할 수 있다.

- SemVer의 major 버전이 바뀌었다고 무조건 못쓰게 되느냐? 아님!
- 내가 import한 library의 전부를 쓰는 것이 아니기 때문에, 내가 사용하는 부분과 필요없는 부분만 변경되었는데 major 버전이 바뀌었을 수 있다.
- 이로 인해서 의존성 지옥에 빠질 수 있음

#### 21.4.2 확실하지 않을 약속일 수 있다.

- SemVer는 강제적 혹은 완벽하게 API를 제어하는 것은 아니기 때문에, 완벽히 믿을 수 없음
- patch 버전이 바뀌었다는 것은 내부 사용 API를 바꾸었다는 뜻인데, 하이럼의 법칙에 따라 외부 사용자가 내부 API를 사용해서 patch버전만 바뀌었는데 소스가 깨질 수 있음...
- SAT solver는 괜찮다고 하는데 소스가 깨질 수 있다

#### 21.4.3 버전업 동기

- 메이저 버전업을 위축시키는 동기가 될 수 있음
- Go나 Closure 메이저 버전업을 완전히 새로운 패키지로 취급함

#### 21.4.4 최소 버전 선택

- 기본적으로는 허용가능한 최신 버전을 선택하여 설치함
- 최소 버전은 명시되어있는 최소 버전을 선택하여 설치 -> 안정성 증가
- 그나마 개발자가 설치해서 동작했던 버전을 설치할 수 있다

#### 21.4.5 그래서 유의적 버전을 이용해도 괜찮은가?

- 유의할 점
  - (SemVer 버전업 시 사람에 의한 오류가 나지 않도록) 의존성 제공자들이 버전을 정확하고 책임감 있게 관리합니다.
  - (지나친 구속과 지나친 약속을 피하기 위해) 의존성들이 충분히 세분화되어 있습니다.
  - (직간접적으로 사용하는 의존성에서 호환된다고 가정하고 진행한 변경임에도 여러분의 코드가 예기치 못한 방식으로 오작동하는 일을 피하기 위해) 모든 API를 제공자의 의도대로 사용합니다.

- 하지만 이같은 노력에도 규모가 커질수록 SemVer의 단점이 드러남

## 21.5 자원이 무한할 때의 의존성 관리

- 현재 업계가 SemVer에 의지하는 이유
  - 내부 정보만 있으면 됨 (API 제공자가 다운스트림 사용자가 어떻게 이용하는지까지 알 필요 없음)
  - 충분한 테스트를 수행할 컴퓨팅 자원, 테스트 결과를 모니터링해줄 CI 시스템이 존재하지 않아도 됨
  - 관행임

- 만약 더 많은 컴퓨팅 자원을 쓸 수 있고 다운스트림 의존성 정보를 더 쉽게 얻을 수 있다면 이를 활용할 수 있다.
  - 실제로 의존성 테스트를 돌려 볼 수 있다면? 통계적으로 안전하다 라고 이야기 할 수 있음
  - 의존성 업데이트를 CI 기반 모델 적용 가능
  - 이를 적용하려면? 갈길이 굉장히 멀다
- 위 모델을 위한 필요조건
  - 모든 의존성이 단위 테스트를 제공해야 함
  - OSS 생태계 대부분의 의존성 네트워크를 이해해야 함
  - CI를 수행하기 위한 컴퓨팅 자원이 충분해야 함
  - 의존성을 고정해 놓은 경우가 많음
  - CI 이력과 평판을 명시하는 것도 좋은 방법일 수 있음
- 결국 헤드에서 지내기 모델로 수렴함

### 21.5.1 의존성 내보내기

- 구글의 경우 gflags와 AppEngine의 사례를 통해 의존성을 밖으로 내보냈다가 큰 손해를 본 적이 있음
- 하지만 구글은 평판으로 인해 deprecated도 쉽게 시킬 수 없었음...

## 21.7 핵심 정리

- 의존성 관리보다는 되도록 버전관리가 되도록 합시다. 더많은 코드를 조직내로 가져와 투명성과 통제력을 높인다면 문제가 훨씬 단순해짐
- 소프트웨어 엔지니어링 프로젝트에서 의존성 추가는 공짜가 아님, 의존성 네트워크를 관리하는 것은 매우 비싸다
- 의존성 역시 하나의 계약임, 주고 받는 것이 있다. 지금 당장뿐 아니라 미래에 대한 약속도 명확하게 포함되어야 함
- SemVer는 변경이 얼마나 위험할지를 '사람'이 '추정'하는 간단하지만 정보가 일부 손실되는 표현법임. 하지만 SAT solver 방식의 패키지 관리자는 이 추정이 틀릴 리 없다고 동작함 -> 의존성 지옥 or 소스깨짐
- 이에 비해 테스트와 CI는 새로운 버전들이 잘 어울려 돌아가는지를 '실제로'보여줌
- SemVer 기반 패키지 관리에 최소 버전 선택 전략을 가미하면 충실성이 올라간다. API제공자와 사용자의 환경이 비슷할 확률이 올라감
- 단위 테스트, CI, 저렴한 컴퓨팅 자원이 의존성 관리를 이해하고 처리하는 방식에 변화를 가져올 수 있음
- 의존성을 제공하는 일 또한 매우 비싸다.

