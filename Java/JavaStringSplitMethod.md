# String의 split() 메서드

String 클래스에서 제공하는 `split()` 메서드는 문자열을 특정 구분자로 나누어 배열로 반환한다. 이 메서드를 사용하다가 발생한 논리적인 문제를 기록하려고 한다.

## 문제 상황

```text
게임에 참여할 사람의 이름을 입력하세요.(쉼표 기준으로 분리)
pobi, jason
```

위와 같은 요구사항을 해결한다고 가정하면, String 클래스에서 제공하는 `split()` 메서드를 사용하려고 할 것이다.

나도 간단하게 `split()` 메서드를 사용하여 문제를 해결하려고 했다.

## 1. 구분만 한다.

```java
public class InputView {

    private static final Scanner scanner = new Scanner(System.in);

    public List<String> readPlayerNames() {
        System.out.println("게임에 참여할 사람의 이름을 입력하세요.(쉼표 기준으로 분리)");
        return splitWithComma(scanner.nextLine());
    }

    private List<String> splitWithComma(String input) {
        return Arrays.stream(input.split(","))
                .toList();
    }
}
```

가독성을 위해 `splitWithComma()` 라는 메서드를 작성했다. Scanner로 입력받은 문자열을 `쉼표(,)`를 기준으로 문자열을 나누려고 했다.
그러나 실행 결과는 예상과 달랐다.

```text
게임에 참여할 사람의 이름을 입력하세요.(쉼표 기준으로 분리)
pobi, jason

실제로 반환된 배열 출력 결과: [pobi,  jason]
```

`split()` 메서드를 사용하면 `쉼표(,)`를 기준으로 문자열을 나누기만 할 뿐, 공백을 제거하지 않는다. 따라서 `jason` 앞에 공백이 포함되어 있다.

### 해결 방법

`,`로 구분된 문자열들을 `trim()` 메서드를 사용하여 공백을 제거한 후, 리스트로 반환하면 된다.

```java
// trim() 사용하기
private List<String> splitWithComma(String input) {
    return Arrays.stream(input.split(","))
            .map(String::trim)
            .toList();
}
```

이렇게 하면 공백이 제거된 결과를 얻을 수 있다.

또는 `split()` 메서드의 인자로 `\\s*,\\s*`를 사용하여 공백을 제거할 수도 있다.

```java
// 정규식을 사용하기
private List<String> splitWithComma(String input) {
    return Arrays.stream(input.split("\\s*,\\s*"))
            .toList();
}
```

> `\\s*`: 0개 이상의 공백 문자  
> `,`: 쉼표  
> `\\s*`: 0개 이상의 공백 문자

이렇게 하면 `split()` 메서드를 사용하면서 공백을 제거할 수 있다.

두 방법 모두 원하는 결과를 얻을 수 있지만, 성능 측면에서는 `trim()` 메서드를 사용하는 것이 정규표현식을 사용하는 것보다 더 효율적이라고 한다.

```text
게임에 참여할 사람의 이름을 입력하세요.(쉼표 기준으로 분리)
pobi,                  jason

실제로 반환된 배열 출력 결과: [pobi, jason]
```

공백이 몇개가 들어가더라도 제거되는 것을 확인할 수 있다.

## 2. 구분자 이후에 아무것도 없는 경우

```text
게임에 참여할 사람의 이름을 입력하세요.(쉼표 기준으로 분리)
,,,

실제로 반환된 배열 출력 결과: []
```

위와 같이 입력한 경우 `split()` 메서드의 반환 배열은 아무것도 없는 배열이다. 이런 결과를 쉽게 설명하면 `split()` 메서드는 구분자 뒤에 아무것도 없는 문자열은 무시한다고 할 수 있다.

![splitDefault](images/splitDefault.png)

위는 구분자만 전달한 경우에 호출되는 `split()` 메서드이다. 구분자만 전달한 경우 limit은 0이 전달되고, withDelimiters는 false로 전달된다.

