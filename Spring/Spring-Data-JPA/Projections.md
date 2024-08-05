# Projections

> Spring Data query methods usually return one or multiple instances of the aggregate root managed by the repository. However, it might sometimes be desirable to create projections based on certain attributes of those types. Spring Data allows modeling dedicated return types, to more selectively retrieve partial views of the managed aggregates. - [Spring Data JPA - Projections](https://docs.spring.io/spring-data/jpa/reference/repositories/projections.html)

간단하게 살펴보면 Spring Data의 쿼리 메서드는 보통 저장소에서 관리하는 집합체의 루트 인스턴스 하나 또는 여러 개를 반환한다고 한다.

```sql
SELECT *
FROM table_name;
```

위와 비슷하게 반환되는 것이다. 즉, 엔티티의 모든 필드를 가져오는 것이다.
Spring Data JPA에서는 `Projections`를 사용하여 엔티티의 일부 필드만 조회할 수 있다. 직접 사용해보면서 알아보자.

## 사용하는 엔티티

```java
@Entity
@Getter
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public User() {

    }
}
```

### 일반적인 조회 쿼리 확인

```java
@SpringBootTest
public class ProjectionTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    @DisplayName("findAll() 쿼리 확인")
    void findAll() {
        User user1 = new User("bang1", 21);
        User user2 = new User("bang2", 22);
        userRepository.saveAll(List.of(user1, user2));

        userRepository.findAll();
    }
}
```

```sql
Hibernate:
    select
        u1_0.id,
        u1_0.age,
        u1_0.name
    from
        user u1_0
```

실행된 쿼리를 보면 `id`, `age`, `name` 필드를 모두 조회하는 것을 확인할 수 있다. (JpaRepository를 상속받은 인터페이스를 사용했다.)

## Interface 기반 Projections (Closed Projections)

읽고 싶은 필드에 대한 접근자 메서드를 노출하는 인터페이스를 선언하는 방법이다. 여기서 중요한 점은 이 인터페이스에 정의된 필드들이 집합체의 루트에 있는 필드들과 정확히 일치해야 한다.

```java
public interface UserName {
    String getName(); // 필드명과 일치해야 한다.
}
```

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<UserName> findAllBy();
}
```

### 결과 확인

```java
@Test
@DisplayName("findAllBy() 쿼리 확인")
void findAllBy() {
    User user1 = new User("bang1", 21);
    User user2 = new User("bang2", 22);
    userRepository.saveAll(List.of(user1, user2));

    userRepository.findAllBy();
}
```

```sql
Hibernate:
    select
        u1_0.name
    from
        user u1_0
```

- `findAllBy()` 메서드를 사용하여 `name` 필드만 조회하는 것을 확인할 수 있다.

## Closed Projections

- 위에서 사용한 `Closed Projections`는 엔티티의 필드와 일치하는 인터페이스를 사용한다. `Spring Data`는 프로젝션 프록시를 지원하는 데 필요한 모든 속성을 알고 있으므로 쿼리 실행을 최적화할 수 있다고 한다.

## Open Projections

- 인터페이스의 접근자 메서드는 `@Value` 어노테이션을 사용하여 새로운 값을 계산하는 데도 사용할 수 있다.

```java
public interface NameAndAge {
    @Value("#{target.name + ' : ' + target.age}")
    String getNameAndAge();
}
```

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<NameAndAge> findAllBy();
}
```

### 결과 확인

```java
@Test
@DisplayName("Open Projection 쿼리 확인")
void openProjection() {
    User user1 = new User("bang1", 21);
    User user2 = new User("bang2", 22);
    userRepository.saveAll(List.of(user1, user2));

    var users = userRepository.findAllBy();

    for (NameAndAge user : users) {
        System.out.println("user name and age : " + user.getNameAndAge());
    }
}
```

```sql
Hibernate:
    select
        u1_0.id,
        u1_0.age,
        u1_0.name
    from
        user u1_0

-- print result
-- user name and age : bang1 : 21
-- user name and age : bang2 : 22
```

