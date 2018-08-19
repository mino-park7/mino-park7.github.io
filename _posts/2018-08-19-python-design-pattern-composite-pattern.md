---
layout: "post"
title: "파이썬 디자인 패턴 - 구조디자인패턴 - 컴포지트 패턴"
date: "2018-08-19 17:02"
category: python Study
tag: [python, designpattern, 구조디자인패턴]
---

## 3. 컴포지트 패턴

* 컴포지트(composite) 패턴은 계층구조에서 어떤 객체가 다른 객체를 포함하는지(계층구조의 일부로) 여부와 상관없이 구조 내 모든 객체의 동일한 처리 방법을 제공하기 위해 고안된 패턴
* 이러한 객체를 **조합체(composite object)** 라고 부르기도 한다.
* 고전적인 접근법에서는 **조합체** 내의 각 객체 및 객체 컬렉션이 모두 동일한 기반 클래스를 가진다.
* 조합 및 비조합 객체는 모두 같은 **핵심 메서드** 를 공유한다.
* 하지만 조합 객체는 자식 객체를 추가, 제거, 순회하기 위한 메서드를 추가로 포함

---

- 이 패턴은 Inkscape 같은 그래픽 프로그램에서 객체의 그룹을 설정하거나 해제하는 기능을 지원하는 데 흔히 사용된다.
  - 이러한 경우 이 패턴은 매우 유용하다. 왜냐하면 그룹을 설정하거나 해제하기 위해 사용자가 구성 요소를 선택할 때 선택 요소 중 일부는 단일 항목 (ex: 사각형)이지만 나머지는 조합된 것 (ex: 여러 다른 도형을 조합한 얼굴 이미지)일 수도 있기 때문이다.

---

- 실례를 살펴보기 위해 비조합 항목과 일부 조합 항목을 생성하고 이 모두를 출력하는 main() 함수를 살펴보자
- 다음은 stationary1.py에 있는 코드이며 실행결과는 아래와 같다

