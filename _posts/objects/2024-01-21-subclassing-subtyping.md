---
layout: "post"
title: "13. 서브클래싱과 서브타이핑"
date: "2024-01-21 01:10+0900"
category: Objects
tag: [Objects]
---

# 13. 서브클래싱과 서브타이핑

- 상속의 두가지 용도
  - **타입 계층 구현** : 부모 클래스는 일반적인 개념 구현 (**일반화 generalization**), 자식 클래스는 특수한 개념 구현(**특수화 specialization**)
  - **코드 재사용** : 간단한 선언만으로 부모 클래스의 코드 재사용 가능, 하지만 부모 클래스와 **자식 클래스가 강하게 결합**되기 때문에 **변경하기 어려운 코드**를 얻게 될 확률이 높다.
- 상속을 사용하는 일차적인 목표는 **코드 재사용이 아니라 타입 계층을 구현하는 것**이어야 함
- 결론부터 말하자면 동일한 메시지에 대해 서로 다르게 활동할 수 있는 다형적인 객체를 구현하기 위해서는 **객체의 행동을 기반으로 타입 계층을 구성**해야 함
- 타입 사이의 관계를 고려하지 않은 채 단순히 코드를 재사용하기 위해 상속을 사용해서는 안된다.

## 13.1 타입

### 개념 관점의 타입

- 우리가 인지하는 세상의 사물의 종류
  - 자바, 루비, 자바스크립트, C : 프로그래밍 언어 -> 프로그래밍 언어라는 **타입**으로 분류
  - 어떤 대상이 타입으로 분류될 때, 그 대상을 타입의 **인스턴스(instance)**라고 부름
  - 일반적으로 타입의 인스턴스를 **객체**라고 부름
- Martin98, Larman04
  - **심볼(symbol)** : 타입에 이름을 붙인 것
  - **내연(intention)** : 타입의 정의로서 타입에 속하는 객체들이 가지는 **공통적인 속성이나 행동**
  - **외연(extension)** : 타입에 속하는 객체들의 집합

### 프로그래밍 언어 관점의 타입

- 연속적인 비트에 의미와 제약을 부여하기 위해 사용

#### 타입에 수행될 수 있는 유효한 오퍼레이션의 집합을 정의

- 자바에서 `+` 연산자는 원시형 숫자 타입이나 문자열 타입의 객체에는 사용할 수 있지만, 다른 클래스의 인스턴스에 대해서는 사용할 수 없음
- 하지만 C++와 C#에서는 **연산자 오버로딩**을 통해 `+`연산자를 사용하는 것이 가능
- 모든 객체지향 언어들을 객체의 타입에 따라 적용 가능한 연산자의 종류를 제한함으로써 프로그래머의 실수를 막아줌

#### 타입에 수행되는 오퍼레이션에 대해 미리 약속된 문맥을 제공

- 예를 들어 자바에서 `a + b`라는 연산이 있을 때 `a`와 `b`의 타입이 `int`라면 두 수를 더할 것, 하지만 `a`와 `b`의 타입이 `String`이라면 두 문자열을 하나의 문자열로 합칠 것
- 객체를 다루는 방법에 대한 문맥을 결정하는 것은 바로 객체의 타입임

#### 객체지향 패러다임 관점의 타입

- 타입이란
  - 개념 관점에서 타입이란 콩통의 특징을 공유하는 대상들의 분류
  - 프로그래밍 언어 관점에서 타입이란 동일한 오퍼레이션을 적용할 수 있는 인스턴스들의 집합
- 객체지향 프로그래밍에서 오퍼레이션은 객체가 수신할 수 있는 메시지를 의미
- **객체의 타입이란 객체가 수신할 수 있는 메시지의 종류를 정의하는 것** -> **퍼블릭 인터페이스**

> 객체의 퍼블릭 인터페이스가 객체의 타입을 결정한다. 따라서 동일한 퍼블릭 인터페이스를 제공하는 객체들은 동일한 타입으로 분류된다