- `@Value` 어노테이션을 사용하여 `name`과 `age` 필드를 합쳐서 출력하는 것을 확인할 수 있다.
- 하지만 실행된 쿼리를 보면 모든 필드를 조회한 후 `@Value` 어노테이션을 사용하여 값을 계산한 것을 확인할 수 있다. 이는 필요한 필드만 조회하고 싶은 경우에는 원하는 바가 아닐 수 있다.
- 이를 개선하기 위해 Closed Projections과 Open Projections을 조합하여 사용할 수 있다.

### Closed Projections과 Open Projections 조합

```java
public interface NameAndAge {
    String getName();
    int getAge();

    default String getNameAndAge() {
        return getName().concat(" : ").concat(String.valueOf(getAge()));
    }
}
```

```sql
Hibernate:
    select
        u1_0.name,
        u1_0.age
    from
        user u1_0

-- print result
-- user name and age : bang1 : 21
-- user name and age : bang2 : 22
```

- 위와 같이 변경 후 테스트를 실행하면 `name`과 `age` 필드만 조회한 후 default 메서드를 사용하여 값을 계산한 것을 확인할 수 있다.

## Class 기반 Projections (DTOs)

> Another way of defining projections is by using value type DTOs (Data Transfer Objects) that hold properties for the fields that are supposed to be retrieved. These DTO types can be used in exactly the same way projection interfaces are used, except that no proxying happens and no nested projections can be applied. - [Spring Data JPA - Projections](https://docs.spring.io/spring-data/jpa/reference/repositories/projections.html)

- `DTO`를 사용하여 프로젝션을 정의하는 또 다른 방법이 있다. 이러한 `DTO` 유형은 프록시화되지 않고 중첩 프로젝션을 적용할 수 없다는 점을 제외하고 프로젝션 인터페이스가 사용되는 방식과 정확히 동일하게 사용할 수 있다고 한다.

```java
public record UserName(String name) {
}
```

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<UserName> findAllBy();
}
```

### 결과 확인

```java
@Test
@DisplayName("Class 기반 Projection 쿼리 확인")
void classBasedProjection() {
    User user1 = new User("bang1", 21);
    User user2 = new User("bang2", 22);
    userRepository.saveAll(List.of(user1, user2));

    userRepository.findAllBy();
}
```

```sql
Hibernate:
    select
        u1_0.name
    from
        user u1_0
```

- 인터페이스 기반 프로젝션에서 사용한 코드를 사용하면 `getName()`과 `getAge()`와 같은 메서드를 가진 프록시 객체를 생성한다. 이 프록시 객체의 메서드를 호출하면, 실제로는 원래 객체의 `name`과 `age` 속성을 가져와 반환한다. 이렇게 프록싱을 사용하면, 사용자는 필요한 속성만을 정의한 인터페이스를 통해 원래 객체를 사용하는 것처럼 특정 속성들만을 효율적으로 접근할 수 있다고 한다.
- 그러나 클래스 기반 프로젝션에서는 이러한 프록싱이 발생하지 않는다. 대신 사용자가 정의한 DTO 클래스의 인스턴스를 직접 생성하여 사용하게 됩니다. 이 경우 중첩된 프로젝션이 적용될 수 없다는 단점이 있지만, 대신 프록시를 생성하고 관리하는 overhead가 없으므로 성능상의 이점이 있을 수 있다고 한다.

## Dynamic Projections

- 지금까지는 프로젝션 타입을 반환 타입이나 컬렉션의 요소 타입으로 직접 지정해 사용했다. 그러나 호출 시점에 사용할 타입을 선택하고 싶다면, 아래와 같이 동적 프로젝션을 사용하여 쿼리 메서드를 작성하자.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    <T> List<T> findAllBy(Class<T> type);
}
```

```java
// 멤버 엔티티 원본을 그대로 반환한다.
List<User> members = userRepository.findAllBy(User.class);

// UserName 인터페이스에 정의된 필드만 가져온다. (name 필드만 가져온다.)
List<UserName> names = userRepository.findAllBy(UserName.class);
```

# 중첩된 구조인 경우?

- 공식문서에는 단일 엔티티에 대한 프로젝션만을 다루고 있지만, 대부분의 프로젝트에서는 엔티티 간의 관계를 맺고 있을 것이다. 여러 시도를 해보면서 확인해보자.

## 사용하는 엔티티

