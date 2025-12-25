# Open-Closed Principle (개방-폐쇄 원칙)

> “확장에는 열려 있고, 변경에는 닫혀 있어야 한다.”

원칙명이 직관적이며, 기존 코드의 변경 없이 새로운 기능을 추가(확장)할 수 있어야 한다고 알고 있다. 새로운 요구사항마다 기존 코드를 전반적으로 수정해야 한다면, 좋은 설계라고 보기 어렵다.

이런 개념보다 더 중요한 것은 "어떻게 하면 기존 코드를 수정하지 않고 기능을 확장할 수 있을까?"이다.

## OCP가 깨지는 전형적인 할인 정책 코드

예시로 OCP를 위반하면 어떤 문제가 발생하는지 살펴보고, 이를 어떻게 해결할 수 있을지 생각해보자.

```java
public class DiscountService {
    
    public Money applyDiscount(Order order) {
        if (order.getDiscountType() == DiscountType.RATE) {
            return order.getAmount().multiply(0.9);
        }
        if (order.getDiscountType() == DiscountType.FIXED) {
            return order.getAmount().minus(1_000);
        }
        return order.getAmount(); 
    }
}
```

비율 할인과 고정 금액 할인을 처리하는 클래스다.
크게 문제가 될 부분은 없어 보이지만, 새로운 요구사항이 생겼다고 가정하자.

> "첫 구매 고객에게만 적용되는 특별 할인 정책이 필요해요."

요구사항을 반영하려면 아래 if 문을 추가해야 한다.

```java
if (order.getDiscountType() == DiscountType.FIRST_ORDER) {
    return order.getAmount().multiply(0.8);
}
```

이처럼 새로운 할인 정책이 생길 때마다 기존 코드를 수정해야 한다.

- 기간 한정 할인
- 회원 등급별 할인
- ...

직관적으로 "확장에는 닫혀 있고, 변경에는 열려 있는 설계"라고 볼 수 있다.
즉, OCP를 위반하고 있다.

## OCP 위반의 본질

여러 아티클을 읽어보면, if-else 문이나 switch 문이 OCP 위반의 전형적인 예시로 자주 등장한다.
하지만 그런 분기문 자체가 문제인 것은 아니다.

본질은 고수준 정책이 저수준 구현에 직접 의존하지 않게 만드는 것이다.
변경 가능성이 높은 부분을 고정된 확장 포인트 뒤로 밀어내야 한다.

> ### 고수준? 저수준?
>
> 고수준(High-level)은 시스템이 어떤 결정을 내려야 하는지, 어떤 규칙과 정책을 따라야 하는지를 표현하는 코드다.
> “무엇을 할 것인가”에 초점이 맞춰져 있으며, 비즈니스 의미가 강하고 변경의 이유도 주로 기획이나 정책 같은 비즈니스 요구에서 발생한다.
>
>저수준(Low-level)은 고수준에서 결정한 정책을 실제로 어떻게 실행할지를 다루는 코드다.
>기술적인 세부 구현, 알고리즘, 인프라와 밀접하며, “어떻게 할 것인가”에 집중한다.
> 이 코드가 바뀌는 이유는 대부분 기술적 제약, 성능 최적화, 라이브러리나 환경 변경과 같은 기술적인 요구에서 비롯된다.

## 어떻게 OCP를 지킬 수 있을까?

1. 인터페이스나 추상 클래스를 정의하여 확장 포인트를 만든다.
2. 새로운 기능이 필요할 때마다 해당 인터페이스를 구현하는 새로운 클래스를 추가한다.
3. 기존 코드는 변경하지 않고, 새로운 클래스를 사용하도록 설정만 변경한다.

### 변하지 않는 역할을 인터페이스로 분리하기

```java
public interface DiscountPolicy {
 
    boolean supports(Order order);
    Money apply(Order order);
}
```

할인을 적용하는 역할을 인터페이스로 고정하고, 어떤 할인인지는 구현체에 맡기며 확장 포인트를 만든다.

