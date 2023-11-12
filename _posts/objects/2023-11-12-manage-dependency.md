---
layout: "post"
title: "8. 의존성 관리하기"
date: "2023-11-12 01:10+0900"
category: Objects
tag: [Objects]
---

# 8. 의존성 관리하기

- 잘 설계된 객체지향 애플리케이션은 작고 응집도 높은 객체들로 구성
  - 작고 응집도 높은 객체 : 책임의 초점이 명확하고 **한 가지 일만 잘 하는 객체**
- 이런 객체들이 단독으로 수행할 수 있는 작업은 거의 없기 때문에 **객체간의 협력**이 발생
- 협력은 필수적이나, 다른 객체에 대한 지식을 요함 -> 의존성 발생
- 과도한 의존성은 애플리케이션을 수정하기 어렵게 만듬
- 협력을 위해 필요한 의존성은 유지하면서도 변경을 방해하는 의존성은 제거하는 것이 좋은 설계

## 1. 의존성 이해하기

### 변경과 의존성

- 의존성은 실행 시점과 구현 시점에 서로 다른 의미를 가짐
  - 실행 시점: 의존하는 객체가 정상적으로 동작하기 위해서는 실행 시에 의존 대상 객체가 반드시 존재하야 한다.
  - 구현 시점: 의존 대상 객체가 변경될 경우 의존하는 객체도 함께 변경된다
- 영화 예매 코드의 `PereiodCondition` 클래스를 이용해 의존성의 개념 설명
  - `PeriodCondition` 클래스의 `isSatisfiedBy` 메서드는 `Screening` 인스턴스에게 `getStartTime` 메시지를 전송항

```java
public class PeriodCondition implements DiscountCondition {
  private DayOfWeek dayOfWeek;
  private LocalTime startTime;
  private LocalTime endTime;

  ...

  public boolean isStisfiedBy(Screening screening) {
    return screening.getStartTime().getDayofWeek().equals(dayofWeek) &&
      startTime.compareTo(screening.getStartTime().toLocalTime()) <= 0 &&
      endTime.compareTo(screening.getStartTime().toLocalTime()) >= 0;
  }
}
```

- 실행 시점에 `PeriodCondition`의 인스턴스가 정상적으로 동작하기 위해서는 Screening의 인스턴스가 존재해야 함
- 만약 `Screening`의 인스턴스가 존재하지 않거나, `getStartTime`의 메시지를 이해할 수 없다면 `PeriodCondition`의 `isSatisfiedBy` 메서드는 예상했던대로 동작하지 않을 것
- 이런 상황을 두 객체 사이의 의존성이 존재한다고 말하며, 의존성은 단방향이다.
- `PeriodCondition` 은 `Screening`에 의존한다 (점선화살표)

---

- 의존성은 변경에 의한 영향의 전파 가능성을 암시
- 예시 코드에서, `PeriodCondition`은 `DayofWeek`, `LocalTime`, `Screening`에 대해 의존성을 가짐
- 어떤 형태로든, `DayofWeek`, `LocalTime`, `Screening`, `DiscountCondition` 이 변경된다면 `PeriodCondition`도 함께 변경될 수 있음
- 그림 8.3을 보면 의존성 종류별로 표현된 클래스 다이어그램을 볼 수 있음

### 의존성 전이

- 의존성 전이 (transitive dependency)가 의미하는 것은 `PeriodCondition`이 `Screening`에 의존할 경우, `PeriodCondition`은 `Screening`이 의존하는 대상에 대해서도 자동적으로 의존하게 되는 것 (그림 8.4)
- 의존성은 함께 변경될 수 있는 *가능성*을 의미하기 때문에 모든 경우에 의존성이 전이되는 것은 아님, 이는 변경의 방향과 캡슐과의 정도에 따라 달라짐
- 의존성을 **직접 의존성**과 **간접 의존성**으로 나누기도 함
  - 특히 간접 의존성은 코드에 명시적으로 드러나지 않음

### 런타임 의존성과 컴파일타임 의존성

- 런타임은 말 그대로 애플리케이션이 실행되는 시점
- 컴파일타임은 미묘함, 작성된 코드를 컴파일하는 시점을 가리키지만, 문맥에 따라서는 코드 그 자체를 가리키기도 함
  - 동적 타입 언어는 컴파일 타임이 존재하지 않음!
