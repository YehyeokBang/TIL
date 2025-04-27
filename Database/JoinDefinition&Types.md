## JOIN의 정의와 종류 이해

[이전 글](Database/SQLDefinition&BasicSyntax.md)에서 크루와 파트를 저장하는 테이블을 만들거나 수정해보고, 조인을 이용하여 크루와 파트의 정보를 조회해보기도 했다. 이번 에는 조인(JOIN)의 개념과 다양한 종류에 대해 정리하고, 각각의 특징과 사용 방법을 살펴본다.

- [JOIN 이란?](#join-이란)
- [INNER JOIN](#inner-join)
- [OUTER JOIN](#outer-join)
  - [LEFT JOIN](#left-outer-join)
  - [RIGHT JOIN](#right-outer-join)
  - [FULL OUTER JOIN](#full-outer-join)
- [CROSS JOIN](#cross-join)
- [EQUI JOIN과 NON-EQUI JOIN](#equi-join과-non-equi-join)
- [USING](#using)
- [NATURAL JOIN](#natural-join)
- [SELF JOIN](#self-join)
- [마무리](#마무리)

## JOIN 이란?

SQL에서 조인은 두 개 이상의 테이블에 있는 데이터를 결합하여 하나의 결과 집합으로 만드는 연산이다. 관계형 데이터베이스에서는 정보가 여러 테이블에 분산되어 저장되므로, 필요한 정보를 얻기 위해 조인을 활용해 연결해야 한다.

조인 방식은 크게 `암시적 조인 (Implicit Join)`과 `명시적 조인 (Explicit Join)`으로 구분된다. 초기의 SQL에서는 암시적 조인 방식을 사용했지만, 가독성 문제로 명시적 조인 방식이 권장되고 있다.

### 암시적 조인 (Implicit Join)

`FROM` 절에 테이블들을 쉼표로 나열하고, `WHERE` 절에서 조인 조건을 지정하는 방식이다. 초기 SQL 방식이지만, 가독성이 떨어지고 복잡한 쿼리에서 실수하기 쉽다.

```sql
SELECT u.name, t.team_name
FROM users u, teams t
WHERE u.team_id = t.id;
```

### 명시적 조인 (Explicit Join)

`FROM` 절에서 `JOIN` 키워드를 사용하고 `ON` 절에 조인 조건을 작성하는 방식이다. 조인 조건과 필터 조건이 분리되어 가독성이 높고 실수 위험을 줄인다.

```sql
SELECT u.name, t.team_name
FROM users u
JOIN teams t
  ON u.team_id = t.id;
```

## INNER JOIN

`INNER JOIN`은 `ON` 절의 조건을 만족하는 두 테이블의 튜플만 결합하여 결과를 생성한다. 조건에 맞지 않는 튜플은 결과에서 제외된다.

```sql
SELECT u.name, t.team_name
FROM users u
INNER JOIN teams t -- INNER 키워드 생략 가능
    ON u.team_id = t.id;
```

**예시**

| id  | name | team_id |     | id  | team_name |
| --- | ---- | ------- | --- | --- | --------- |
| 1   | 벨로 | 1       |     | 1   | Team A    |
| 2   | 도기 | 2       |     | 2   | Team B    |
| 3   | 머랭 | NULL    |     | 3   | Team C    |
| 4   | 가콩 | 3       |     | 4   | Team D    |

**결과**

```
 name | team_name
------+-----------
  벨로 | Team A
  도기 | Team B
  가콩 | Team C
```

> ### 참고: NULL과 Three-Valued Logic
>
> SQL에서 NULL과 비교 연산을 하게 되면 그 결과는 UNKNOWN 이다.  
> UNKNOWN은 'TRUE 일수도 있고 FALSE 일수도 있다' 라는 의미이다.  
> three-valued logic : 비교/논리 연산의 결과로 TRUE, FALSE, UNKNOWN을 가진다

## OUTER JOIN

OUTER JOIN은 조인 조건을 만족하지 않아도 한쪽 테이블의 모든 레코드를 결과에 포함시킨다. 방향에 따라 LEFT, RIGHT, FULL로 나뉜다.

### LEFT OUTER JOIN

왼쪽 테이블의 모든 튜플을 포함하고, 오른쪽에서 매칭되는 튜플이 있으면 결합한다.

```sql
SELECT u.name, t.team_name
FROM users u
LEFT JOIN teams t
  ON u.team_id = t.id;
```

**결과**

```
 name | team_name
------+-----------
  벨로 | Team A
  도기 | Team B
  머랭 | NULL
  가콩 | Team C
```

### RIGHT OUTER JOIN

오른쪽 테이블의 모든 튜플을 포함하고, 왼쪽에서 매칭되는 튜플이 있으면 결합한다.

```sql
SELECT u.name, t.team_name
FROM users u
RIGHT JOIN teams t
  ON u.team_id = t.id;
```

**결과**

```
 name | team_name
------+-----------
  벨로 | Team A
  도기 | Team B
  가콩 | Team C
  NULL | Team D
```

### FULL OUTER JOIN

양쪽 테이블의 모든 튜플을 포함하며, 매칭되지 않는 쪽의 컬럼은 `NULL`로 채워진다.

> MySQL은 `FULL OUTER JOIN`을 지원하지 않으므로, `LEFT JOIN`과 `RIGHT JOIN` 결과를 결합해야 한다.

#### UNION 사용 (중복 자동 제거)

```sql
(SELECT u.name, t.team_name
 FROM users u
 LEFT JOIN  teams t ON u.team_id = t.id)
UNION
(SELECT u.name, t.team_name
 FROM users u
 RIGHT JOIN teams t ON u.team_id = t.id);
```

- UNION은 두 결과 집합을 합치면서 중복 행을 자동으로 제거된다.
- 매칭된(ON 조건 만족) 레코드는 LEFT/RIGHT 양쪽에 모두 등장하지만, 최종 결과에는 한 번만 나타난다.

#### UNION ALL + DISTINCT 사용 (중복 제어)

```sql
SELECT DISTINCT name, team_name
FROM (
  SELECT u.name, t.team_name
  FROM users u
  LEFT JOIN  teams t ON u.team_id = t.id
  UNION ALL
  SELECT u.name, t.team_name
  FROM users u
  RIGHT JOIN teams t ON u.team_id = t.id
) AS full_outer;
```

- UNION ALL은 중복을 그대로 보존된다.
- 내부 서브쿼리에서 매칭된 레코드가 두 번 등장하므로, 외부 SELECT DISTINCT로 중복을 제거해줘야 한다.

## CROSS JOIN

두 테이블의 **카티션 프로덕트**(모든 조합)를 생성한다. 한쪽 3행, 다른쪽 4행일 때 12행이 결과로 나온다.

```sql
-- 명시적
SELECT u.name, t.team_name
FROM users u
CROSS JOIN teams t;

-- 암시적
SELECT u.name, t.team_name
FROM users u, teams t;
```

> MySQL에서 `INNER JOIN`은 `ON` 또는 `USING`이 필수이므로, 조건 없이 쓰면 문법 오류가 발생한다.

## EQUI JOIN과 NON-EQUI JOIN

- `EQUI JOIN`: `=` 연산자를 사용하는 조인. 일반적인 이너/아우터 조인은 모두 EQUI JOIN이다.

  ```sql
  SELECT u.name, t.team_name
  FROM users u
  INNER JOIN teams t
    ON u.team_id = t.id;
  ```

- `NON‑EQUI JOIN`: `>`, `<`, `>=`, `<=`, `BETWEEN`, `<>` 등 비등가 연산자를 사용하는 조인이다.

## USING

동일 이름의 컬럼을 기준으로 EQUI JOIN을 수행하며, 해당 컬럼을 결과에 한 번만 표시한다.
즉, 이름이 동일한 경우에 ON 절 대신 사용할 수 있는 구문이다.

```sql
SELECT *
FROM users u
INNER JOIN teams t
  USING (team_id);
```

```sql
-- 결과 컬럼: team_id, u.id AS id, u.name, t.team_name
```

결과 집합에는 해당 컬럼이 한 번만 나타나며, 다른 컬럼들보다 앞에 위치한다.

> 동명이 컬럼이 많다면, 별칭(alias)을 통해 구분하는 것이 좋다.

## NATURAL JOIN

이름이 같은 모든 컬럼을 자동으로 EQUI JOIN하고, 조인 키를 결과에 한 번만 표시한다.  
편리하지만, 의도치 않은 컬럼이 조인 키로 사용될 수 있어 주의해야 한다.

```sql
-- 가정: teams.id를 team_id로 변경했다고 가정한다.
SELECT *
FROM users
NATURAL INNER JOIN teams;
```

```sql
-- 명시적 조인과 동등하다.
SELECT *
FROM users
INNER JOIN teams
  ON users.team_id = teams.team_id;
```

아무튼 현재 예시 테이블 구조처럼 컬럼 이름이 다를 경우, 내추럴 조인은 적합하지 않고 명시적 이너 조인을 사용하는 것이 좋다.

나아가 테이블 구조가 변경될 가능성이 있기 때문에, NATURAL JOIN은 예기치 않은 오류를 초래할 수 있다. 따라서 명시적으로 조인 조건을 작성하는 것이 안전하다.

## SELF JOIN

셀프 조인은 하나의 테이블을 자신과 다시 조인하는 방법이다.
주로 계층 구조 데이터(예: 조직 구조에서 상하 관계를 표현할 때)를 다룰 때 유용하다고 한다.

셀프 조인을 수행하려면, `동일한 테이블을 FROM 절에 두 번 명시하고, 각각 다른 별칭(alias) 을 부여`한 뒤 ON 절로 조인 조건을 정의한다.

### 팀 내 다른 사용자 조회

```sql
SELECT u1.name AS UserName, u2.name AS TeammateName
FROM users u1
JOIN users u2
  ON u1.team_id = u2.team_id
WHERE u1.id <> u2.id;
```

### 매니저‑팀원 관계 조회 (가정)

```sql
SELECT u.name AS UserName, m.name AS ManagerName
FROM users u
JOIN users m
  ON u.manager_id = m.id;
```

> 조인 조건에 자주 사용되는 컬럼에 인덱스를 생성하면 성능이 향상된다.

### 셀프 조인은 이럴 때 유용하다 (예시)

- 조직도와 같은 계층 구조 데이터 조회
- 동일 테이블 내 레코드 간의 관계 분석
- 순위나 그룹 내 비교 연산 수행

## 마무리

그동안은 단순히 데이터가 중복되면 테이블을 분리하고, 외래 키를 기준으로 조인하는 방식에만 익숙했다.
하지만 이번에 쉬운코드 님의 영상을 보면서 다양한 조인 방식과 조건을 학습하게 되었다.

이번 학습을 통해 조인의 기본 원리부터 다양한 활용법까지 체계적으로 정리할 수 있었다. 앞으로 복잡한 관계형 데이터베이스를 설계하거나 최적화할 때, 조인 방식을 상황에 맞게 선택하는 것이 얼마나 중요한지 다시 한번 깨달았다

앞으로는 조인에 사용하는 컬럼에 인덱스를 적절히 설정하고, 필요한 데이터만 조회하는 쿼리를 작성하는 연습을 하려고 한다.

### 참고 자료

- [쉬운코드: join의 의미와 여러 종류의 join들을 쉽게 정리해서 설명합니다! (YouTube)](https://youtu.be/E-khvKjjVv4)