![stationary main](http://mino-park7.github.io/assets/images/2018/08/stationary-main.png)

![result](http://mino-park7.github.io/assets/images/2018/08/result.png)

- 각 `SimpleItem`에는 이름과 가격이 있다.
- 각 `CompositeItem`은 이름과 임의 개수의 `SimpleItem`이나 `CompositeItem`을 포함하고 있다.
- 따라서 조합 항목은 무제한으로 중첩될 수 있다.
- 조합 항목의 가격은 포함된 항목 가격의 총합이다.

- 이 예제에서 연필 세트는 연필, 자, 지우개로 구성된다.
  - 포장된 연필 세트는
    - 먼저 포장 상자를 만들고
    - 그 안에 연필 세트를 넣고
    - 추가로 연필하나를 넣는다.
- 여기서는 컴포지트 패턴을 구현하는 두 가지 방법을 살펴보겠다.
  - 하나는 전통적인 접근법
  - 다른 하나는 조합 및 비조합 객체를 모두 표현하는 단일 클래스를 활용하는 방법


---


### 3.1 전통적인 조합/비조합 계층구조

* 전통적인 접근법은 모든 종류의 항목에 조합 여부에 상관없이 공통의 추상 기반 클래스를 제공
* 조합체에 대해서는 추가적인 추상 기반 클래스를 제공하는 것
* 클래스 구조는 아래와 같다

![class structure](http://mino-park7.github.io/assets/images/2018/08/class-structure.png)

---

- 기반 클래스인 `AbstractItem` 부터 살펴보자

![AbstractItem](http://mino-park7.github.io/assets/images/2018/08/abstractitem.png)

- 이 클래스에서 파생될 하위 클래스가 자신이 복합 객체인지 여부를 알려주게 하고싶다.
- 또한 모든 하위 클래스가 순회 가능하게 만들 것이다.
  - 이때 기본 동작은 빈 시퀀스에 대한 반복자를 돌려주는 것이다.

-  `AbstractItem` 클래스에는 추상 메서드나 프로퍼티가 적어도 하나는 포함돼 있기 때문에 `AbstractItem` 클래스의 객체는 생성할 수 없다

---

![SimpleItem class](http://mino-park7.github.io/assets/images/2018/08/simpleitem-class.png)

- 위 코드는 비조합 항목에 대한 `SimpleItem` 클래스다.
  - `name`과 `price` 애트리뷰트가 있다.
- `SimpleItem`은 `AbstractItem`을 상속하므로 모든 추상 애트리뷰트 메서드를 재구현해야 한다.
  - `composite` 애트리뷰트만 구현하면 된다.
  - `AbstractItem`의 `__iter__()` 메서드는 추상메서드가 아니므로 여기서 재구현 할 필요가 없고, 기반 클래스의 메서드를 그대로 가져와 빈 시퀀스에 대한 이터레이터를 반환하면 된다.
- `SimpleItem`은 비조합 객체지만 `SimpleItem`이나 `CompositeItem` 모두 같은 방식으로 다룰 수 있으므로 (적어도 순회할 때는) 이러한 방법이 적절하다.
  - ex: 이러한 객체를 섞어서 `itertools.chain()` 함수에 전달할 수 있다.
- `print()` 메서드는 조합 및 비조합 객체 항목을 출력한다.
  - 포함된 항목을 계층구조 내에서의 단계에 따라 들여쓰기 한다.
---

![AbstractCompositeItem class](http://mino-park7.github.io/assets/images/2018/08/abstractcompositeitem-class.png)

* 이 클래스는 `CompositeItem`의 기반 클래스 역할을 하며, 조합 객체에 대한 추가, 제거 ,순회 기능을 제공
* `AbstractCompositeItem`의 인스턴스를 직접 만들 수는 없다.
  - 추상 애트리뷰트인 `composite()`를 상속했지만 구현하지는 않았기 때문
* `add()` 메서드는 하나 이상의 항목(`SimpleItem`과 `CompositeItem` 모두 가능)을 받아 이것들을 조합객체의 자식 목록에 추가한다.
  - first 인자를 생략하고 `*items`만 사용할 수는 없다.
  - 이렇게 만들경우 빈 리스트를 추가할 수 있게 되어, 사용자 코드에 논리오류가 남을 여기가 있기 때문
  - 순환 참조를 막기위한, 즉 조합 항목에 자기 자신을 추가하는 등의 문제를 막기 위한 조치는 여기서 취하지 않았다.
* `remove()` 메서드는 항목을 제거할 때 한 번에 하나만 제거하는 간단한 접근법을 취함
  - 물론 제거한 항목이 조합 항목이라면, 해당 항목을 제거하는 것은 그 자식 또는 자손 항목을 모두 제거하는 셈이 된다.
* `__iter__()` 특수 메서드를 재구현 하면 조합 객체의 자식 항목을 for루프, (리스트) 내장 구문, 제너레이터 등에서 순회 할 수 있다.
  - 대부분의 경우 메서드 본문을 `for item in self.children: yield item` 이라고 쓸 수도 있지만 `self.children`은 시퀀스(리스트)이기 때문에 그러한 경우에 대한 내장함수 `iter()`를 활용할 수 있다.

---

![CompositeItem clas](http://mino-park7.github.io/assets/images/2018/08/compositeitem-clas.png)

- 이 클래스는 실제 조합 항목을 나타낸 것이다.
  - 자신의 `name` 애트리뷰트를 가지고 있지만 조합(자식 항목의 추가, 제거, 순회등)과 관련된 모든 처리 코드는 기반 클래스에 있다.
  - 기반 클래스의 추상 애트리뷰트인 `composite()`를 구현했기 때문에 `CompositeItem`의 인스턴스를 생성할 수 있다.
- `price()` 는 읽기전용 애트리뷰트이다.
  - 조합 항목의 가격을 모든 자식 항목 가격의 합계로 누적해서 계산하기 위해 내장함수 `sum()`의 인자로 제너레이터 표현식을 전달
  - 이때 자식 항목에 조합 항목이 있다면, 마찬가지로 그 아래 자식 가격의 합계를 **재귀적**으로 누적
  - `for item in self` 표현식은 실제로는 `iter(self)`를 호출해서 `self`에 대한 이터레이터를 얻는다.
    - 그 결과로 `__iter__()` 특수 메서드를 호출하며, 이 메서드는 `self.children`에 대한 이터레이터를 반환
- `print()` 메서드의 첫 부분은 `SimpleItem.print()` 메서드와 같다
  - `child`에 대한 처리를 추가함

---

![stationary1.py structure](http://mino-park7.github.io/assets/images/2018/08/stationary1-py-structure.png)

- 위 예제의 `SimpleItem`과 `CompositeItem`은 모두 대부분의 사례에서 그대로 활용할 수 있다.
  - 하지만 더 계층구조를 다듬을 필요가 있다면, 이 클래스나 해당 추상 기반 클래스를 상속한 클래스를 활용할 수도 있다.
- 여기서 `AbstractItem`, `SimpleIterm`, `AbstractCompositeItem`, `CompositeItem` 클래스는 모두 완벽하게 잘 동작
  - 그러나 **필요이상으로 고드가 길고**, **비조합 클래스에는 없는 `add()`나 `remove()` 같은 메서드** 가 조합클래스에 있기 때문에, 단일 인터페이스를 사용하는 것도 아니다.
- 이를 해결하기 위한 방법을 다음 절에서 소개

---
---

### 3.2 (비)조합 객체를 위한 단일 클래스

* 앞에 있는 네 개의 클래스 (추상 클래스 둘, 구상 클래스 둘)에는 해줘야 할게 많음
* 그리고 조합 클래스에서만 `add()`와 `remove()` 메서드를 제공하기 때문에 완변한 공통 인터페이스도 제공하지 않는다.
* 약간의 추가 비용(비조합 항목마다 빈 리스트 애트리뷰트 하나(children list), 조합 항목마다 float값 하나(price attribute))을 감수 할 수 있다면 조합 및 비조합 항목을 표현하는 단일 클래스를 활용할 수 있다.
  - 이렇게 하면 인터페이스가 완벽히 동일해진다는 이점을 얻을 수 있는데, 조합 항목이 아닌 어떤 항목이든 `add()`와 `remove()` 메서드를 호출해 예상한 결과를 얻을 수 있기 때문

---

- 이번 절에서는 다른 클래스를 필요로 하지 않는, 조합 및 비조합 항목 모두를 위한 새로운 Item 클래스를 만든다
- 인용할 코드는 `stationery2.py` 이다

---

![Item __init__](http://mino-park7.github.io/assets/images/2018/08/item-init.png)

- `__init__()` 메서드의 인자가 다소 복잡해 보인다.
  - 항목을 생성하는데 `Item()`을 직접 호출할 것이 아니기 때문에 괜찮다
- 각 항목에는 name이 필요하다. 각 항목은 price가 붙어 있으며, 기본값(0.00)을 갖는다
  - 또한 모든 항목은 0개 이상의 자식 항목(`*items`)를 가질 수 있으며, 이 값은 `self.children`에 저장된다.
    - 이 때 비조합 항목은 빈 리스트를 갖는다.

---

![create compose](http://mino-park7.github.io/assets/images/2018/08/create-compose.png)

- 클래스 객체를 호출해서 항목을 생성하는 대신 더 깔끔하게 인자를 취해 Item을 반환하는 두 종류의 팩터리 클래스 메서드를 제공
  - ~~`SimpleItem("Ruler", 1.60)`~~ or ~~`CompositeItem("Pencil Set", pencil, ruler, eraser)`~~
  - `Item.create("Ruler", 1.60)` 이나 `Item.compose("Pencil Set", pencil, ruler, eraser)`로 사용
- `Item()`을 직접 사용 가능 (`__init__()`)

---

![make methods](http://mino-park7.github.io/assets/images/2018/08/make-methods.png)

- 클래스 메서드와 같은 역할을 하는 두 팩터리 함수를 제공
- 이런 함수는 모듈을 활용할 때 편리
  - ex: `Item`클래스가 `Item.py` 모듈에 있다면
    - ~~`Item.Item.create("Ruler", 1.60)`~~
    - `Item.make_item("Ruler", 1.60)`

---

![composite](http://mino-park7.github.io/assets/images/2018/08/composite.png)

- 이 프로퍼티는 앞의 것과는 다른데, 어떤 항목이 조합 항목이거나 조합 항목이 아닐 수도 있기 때문
  - `Item` 클래스에서 조합 항목은 `self.children` 리스트가 비어 있지 않은 것이다.

![add method](http://mino-park7.github.io/assets/images/2018/08/add-method.png)

- `add()` 메서드를 이전과는 약간 다르게 더 효율적인 방식으로 작성
  - `itertools.chain()` : 임의 개수의 iterable 을 받아, 함수에 전달된 모든 iterable을 연결한 것과 같은 하나의 iterable을 반환
- 이 메서드는 조합 여부에 관계없이 어떤 항목에서도 호출 가능
  - 비조합 항목에서 이 함수를 호출하면 해당 항목이 조합 항목이 된다.
- 비조합 항목을 조합 항목으로 변경할 때의 미묘한 부작용이 존재
  - 항목 자기 자신의 가격이 감춰진다는 점
  - 조합 항목의 가격은 자식 항목 가격의 합계이기 때문
  - 물론 설계 관점에 따라 다른 결정(자신의 가격값을 유지하는 것과 같은)도 가능

![remove method](http://mino-park7.github.io/assets/images/2018/08/remove-method.png)

- 조합 항목의 마지막 자식 항목을 삭제하면 해당 항목은 이제 비조합 항목이 된다.
- 그런데 조금 미묘한 부분은 이제 이 항목의 가격은 자식 항목 가격(이제는 존재하지 않는) 의 합계가 아닌 비공개 `self.__price` 애트리뷰트의 값이 된다는 점
  - 이에 대응하기 위해 `__init__()` 메서드에서 모든 항목에 가격 기본값을 설정

![__iter__ method](http://mino-park7.github.io/assets/images/2018/08/iter-method.png)

- 이 메서드는 조합 항목이라면 자식에 대한 이터레이터를, 비조합 항목이라면 빈 시퀀스를 반환

![property](http://mino-park7.github.io/assets/images/2018/08/property.png)

- `price` 프라퍼티는 조합 항목이든(자식 항목 가격의 합계), 비조합 항목이든(항목가격) 모두 잘 동작해야 한다.

![print method](http://mino-park7.github.io/assets/images/2018/08/print-method.png)

- 마찬가지로, `print()` 메서드도 조합이나 비조합 항목에 대해 모두 잘 동작 해야 한다.
- 앞 절의 `CompositeItem.print()` 메서드와 동일
- 비조합 항목을 방문할 때 빈 시퀀스에 대한 이터레이터를 반환하기 때문에 항목의 자식을 방문할 때 무한 재귀 호출이 일어나지 않는다.