- 런타임의 주인공은 객체! 따라서 런타임 의존성이 다루는 주제는 **객체 사이의 의존성**
- 코드 관점에서의 주인공은 클래스. 따라서 컴파일타임 의존성이 다루는 주재는 **클래스 사이의 의존성**
- 이 둘은 다를 수 있다.
  - `Movie`는 `AmountDiscountPolicy`와 `PercentDiscountPolicy`모두와 협력할 수 있어야 함 -> 추상**클래스** 사용
  - `Movie`클래스는 오직 추상 클래스인 `DiscountPolicy` 클래스에만 의존

```java
public class Movie {
  ...
  private Discount Policy discountPolicy;

  public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
    ...
    this.discountPolicy = discountPolicy;
  }

  public Money calculateMovieFee(Screening screening) {
    return fee.minus(discountPolicy.calculateDiscountAmount(screening));
  }
}
```

---

- 하지만 런타임 의존성을 살펴보면 상황이 완전히 달라짐
- 금액 할인 정책을 적용하기 위해서는 **인스턴스**와 협력해야 함
- 컴파일 시점에는 전혀 모르던 `AmountDiscountPolicy`와 `PercentDiscountPolicy` 인스턴스와 협력해야 함
- 컴파일타임 의존성을 런타임 의존성으로 잘 대체해야 함
- 어떤 클래스의 인스턴스가 다양한 클래스의 인스턴스와 협려가기 위해서는 협력할 인스턴스의 **구체적인 클래스**를 알아서는 안된다
- 클래스가 협력할 객체의 클래스를 명시적으로 드러내고 있다면 **다른 클래스의 인스턴스와 협력할 가능성 자체가 사라짐**

### 컨텍스트 독립성

- 구체 클래스에 대해 의존하는 것은 클래스의 인스턴스가 어떤 문맥에서 사용될 것인지를 구체적으로 명시하는 것과 같다
- `Movie` 클래스 안에 `PercentDiscountPolicy` 클래스에 대한 컴파일타임 의존성을 명시적으로 표현하는 것은 `Movie`가 비율 할인 정책이 적용된 영화의 요금을 계산하는 문맥에서 사용될 것이라는 것을 가정
- 클래스가 특정한 문맥에 강하게 결합될수록 **다른 문맥에서 사용하기는 더 어려워짐**
- 클래스가 사용된 특정한 문맥에 대해 최소한의 가정만으로 이뤄져 있다면 다른 무낵에서 재사용하기가 더 수월해진다 -> **컨텍스트 독립성**

### 의존성 해결하기

- 컴파일타임 의존성은 구체적인 런타임 의존성으로 대체돼야 함 -> **의존성 해결**
  - 객체를 생성하는 시점에 생성자를 통해 의존성 해결
  - 객체 생성 후 `setter` 메서드를 통해 의존성 해결
  - 메서드 실행 시 인자를 이용해 의존성 해결
- ex, 어떤 영화의 요금 계산에 금액 할인 정책을 적용하고 싶다고 가정해보자, 다음과 같이 `Movie` 객체를 생성할 때 `AmountDiscountPolicy`의 인스턴스를 `Movie`의 생성자에 인자로 전달하면 됨

```java
Movie avatar = new movie("아바타",
  Duration.ofMinutes(120),
  Money.wons(10000),
  new AmountDiscountPolicy(...));
```

- Movie의 생성자에 `PercentDiscountPolicy`의 인스턴스를 전달하면 비율 할인 정책에 따라 요금을 계산하게 될 것이다

```java
Moview starWars = new Movie("스타워즈",
  Duration.ofMinutes(180),
  Money.wons(11000),
  new PercentDiscountPolicy(...));
```

- 이를 위해 `Movie`클래스의 생성자 코드는 다음과 같다

```java
public class Movie {
  public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
    ...
    this.discountPolicy = discountPolicy;
  }
}
```

- 클래스 생성 후 메서드를 이용하는 방법

```java
Movie avatar = new Movie(...);
avatar.setDiscountPolicy(new AmoutDiscountPolicy(...));

public class Movie {
  ...
  public void setDiscountPolicy (DiscountPolicy discountPolicy) {
    this.discountPolicy = discountPolicy;
  }
}
```

