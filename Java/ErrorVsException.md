# Java Error & Exception

자바를 사용하다 보면 프로그램이 예상치 못하게 멈추거나 오류가 발생하는 경우가 있다.
이러한 오류들을 어떻게 이해하고 처리해야 하는지 명확히 알고 싶어서 이번에는 자바의 에러와 예외 체계에 대해 확실히 정리해보려고 한다.

- [프로그램 오류의 종류](#프로그램-오류의-종류)
- [Error vs Exception](#error-vs-exception)
- [예외 클래스 구조](#예외-클래스-구조)
- [Checked vs Unchecked Exception](#checked-vs-unchecked-exception)
- [예외 처리 전략](#예외-처리-전략)
- [마무리](#마무리)

## 프로그램 오류의 종류

자바로 프로그래밍할 때 마주하는 오류는 크게 컴파일 타임과 런타임으로 나눌 수 있다.

### 컴파일 타임 오류 (Compile-time Error)

컴파일 단계에서 발생하는 오류로, IDE나 컴파일러가 즉시 알려주기 때문에 비교적 쉽게 수정할 수 있다.

- 세미콜론 누락
- 타입 불일치
- 존재하지 않는 메서드 호출
- 문법 오류

### 런타임 문제

프로그램 실행 중에 드러나는 문제들이고, 두 가지로 나눌 수 있다.

#### 논리 오류 (Logic Error)

컴파일도 되고 실행도 되지만, 프로그래머가 의도한 대로 동작하지 않는 경우다.
자바는 이를 감지하거나 예외로 던지지 않는다. 
개발자가 테스트나 디버깅을 통해 직접 찾아내야 한다.

```java
// 할인가를 계산하려 했지만 로직이 잘못됨
int discountPrice = originalPrice / discountRate; // 나눗셈이 아니라 곱셈이어야 함
```

#### Throwable 계층의 오류

자바가 런타임에 발생하는 특정 문제들을 객체로 모델링한 것이다.
이것이 바로 `Error`와 `Exception`이며, 모두 `Throwable` 클래스를 상속받는다.

```
Object
 └── Throwable
      ├── Error
      └── Exception
```

이 Throwable 계층의 오류들은 자바가 감지하고 던져주기 때문에, 개발자가 이를 잡아서(catch) 처리하거나 전파할 수 있다.

## Error vs Exception

자바에서는 런타임에 발생할 수 있는 오류를 두 가지로 구분한다.

```java
Object
 └── Throwable
      ├── Error
      └── Exception
```

모든 오류와 예외는 [`Throwable`](https://docs.oracle.com/javase/8/docs/api/java/lang/Throwable.html) 클래스를 상속받는다.
`Throwable`은 오류나 예외에 대한 메시지와 스택 트레이스 정보를 담고 있으며, [`Error`](https://docs.oracle.com/javase/8/docs/api/java/lang/Error.html)와 [`Exception`](https://docs.oracle.com/javase/8/docs/api/java/lang/Exception.html)은 이를 상속받아 구현된다.

### Error

`Error`는 JVM 수준에서 발생하는 심각한 오류를 의미한다.
이러한 오류는 시스템 레벨에서 발생하기 때문에 개발자가 미리 예측하거나 처리할 수 없다.

**대표적인 Error의 예시**
- `OutOfMemoryError`: JVM의 메모리가 부족할 때 발생한다.
- `StackOverflowError`: 재귀 호출이 너무 깊어져 스택이 넘칠 때 발생한다.
- `NoClassDefFoundError`: 클래스 파일을 찾을 수 없을 때 발생한다.

일반적으로 애플리케이션 레벨에서 복구를 기대하지 않는다. 보통 프로세스를 재시작하거나 환경을 수정해야 한다.

> ### Error도 catch할 수 있을까?
>
> 기술적으로 `Error`는 `Throwable`의 하위 클래스이므로 `catch (Error e)`나 `catch (Throwable t)`로 잡을 수 있다. 
> 하지만 권장되지 않는다.
> 
> 예를 들어 `StackOverflowError`가 발생한 시점에는 이미 스택 메모리가 바닥난 상태이므로, catch 블록 내부의 코드를 실행할 여유조차 없을 가능성이 높다. 즉, 복구 시도 자체가 시스템을 더 불안정하게 만들 수 있다.

### Exception

`Exception`은 개발자가 코드로 처리 가능한 오류를 의미하며, 개발자가 처리 전략을 설계할 수 있다.

**대표적인 Exception의 예시**
- `IOException` : 파일 입출력 중 발생하는 예외다.
- `SQLException` : 데이터베이스 작업 중 발생하는 예외다.
- `NullPointerException` : null 객체에 접근할 때 발생하는 예외다.

`Exception`은 `try-catch`로 잡아서 처리할 수 있으며, 필요에 따라 예외를 상위로 전파할 수도 있다.

### 핵심 차이

| 구분            | Error                                  | Exception                      |
| ------------- | -------------------------------------- | ------------------------------ |
| **심각도**       | 매우 심각 (시스템 레벨)                         | 비교적 경미 (애플리케이션 레벨)             |
| **복구 가능성**    | 애플리케이션 레벨 복구를 기대하지 않음                                    | 가능                             |
| **발생 원인**     | JVM, 시스템 리소스 문제                        | 프로그래밍 로직, 외부 리소스 문제            |
| **처리 방법**     | 개발자가 제어 불가                             | try-catch 등으로 대응 가능            |
| **예시**        | OutOfMemoryError, StackOverflowError | IOException, NullPointerException |

## 예외 클래스 구조

자바의 모든 예외는 다음과 같은 계층 구조를 따른다.

```
Object
 └── Throwable
      ├── Error
      │    ├── OutOfMemoryError
      │    ├── StackOverflowError
      │    └── ...
      └── Exception
            ├── IOException
            ├── SQLException
            ├── RuntimeException
            │    ├── NullPointerException
            │    ├── IllegalArgumentException
            │    ├── ArrayIndexOutOfBoundsException
            │    └── ...
            └── ...
```

이 구조에서 가장 중요한 분기점은 [`RuntimeException`](https://docs.oracle.com/javase/8/docs/api/java/lang/RuntimeException.html)이다.
`Exception`을 상속받는 예외 중에서 `RuntimeException`을 상속받는지 여부에 따라 예외를 `Checked Exception`과 `Unchecked Exception`으로 구분한다.

## Checked vs Unchecked Exception

> ### 핵심 차이
>
> 복구 가능성이 아니라 “컴파일러가 호출자에게 처리 책임을 강제하는가”의 차이이다.

### Checked Exception

> ### 의도
>
> 호출자가 반드시 인지하고 대응 전략을 고민하도록 만들기 위한 것이다.

`RuntimeException`을 상속받지 않는다.

```java
public class CheckedExceptionExample {

    public static void main(String[] args) {
        // 컴파일 에러 발생 - 예외 처리가 강제됨
        // readFile(); // Unhandled exception: IOException
        
        // 방법 1: try-catch로 처리
        try {
            readFile();
        } catch (IOException e) {
            System.out.println("파일을 읽을 수 없습니다: " + e.getMessage());
        }
    }
    
    // 방법 2: throws로 예외를 상위로 전파
    public static void readFile() throws IOException {
        FileReader file = new FileReader("test.txt");
        // 파일 읽기 로직
    }
}
```

Checked Exception은 반드시 throws 혹은 try-catch로 처리해야 한다.
주로 I/O, 네트워크, DB 등 외부 자원과 관련된 예외에서 사용된다.

호출자가 합리적으로 복구 전략을 선택할 수 있다면 Checked Exception이 적합하다. 예를 들어, 파일이 없을 때 사용자에게 알리고 다른 파일을 선택하도록 유도하는 등의 전략이 가능하다.

### Unchecked Exception

> ### 의도
>
> 정상적인 흐름이 아닌 경우 굳이 모든 호출자에게 강제하지 않아도 된다는 것이다.

`RuntimeException`을 상속받는 예외다.

```java
public class UncheckedExceptionExample {

    public static void main(String[] args) {
        // 컴파일은 정상적으로 되지만, 런타임에 예외 발생
        String str = null;
        System.out.println(str.length()); // NullPointerException 발생
        
        int[] arr = new int[5];
        System.out.println(arr[10]); // ArrayIndexOutOfBoundsException 발생
    }
}
```

Unchecked Exception은 주로 프로그래밍 오류로 인해 발생한다.
예를 들어, null 체크를 하지 않았거나, 배열의 범위를 벗어나는 등의 경우다.
이러한 예외는 코드를 수정하여 예방할 수 있기 때문에, 컴파일러가 강제하지 않는다.

현대 서버 애플리케이션에서는 Unchecked Exception을 비즈니스 로직에서 발생하는 예외로 많이 사용한다. `@Transactional`이 적용된 메서드에서 Unchecked Exception이 발생하면 트랜잭션이 롤백되도록 설계되어 있고, 글로벌 예외 처리기를 통해 일관된 방식으로 예외를 처리할 수 있기 때문이다.

### 비교 정리

| 구분        | Checked Exception                  | Unchecked Exception                            |
| --------- | ---------------------------------- | ---------------------------------------------- |
| **상속 구조**  | RuntimeException을 상속받지 않음          | RuntimeException을 상속받음                         |
| **처리 강제**  | 반드시 처리해야 함 (컴파일 에러)               | 처리 선택 가능                                       |
| **확인 시점**  | 컴파일 시점                             | 런타임 시점                                         |
| **발생 원인**  | 외부 요인 (파일, 네트워크, DB 등)            | 프로그래밍 오류 또는 도메인 정책 위반                                       |
| **예방 방법**  | 예외 처리 코드 작성                        | 코드 수정으로 예방 가능                                  |
| **예시**     | IOException, FileNotFoundException | NullPointerException, IllegalArgumentException |

## 예외 처리 전략

예외를 처리할 때는 단순히 예외를 잡는 것뿐만 아니라, 시스템의 안정성과 유지보수성을 고려한 전략이 필요하다.

### 예외를 무시하지 말자

```java
// 나쁜 예 - 예외를 삼킴
try {
    processPayment(order);
} catch (PaymentException e) {
    // 아무것도 하지 않음 - 결제 실패를 숨김
}

// 좋은 예
try {
    processPayment(order);
} catch (PaymentException e) {
    logger.error("결제 실패: orderId={}", order.getId(), e);
    throw new OrderProcessException("결제 처리 중 오류가 발생했습니다", e);
}
```

catch 블록에서 예외를 잡고 아무런 조치를 취하지 않는 것은 좋지 않다.
최소한 로그를 남기거나, 사용자에게 알리는 등의 조치를 취해야 한다.

### 구체적인 예외를 잡고 의미 있는 메시지를 포함하자

가능한 한 구체적인 예외 클래스를 catch하는 것이 좋다.
`catch (Exception e)`처럼 너무 포괄적으로 잡는 것은 디버깅을 어렵게 만들 수 있다.

예외를 상위로 전파할 때는, 예외 객체에 의미 있는 메시지를 포함시키는 것이 좋다.
이렇게 하면 나중에 로그를 분석할 때 어떤 상황에서 예외가 발생했는지 쉽게 파악할 수 있다.

```java
// 나쁜 예 - 원본 예외 정보 손실
try {
    userRepository.findById(userId);
} catch (SQLException e) {
    throw new UserNotFoundException("사용자를 찾을 수 없습니다");
    // SQLException의 스택 트레이스가 사라짐!
}

// 좋은 예 - Exception Chaining
try {
    userRepository.findById(userId);
} catch (SQLException e) {
    throw new UserNotFoundException("사용자를 찾을 수 없습니다: " + userId, e);
    // 원본 SQLException을 cause로 포함
}
```

### 자원 정리는 finally 블록에서

파일, 네트워크 연결 등과 같은 자원을 사용하는 경우, 예외가 발생하더라도 자원이 제대로 정리되도록 `finally` 블록을 활용하는 것이 좋다.
Java 7 이상에서는 `try-with-resources` 구문을 사용하면 자동으로 자원을 닫아주기 때문에 더욱 편리하다.

```java
// 예전 방식 - finally 블록
FileReader fr = null;
try {
    fr = new FileReader("test.txt");
    // 파일 처리
} catch (IOException e) {
    // 예외 처리
} finally {
    if (fr != null) {
        try {
            fr.close(); // 여기서도 예외 처리 필요
        } catch (IOException e) {
            // ...
        }
    }
}

// try-with-resources - 자동으로 close() 호출
try (FileReader fr = new FileReader("test.txt")) {
    // 파일 처리
} catch (IOException e) {
    // 예외 처리
}
```

## 마무리

자바의 Error와 Exception 체계를 학습하면서, 단순히 문법을 이해하는 것을 넘어 설계 관점에서 예외를 바라보는 것이 중요하다는 것을 깨달았다. 

좋은 예외 처리는 단순히 에러를 잡는 것이 아니라, 시스템의 안정성과 유지보수성을 높이는 핵심 설계 요소다.
예외 처리를 통해 사용자에게 명확한 피드백을 제공하고, 디버깅을 쉽게 만들며, 시스템의 복구 가능성을 높일 수 있다.

앞으로 예외를 설계할 때는 "이 예외가 발생했을 때 누가, 어떻게 처리해야 하는가?"라는 고민과 함께 적절한 처리 전략을 세우는 것을 목표로 삼아야겠다.

나아가 Spring Framework에서는 어떻게 예외를 다루는지도 함께 공부해보면 좋을 것 같다.

## 참고

- [Oracle Docs - The Java Tutorials - Exceptions](https://docs.oracle.com/javase/tutorial/essential/exceptions/)
- [Oracle Docs - Java SE API - java.lang.Throwable](https://docs.oracle.com/javase/8/docs/api/java/lang/Throwable.html)