- 객체에게 중요한 것은 속성이 아니라 행동
- 객체들이 동일한 상태를 가지고 있더라도 퍼블릭 인터페이스가 다르다면 이들은 서로 다른 타입으로 분류
- 객체가 외부에 제공하는 행동에 초점을 맞춰야 한다

## 13.2 타입 계층

### 타입 사이의 포함관계

![프로그래밍 언어 타입의 인스턴스 집합](https://mino-park7.github.io/assets/image/2024/01/IMG_5166.jpeg)

![더 세분화된 타입을 부분집합화](https://mino-park7.github.io/assets/image/2024/01/IMG_5178.jpeg)

- 프로그래밍 언어 타입은 객체지향 언어 타입과 절차적 언어 타입을 포함
- 객체지향 언어 타입은 클래스 기반 언어 타입과 프로토타입 기반 언어 타입을 포함
- 동인한 인스턴스가 하나 이상의 타입으로 분류되는 것도 가능 -> 자바는 프로그래밍 언어인 동시에 객체지향 언어에 속하며 크래스 기반 언어이다
- 타른 타입을 포함하는 타입은 포함되는 타입보다 좀 더 **일반화**된 의미를 표현할 수 있음
- 다른 타입을 포함하는 타입은 포함되는 타입보다 더 많은 인스턴스를 가짐
- 포함하는 타입은 **외연 관점에서는 더 크고 내연 관점에서는 더 일반적**
- 포함되는 타입은 **외연 관점에서는 더 작고 내연 관점에서는 더 특수함**

![프로그래밍 언어 타입 계층](https://mino-park7.github.io/assets/image/2024/01/IMG_5179.jpeg)

- 타입 계층을 구성하는 두 타입 관계에서 더 일반적인 타입을 **슈퍼타입(supertype)**
- 더 특수한 타입을 **서브타입(subtype)**
- 일반화 -> 좀 더 보편적이고 추상적으로 만드는 과정
- 특수화 -> 어떤 타입의 정의를 좀 더 구체적이고 문맥 종속적으로 만드는 과정
- Martin 98
  - 일반화는 다른 타입을 와전히 포함하거나 내포하는 타입을 식별하는 행위 또는 그 행위의 결과를 가리킴
  - 특수화는 다른 타입 안데 전체적으로 포함되거나 완전히 내포되는 타입을 식별하는 행위 또는 그 행위의 결과를 가리킴

### 객체지향 프로그래밍과 타입 계층

- **슈퍼타입**이란 서브타입이 정의한 퍼블릭 인터페이스를 일반화시켜 상대적으로 범용적이고 넓은 의미로 정의한 것
- **서브타입**이란 슈퍼타입이 정의한 퍼블릭 인터페이스를 특수화시켜 상대적으로 구체적이고 좁은 의미로 정의한 것
- 서브타입의 인스턴스는 슈퍼타입의 인스턴스로 간주될 수 있다 -> 핵심, 상속과 다형성의 관계를 이해하기 위한 출발점

## 13.3 서브클래싱과 서브타이핑

- 객체지향 프로그래밍 언어에서 타입 구현 -> 클래스 사용, 타입 계층 구현 -> 상속 이용

### 언제 상속을 사용해야 하는가?

- 다음 두 가지 질문에 모두 "예"라고 대답 가능 해야 함

#### 상속 관계가 *is-a* 관계를 모델링 하는가?

- 일반적으로 "자식 클래스는 부모 클래스다" 라고 말해도 이상하지 않다면 상속을 사용할 후보로 간주 가능

#### 클라이언트 입장에서 부모 클래스의 타입으로 자식 클래스를 사용해도 무방한가?

- 상속 계층을 사용하는 클라이언트의 입장에서 부모 클래스와 자식 클래스의 차이점을 몰라야 한다
- 이를 자식 클래스와 부모 클래스 사이의 **행동 호환성**이라고 부름
- 클라이언트의 관점에서 두 클래스에 대해 기대하는 행동이 다르다면 비록 그것이 어휘적으로 is-a 관계로 표현할 수 있다고 해도 상속을 사용해서는 안됨

### is-a 관계

- 새는 펭귄이지만 날 수 없음
- 클라이언트가 새에 대해 기대하는 행동이 "나는 것"이라면 펭귄은 새를 상송해서는 안됨

### 행동 호환성

- 행동의 호환 여부를 판단하는 기준은 **클라이언트의 관점**
- `Penguin`이 `Bird`의 서브타입이 아닌 이유는 클라이언트의 입장에서 모든 새가 날 수 있다고 가정하기 때문

```java
public void flyBird(Bird bird) {
  // 인자로 전달된 모든 bird는 날 수 있어야 함
  bird.fly();
}
```

- 현재 `Penguin`은 `bird`의 자식 클래스이기 때문에 컴파일러는 업캐스팅을 허용
- 따라서 `flyBird` 메서드의 인자로 `Penguin`의 인스턴스가 전달되는 것을 막을 수 있는 방법이 없음
- 하지만 `Penguin`은 날 수 없고, 클라이언트는 모든 `bird`가 날 수 있기를 기대하기 때문에 `flyBird` 메서드로 전달돼서는 안됨
- 따라서 이 둘을 상속 관계로 연결한 위 설계는 수정되어야함
- 어거지로 수정 안하고 쓴다면?

#### 방법 1. 오버라이딩해서 내부 구현 비우기

```java
public class Penguin extends Bird {
  ...
  @Override
  public void fly() {}
}
```

- 이 방법은 어떤 행동도 수행하지 않기 때문에 모든 `bird`가 날 수 있다는 클라이언트의 기대를 만족시키지 못함

#### 방법 2. Penguindml fly 메서드를 오버라이딩한 후 예외를 던짐

```java
pbulic class Penguin extends Bird {
  ...
  @Override
  public void fly() {
    throw new UnsupprotedOperationException();
  }
}
```

- 클라이언트는 `flyBird`에서 예외가 발생할 것을 기대하지 않음

#### 방법 3. flyBird 수정해서 인자가 `Penguin`이 아닐 경우에만 fly 메시지 전송

```java
public boid flyBird(Bird bird) {
  // 인자로 전달된 모든 birtd가 Penguin의 인스턴스가 아닐 경우에만
  // fly() 메시지 전송
  if (!(bird instanceof Penguin)) {
    bird.fly();
  }
}
```

- 또 다른 날지 못하는 새가 상속 계층에 추가된다면? if문 추가... -> 개방-폐쇄 원칙 위반

### 클라이언트의 기대에 따라 계층 분리하기

- 다음 코드는 새에는 날 수 있는 새와 없는 새 두 분류가 존재하며, 그 중 펭귄은 날 수 없는 새에 속한다는 사실을 표현

```java
public class Bird {
  ...
}

public class FlyingBird extends Bird {
  public void fly() {...}
  ...
}

public class Penguin extends Bird{
  ...
}

public void flyBird(FlyingBird bird) {
  bird.fly();
}
```

- 이제 `flyBird` 메서드는 `FlyingBird` 타입을 이용해 날 수 있는 새만 인자로 전달돼야 한다는 사실을 코드에 명시 가능

---

- 이 문제를 해결하는 다른 방법은 클라이너트에 따라 인터페이스를 분리하는 것
- 만약 `Bird`가 날 수 있으면서 걸을 수도 있어야 하고, `Penguin`은 오직 걸을 수만 있다고 가정
- 가장 좋은 방법은 `fly` 오퍼레이션을 가진 `Flyer` 인터페이스와 `walk` 오퍼레이션을 가진 `Walker` 인터페이스로 분리하는 것

![클라이언트 기대에 따른 인터페이스 분리](https://mino-park7.github.io/assets/image/2024/01/IMG_5180.jpeg)

---

- 더 좋은 방법은 합성을 사용하는 것

![합성을 이용한 코드 재사용](https://mino-park7.github.io/assets/image/2024/01/IMG_5181.jpeg)

---

- 이처럼 인터페이스를 클라이언트 기대에 따라 분리함으로써 변경에 의해 영향을 제어하는 설계 원칙을 **인터페이스 분리 원칙 (Interface Segregation Principle, ISP)** 이라고 부름
- 자연어에 현혹되지 말고 요구사항 속에서 클라이언트가 기대하는 행동에 집중하라
  - 크래스의 이름 사이에 어떤 연관성이 있다는 사실은 아무런 의미도 없다
  - 두 클래스 사이에 행동이 호환되지 않는다면 올바른 타입 계층이 아니기 때문에 상속을 사용해서는 안된다

### 서브클래싱과 서브타이핑

#### 서브클래싱(subclassing)

- 다른 클래스의 코드를 재사용할 목적으로 상속을 사용하는 경우
- 자식 클래스와 부모 클래스의 행동이 호환되지 않기 때문에 자식 클래스의 인스턴스가 부모 클래스의 인스턴스를 대체할 수 없음
- **구현 상속 (implementation inheritance)** 또는 **클래스 상속(class inheritance)**이라고 부르기도 함

#### 서브타이핑(subtyping)

- 타입 계층을 구성하기 위해 상속을 사용하는 경우
- 서브타이핑에서는 자식 클래스와 부모 클래스의 행동이 호환되기 때문에 자식 클래스의 인스턴스가 부모 클래스의 인스턴스를 대체할 수 있음
- 서브타이핑을 **인터페이스 상속(interface inheritance)** 이라고 부르기도 함

---

- 서브 클래싱과 서브 타이핑을 나누는 기준은 상속을 사용하는 목적
- 사실 10장에서 나쁜 설계의 예로 든 대부분의 상속은 구현을 재사용하기 위해 사용된 서브클래싱에 속함

> 추상 클래스를 상속한다는 것은 단순한 코드의 재사용을 위한 상속이 아니라 추상 클래스가 정의하고 있는 인터페이스를 상속하겠다는 의미인 것이다 [GOF 1994]

- 슈퍼타입과 서브타입 사이의 관계에서 가장 중요한 것은 퍼블릭 인터페이스이다.
- 슈퍼타입 인스턴스를 요구하는 모든 곳에서 서브타입의 인스턴스를 대신 사용하기 위해 만족해야 하는 최소한의 조건은 서브타입의 퍼블릭 인터페이스가 슈퍼타입에서 정의한 퍼블릭 인터페이스와 동일하거나 더 많은 오퍼레이션을 포함해야 함
- 서브타이핑 관계가 유지되지 위해서는 서브타입이 슈퍼타입이 하는 모든 행동을 동일하게 할 수 있어야 한다 -> 행동 호환성 만족, 대체 가능성(substitutability)
- 위 지침은 **리스코프 치환 원칙**이라는 이름으로 정리되어 소개되어 왔다

## 13.4 리스코프 치환 원칙

- 상속 관계로 연결한 두 클래스가 서브 타이핑 관계를 만족 시키려면

> 여기서 요구되는 것은 다음의 치환 속성과 같은 것이다. S형의 각 객체 o1에 대해 T형의 객체 o2가 하나 있고, T에 의해 정의된 모든 프로그램 P에서 T가 S로 치환될 때, P의 동작이 변하지 않으면 S는 T의 서브타입이다.

- 서브타입은 그것의 기반 타입에 대해 대체 가능해야 한다
- 클라이언트가 차이점을 인식하지 못한 채 기반 클래스의 인터페이스를 통해 서브플래스를 사용할 수 있어야 한다
- 10장에서 살펴본 `Stack`과 `Vector`는 리스코프 치환 원칙을 위반하는 전형적인 예다
  - 클라이언트가 부모 클래스인 `Vector`에 대해 기대하는 행동을 `Stack`에 대해서는 기대할 수 없기 때문에 행동 호환성을 만족시키지 않기 떄문
- 정사각형은 직사각형이다 예 -> 454p ~ 457p 참조

### 클라이언트와 대체 가능성

- 클라이언트 입장에서 정사각형을 추상화한 `Square`는 직사각형을 추상화한 `Rectangle`과 동일하지 않다는 점
- 클라이언트 코드에서 `Rectangle`을 `Square`로 대체할 경우 `Rectangle`에 대해 세워진 가정을 위반할 확률이 높다
- `Stack`과 `Vector`가 서브 클래싱 관계인 이유도 상속으로 인해 `Stack`에 포함돼서는 안되는 `Vector`의 퍼블릭 인터페이스가 `Stack`의 퍼블릭 인터페이스에 포함됐기 때문
- 클라이언트와 격리한 채로 본 모델을 의미 있게 검증하는 것이 불가능하다[Martin02]
- 대체 가능성을 결정하는 정은 클라이언트다

### is-a 관계 다시 살펴보기

- is-a는 클라이언트 관점에서 is-a일 때만 참이다
- is-a 관계 역시 행동이 우선임

### 리스코프 치환 원칙은 유연한 설계의 기반이다

- 새로운 자식 클래스를 추가하더라도 클라이언트 입장에서 동일하게 행동하기만 한다면 클라이언트를 수정하지 않고도 상속 계층을 확장 할 수 있다
- 클라이언트의 입장에서 퍼블릭 인터페이스의 행동 방식이 변경되지 않는다면 클라이언트의 코드를 변경하지 않고도 새로운 자식 클래스와 협력할 수 있다

![DIP, LSP, OCP가 조합된 유연한 설계](https://mino-park7.github.io/assets/image/2024/01/IMG_5182.jpeg)

- **의존성 역전 원칙**
- **리스코프 치환 원칙**
- **개방-폐쇄 원칙**

### 타입 계층과 리스코프 치환 원칙

- 클래스 상속은 타입 계층을 구현할 수 있는 다양한 방법 중 하나일 뿐
  - 자바와 C#의 인터페이스, 스칼라의 트레이트, 동적 타입 언더의 덕 타이핑 등의 기법을 사용하면 클래스 사이의 상속을 사용하지 않고 서브파이핑 관계를 구현할 수 있음
- 리스코프 치환 원칙을 위반하는 예를 설명하는 데 클래스의 상속을 자주 사용하는 이유는 대부분의 객체 지향 언어가 구현 단위로서 클래스를 사용하고 코드 재사용의 목적으로 상속을 지나치게 남용하는 경우가 많기 때문

## 13.5 계약에 의한 설계와 서브타이핑

- 클라이언트와 서버 사이읭 협력을 의무(obligation)와 이익(benefit)으로 구성된 계약의 관점에서 표현하는 것을 **계약에 의한 설계(Design By Contract, DBC)**라고 부른다
  - 클라이언트가 정상적으로 메서드를 실행하기 위해 만족시켜야하는 **사전조건(precondition)**과 메서드가 실행된 후에 서버가 클라이언트에게 보장해야 하는 **사후조건(postcondition)**, 메서드 실행 전과 실행 후에 인스턴스가 만족시켜야 하는 **클래스 불변식(class invariant)**의 세 가지 요소로 구성됨
- 리스코프 치환 원칙과 계약에 의한 설계 사이의 관계를 다음과 같은 한 문장으로 요약 가능

> 서브타입이 리스코프 치환 원칙을 만족시키기 위해서는 클라이언트와 슈퍼타입 간에 체결된 "계약"을 준수해야 한다

- 이해를 돕기 위해 영화 예매 시스템에서 `DiscountPolicy`와 협력하는 `Movie` 클래스를 예로 들어보자

```java
public class Movie {
  ...
  public Money calculateMovieFee(Screening screening) {
    return fee.minus(discountPolicy.calculateDiscountAmount(screening));
  }
}
```

`Movie`는 `DiscountPolicy`의 인스턴스에게 `calculateDiscountAmount` 메시지를 전송하는 클라이언트다. `DiscountPolicy`는 `Movie`의 메시지를 수신한 후 할인 가격을 계산해서 반환한다

```java
public abstract class DiscountPolicy {
  public Money calculateDiscountAmount(Screening screening) {
    for(DiscountCondition each : conditions) {
      if (each.isSatisfiedBy(Screening)) {
        return getDiscountAmount(screening);
      }
    }

    return screening.getMovieFee();
  }

  abstract protected Money getDiscountAmount(Screening screening);
}
```

- DBC에 따르면 협력하는 클라이언트와 슈퍼타입의 인스턴스 사이에는 어떤 계약이 맺어져 있다. 클라이언트와 슈퍼타입은 이 계약을 준수할 때만 정상적으로 협력할 수 있음
- 서브타입이 슈퍼타입처럼 보일 수 있는 유일한 방법은 클라이언트가 슈퍼타입과 맺은 계약을 서브타입이 준수하는 것 뿐
- 위 코드에서 암묵적인 사전조건과 사후조건이 존재
  - 사전 조건
    - `DiscountPolicy`의 `calculateDiscountAmount` 메서드는 인자로 전달된 `screening`이 `null`인지 여부를 확인하지 않음
    - 하지만 `screening`에 `null`이 전달된다면 `screening.getMovieFee()`가 실행될 때 `NullPointerException` 예외가 던져질 것
    - 단정문을 통해 사전조건 표현 가능 `assert screening != null && screening.getStartTime().isAfter(LocalDateTime.now())`
  - 사후 조건
    - `Movie`의 `calcualationMovieFee` 메서드를 살펴보면 `DiscountPolicy`의 `calculationDiscountAmount` 메서드의 반환값은 항상 `null`이 아니어야 한다. 추가로 반환되는 값은 청구되는 요금이기 때문에 0원보다 커야 한다
    - `assert amount != null && amount.isGreaterThanOrEqual(Money.ZERO);`

```java
public abstract class DiscountPolicy {
  public Money calculateDiscountAmount(Screening screening) {
    checkPrecondition(screening);

    Money amount = Money.ZERO;
    for(DiscountCondition each : conditions) {
      if (each.isSatisfiedBy(Screening)) {
        amount = getDiscountAmount(screening);
        checkPostcondition(amount);
        return amount;
      }
    }

    amount = screening.getMovieFee();
    checkPostcondition(amount);
    return amount;
  }

  protected void checkPrecondition(Screening screening) {
    assert screening != null && screening.getStartTime().isAfter(LocalDateTime.now());
  }

  protected void checkPostcondition(Money amount) {
    assert amount != null && amount.isGreaterThanOrEqual(Money.ZERO);
  }
  abstract protected Money getDiscountAmount(Screening screening);
}
```

```java
public class Movie {
  ...
  public Money calculateMovieFee(Screening screening) {
    if (screening == null || screening.getStartTime().isBefore(LocalDateTime.now())) {
      throw new InvalidScreeningException();
    }
    return fee.minus(discountPolicy.calculateDiscountAmount(screening));
  }
}
```

### 서브타입과 계약

- 계약의 관점에서 상속이 초래하는 가장 큰 문제는 자식 클래스가 부모 클래스의 메서드를 오버라이딩할 수 있다는 것
- 서브타입이 계약조건을 만족하려면?

> 서브타입에 더 강력한 사전조건을 정의할 수 없다
> 서브타입에 슈퍼타입과 같거나 더 약한 사전조건을 정의할 수 있다.
> 서브타입에 슈퍼타입과 같거나 더 강한 사후조건을 정의할 수 있다
> 서브타입에 슈퍼타입과 더 약한 사후조건을 정의할 수 없다