메서드를 쭉 따라가면 아래와 같은 로직을 볼 수 있다.

![splitWhenLimitZero](images/splitWhenLimitZero.png)

limit이 0인 경우, 구분자 뒤에 아무것도 없는 문자열(trailing empty strings)은 결과에서 제외된다.

예를 들어, `,,pobi,,,`와 같은 문자열을 `split(",")`로 나누면 결과는 `[, , pobi]`가 된다.

즉, `,,,`와 같은 문자열은 모두 해당 반복문에서 버려진다고 할 수 있다.

이러한 동작은 많은 경우에 유용할 수 있겠지만, 나의 사용 사례에서는 문제가 될 수 있다. 사용자가 `,`를 입력했다는 것은 게임에 참여할 사람의 이름을 입력하고 구분했다고 간주해야 하는데, 이러한 정보가 split() 내부에서 무시되는 것은 논리적으로 맞지 않다는 관점이다.

무시되는 것이 아니라 차라리 이름은 공백일 수 없다고 예외를 발생시키는 것이 더 좋은 방법이라고 생각한다.

### 해결 방법

limit을 -1로 전달하면 구분자 사이에 빈 문자열도 포함하여 반환한다.

```java
private List<String> splitWithComma(String input) {
    return Arrays.stream(input.split(",", -1))
            .map(String::trim)
            .toList();
}

// 이후 이름을 검증하는 로직을 추가할 수 있다.
```

이제 사용자가 4명의 플레이어를 의도했지만 이름을 비워둔 것으로 정확하게 해석할 수 있다. 이후에 추가적인 검증 로직을 적용하여 빈 이름에 대한 처리를 할 수 있다.

```text
게임에 참여할 사람의 이름을 입력하세요.(쉼표 기준으로 분리)
,,,

실제로 반환된 배열 출력 결과: [, , , ] (size: 4)
```

사용자는 4명의 플레이어 이름을 공백으로 입력했다고 간주할 수 있다. 더욱 자연스러운 결과를 얻을 수 있게 된다.

## limit의 의미

split() 메서드의 limit 매개변수는 문자열 분할 결과의 최대 개수를 제한한다. limit의 값에 따라 다음과 같은 동작을 한다.

`limit > 0`: 최대 limit개의 문자열로 분할한다. 마지막 부분 문자열에는 나머지 모든 내용이 포함된다.

```java
"a,b,c,d".split(",", 2) // 결과: ["a", "b,c,d"]
```

`limit = 0(기본값)`: 가능한 많이 분할하고, 뒤에 따라오는 빈 문자열은 버린다.

```java
"a,b,c,,".split(",") // 결과: ["a", "b", "c"]
```

`limit < 0`: 가능한 많이 분할하고, 모든 빈 문자열을 유지한다.

```java
"a,b,c,".split(",", -1) // 결과: ["a", "b", "c", ""]
```

## withDelimiters의 의미

split() 메서드의 withDelimiters 매개변수는 구분자를 결과 배열에 포함할지 여부를 결정한다.

`withDelimiters = true`: 구분자를 결과 배열에 포함한다.

```java
"a,b,c".split(",", true) // 결과: ["a", ",", "b", ",", "c"]
```

`withDelimiters = false(기본값)`: 구분자를 결과 배열에 포함하지 않는다.

```java
"a,b,c".split(",") // 결과: ["a", "b", "c"]
```

## 정리

`split()` 메서드를 사용할 때, limit과 withDelimiters 매개변수를 사용하여 원하는 결과를 얻을 수 있다. 또한, 공백을 제거하거나 빈 문자열을 포함하도록 설정하여 논리적인 문제를 해결할 수 있다.

자바에서 기본으로 제공하니 별 생각 없이 사용했었는데, 이렇게 내부적으로 어떻게 동작하는지 알게 되어서 좋았다. 앞으로도 항상 의심하고 검증하는 습관을 가지도록 해야겠다.