> ### 셀프 반박, supports() 기반 선택도 결국 분기 아닌가
>
> supports() 메서드도 결국 if-else 문이나 switch 문과 같은 분기문 아닌가?  
> 맞다. if, switch(분기)를 완전히 제거한 것은 아니다. 
> 분기 책임을 DiscountService → 각 Policy 구현체로 옮긴 것에 가깝다고 볼 수 있다.

### 구체적인 할인 정책 구현하기

```java
public class RateDiscountPolicy implements DiscountPolicy {

    @Override
    public boolean supports(Order order) {
        return order.getDiscountType() == DiscountType.RATE;
    }

    @Override
    public Money apply(Order order) {
        return order.getAmount().multiply(0.9);
    }
}

public class FixedDiscountPolicy implements DiscountPolicy {

    @Override
    public boolean supports(Order order) {
        return order.getDiscountType() == DiscountType.FIXED;
    }

    @Override
    public Money apply(Order order) {
        return order.getAmount().minus(1_000);
    }
}
```

### 고수준 정책에서 확장 포인트 사용하기

```java
@Service
public class DiscountService {

    private final List<DiscountPolicy> policies;

    public DiscountService(List<DiscountPolicy> policies) {
        this.policies = policies;
    }

    public Money applyDiscount(Order order) {
        return policies.stream()
            .filter(policy -> policy.supports(order))
            .findFirst()
            .map(policy -> policy.apply(order))
            .orElse(order.getAmount());
    }
}
```

DiscountService는 어떤 할인 정책이 있는지 알 필요가 없다.  
&nbsp;&nbsp;→ DiscountService는 할인 정책을 선택하고 조합하는 고수준 정책이다.  
즉, 새로운 할인 정책이 추가되더라도 DiscountService 코드는 변경되지 않는다.  
&nbsp;&nbsp;→ DiscountPolicy 구현체는 변경 가능성이 높은 저수준 정책이다.

Spring 환경에서는 Bean 추가만으로 확장이 가능하기 때문에, 새로운 요구사항이 생길 때마다 기존 코드를 수정할 필요가 없을 것이다.

이 구조가 확장에는 열려 있고, 변경에는 닫혀 있는 설계라고 할 수 있다.

> ### 셀프 반박, 정책이 늘수록 조합은 어려워지지 않나?
>
> 정책이 많아지면 이런 문제가 생길 수 있다.
> - 등급 할인 + 기간 할인 + 첫 구매 할인
> - 동시에 적용 가능한 정책 조합  
> 
> 이 경우 단순한 supports() 구조는 한계가 있다.
> - 우선순위 문제
> - 중복 적용 문제
> - 정책 간 의존 관계
> 
> 즉, OCP를 지켰다고 해서 도메인 복잡성이 사라지는 것은 아니다.

## 정리

핵심은 새로운 요구사항이 생겼을 때, 어디를 바꾸게 될 것인가를 설계하는 것이다.

조건 분기(if, switch)가 많아지는 문제가 아니라, 고수준 정책이 변경 가능성이 높은 구현 세부사항에 직접 의존하고 있다는 점이 문제다.
이 구조에서는 기능을 추가할 때마다 기존 코드의 수정이 반복되고, 변경 범위도 함께 커진다.

OCP를 지킨다는 것은 변하지 않아야 할 역할과, 자주 바뀔 수밖에 없는 로직을 분리하고 변경 가능성이 높은 축을 인터페이스 뒤로 밀어내는 것이다.

고수준 정책은 안정적으로 유지하고, 새로운 요구사항은 기존 코드의 수정이 아니라 새로운 구현의 추가로 해결한다.

이때 확장은 자연스럽게 이루어지고, 변경은 특정 구현체에 국한된다.

결국 OCP는 문법이나 패턴의 문제가 아니라,
의존성의 방향과 확장 지점을 어떻게 설계할 것인가에 대한 원칙이다.

## 참고

- [모든 개발자가 알아야 할 SOLID의 진실 혹은 거짓, KakaoBank Tech, Loopy](https://tech.kakaobank.com/posts/2411-solid-truth-or-myths-for-developers/)
