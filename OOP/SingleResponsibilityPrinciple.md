# Single Responsibility Principle (단일 책임 원칙)

원칙명에 있는 책임이라는 단어는 매우 주관적이다.  
특히 기능, 역할, 업무 등으로 해석될 수 있는 소프트웨어 개발에서 “이 클래스는 책임이 하나인가?”라는 질문은 사람마다 다른 답변을 하게 만든다.

그래서 [밥 아저씨](https://en.wikipedia.org/wiki/Robert_C._Martin)는 SRP는 책임이라는 모호한 개념 대신, 단일 모듈은 변경의 이유가 오직 하나뿐이어야 한다고 말한다.

여기서 중요한 것은 무엇을 하는가가 아니라 왜 바뀌는가이다.
SRP는 기능의 개수나 클래스의 크기를 제한하는 원칙이 아니라, 서로 다른 이유로 바뀌는 것들이 한 모듈에 섞이지 않도록 하라는 설계 기준에 가깝다.

메서드가 여러 개여도 항상 함께 바뀐다면 하나의 책임일 수 있고, 반대로 코드가 작아 보여도 서로 다른 이유로 수정된다면 SRP를 위반하고 있을 수 있다.

여기서 말하는 모듈은 클래스 하나일 수도 있고, 하나의 컴포넌트나 패키지 단위일 수도 있다.

## 변경의 이유를 식별하는 현실적인 기준: 변경 주체

변경의 이유는 추상적인 개념이지만, 프로젝트를 진행하다 보면 누가 이 변경을 요구하는지에 따라 그 이유가 달라지는 경우가 많다.

- 관리자 정책 변경으로 바뀌는 코드
- 전시/조회 성능 이슈로 바뀌는 코드
- 운영, 정책, 법무, CS 요구로 바뀌는 코드

이처럼 변경을 요구하는 주체(팀, 이해관계자)가 다르다면, 그 코드는 이미 서로 다른 변경 이유를 가진 책임을 함께 가지고 있을 가능성이 높다.

물론 변경 주체가 다르다고 해서 항상 분리해야 하는 것은 아니다.  
변경 주체는 SRP를 판단하기 위한 실용적인 출발점일 뿐이며, 최종적으로는 실제 변경이 함께 일어나는지, 즉 변경의 결합도가 어떤지를 함께 살펴봐야 한다.

> ### 셀프 반박, 변경 주체가 바뀌면?
>
> 조직이 바뀌면 설계의 옳고 그름도 바뀌는가?  
> 같은 코드라도 회사/팀마다 SRP 위반 여부가 달라지는가?  
> 그렇다면 조직 안정성도 설계 판단 기준에 포함되어야 하나? 자주 조직이 변경되는 팀이라면 변경 주체 대신 다른 기준을 써야 하나?

## SRP는 “나눠야 한다”가 아니라 “섞이지 말아야 한다”

처음에는 SRP를 모든 것을 잘게 쪼개야 하는 규칙처럼 이해했었다.
하지만 SRP의 목적은 분리가 아니라 변경의 전염을 막는 데 있다.

하나의 변경이 의도하지 않은 다른 동작을 깨뜨리고, 다른 팀의 테스트와 릴리즈를 위협한다면 그 설계는 SRP를 위반하고 있다고 볼 수 있다.

반대로, 같은 주체가 같은 정책과 맥락에서 함께 바뀌는 코드라면 기능이 여러 개여도 하나의 책임으로 볼 수 있다.

SRP의 핵심은 얼마나 잘게 나누었는지가 아니라, 서로 다른 변경 이유가 한 모듈에 섞이지 않았는지에 있다.

> ### 셀프 반박, 섞이지 말아야 한다?
>
> 모든 것을 섞이지 않게 하면 시스템은 어쩔 수 없이 더 많은 경계를 만든다.
> 그 결과 더 많은 클래스, 호출 흐름 파편화, 온보딩 비용 증가가 발생할 수 있다.
> 어느 순간에는 변경 전염을 줄이는 것보다 변경 위치를 빠르게 찾는 게 더 중요할 수도 있다.

## 사고를 확장하는 질문들

- 이 모듈을 수정하라는 요청은 항상 같은 종류의 이유에서 오는가?
- 이 코드가 바뀔 때 영향을 받는 테스트와 사용자는 누구인가?
- 한 요구사항의 변경이, 전혀 다른 관심사의 코드를 함께 수정하게 만들지는 않는가?
- “공통 로직”이라는 이유로 묶인 코드가 사실은 서로 다른 정책을 공유하고 있지는 않은가?

이 질문에 대한 답이 하나로 수렴하지 않는다면, 그 모듈은 SRP 관점에서 분리 후보일 가능성이 높다.

## 결제 금액 계산 클래스 예시

```java
class PaymentCalculator {

    // 기본 결제 금액 계산
    public Money calculateOrderPayment(Order order) {
        Money baseAmount = order.getTotalPrice();
        return applyDiscount(baseAmount, order.getMemberGrade());
    }

    // 기본 환불 금액 계산
    public Money calculateRefundPayment(Order order) {
        Money baseAmount = order.getRefundPrice();
        return applyDiscount(baseAmount, order.getMemberGrade());
    }

    // 등급별 할인 정책 적용
    private Money applyDiscount(Money amount, MemberGrade grade) {
        if (grade == MemberGrade.VIP) {
            return amount.multiply(0.9);
        }
        return amount;
    }
}
```

결제 금액을 계산하는 클래스가 있다고 가정해보자.
처음에는 `결제 금액 계산`이라는 하나의 책임만 가진 것처럼 보인다.

### 실제 사용 맥락

| 메서드 | 변경 이유 | 변경 주체 |
|------|----------|----------|
| calculateOrderPayment | 주문 할인 정책, 프로모션 | 주문/결제 팀 |
| calculateRefundPayment | 환불 정책, 정산 규칙 | CS / 정산 팀 |
| applyDiscount | 두 정책을 동시에 만족시켜야 함 | ❌ (충돌 지점) |

### 새로운 요구사항이 생겼다

> “환불 금액에는 VIP 할인을 적용하지 말아주세요.”

이를 위해 아래와 같이 코드를 수정해야 한다.

```java
// 기존 할인 적용 메서드
private Money applyDiscount(Money amount, MemberGrade grade) {
    if (grade == MemberGrade.VIP) {
        return amount.multiply(0.9);
    }
    return amount;
}

// 수정된 할인 적용 메서드
private Money applyDiscount(Money amount, MemberGrade grade, boolean isRefund) {
    if (isRefund) {
        return amount;
    }
    if (grade == MemberGrade.VIP) {
        return amount.multiply(0.9);
    }
    return amount;
}
```

```java
public Money calculateOrderPayment(Order order) {
    Money baseAmount = order.getTotalPrice();
    return applyDiscount(baseAmount, order.getMemberGrade(), false);
}

public Money calculateRefundPayment(Order order) {
    Money baseAmount = order.getRefundPrice();
    return applyDiscount(baseAmount, order.getMemberGrade(), true);
}
```
applyDiscount() 호출부가 모두 수정되었다.
분명 환불 정책 변경인데 주문 결제 코드까지 함께 수정되었다.
테스트 영향 범위도 넓어졌으며, “왜 환불 때문에 주문 테스트가 깨지지?” 라는 의문이 생길 수 있다.

### 문제 원인은 SRP 위반

이 클래스의 문제는 메서드 수가 아니다.
`주문 결제 정책 변경`과 `환불 정책 변경`이라는 서로 다른 변경 이유가 하나의 private 메서드에 섞여 있다는 점이다.

applyDiscount()는 공통 로직처럼 보이지만, 실제로는 서로 다른 정책을 동시에 만족시켜야 하는 지점이 된다.

```
주문 할인 정책 변경 → OrderPaymentCalculator  
환불 정책 변경 → RefundPaymentCalculator
```

이렇게 분리한다면 각 변경 주체가 자신의 관심사에만 집중할 수 있고, 변경이 다른 영역에 전염되지 않을 것이다.

> ### 셀프 반박, 응집도 관점
>
> SRP를 책임 분리보다는 응집도 관점으로도 생각해보자.
> 변경 이유가 다르다는 이유만으로 분리하여 도메인 개념이 파편화된 것은 아닌가?
>
> 할인 로직의 중복은 어떻게 처리하나? 
> 나중에 "환불도 VIP 할인 적용"으로 정책이 바뀌면 다시 합쳐야 하나?

## 정리

SRP는 “클래스는 하나의 책임만 가져야 한다”는 규칙이라기보다, “하나의 모듈은 하나의 변경 이유만 가져야 한다”는 설계 판단 기준에 가깝다.

변경 이유를 식별하기 위한 실용적인 출발점 중 하나가 변경 주체(팀, 이해관계자)다.
어떤 코드가 누구의 요구에 의해, 어떤 맥락에서 바뀌는지를 살펴보면 서로 다른 변경 이유가 한 모듈에 섞여 있는지 비교적 빠르게 드러난다.

구조가 잘게 나뉘어 있는 설계가 아니라, 변경이 발생했을 때 어디를 고쳐야 할지 명확하고, 그 변경이 다른 관심사로 불필요하게 전염되지 않는 구조가 SRP를 잘 지키고 있는 설계라고 할 수 있을 것이다.

### 셀프 반박 정리

- 변경 주체는 조직 구조에 의존적인 기준일 수 있다.  
- “섞이지 말아야 한다”는 원칙은 과도한 분리로 이어질 위험도 있다.  
- 응집도 관점에서는 하나의 개념으로 묶여 있는 편이 더 자연스러운 경우도 있다.

그래서 SRP는 처음부터 지켜야 할 절대적인 규칙이라기보다, 변경이 반복되며 코드가 복잡해질 때 어디에 경계를 세울지 판단하기 위한 도구에 가깝다고 생각한다.

아직 변경이 거의 없거나, 팀과 도메인이 충분히 작을 때는 SRP보다 명확성과 개념적 응집을 우선하는 선택이 더 나을 수도 있다.

결국 SRP는 정답의 문제가 아니라, 변경 비용과 이해 비용 사이에서 어떤 트레이드오프를 선택할지에 대한 문제인 것 같다.

## 참고 자료

- [모든 개발자가 알아야 할 SOLID의 진실 혹은 거짓, KakaoBank Tech, Loopy](https://tech.kakaobank.com/posts/2411-solid-truth-or-myths-for-developers/)
