# SQL의 정의와 기본 구문 이해 (DDL, DML, DCL)

SQL(Structured Query Language)은 중요하기 때문에 배워야 한다고 생각했지만, 지금까지도 SQL이 정확히 무엇인지 모르고 사용하고 있었던 것 같다.

SQL이 무엇인지 이해하고 기본 구문에 대해 학습하려고 한다.

## SQL의 역할과 역사적 배경

SQL(Structured Query Language)은 관계형 데이터베이스에서 데이터를 정의하고 조작하고 제어하기 위한 표준 언어다.
1970년대 IBM에서 [Edgar F. Codd](https://en.wikipedia.org/wiki/Edgar_F._Codd)가 정립한 관계형 데이터 모델(Relational Model) 개념을 기반으로 만들어졌다고 한다.

최초의 SQL은 IBM의 System R 프로젝트에서 개발된 `SEQUEL(Structured English Query Language)`이었고, 이후 ANSI(미국표준협회)와 ISO(국제표준화기구)에 의해 표준 SQL 언어로 채택되었다.

지금은 다양한 RDBMS에서 SQL을 기반으로 한 쿼리 언어를 사용하고 있으며, 각 RDBMS마다 약간의 차이가 있지만 기본적인 SQL 문법은 동일하다.

SQL은 사용자가 직접 데이터를 조작하지 않고도,
원하는 데이터를 정확하게 질의(query) 할 수 있도록 도와준다.
사용자는 무엇을 원한다는 `의도`만 명확히 표현하면, 나머지는 DBMS가 알아서 처리하는 구조라고 할 수 있다.

```sql
-- "나이가 20살보다 많은 학생의 이름이 필요하다." 라고 말하는 것 같다.
SELECT name FROM students WHERE age > 20;
```

## SQL을 사용하는 이유: 선언형 언어 vs 절차형 언어

SQL은 `선언형 언어(declarative language)`로 분류된다.
`어떻게(How)` 보다는 `무엇을(What)` 에 초점을 맞춰서 작성된다.

```
선언형 언어 SQL: 결과만 기술하며, 내부 처리 방식은 DBMS가 결정한다.
절차형 언어 Java, Python 등: 순서대로 로직을 명시적으로 기술해야 한다.
```

선언형 언어의 장점은 간결하고 읽기 쉬우며, 최적화는 DBMS가 알아서 수행하기 때문에
개발자가 복잡한 내부 구현을 몰라도 원하는 결과를 얻을 수 있다는 것이다.

```sql
-- "평균 점수가 필요해" 라는 의도만 전달하면, DBMS가 알아서 실행 계획을 수립하고 처리해준다.
SELECT AVG(score) FROM tests;
```

## SQL이 데이터베이스와 상호작용하는 방식

사용자가 SQL 문을 작성하여 실행하면, DBMS는 내부적으로 여러 단계를 거쳐 해당 쿼리를 처리하고 결과를 반환한다.

```sql
-- 예시: 가격이 10,000원 초과인 상품을 조회하는 SQL 문
SELECT * FROM products WHERE price > 10000;
```

사용자가 작성한 SQL 문은 단순히 실행되는 것이 아니라, DBMS 내부에서 여러 단계를 거쳐 처리된다.

```
파싱 → 쿼리 변환 및 재작성 → 최적화 → 실행 → 결과 반환
```

### 1. SQL 문을 파싱하여 문법을 분석한다.

먼저 `파서(Parser)`가 문법을 분석한다. 쿼리의 구조를 분석하여 `파스 트리(Parse Tree)`를 생성하며, 문법 오류 여부, 권한 체크 등을 수행한다.

### 2. 쿼리 변환 및 재작성(Query Rewrite)

일부 DBMS는 파싱 후 논리적인 쿼리 재작성 단계를 수핸한다. 예를 들어, `뷰(View)`가 포함된 쿼리를 기반 테이블 쿼리로 변환하거나, `규칙 기반 최적화(Rule-based optimization)` 등을 적용할 수 있다.

### 3. 최적화(Optimization)

`옵티마이저(Optimizer)`는 다양한 실행 경로 중에서 가장 효율적인 경로(최적의 실행 계획)를 선택한다. 통계 정보, 인덱스 존재 여부, 조인 순서 등을 고려하여 `비용 기반으로 평가`하며, 결과적으로 `실행 계획(Execution Plan)`이 생성된다.

### 4. 실행(Execution)

생성된 `실행 계획`을 기반으로, `실행기(Executor)`가 실제로 데이터를 읽고 조작한다. 디스크 I/O, 메모리 버퍼 사용, 인덱스 탐색 등 물리적인 작업을 수행한다.

### 5. 결과 반환

실행 결과는 메모리나 네트워크 버퍼를 통해 클라이언트에게 반환한다. 사용자는 최종 결과만을 확인하게 되며, 내부적인 처리 과정을 몰라도 SQL 한 줄로 원하는 데이터를 얻을 수 있다.

SQL은 내부적으로 파서(Parser), 옵티마이저(Optimizer), 실행기(Executor) 같은 컴포넌트를 통해 처리되고,
사용자는 복잡한 내부 구현을 몰라도 원하는 결과를 얻을 수 있다.

## SQL 유형

SQL은 다양한 명령어를 제공하여 데이터베이스와 상호작용할 수 있도록 한다.
모든 명령은 `DBMS(Database Management System)`에 전달되고,
DBMS는 해당 명령을 처리하여 결과를 반환하거나, 데이터베이스를 수정하는 등의 작업을 수행한다.

SQL은 크게 네 가지 카테고리로 나눌 수 있다.

- DDL (Data Definition Language)
- DML (Data Manipulation Language)
- DCL (Data Control Language)
- TCL (Transaction Control Language)

### DDL(Data Definition Language, 데이터 정의어)

DDL은 데이터베이스의 구조를 정의하는 명령어이다. 테이블을 생성, 수정, 삭제 작업을 수행한다.

- `CREATE`: 새로운 테이블을 생성한다.
- `ALTER`: 기존 테이블의 구조를 변경한다.
- `DROP`: 테이블을 삭제한다.
- `TRUNCATE`: 테이블의 모든 데이터를 삭제하지만, 테이블 구조는 유지한다.
- `RENAME`: 테이블의 이름을 변경한다.

### DML(Data Manipulation Language, 데이터 조작어)

DML은 데이터베이스에 저장된 데이터를 조작하는 명령어이다. 데이터의 삽입, 수정, 삭제 작업을 수행한다.

- `SELECT`: 데이터베이스에서 데이터를 조회한다.
- `INSERT`: 새로운 데이터를 테이블에 삽입한다.
- `UPDATE`: 기존 데이터를 수정한다.
- `DELETE`: 데이터를 삭제한다.

### DCL(Data Control Language, 데이터 제어어)

DCL은 데이터베이스에 대한 접근 권한을 제어하는 명령어이다. 사용자에게 권한을 부여하거나 회수하는 작업을 수행한다.

- `GRANT`: 사용자에게 특정 권한을 부여한다.
- `REVOKE`: 사용자에게 부여된 권한을 회수한다.

### TCL(Transaction Control Language, 트랜잭션 제어어)

TCL은 데이터베이스 트랜잭션을 제어하는 명령어이다. 별도의 TCL로 분류하지 않고, DCL에 포함시키기도 한다.

- `COMMIT`: 트랜잭션을 확정한다. 데이터베이스에 변경 사항을 영구적으로 반영한다.
- `ROLLBACK`: 트랜잭션을 취소한다. 데이터베이스에 변경 사항을 반영하지 않는다.
- `SAVEPOINT`: 트랜잭션 내에서 특정 시점을 저장한다. 이후 ROLLBACK을 통해 해당 시점으로 되돌릴 수 있다.

## 예시로 살펴보는 SQL 구문

크루 정보를 데이터베이스에 저장하기 위해 테이블을 생성하는 것부터 시작해보자.

### 1. `crew` 테이블 생성하기

우선 크루 정보를 저장할 `crew` 테이블을 만들어보자.

```sql
-- DDL
CREATE TABLE crew (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nickname VARCHAR(50) NOT NULL,
    part VARCHAR(50) NOT NULL
);
```

이 테이블은 세 가지 속성을 가진다.

- `id`: 크루의 고유 식별자. 정수형이며 자동 증가한다.
- `nickname`: 크루의 닉네임. NULL을 허용하지 않는다.
- `part`: 크루의 파트. 문자열로 저장하며 NULL을 허용하지 않는다.

### 테이블 이름 실수 예시

테이블 이름을 실수로 `creww`로 작성했다면 어떻게 수정할 수 있을까?

```sql
-- 잘못된 테이블 이름으로 생성
CREATE TABLE creww (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nickname VARCHAR(50) NOT NULL,
    part VARCHAR(50) NOT NULL
);

-- 테이블 이름 수정
RENAME TABLE creww TO crew;
```

이처럼 실수했을 때는 `RENAME TABLE` 구문으로 테이블 이름을 수정할 수 있다.

### 2. 데이터 삽입 및 조회

합격한 크루 정보를 `crew` 테이블에 삽입해보자.

```sql
-- DML
INSERT INTO crew (nickname, part)
VALUES ('벨로', 'BE'),
       ('도기', 'BE'),
       ('레몬', 'FE'),
       ('오이', 'AN');
```

```sql
-- DML
SELECT * FROM crew;
```

| id  | nickname | part |
| --- | -------- | ---- |
| 1   | 벨로     | BE   |
| 2   | 도기     | BE   |
| 3   | 레몬     | FE   |
| 4   | 오이     | AN   |

### 3. 데이터 수정 및 삭제

레몬의 파트가 FE가 아니라 BE라고 하자. `UPDATE`로 수정할 수 있다.

```sql
UPDATE crew
SET part = 'BE'
WHERE nickname = '레몬';
```

수정하자마자 레몬이 퇴소한다고 말했다.
우리는 `DELETE`로 제거할 수 있다.

```sql
DELETE FROM crew
WHERE nickname = '레몬';
```

### 4. 정규화가 필요한 순간

기획이 바뀌어, BE/FE/AN이 아니라 `BACKEND`, `FRONTEND`, `ANDROID`로 표기하자고 기획이 바뀌었다.

```sql
-- DML
UPDATE crew
SET part = 'BACKEND'
WHERE part = 'BE';

UPDATE crew
SET part = 'FRONTEND'
WHERE part = 'FE';
-- ...
```

이처럼 `crew` 테이블 안에 반복되는 문자열을 전부 수정해야 한다.
유지보수가 어려운 설계라고 볼 수 있다. 이럴 땐 테이블을 분리하자.

### 5. `part` 테이블 분리 (정규화)

`part`를 별도의 테이블로 분리하고, `crew` 테이블은 `part_id`로 참조하게 만든다.

```sql
-- DDL
CREATE TABLE part (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL
);
```

```sql
-- DML
INSERT INTO part (name)
VALUES ('BE'), ('FE'), ('AN');
```

이제 `crew` 테이블의 `part` 컬럼을 `part_id`로 바꾸자. (외래 키 연결 준비)

```sql
-- DDL
ALTER TABLE crew
CHANGE part part_id INT NOT NULL;
```

```sql
-- DDL
ALTER TABLE crew
ADD CONSTRAINT fk_part
FOREIGN KEY (part_id) REFERENCES part(id);
```

`CHANGE` 구문은 컬럼 이름과 타입을 모두 바꿀 수 있다.
여기서는 `VARCHAR` → `INT`로 바꾸며, `part_id`를 외래 키로 연결하기 위한 준비이다.

### 6. 정규화의 이점

이제 `part` 이름이 바뀌더라도 `part` 테이블만 수정하면 된다.

```sql
-- DML
UPDATE part
SET name = 'BACKEND'
WHERE name = 'BE';

UPDATE part
SET name = 'FRONTEND'
WHERE name = 'FE';

UPDATE part
SET name = 'ANDROID'
WHERE name = 'AN';
```

### 7. JOIN으로 데이터 조회

정규화 이후에는 조인(JOIN)으로 데이터를 함께 조회해야 한다.

```sql
-- BE → BACKEND 파트에 소속된 크루 조회
SELECT c.nickname, p.name AS part
FROM crew c
JOIN part p ON c.part_id = p.id
WHERE p.name = 'BACKEND';
```

| nickname | part    |
| -------- | ------- |
| 벨로     | BACKEND |
| 도기     | BACKEND |

### 8. 외래 키의 삭제 옵션

`part`가 삭제되면 `crew` 테이블의 `part_id`는 어떻게 될까?

옵션을 지정하지 않으면 오류가 발생하지만, 다음과 같이 설정하면 유연하게 처리할 수 있다.

```sql
-- 파트가 삭제되면 crew의 part_id를 NULL로 설정
ALTER TABLE crew
ADD CONSTRAINT fk_part
FOREIGN KEY (part_id) REFERENCES part(id)
ON DELETE SET NULL;
```

혹은 아래처럼 연결된 크루도 함께 삭제할 수도 있다.

```sql
-- 파트가 삭제되면 관련 크루도 삭제
ON DELETE CASCADE;
```

이렇게 정규화를 통해 구조를 개선하면, 유지보수는 쉬워지고 의미도 더 명확해진다.

## 마무리

SQL을 잘 모르더라도 기본적인 작업 내용은 이해할 수 있다는 것이 꽤나 큰 장점인 것 같다.
또한, SQL은 선언형 언어이기 때문에, 복잡한 내부 구현을 몰라도 원하는 결과를 얻을 수 있다는 점이 매력적이다.
