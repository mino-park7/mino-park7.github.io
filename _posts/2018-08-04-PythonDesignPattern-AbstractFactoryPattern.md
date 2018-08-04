---
layout: post
title: 파이썬 디자인 패턴 - 생성 디자인 패턴 - 추상 팩터리 패턴
category: python Study
tag: python, designpattern
---


# 1장 파이썬 생성 디자인 패턴

* 생성 디자인 패턴은 객체를 생성하는 방법에 대한 패턴
  * 보통은 객체를 생성할 때 생성자를 활용
  * 생성 디자인 패턴을 활용하면 객체 생성을 유연하게 할 수 있다


***

## 1. 추상 팩터리 패턴

* 여러 객체로 구성된 복합 객체를 만들어야 하는데, 포함된 객체가 모두 틀별한 한 **계통** 일 때 사용 가능.
* 예를 들어, Mac, Xfce, Window에서 위젯을 만들어주는 3 개의 concrete subclass factory 가 있는 Abstract widget factory가 있다고하자.
  * 세 구상 팩터리 모두 동일한 객체를 생성하는 메서드를 제공하지만, 각 결과물은 플랫폼에 따라 다른 스타일을 띠어야 한다.
  * 이러한 메서드를 활용하면 팩터리 인스턴스를 인자로 받아 해당 인자에 따라 다른 대화상자를 만드는 제네릭함수를 만들 수 있다.

### 1.1 고전적인 추상 팩터리


***

![동일부분](/{{site_url}}/_posts/assets/markdown-img-paste-20180804150908111.png)
* diagram1.py 와 diagram2.py 의 동일 부분을 보자. 공통적으로 txtDiagram과 svgDiagram을 생성하여 save한다.

***

![create-diagram at diagram1.py](/{{site_url}}/_posts/assets/markdown-img-paste-20180804151338458.png)

* 위 함수는 diagram factory 만을 받아 이를 활용해 요청받은 다이어그램을 생
* 다이어그램 팩터리 인터페이스(make_diagram, make_rectangle, make_text...의 def를 지원)를 지원하는 한 어떤 종류의 팩터리를 인자로 받은 관계 없음

***

![DiagramFactory](/{{site_url}}/_posts/assets/markdown-img-paste-20180804152116807.png)

* 패턴 이름에 **추상**이란 단어가 포함돼 있음에도 한 클래스가 인터페이스를 지원하는 기반 클래스이자 그 자체로 구상 클래스인 경우는 흔히 접할 수 있다. (DiagramFactory 클래스가 기반 클래스이지만, 내부에 모든 def가 구현되어 있음) ([추상클래스? 구상클래스? 정리본](http://e2xist.tistory.com/581) )

***

![SvgDiagramFactory](/{{site_url}}/_posts/assets/markdown-img-paste-20180804153605407.png)
* DiagramFactory의 make_diagram() 과의 유일한 차이점은 SvgDiagram 객체를 반환한다는 것 뿐이다. 나머지 메서에도 동일하다

***

* 각 클래스가 동일한 인터페이스를 제공하긴 하지만 (Diagram과 SvgDiagram 모두 동일한 **메서드**를 가지고 있음) 일반 텍스트 버전의 Diagram, Rectangle, Text 클래스 구현은 Svg버전의 그것과는 근본적으로 다르다.
* 이는 서로 다른 계열의 클래스 (즉, Rectangle과 SvgText)를 혼합할 수 없음을 의미, 이것이 팩터리 클래스에 자동적으로 적용되는 제약사항
* 일반 텍스트 다이어그램 객체는 내부데이터를 `공백, +, -, |` 로 구성된 한 글자짜리 문자열 리스트로 가지고 있다. 그것들을 지정된 위치에 맞춰 생성한다.

***

### 1.2 더 파이썬다운 추상 팩터리

***

* 위의 diagram1.py의 구현에는 몇가지 미흡한 점이 있다.


  1. 어느 팩터리는 상태 정보를 가지고 있을 필요가 없으므로, 실제 팩터리 인스턴스를 생성할 필요가 없다.
  2. SvgDiagramFactory 코드는 DiagramFactory 코드와 거의 동일하고, 유일한 차이는 **Text** 대신 **SvgText** 인스턴스를 돌려준다는 것 뿐이라 *불필요한* **중복**이 존재한다.
  3. 최상위 네임스페이스에 모든 클래스가 모여있다, 하지만 우리는 단지 두개의 팩터리에만 접근하면 된다. 게다가 이름의 충돌을 피하기위해 Svg클래스 이름에 접두어를 붙여야만 했는데 지저분하다.

***

* diagram2.py에서 위의 미흡한 부분을 모두 해결할 예정

  1. Diagram, Rectangle, Text 클래스를 DiagramFactory 클래스의 내부 클래스로 만든다. 이렇게 하면 클래스에 DiagramFactory.Diagram 과 같은 식으로 접근 가능하므로, SvgDiagramFactory내에 Diagram 과 같이 이름을 동일하게 만들 수 있다.(클래스 이름 충돌 x)
  2. 최상위 네임스페이스에는 `main(), create_diagram(), DiagramFactory(), SvgDiagramFactory()` 만이 남는다.

***

![DiagramFactory classmetho](/{{site_url}}/_posts/assets/markdown-img-paste-20180804161349402.png)
* `make_...()` 메서드는 이제 모두 *클래스메서드*이다.
  - 이는 이 메서드를 호출할 때 (일반 메서드에 self가 전달되는 것과 달리) 첫 번째 인자로 **클래스 정보**를 전달한다는 것을 의미
  - 따라서 `DiagramFactory.make_text()`를 호출하면 DiagramFactory가 Class 값으로 전달되 `DiagramFactory.Text` 객체가 생성되어 반환된다.
* 이러한 변화 때문에 `DiagramFactory`를 상속한 하위 클래스인 `SvgDiagramFactory`에는 더이상 `make_...()` 메서드가 필요하지 않다.
  - 예를들어 `SvgDiagramFactory.make_rectangle()` 호출 -> `SvgDiagramFactory`에 해당 메서드가 없음 -> 상위(기반) 클래스의 `DiagramFactory.make_rectangle` 메서드를 대신 호출 -> 이때 Class 값에는 `SvgDiagramFactory` 가 전달 -> `SvgDiagramFactory.Rectangle` 객체가 생성되어 반환

***

* 나머지 코드는 이전과 거의 동일하다. 핵심적인 변경사항은 이제 상수값과 팩터리가 아닌 클래스들이 모두 팩터리 안에 내장됐으므로 앞으로 이것들을 사용하려면 팩터리 이름을 붙여야 한다는 것이다.


![SvgText와 SvgDiagramFactory.Text](/{{site_url}}/_posts/assets/markdown-img-paste-20180804162902512.png)

* 위와 같이 변경되었다.