- `setter` 메서드가 있으면 인스턴스를 중간에 바꿀 수 있음
- 실행 시점에 의존 대상을 변경할 수 있으므로, 설계를 좀 더 유연하게 만들 수 있음
- 단점은 객체가 생성된 후에 협력에 필요한 의존 대상을 설정하기 때문에 객체를 생성하고 의존 대상을 설정하기 전까지는 객체의 상태가 불완전할 수 있다.
- 둘 다 쓰면 된다!

## 2. 유연한 설계

### 의존성과 결합도

- 객체들이 협력하기 위해서는 서로의 존재와 수행 가능한 책임을 알아야 함
- 이런 지식들이 객체 사이의 의존성을 낳는다
  - 의존성이 나쁜 것은 아니지만, 과하면 문제가 될 수 있다
- `Movie`가 비율 할인 정책을 구현하는 `PercentDiscountPolicy`에 직접 의존한다면?

```java
public class Movie {
  ...
  private PercentDisplayPolicy percentDiscountPolicy;

  public movie(String title, Duration runningTime, Money fee, 
  PercentDiscountPolicy percentDiscountPolicy) {
    ...
    this.percentDiscountPolicy = percentDiscountPolicy;
  }

  public Money calculateMovieFee(Screening screening) {
    return fee.minus(percentDiscountPolicy.calculateDiscountAmount(screening));
  }
}
```

- 이 코드에서는 명시적으로 `Movie`가 `PercentDiscountPolicy`에 의존하고 있음을 보여줌
- 의존성 자체가 문제는 아니지만, **너무 구체적인 클래스**에 의존하고 있음
- 자신이 전송하는 `calculateDiscountAmount` 메시지를 이해할 수 있고 할인된 요금을 계산할 수만 있다면 어떤 타입의 객체와 협력하더라도 상관 없음 -> 추상 클래스 `DiscountPolicy` 사용으로 해결
- `PercentDiscountPolicy`에 대한 의존은 부적절, `DiscountPolicy`에 대한 의존은 적절
- 바람직한 의존성이란? **재사용성**과 관련됨, 다시 말해 **컨텍스트에 독립적**이여야 함
- 세련된 표현이 존재: **결합도**
  - **loose coupling, weak coupling** : 바람직한 의존성
  - **tight coupling, strong coupling** : 바람직하지 못한 의존성
- 바람직한 의존성이란 설계를 **재사용하기 쉽게** 만드는 의존성

### 지식이 결합을 낳는다

- 결합도의 정도는 한 요소가 자신이 의존하고 있는 다른 요소에 대해 알고 있는 정보의 양으로 결정
- 서로에 대해 알고 있는 **지식의 양**이 결합도를 결정
- 더 많이 알고 있다는 것은 **더 적은 컨텍스트에서 재사용 가능**하다는 것을 의미
- 기존 지식에 어울리지 않는 컨텍스트에서 클래스의 인스턴스를 사용하기 위해서 할 수 있는 유일한 방법은 클래스를 수정하는 것뿐
- 이 목적을 달성할 수 있는 가장 효과적인 방법은 **추상화**

### 추상화에 의존하라

- 추상화란 어떤 양상, 세부사항, 구조를 좀 더 명확하게 이해하기 위해 특정 절차나 물체를 의도적으로 생략하거나 감춤으로써 복잡도를 극복하는 방법
- 대상에 대해 알아야 하는 지식의 양을 줄일 수 있기 때문에 결합도를 느슨하게 유지할 수 있음
- 일반적으로 추상화와 결합도의 관점에서 의존 대상을 다음과 같이 구분하는 것이 유용, 아래로 갈 수록 클라이언트가 알아야 하는 지식의 양이 적어지기 때문에 결합도가 느슨해짐
  - 구체 클래스 의존성 (concrete class dependency)
  - 추상 클래스 의존성 (abstract class depecdency)
  - 인터페이스 의존성 (interface dependency)
- 추상 클래스의 클라이언트는 여전히 협력하는 대상이 속한 클래스 상속 계층이 무엇인지에 대해서는 알고있어야함
- 인터페이스에 의존하면 상속 계층을 모르더라도 협력이 가능해짐

