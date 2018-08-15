---
layout: "post"
title: "파이썬 디자인 패턴 - 구조디자인패턴 - 어댑터 패턴"
date: "2018-08-15 15:45"
ategory: python Study
tag: [python, designpattern, 구조디자인패턴]
---




# 2.구조 디자인 패턴
 * 구조 디자인 패턴은 주로 어떻게 기존 객체를 **조합**해 새로운 **더 큰 객체**를 만드는가에 대한 패턴
 * 세가지 주요 주제
   - 인터페이스 맞추기
   - 기능 추가
   - 객체 컬렉션 추가




***

## 1. 어댑터 패턴

* 어댑터 패턴은 **인터페이스**를 맞춰서 한 객체가 호환되지 않은 인터페이스를 가진 다른 객체를 활용 할 수 있게하되, 어떤 클래스도 변경하지 않는 기법
* 원래 의도한 상황이 아닌 곳에서 변경 불가능한 클래스를 활용하고 싶은 때 유용

***

제목, 본문, 그리기용 인스턴스를 가지고 페이지를 그려주는 간단한 Page 클래스가 있다고 해보자

![Page class](http://mino-park7.github.io/assets/images/2018/08/page-class.png)

* Page 클래스는 그리기용 클래스(`renderer`)가 페이지 그리기 인터페이스로 `header(str)`, `paragraph(str)`, `footer()`세 메서드를 제공하는 한, 실제 클래스가 무엇인지는 알지도 못하고, 신경 쓰지도 않는다.
* 이때 전달된 그리기 객체가 `Renderer`인스턴스임을 보장하고 싶다
  - 간단하지만 약간 문제가 있는 해결책 : `assert isinstance(renderer, Renderer)`
    - 문제점 1. 더 명확한 TypeError 대신 AssersionError가 발생
    - 문제점 2. 프로그램을 -O(optimize, 최적화) 옵션을 주고 실행하면 assert 구문이 무시되므로 `render()` 메서드에서 AttributeError오류가 발생
  - 위 코드의 `if not isinstance()` 구문은 올바른 TypeError를 발생 시키고, -O 옵션과도 상관없이 동작함


***

* 위 방법의 문제점 하나는 모든 그리기 객체를 `Renderer` 클래스의 하위 클래스로 만들어야만 할 것 같다는 점이다. C++ 이라면 분명히 그렇다.
* 그러나 파이썬의 abc(Abstract Base Class, 추상기반클래스) 모듈이 대안이 될 수 있다.
  - abc는 추상 기반 클래스가 제공하는 **인터페이스 점검** 기능에 duck typing의 유연함을 결합함
  - 이는 특정한 **인터페이스**를 만족한다는 것 (즉, 특정 API를 제공)이 보장되지만 특정 기반 클래스의 하위 클래스일 필요는 없는 객체를 생성할 수 있다는 의미다

***

![Renderer class](http://mino-park7.github.io/assets/images/2018/08/renderer-class.png)

* 여기서는 `Renderer` 클래스의 `__subclasshook__()` 특수 메서드를 재구현했다.
  - 이 메서드는 내장 함수인 `isinstance()` 가 활용하며, 첫 번째 인자로 전달된 객체가 두 번째 인자로 전달된 클래스(또는 클래스 튜플 가운데 어느 하나)의 상위 클래스인지 판단한다.
  - `collections.ChainMap()` 클래스를 활용 (파이썬 3 코드)
  - `__subclasshook__()` 특수 메서드는 호출 대상 클래스가 `Renderer` 인지 검사하면서 시작하고, 만약 그렇지 않다면 NotImplemented를 반환
    - 이는 `Renderer`의 하위 클래스가 __subclasshook__행위를 상속하지 않는다는 의미
    - 이렇게 하는 이유는 추상 기반 클래스를 상속하는 목적은 **행위를 상속하는데 있지 않고**, **조건을 덧붙여 인터페이스를 더 세분화** 하는데 있기 때문
    - 원한다면 하위 클래스에서 정의한 `__subclasshook__()` 안에서 `Renderer.__subclasshook__()`을 명시적으로 호출해서 `Renderer`의 행위를 상속 할 수 있다
  - 이 함수가 True나 False를 반환하면 추상 기반 클래스 쪽에서 코드 실행이 멈추고 bool 값을 반환한다.
  - 만약 NotImplemented를 반환하면 일반적인 상속 기능(하위 클래스, 명시적으로 등록된 클래스의 하위 클래스, 하위 클래스의 하위 클래스 등)이 동작한다.
  - if문의 조건이 만족되면 Subclass 가 상속한 모든 클래스(자기 자신도 포함)를 `__mro__()` 특수 메서드를 통해 순회해 각 클래스의 비공개 딕셔너리 (`__dict__`)에 접근한다.
    - `__mor__()`? : mro는 **Method Resolution Order**의 약자로 어떤 개게의 메서드를 호출하면 이 **순서에 따라 차례대로 메서드가 존재하는지를 검사**한다.
  - 이 for문을 시퀀스풀기 (* 연산자)를 활용해 즉시 풀어서 dictionary로 이뤄진 튜플(tuple)을 `collections.ChainMap()` 함수에 전달
    - 이 함수는 임의 개수의 맵Map(dict나 그와 비슷한)을 인자로 받아 모든 맵을 한친 것 가튼 단일 맵 뷰를 반환
    - 검사 대상 메서드의 이름으로 구성된 튜플을 생성
    - 모든 메서드가 attributes 맵에 있는지 검사
      - attributes 맵의 키는 Subclass 또는 모든 Superclass에 있는 모든 메서드나 애트리뷰트의 이름
    - 모든 메서드가 맵에 있다면 True를 반환
  - 참고로 이 함수에서는 하위 클래스(또는 기반 클래스 가운데 아무거나)에 필요한 메서드와 같은 이름의 애트리뷰트가 있는지만 검사
    - 그래서 메서드 대신 애트리뷰트가 매칭 될 수도 있다.
    - 메서드만 확실하게 매칭하고 싶다면 `method in attribues`에 `and callable(method)`를 추가해 검사 (이렇게까지 할 필욘없다)
* 인터페이스 검사를 제공하기 위해 `__subclasshook__()`를 포함한 클래스를 생성하는 것은 매우 유용
* 하지만 기반 클래스와 지원 메서드만 차이 나는데도 십여 줄의 코드를 작성해야 한다는 것은 피해야 할 코드 중복으로 볼 수 있다

***

![TextRenderer class](http://mino-park7.github.io/assets/images/2018/08/textrenderer-class.png)

* 위 코드는 페이지 그리기 인터페이스를 지원하는 간단한 클래스이다
* `header()` 메서드는 지정된 폭만큼 중앙에 제목을 출력하고, 다음 줄에는 제목 아래에 = 글자를 출력
* `paragraph()` 메서드는 파이썬 표준 라이브러리의 texwrap 모듈을 활용해 지정된 줄 폭에 맞춰 줄바꿈한 단락을 출력
* `footer()` 메서드는 아무것도 하지 않지만 인터페이스의 일부이므로 반드시 있어야한다.

***

![HtmlWriter class](http://mino-park7.github.io/assets/images/2018/08/htmlwriter-class.png)

* HtmlWriter 클래스에서는 간단한 HTML 페이지를 출력
* 이 클래스에 `header()`, `footer()`메서드가 있긴 하지만, 페이지 그리기 인터페이스에 약속된 행동과는 다르게 동작 (html방식으로)
* 따라서 TextRenderer와는 달리 Page 인스턴스에 페이지 출력을 위해 HtmlWriter를 직접 전달할 수 없다.

***

* 해결책 중 하나는 HtmlWriter를 상속한 클래스에 페이지 그리기 인터페이스 메서드를 추가하는 것
  - 하지만 이 방법은 다소 취약한 방식
  - 결과 클래스에 HtmlWriter의 메서드와 페이지 그리기 인터페이스의 메서드가 섞여있기 때문
* 더 나은 해결책은 **어댑터**를 만드는 것
  - 어댑터는 우리가 사용해야 하는 클래스를 내부에 가지고 있는 클래스
  - 필요한 인터페이스를 제공하고 내부의 클래스와 외부 인터페이스 사이를 중계하는 데 필요한 작업을 처리

![page renderer adapter](http://mino-park7.github.io/assets/images/2018/08/page-renderer-adapter.png)

***

![htmlrenderer class](http://mino-park7.github.io/assets/images/2018/08/htmlrenderer-class.png)

* 어댑터 클래스의 코드다
* 이 클래스의 객체를 생성할 때 HtmlWriter 타입의 htmlWriter를 얻고, 이에 대한 페이지 그리기 인터페이스 메서드를 제공
* 실제 작업은 합성된 HtmlWriter 객체에서 위임하므로, HtmlRenderer 클래스는 기존 HtmlWriter 클래스에 새로운 인터페이스를 제공하는 wrapper에 불과하다

***

![main method](http://mino-park7.github.io/assets/images/2018/08/main-method.png)

* 실제 Page 클래스 인스턴스를 각 그리기 객체를 가지고 생성하는 코드
* TextRenderer에는 줄당 22자의 폭을 주었다.
* HtmlRenderer가 사용하는 HtmlWriter에는 결과를 출력할 인자로 열린 파일 객체 전달
