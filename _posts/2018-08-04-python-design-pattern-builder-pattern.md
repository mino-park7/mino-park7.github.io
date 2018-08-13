---
layout: post
title: Python Design Pattern - Builder Pattern
date: '2018-08-04 18:54'
category: python Study
tag: 'python, designpattern'
---

## 2. 빌더 패턴

***

* 빌더 패턴은 다른 객체를 조합한 복합 객체 (compled object)를 생성하기 위해 고안됐다는 점에서 추상 팩터리 패턴과 비슷하다.
* 빌더 패턴은 복합 객체를 구축하는 메서드를 제공할뿐더러 전에 복합 객체 자체의 표현을 보유한다는 점에서 팩터리와 구별된다.
* 이 패턴은 **복합 객체**가 하나 또는 여러 개의 더 간단한 객체로부터 만들어진다는 점에서 추상 팩터리 패턴과 동일한 조합 방법을 제공한다.
* 그러나 이 패턴은 특히 복합 객체의 표현을 조합 알고리즘과 별도로 유지해야 할 때 적합하다.

***

가장먼저 최상위 코드
![highest code](https://mino-park7.github.io/assets/images/2018/08/highest-code.png)

* 각 폼을 생성한 후 파일에 저장
* 두 경우 모두 동일한 생성함수 `create_login_form()` 에 적절한 빌더 객체를 인자로 전달해 호출

***

![create_login_form](https://mino-park7.github.io/assets/images/2018/08/create-login-form.png)

* `create_login_form()`함수는 임의의 HTML이나 Tkinter 폼을 생성할 수 있다. 또는 알맞은 빌더만 있다면 어떤 종류의 폼이든 생성 가능하다.

***

HtmlFormBuilder와 TkFormBuilder 모두 추상 기반 클래스인 AbstractFormBuilder를 상속한다.

![ABSFormBuilder](https://mino-park7.github.io/assets/images/2018/08/absformbuilder.png)

* 이 클래스를 상속하는 클래스는 추상 메서드를 모두 구현해야 한다.
* 참고로 `AbstractFormBuilder`는 `abc.ABCMeta` 메타클래스를 받아 abs 모듈의 @abstractmethod 데코레이터를 사용한다
* 어떤 클래스에 `abc.ABCMeta`의 메타클래스를 전달하면 해달 클래스의 인스턴스를 만들 수 없기 때문에 추상 기반 클래스로만 사용해야 한다.
* 이는 C++ 나 자바에서 이식된 코드에 유의미하나, 실행 시 약간의 비용이 더 든다.
* 하지만 대다수의 파이썬 프로그래머는 문제가 될 수 있는 더 안일한 방법을 활용 -> 메타클래스를 활용하지 않고, 해당 클래스를 추상 기반 클래스로 사용해야 한다고 문서화 함

***

![HtmlFormBuilder](https://mino-park7.github.io/assets/images/2018/08/htmlformbuilder.png)

* `add_title()` 메서드는 추상 메서드 이므로 재구현(reimplement)해야 한다. 그러나 원래의 추상 메서드에도 구현 코드가 있으므로, 그 메서드를 호출해 처리한다 (`super.add_title`) (`escape` 는 html module에 있는 메서드)

***

![HtmlFormBuilder.form](https://mino-park7.github.io/assets/images/2018/08/htmlformbuilder-form.png)

* `HtmlFormBuilder.form()` 메서드는 HTML페이지를 생성한다.

***

![TkFormBuilder](https://mino-park7.github.io/assets/images/2018/08/tkformbuilder.png)

* 위 코드는 TkFormBuilder 클래스의 일부다.
*