### 명시적인 의존성

- 아래 코드는 한 가지 실수로 인해 결합도가 불필요하게 높아짐, 실수가 뭘까?

```java
public class Movie {
  ...
  private DiscountPolicy discountPolicy;

  public Movie(String title, Duration runningTime, Money fee) {
    ...
    this.discountPolicy = new AmountDiscountPolicy(...);
  }
}
```

- 생성자가 잘못됨
- 타입만 추상화 한다고 끝이 아니라, 클래스 안에서 구체 클래스에 대한 모든 의존성을 제거해야 함

```java
public class Movie {
  ...
  private DiscountPolicy discountPolicy;

  public Movie(String title, Duration RunningTime, Money fee, DiscountPolicy discountPolicy) {
    ...
    this.discountPolicy = discountPolicy;
  }
}
```

- 생성자의 인자가 추상 클래스 타입으로 선언됐기 때문에, 자식 클래스라면 어떤 것이라도 전달 가능
- 이렇게 퍼블릭 인터페이스에 인자로 의존성을 드러내는 경우를 **명시적인 의존성 (explicit dependency)** 이라고 부름
- 반면 `Movie` 내부에서 `AmountDiscountPolicy`의 인스턴스를 직접 생성하는 방식은 `Movie`가 `DiscountPolicy`에 의존한다는 사실을 감춤 -> **숨겨진 의존성 (hidden dependency)**
- 숨겨진 의존성을 파악하는 것은 매우 고통스러운 일 (코드 구현을 다 봐야함)
- 더 큰 문제는 클래스를 다른 컨텍스트에서 재사용하기 위해 내부 구현을 직접 변경해야 하는 것 -> 버그의 발생 가능성을 내포함
- 의존성은 **명시적으로 표현되어야 함**
- 경계해야 할 것은 의존성 자체가 아니라 **의존성을 감추는 것**

### new는 해롭다

- `new`를 잘못 사용하면 클래스 사이의 결합도가 극단적으로 높아짐
  - `new`연산자를 사용하기 위해서는 **구체 클래스의 이름을 직접 기술**해야 한다. 따라서 `new`를 사용하는 클라이언트는 추상화가 아닌 구체 클래스에 의존할 수 밖에 없기 때문에 결합도가 높아짐
  - `new` 연산자는 생성하려는 구체 클래스뿐만 아니라 어떤 인자를 이용해 클래스의 생성자를 호출해야 하는지도 알아야 한다. 따라서 `new`를 사용하면 클라이언트가 알아야 하는 지식의 양이 늘어나기 때문에 결합도가 높아짐
- 272p. 예시 참고
- `new`는 결합도를 높이기 때문에 해롭다. `new`는 여러분의 클래스를 구체 클래스에 결합시키는 것만으로 끝나지 않음
  - 협력할 클래스의 인스턴스를 생성하기 위해 어떤 인자들이 필요하고 그 인자들을 어떤 순서로 사용해야 하는지에 대한 정보도 노출시킬뿐만 아니라 인자로 사용되는 구체 클래스에 대한 의존성을 추가함
- 해결 방법
  - 인스턴스를 생성하는 로직과 생성된 인스턴스를 사용하는 **로직을 분리하는 것**
  - `Movie`는 외부로부터 이미 생성된 `AmountDiscountPolicy`의 인스턴스를 전달받아야 함
  - 사용과 생성의 책임을 분리해서 `Movie`의 결합도를 낮추면 설계를 유연하게 만들 수 있음.
  - 객체를 생성하는 책임을 **객체 내부가 아니라 클라이언트로 옮기는 것에서 시작**

### 가끔은 생성해도 무방하다

- 협력하는 **기본 객체**를 설정하고 싶은 경우, 객체 내에서 인스턴스를 직접 생성하는 방식이 유용할 수 있음

```java
public class Movie {
  private DiscountPolicy discountPolicy;

  public Movie(String title, Duration runningTime, Money fee) {
    this(title, runningTime, fee, new AmountDiscountPolicy(...));
  }
  
  public Movie(String title, Duration runningTime, MOney fee, DiscountPolicy discountPolicy) {
    ...
    this.discountPolicy = discountPolicy;
  }
}
```

