# Interface Segregation Principle (인터페이스 분리 원칙)

> 사용하지 않는 것에 의존하지 않아야 한다.

인터페이스가 제공하는 기능 중 특정 구현체나 호출자가 사용하지 않는 메서드가 있다면, 해당 인터페이스는 분리 대상이다.

## 게임 캐릭터 인터페이스 예시

```java
interface Character {

    void move(Position to);
    void talk(String message);
    void attack(Target target); // 일부는 필요 없음
}
```

Character 인터페이스는 이동, 대화, 공격 기능을 모두 포함하고 있다.
하지만 모든 캐릭터가 공격 기능을 필요로 하는 것은 아니다.

```java
// 상점 NPC 클래스
class MerchantNpc implements Character {

    @Override
    public void move(Position to) { /* ... */ }

    @Override
    public void talk(String message) { /* ... */ }

    @Override
    public void attack(Target target) {
        // 아무것도 하지 않거나 예외 발생
    }
}
```

Character라는 인터페이스 때문에 캐릭터면 공격 가능이라는 오해를 만든다.
어떤 구현체는 아무것도 하지 않고, 어떤 구현체는 예외 → 호출부에 분기 및 예외 처리 코드가 전파된다.

시간이 지나면 attack() 호출하는 코드가 여기저기 생기고, NPC에서 버그/예외 터진다.
즉, Character는 공격할 수도 있고 아닐 수도 있다라는 애매한 타입이 되어버린다.

## 인터페이스 분리하기

```java
interface Movable {
    void move(Position to);
}

interface Talkable {
    void talk(String message);
}

interface Attackable {
    void attack(Target target);
}
```

이제 구현체는 필요한 것만 구현한다.

```java
// 전사 클래스
class Warrior implements Movable, Talkable, Attackable {
 
    public void move(Position to) { /* ... */ }
    public void talk(String message) { /* ... */ }
    public void attack(Target target) { /* ... */ }
}

// 상점 NPC 클래스
class MerchantNpc implements Movable, Talkable {
    
    public void move(Position to) { /* ... */ }
    public void talk(String message) { /* ... */ }
}
```

이제 상점 NPC는 공격 기능이 없으며, 호출부에서도 공격 기능을 기대하지 않는다.

```java
class Game {

    // NPC는 타입이 안 맞으니 컴파일 단계에서 들어올 수가 없다.
    void hit(Attackable attacker, Target target) {
        attacker.attack(target);
    }
}
```

> ### 셀프 반박, 인터페이스가 너무 많아지지 않나?
>
> 인터페이스 수가 많아지면 관리가 어려워지고, 도메인 파편화가 발생할 수 있지 않은가?
>
> ISP를 적용하면 인터페이스 수는 늘어날 수 있다.
> 하지만 이는 설계를 복잡하게 만드는 비용이 아니라,
> 잘못된 의존성이 호출부 전체로 확산되는 것을 막기 위해 지불하는 비용이다.
> 핵심은 인터페이스 수가 아니라, 각 인터페이스가 어떤 호출자를 위한 것인지 명확한가이다.

## 정리

ISP는 인터페이스를 많이 만드는 것이 목적이 아니다.
호출자가 필요한 것만 의존하도록 구현체가 필요 없는 메서드를 억지로 갖지 않도록 만드는 것이 목적이다.

ISP를 지키지 않는다면, 자연스럽게 [LSP](./LiskovSubstitutionPrinciple.md)도 위반할 가능성이 높아진다.
- ISP는 호출자가 불필요한 기능을 의존하게 만들지 마라. (잘못된 계약을 만들지 마라)
- LSP는 그 계약을 믿고 치환해도 행위가 깨지지 않아야 한다. (계약을 배신하지 마라)

## 참고

- [모든 개발자가 알아야 할 SOLID의 진실 혹은 거짓, KakaoBank Tech, Loopy](https://tech.kakaobank.com/posts/2411-solid-truth-or-myths-for-developers/)