```java
@Entity
@Getter
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    private String content;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User author;

    public Post(String title, String content, User author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }

    public Post() {

    }
}

@Entity
@Getter
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private int age;

    @OneToMany(mappedBy = "author", cascade = CascadeType.PERSIST)
    private List<Post> posts = new ArrayList<>();

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public User() {

    }
}
```

- `Post` 엔티티는 `User` 엔티티와 `ManyToOne` 관계를 맺고 있다.
- `User` 엔티티는 `Post` 엔티티와 `OneToMany` 관계를 맺고 있다.

## 게시글의 제목과 작성자의 이름을 가져와보기.

```java
public interface NestedProjections {

    String getTitle();
    AuthorInfo getAuthor();

    interface AuthorInfo {
        String getName();
    }
}
```

```java
public interface PostRepository extends JpaRepository<Post, Long> {

    <T> List<T> findProjectionsBy(Class<T> tClass);
}
```

- Post 엔티티의 `title` 필드와 `author` 필드(User 엔티티)의 `name` 필드를 조회하는 프로젝션을 정의했다.
- Dynamic Projections를 사용하여 호출 시점에 사용할 타입을 선택하도록 구현했다.

### 결과 확인

```java
@Test
@DisplayName("중첩 프로젝션 사용해보기")
void nestedProjections() {
    User user1 = new User("bang1", 21);
    User user2 = new User("bang2", 22);
    User user3 = new User("bang3", 23);

    Post post1 = new Post("title1", "content1", user1);
    Post post2 = new Post("title2", "content2", user2);
    Post post3 = new Post("title3", "content3", user3);

    userRepository.saveAll(List.of(user1, user2, user3));
    postRepository.saveAll(List.of(post1, post2, post3));

    List<NestedProjections> result = postRepository.findProjectionsBy(NestedProjections.class);
    for (NestedProjections nestedProjections : result) {
        System.out.println("nestedProjections.getTitle() = " + nestedProjections.getTitle());
        System.out.println("nestedProjections.getAuthor().getName() = " + nestedProjections.getAuthor().getName());
    }
}
```

```sql
Hibernate:
    select
        p1_0.title,
        a1_0.id,
        a1_0.age,
        a1_0.name
    from
        post p1_0
    left join
        user a1_0
            on a1_0.id=p1_0.user_id

-- print result
-- nestedProjections.getTitle() = title1
-- nestedProjections.getAuthor().getName() = bang1
-- nestedProjections.getTitle() = title2
-- nestedProjections.getAuthor().getName() = bang2
-- nestedProjections.getTitle() = title3
-- nestedProjections.getAuthor().getName() = bang3
```

- `Post` 엔티티의 `title` 필드와 `author` 필드의 `name` 필드를 조회하는 것을 확인할 수 있다.
- 자동으로 `JOIN` 쿼리가 생성되어 `author` 필드의 `name` 필드를 조회하는 것을 확인할 수 있다.

## 정리

- `Projections`를 사용하여 엔티티의 일부 필드만 조회할 수 있다.
- `Dynamic Projections`는 호출 시점에 사용할 타입을 선택하고 싶을 때 사용한다.
- `Entity` 자체를 반환하는 것보다 필요한 필드만 조회하여 반환하는 것이 성능상 이점이 있다.
- 중첩된 구조인 경우에도 `Projections`를 사용하여 필요한 필드만 조회할 수 있다.

## 생각

필요하지 않는 컬럼의 데이터 크기가 큰 경우 `Projections`를 사용하여 필요한 필드만 조회하는 것이 성능상 이점이 있을 것 같다. 하지만 `Projections`을 통해 조회한 데이터는 영속성 컨텍스트에서 관리되지 않기 때문에 조회 기능에만 사용하는 것이 좋을 것 같다. 또한 `Projections`을 화면에 핏하게 맞춰서 사용할 때 주로 사용하게 될 것 같은데. 화면이 자주 변경되는 경우 유연성이 떨어지고 유지보수성이 낮아질 수 있을 것 같다. 실제 사용하게 된다면, 장단점과 함께 성능 테스트를 비교해보면서 사용해야 할 것 같다.

## 참고 자료

[Spring Data JPA - Projections](https://docs.spring.io/spring-data/jpa/reference/repositories/projections.html)
