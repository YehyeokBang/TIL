# Overloading & Overriding

누군가 나에게 오버로딩과 오버라이딩에 대해 설명해줄 수 있냐고 물어본다면, 당당하게 "물론이죠" 라고 대답하고 싶다. 그렇기 위해 이번에는 오버로딩과 오버라이딩에 대해 확실히 이해해보려고 한다.

## Overloading(오버로딩)

오버로딩은 같은 이름을 가진 메서드를 여러 개 정의하는 것이다. 이때 메소드의 시그니처(매개변수의 개수, 매개변수의 타입)가 달라야 한다.
즉, 같은 이름의 메서드라도 매개변수의 개수나 타입이 다르면 다른 메서드로 인식한다.

```java
class Person {
    void eat() {
        System.out.println("아무거나 먹는다.");
    }

    void eat(String food) {
        System.out.println(food + "을(를) 먹는다.");
    }

    void eat(String food, int count) {
        System.out.println(count + "개의 " + food + "을(를) 먹는다.");
    }
}

public class Overloading {
    public static void main(String[] args) {
        Person person = new Person();
        person.eat();
        person.eat("볶음밥");
        person.eat("김밥", 3);
    }
}

// 결과
// 아무거나 먹는다.
// 볶음밥을(를) 먹는다.
// 3개의 김밥을(를) 먹는다.
```

위 코드에서 `Person` 클래스에 `eat()` 메서드를 세 번 정의했다. 이때 메서드의 시그니처가 다르기 때문에 오버로딩이 가능한 것이다.
오버로딩은 메서드의 이름을 동일하게 유지하면서 메서드의 기능을 확장할 수 있기 때문에 메서드의 이름을 직관적으로 지을 수 있다. 이로 인해 코드의 가독성이 높아진다.

```java
// 오버로딩이 없었다면?
class Person {
    void eat() {
        System.out.println("아무거나 먹는다.");
    }

    void eatFood(String food) {
        System.out.println(food + "을(를) 먹는다.");
    }

    void eatFoodCount(String food, int count) {
        System.out.println(count + "개의 " + food + "을(를) 먹는다.");
    }
}
```

만약 오버로딩이 불가능하다면 위 코드처럼 메서드의 이름을 다르게 지어야 하기 때문에 메서드의 이름이 길어지고 복잡해질 수밖에 없다.

### 오버로딩 정리

- 같은 이름의 메서드를 여러 개 정의하는 것
- 메서드의 시그니처(매개변수의 개수, 매개변수의 타입)가 달라야 한다.
- 확장성을 가지면서 메서드의 이름을 직관적으로 지을 수 있어 가독성이 높아진다.
- 단, 반환 타입은 시그니처에 포함되지 않기 때문에 반환 타입만 다른 경우 오버로딩이 불가능하다.

## Overriding(오버라이딩)

오버라이딩은 상위 클래스의 메서드를 하위 클래스에서 재정의하는 것이다.
즉, 상위 클래스의 메서드와 동일한 시그니처를 가진 메서드를 하위 클래스에서 정의하는 것이다.

```java
class Person {
    String name;

    void introduce() {
        System.out.println("안녕하세요, 저는 " + name + "입니다.");
    }
}

class Student extends Person {
    String school;

    @Override
    void introduce() {
        System.out.println("안녕하세요, 저는 " + school + "에 다니는 " + name + "입니다.");
    }
}

class Teacher extends Person {
    String subject;

    @Override
    void introduce() {
        System.out.println("안녕하세요, 저는 " + subject + "과목을 가르치는 " + name + "입니다.");
    }
}

public class Overriding {
    public static void main(String[] args) {
        Student student = new Student();
        student.name = "홍길동";
        student.school = "서울대학교";
        student.introduce();

        Teacher teacher = new Teacher();
        teacher.name = "이순신";
        teacher.subject = "수학";
        teacher.introduce();
    }
}

// 결과
// 안녕하세요, 저는 서울대학교에 다니는 홍길동입니다.
// 안녕하세요, 저는 수학과목을 가르치는 이순신입니다.
```

위 코드에서 `Person` 클래스의 `introduce()` 메서드를 `Student` 클래스와 `Teacher` 클래스에서 오버라이딩했다. 이때 메서드의 시그니처가 동일하기 때문에 오버라이딩이 가능한 것이다.
오버라이딩을 통해 상위 클래스의 메서드를 하위 클래스에서 재정의할 수 있기 때문에 다형성을 구현할 수 있다.

```java
// 다형성 예시
public class Overriding {
    public static void main(String[] args) {
        Student student1 = new Student();
        student.name = "홍길동";
        student.school = "서울대학교";

        Student student2 = new Student();
        student.name = "고길동";
        student.school = "연세대학교";

        Teacher teacher = new Teacher();
        teacher.name = "이순신";
        teacher.subject = "수학";

        List<Person> persons = Arrays.asList(student1, student2, teacher);
        for (Person person : persons) {
            person.introduce();
        }
    }
}

// 결과
// 안녕하세요, 저는 서울대학교에 다니는 홍길동입니다.
// 안녕하세요, 저는 연세대학교에 다니는 고길동입니다.
// 안녕하세요, 저는 수학과목을 가르치는 이순신입니다.
```

위 코드에서 `Student` 클래스와 `Teacher` 클래스는 `Person` 클래스를 상속받기 때문에 `Person` 타입의 리스트에 `Student` 객체와 `Teacher` 객체를 모두 담을 수 있으면서 `introduce()` 메서드를 호출할 수 있다. 이때 `Student` 클래스와 `Teacher` 클래스에서 오버라이딩한 `introduce()` 메서드가 호출된다. 이렇게 하나의 메서드가 여러 클래스에서 다양하게 동작할 수 있기 때문에 코드의 가독성이 높아진다.

### 오버라이딩 정리

- 상위 클래스의 메서드를 하위 클래스에서 재정의하는 것
- 상위 클래스의 메서드와 동일한 시그니처를 가진 메서드를 하위 클래스에서 정의하는 것
- 다형성을 구현할 수 있다.
- 오버라이딩된 메서드는 상위 클래스의 메서드를 가리게 되므로 상위 클래스의 메서드는 숨겨진다.

## 마무리

오버로딩과 오버라이딩은 자바에서 가장 기본적이면서도 중요한 개념이다. 오버로딩은 같은 이름의 메서드를 여러 개 정의하는 것으로 메서드의 이름을 직관적으로 지을 수 있어 가독성이 높아진다. 오버라이딩은 상위 클래스의 메서드를 하위 클래스에서 재정의하는 것으로 다형성을 구현할 수 있다. 이러한 오버로딩과 오버라이딩을 잘 이해하고 활용하면 코드의 가독성을 높일 수 있고 다형성을 구현할 수 있다. 이제 누가 오버로딩과 오버라이딩에 대해 설명해달라고 하면 당당하게 설명할 수 있을 것 같다.
