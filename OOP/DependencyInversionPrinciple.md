# Dependency Inversion Principle (의존성 역전 원칙)

> 추상화에 의존해야 하며, 구체화에 의존하면 안 된다.

DIP는 이름만 보면 의존성을 뒤집는다(역전)는 말이 먼저 보이지만, 진짜 핵심은 의존성의 방향을 클래스 화살표가 아니라 변경이 전파되는 방향으로 보라는 데 있다.

즉, DIP는 인터페이스를 쓰자가 아니라, 변동성이 큰 구현(저수준)을 바꾸더라도 변하지 말아야 하는 정책(고수준)은 흔들리지 않게 변경 전파를 차단하는 설계 원칙이다.

## 고수준과 저수준

- 고수준(High-level): 비즈니스 정책, 규칙, 유스케이스 흐름
    - 예) OrderService, PaymentService
- 저수준(Low-level): DB, 네트워크, 파일 I/O, 외부 API, 프레임워크 세부 구현
    - 예) MysqlOrderRepository, RestTemplateClient

문제는 정책(고수준)이 세부 구현(저수준)의 변경에 끌려다니는 구조가 만들어진다는 점이다.
이 구조를 끊어내기 위해 DIP를 적용한다.

## DIP 위반 예시

```java
class OrderService {
    
    private final MysqlOrderRepository repository = new MysqlOrderRepository();
}
```

OrderService는 MysqlOrderRepository라는 구체적인 구현에 직접 의존하고 있다.

```
OrderService ───▶ MysqlOrderRepository
```

즉, 고수준이 저수준에 직접 의존하고 있다.
- 여기서 문제는 new 자체가 아니라, OrderService가 구체 타입(MysqlOrderRepository)을 알아야만 동작하는 구조라는 점이다.

만약 DB를 Oracle로 바꾸거나, JPA를 도입, 캐시 레이어 추가 등 구현이 바뀌면 OrderService도 함께 수정해야 한다.

## DIP 준수하기

```java
interface OrderRepository {

    Order findById(Long id);
}

class OrderService {

    private final OrderRepository repository;

    OrderService(OrderRepository repository) {
        this.repository = repository;
    }
}

class MysqlOrderRepository implements OrderRepository { 
    // ...
}
```

이제 OrderService는 OrderRepository라는 추상화에만 의존한다.

```
OrderService ───▶ OrderRepository ◀─── MysqlOrderRepository
```

OrderService는 더 이상 MySQL을 모른다.
오직 “주문을 조회할 수 있다”는 역할(정책) 만 알고 있다.

## 왜 역전이라고 부를까?

처음에는 구현체 대신 인터페이스를 보게 되어서 역전인건가 싶었다.
하지만 DIP에서 말하는 역전의 핵심은 그게 아니다.

> DIP에서 역전된 것은 `누가 누구의 변경에 영향을 받는가의 방향`이다.
>
> 즉, 역전의 기준 = 변경 전파 방향

```java
// 원래 구조
[정책] OrderService ───▶ [세부] MysqlOrderRepository

// DIP 적용 후 구조
[정책] OrderService ───▶ [추상] OrderRepository
[세부] MysqlOrderRepository ───▶ [추상] OrderRepository
```

전통적인 구조  
→ 정책이 구현에 종속됨 (구현 바뀌면 정책도 수정)  

DIP 구조  
→ 구현이 정책이 정의한 규칙에 종속됨 (구현 바뀌어도 정책은 수정 불필요)

즉, 의존성의 주도권이 바뀐 것이다.
그래서 의존성 역전(Dependency Inversion)이라고 부른다.

## 근데 실행 흐름에서는 똑같지 않나?

맞다. 실행 흐름은 변하지 않는다.

```
Controller → OrderService → MysqlOrderRepository
```

실제로 DB 호출은 여전히 구현체가 한다.

하지만 Compile-time Dependency 관점에서 보면 다르다.

```
OrderService ───▶ OrderRepository ◀─── MysqlOrderRepository
```

- 정책은 구현체가 아니라 추상에 의존
- 구현체가 그 추상을 구현

실행 흐름은 그대로인데, 코드 의존성만 제어 흐름과 반대 방향으로 바뀐다.
그래서 런타임에서는 구현체를 호출하지만, 컴파일 타임에는 구현체를 모르는 구조가 된다.

> ### 그럼 인터페이스는 누가 정의할까?
>
> DIP에서 인터페이스는 구현이 필요로 하는 계약이 아니라 고수준 정책이 필요로 하는 계약이다.
> 따라서 인터페이스는 고수준 모듈이 정의하고, 저수준 모듈이 그것을 구현한다.

## DIP가 주는 실질적인 이점

- 변경 전파 차단: 구현 변경이 정책까지 흔들리지 않는다.
- 테스트 용이성: 정책을 구현과 분리해서 테스트할 수 있다.
- 확장 가능한 구조: 새로운 구현을 추가해도 기존 정책은 수정되지 않는다.

> ### 셀프 반박, 인터페이스를 만들면 오히려 더 복잡해지지 않나?
> 
> DIP는 대부분 `인터페이스 + 주입` 형태로 구현되기 때문에, 클래스 수가 늘고 구조가 복잡해질 수 있다.
>
> 하지만 DIP는 복잡도를 없애는 원칙이 아니라, 복잡도를 바깥(변경이 잦은 곳)으로 밀어내는 원칙이다.
> 정책을 안정적으로 유지하고 싶다면, 그 비용을 지불할 가치가 있는지 판단해야 한다.

## 정리

DIP는 단순히 인터페이스를 쓰자는 원칙이 아니다.
고수준 정책이 저수준 구현의 변경에 끌려다니지 않도록, 의존성의 방향을 바꾸는 설계 원칙이다.
이를 통해 변경 전파를 차단하고, 테스트와 확장에 유리한 구조를 만들 수 있다.
