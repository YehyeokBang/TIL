# Liskov Substitution Principle (리스코프 치환 원칙)

다른 원칙들과 달리 사람의 이름을 따서 만들어졌다.

> S 타입의 객체 o1과 T 타입의 객체 o2가 있을 때, T 타입을 이용해서 정의한 프로그램 P 에서 o2를 o1으로 치환해도 P의 행위가 변하지 않는다면, S는 T의 하위타입이다.
> 
> [바바라 리스코프](https://en.wikipedia.org/wiki/Barbara_Liskov)

1. 타입 T가 있다.
2. 서브타입 S가 있다.
3. T 타입으로 만든 프로그램 P가 있다.
4. 프로그램 P에서 T 타입을 서브타입 S로 치환한다.
5. 프로그램의 행위가 변하지 않고 정상적으로 동작한다면, S는 T의 하위타입이다.

리스코프 치환 원칙에서 가장 중요한 부분은 치환해도 `프로그램의 행위가 변하지 않는다`는 것이다. 
하위 타입 구현 시, 부모 타입의 기능을 무작위로 수정한다면, 타입을 치환해서 호출하려 할 때 동작 여부를 확신할 수 없기에, 타입을 서로 치환할 수 없게 된다. 
따라서 LSP는 하위 타입이 상위 타입이 외부에 약속한 행위 계약을 변경해서는 안 된다는 원칙이다.

## LSP를 위반하는 정사각형/직사각형 문제

아래와 같이 직사각형(Rectangle)의 너비와 높이를 설정하고 넓이를 계산하는 Rectangle 클래스가 있다고 가정해보자.

```java
public class Rectangle {
    
    private int width;
    private int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}
```

이를 상속하는 정사각형(Square) 클래스는 너비와 높이가 항상 같아야 한다는 제약 조건이 있다.

```java
public final class Square extends Rectangle {

    @Override
    public void setWidth(int width) {
        super.setWidth(width);
        super.setHeight(width);
    }

    @Override
    public void setHeight(int height) {
        super.setWidth(height);
        super.setHeight(height);
    }
}
```

정사각형 클래스는 너비나 높이를 설정할 때 모두 같은 값이 되도록 부모 클래스의 setHeight()와 setWidth() 메서드를 재정의했다.
부모 클래스의 기능을 변경함으로써 LSP를 위반한 것이다.

이제 Rectangle 타입을 사용하는 아래와 같은 프로그램이 있다고 가정해보자.

```java
public void test(Rectangle rectangle) {
    rectangle.setWidth(5);
    rectangle.setHeight(10);
    assert rectangle.getArea() == 50; // 직사각형의 넓이는 50이어야 한다.
}

Rectangle rectangle = new Rectangle();
test(rectangle); // 정상 동작

Square square = new Square();
test(square); // 정사각형이므로 넓이는 100이 된다. LSP 위반
```

정사각형은 모든 변의 길이가 같아야 하기 때문에, 정사각형을 직사각형의 하위 타입으로 만들면 setWidth()나 setHeight() 메서드를 호출할 때 문제가 생긴다.
너비나 높이를 따로 설정할 수 없기 때문에, 정사각형이 직사각형의 동작(계약)을 수행할 수 없게 된다.

위 문제를 막기 위해 분기문을 사용하여 객체의 타입에 따라 다르게 동작하도록 만들 수도 있다.
그러나 이는 하위 타입을 알아야 한다는 점에서 OCP(Open-Closed Principle, 개방-폐쇄 원칙)도 위반하게 된다.

> ### 궁금증, 정사각형은 수학적으로 직사각형의 하위 개념인데, 왜 코드에서는 안 될까?
>
> 수학에서 말하는 `정사각형 ⊂ 직사각형`은 속성 정의 기준이다. 
> 직사각형은 네 각이 직각, 정사각형은 네 각이 직각 + 네 변의 길이가 같다로 제약을 하나 더 추가한 관계이다.
> 수학에서는 행동(행위)이 존재하지 않는다.
> 오직 성질(불변 조건)만 있다.
>
> 반면, 객체지향 프로그래밍에서는 타입이 제공하는 행동(메서드)이 중요하다.
> 정사각형이 직사각형의 하위 타입이 되려면, 직사각형이 제공하는 모든 행동을 정사각형이 동일하게 제공할 수 있어야 한다는 차이가 있다.

지금까지의 내용을 보면 LSP는 단순히 상속 관계에서의 하위 타입을 설명하는 원칙이라고 생각될 수 있고, SOLID의 창시자인 밥 아저씨도 그렇게 생각했다고 한다.
그러나 이후 잘못된 이해였다는 사실을 깨달았고 현재의 LSP는 인터페이스와 구현체에도 적용되는 더 광범위한 설계 원칙이 되었다.

### 인터페이스와 구현체에서의 LSP

리스코프 치환 원칙은 상속 관계뿐만 아니라 인터페이스와 구현체 관계에도 적용할 수 있다.
인터페이스를 구현하는 클래스는 인터페이스가 정의한 계약을 지켜야 하며, 계약을 지키지 못하는 구현체는 인터페이스의 하위 타입이 되면 안 된다.

```java
interface PaymentProcessor {

    // 계약:
    // 결제 요청을 처리하고 성공 시 PaymentResult.SUCCESS를 반환, 실패 시 PaymentResult.FAILURE 반환
    PaymentResult pay(PaymentRequest request);
}

class BadPaymentProcessor implements PaymentProcessor {
    
    @Override
    public PaymentResult pay(PaymentRequest request) {
        throw new RuntimeException("결제 실패"); // 상위 타입의 계약 위반
    }
}
```

호출자는 결과 객체로 분기한다고 믿고 있는데, 위와 같은 구현체가 추가된다면, 결제 흐름 전체가 구현체를 의심하게 되고, 결제 요청을 처리하는 모든 코드가 예외 처리를 강제당하게 된다.

이외에도 다음과 같은 경우에도 LSP가 위반될 수 있다.

- 상위 타입의 계약은 빈 Collection을 반환하는 것이지만, 하위 타입이 null을 반환하는 경우
- 상위 타입의 계약은 어떤 작업을 수행하는 것이지만, 하위 타입이 아무 작업도 수행하지 않는 경우
    - 아무 작업도 수행하지 않는 것이 계약에 어긋나지 않는다면 괜찮다. 핵심은 계약을 지키느냐이다.

모두 추상화를 제대로 누릴 수 없게 만들어 코드의 복잡도를 높이거나 버그를 유발할 수 있다.

> ### 셀프 반박, 자바 인터페이스에는 계약이 코드로 강제되지 않는다.
>
> 그렇다면 주석이나, 네이밍, 반환 타입 등으로 계약을 명시하는 수밖에 없을 것 같은데, 어디까지 계약이라고 볼 수 있을까?  
> 자바 인터페이스에는 계약이 코드로 강제되지 않는다는 점에서 완벽한 계약을 기대하기 어려울 것 같다.

### (추가) 직사각형/정사각형 문제를 해결하려면?

일단, 정사각형은 직사각형의 하위 타입으로 두기 어렵다.
이유는 Rectangle이라는 클래스가 외부에 제공하는 계약(폭/높이를 독립적으로 설정 가능)을 Square가 지킬 수 없기 때문이다.

Rectangle 타입을 쓰는 코드가 기대하는 행위(폭만 바꾸면 높이는 유지, 높이만 바꾸면 폭은 유지)가 있는데, Square는 그 기대를 충족시켜주지 못한다는 것이다.

만약 이 문제를 해결하고 싶다면, Rectangle보다 더 추상적인 Shape(도형)이라는 상위 타입을 만들고, Rectangle과 Square가 Shape를 구현하도록 바꾸는 방법이 있다.

```java
public interface Shape {
    int getArea();
}

public final class Rectangle implements Shape {
    
    private final int width;
    private final int height;

    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public int getArea() {
        return width * height;
    }
}

public final class Square implements Shape {
    
    private final int sideLength;

    public Square(int sideLength) {
        this.sideLength = sideLength;
    }

    @Override
    public int getArea() {
        return sideLength * sideLength;
    }
}
```

이렇게 하면 Shape이 요구하는 계약(넓이 계산)을 두 클래스가 모두 지킬 수 있으며, 서로의 계약을 강제하지 않게 된다.
추가로 도형의 너비와 높이를 독립적으로 바꿔야 한다는 요구사항이 없다면, 불변 객체로 만드는 것이 더 나은 설계가 될 수 있다.

## 정리

리스코프 치환 원칙은 상속 관계뿐만 아니라 인터페이스와 구현체 관계에도 적용되는 중요한 설계 원칙이다.
이 타입이면 이런 행동을 할 것이다. 라는 호출자의 신뢰가 깨지는 순간이 LSP가 위반되는 순간이라고 할 수 있다.

이를 막기 위해서는 하위 타입이 상위 타입의 계약을 온전히 지키도록 주의 깊게 설계하고 구현해야 한다.

## 참고

- [모든 개발자가 알아야 할 SOLID의 진실 혹은 거짓, KakaoBank Tech, Loopy](https://tech.kakaobank.com/posts/2411-solid-truth-or-myths-for-developers/)
