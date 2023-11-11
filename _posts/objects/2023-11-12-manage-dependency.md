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