- 위와 같이 대부분 `AmountDiscountPolicy`를 사용하는 경우는, 따로 생성자를 갖는 것도 유용할 수 있음
- 메서드를 오버로딩하는 경우에도 사용 가능
- 다음과 같이 `DiscountPolicy`의 인스턴스를 인자로 받는 메서드와 기본값을 생성하는 메서드를 함께 사용한다면 클래스의 사용성을 향상시키면서도 다양한 컨텍스트에서 유연하게 사용될 수 있는 여지를 제공가능

```java
public class Movie {
  public Money calculateMovieFee(Screening screening) {
    return calculateMovieFee(screening, new AmountDiscountPolicy(...));
  }

  public Money calculateMovieFee(Screening screening, DiscountPolicy discountPolicy) {
    return fee.minus(discountPolicy.calculateDiscountAmount(screening));
  }
}
```

- 설계는 트레이트오프, 재사용성을 높이고 결합도를 높인다면, 이것도 선택가능하다
- 종종 모든 결합도가 모이는 새로운 클래스를 추가함으로써 사용성과 유연성이라는 두 마리 토끼를 잡을 수 있는 경우도 있음 -> `FACTORY`

### 표준 클래스에 대한 의존은 해롭지 않다

- 변경될 확률이 거의 없는 클래스라면 의존성이 문제가 되지 않음
- 구체 클래스에 의존해도 됨!

### 컨텍스트 확장하기

- 할인 혜택을 제공하지 않는 영화의 예매 요금을 계산하는 경우

```java
public class Movie {
  public Movie(String title, Duration runningTime, Money fee) {
    this(title, runningTime, fee, null);
  }

  public Movie(String title, Duration running Time, Money fee, DiscountPolicy discountPolicy) {
    ...
    this.discountPolicy = discountPolicy;
  }

  public Money calculateMovieFee(Screening screening) {
    if (discountPolicy == null) {
      return fee;
    }

    return fee.minus(discountPolicy.calculateDiscountAmount(screening));
  }
}
```

- 위 코드는 적절하지 않음, 왜냐하면, 할인 정책이 없는 경우를 위해 코드 내부를 수정했기 때문
- 할인 정책이 없는 경우도 할인 정책으로 만든다면?

```java
public class NoneDiscountPolicy extends DiscountPolicy {
  @Override
  protected Money getDiscountAmount(Screening screening) {
    return Money.ZERO;
  }
}
```

- 이제 `Movie` 클래스에 수정을 가하지 않고, 인스턴스 생성 시에 `NoneDiscountPolicy`를 인자로 주면 됨

---

- 두 번째 예는 중복 적용이 가능한 할인 정책 구현
- 이를 위해서는 Movie가 하나 이상의 DiscountPolicy와 협력할 수 있어야 함
- `List`를 인자로 받게 바꾼다? -> `Movie` 클래스가 수정되야함
- 이것 또한 중복 할인 정책을 할인 정책의 한가지로 간주하자

```java
public class OverlappedDiscountPolicy extends DiscountPolicy {
  private List<DiscountPolicy> discountPolicies = new ArrayList<>();

  public OverlappedDiscountPolicy(DiscountPolicy ... discountPolicies) {
    this.discountPolicies = Arrays.asList(discountPolicies);
  }

  @Overried
  protected Money getDiscountAmount(Screening screening) {
    Money result = Money.ZERO;
    for (DiscountPolicy each : discountPolicies) {
      result = result.plus(each.caculateDiscountAmount(screening));
    }
    return result;
  }
}
```

### 조합 가능한 행동

- 다양한 종류의 할인 정책이 필요한 컨텍스트에서 `Movie`를 재사용할 수 있었던 이유는 **코드를 직접 수정하지 않고**도 협력 대상인 `DiscountPolicy` 인스턴스를 교체할 수 있었기 때문
- 어떤 객체와 협력하느냐에 따라 객체의 행동이 달라지는 것은 유연하고 재사용 가능한 설계가 가진 특징
- 유옇나고 재사용 가능한 설계는 **객체가 어떻게(how) 하는지를 장황하게 나영하지 않고도 객체들의 조합을 통해 무엇(what)을 하는지를 표현하는 클래스들로 구성**
- How가 아닌 What에 초첨!