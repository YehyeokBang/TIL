# SELECT 절의 처리 순서

SQL은 내가 작성한 문법 순서대로 실행될 것이라 막연히 생각했었다.
사실 어쩌면 순서에 대해 깊게 생각해보지 않았던 것 같다.
하지만 곰곰이 생각해보면 SELECT가 가장 먼저 실행될 수는 없다는 걸 알 수 있다.
궁금증이 생겨서 SQL이 실제로 어떤 순서로 쿼리를 처리하는지 학습하려고 한다.

- [왜 순서를 알아야 할까?](#왜-순서를-알아야-할까)
- [예시로 확인하기](#예시로-확인하기)
  - [1. FROM + 2. JOIN / ON](#1-from--2-join--on)
  - [3. WHERE](#3-where)
  - [4. GROUP BY](#4-group-by)
  - [5. HAVING](#5-having)
  - [6. SELECT + 7. DISTINCT](#6-select--7-distinct)
  - [8. ORDER BY](#8-order-by)
  - [9. LIMIT](#9-limit)
- [실행 순서를 이해하면 쿼리 작성이 수월해진다.](#실행-순서를-이해하면-쿼리-작성이-수월해진다)
- [마무리](#마무리)

## SELECT 절의 처리 순서?

> ### 작성 순서와 처리 순서는 다르다.
>
> 우리가 작성하는 SQL은 사람이 읽기 쉬운 형태고,  
> 실제로 DB는 처리하기 좋은 순서로 실행한다.

SQL은 일반적으로 `SELECT → FROM → WHERE` 순으로 작성하지만, DB 엔진이 실제로 처리하는 순서는 다르다.

이것을 `SELECT 절의 처리 순서` 또는 `Logical Query Processing Order`라고 한다.

```
1. FROM: 테이블 또는 뷰를 가져온다.
2. JOIN / ON: 조인 조건에 맞는 행을 연결한다.
3. WHERE: 조건에 맞는 행을 필터링한다.
4. GROUP BY: 데이터를 그룹화한다.
5. HAVING: 그룹화된 결과에 조건을 건다.
6. SELECT: 최종으로 보여줄 컬럼을 선택한다.
7. DISTINCT: 중복 제거
8. ORDER BY: 정렬
9. LIMIT: 결과 개수 제한
```

> ### 왜 순서를 알아야 할까?
>
> 예시로 `SELECT COUNT(*)` 쿼리에서 `WHERE`과 `HAVING`을 헷갈리면 잘못된 결과를 도출할 수 있다고 한다. 또한, `SELECT`에서 사용하는 `별칭(alias)`은 `WHERE`에서 사용할 수 없는데, 이는 `SELECT`가 `WHERE`보다 나중에 처리되기 때문이라고 한다. 즉, 순서를 이해하고 있다면, 쿼리문 작성 또는 예측이 더 수월해질 것이다.

## 예시로 확인하기

```sql
-- 테이블 구조
crew(id, nickname, part_id) -- 크루
part(id, name) -- 파트
mission(id, name, deadline) -- 미션
mission_record(crew_id, mission_id, completed) -- 미션 기록
```

```sql
-- 최대한 다양한 키워드를 위해 구성된 예시이다.
-- 미션 마감 기한이 지난 미션 중,
-- 모든 크루가 완료한 미션의 파트 목록을 완료한 파트 이름 기준으로 정렬해서 3개만 조회
SELECT DISTINCT p.name AS part_name
FROM part p
JOIN crew c ON p.id = c.part_id
JOIN mission_record mr ON c.id = mr.crew_id
JOIN mission m ON mr.mission_id = m.id
WHERE m.deadline < CURRENT_DATE
GROUP BY p.name, mr.mission_id
HAVING COUNT(DISTINCT c.id) = (SELECT COUNT(*)
                               FROM crew
                               WHERE part_id = p.id)
ORDER BY p.name
LIMIT 3;
```

### 1. FROM + 2. JOIN / ON

```sql
FROM part p
JOIN crew c ON p.id = c.part_id
JOIN mission_record mr ON c.id = mr.crew_id
JOIN mission m ON mr.mission_id = m.id
```

`FROM` 절에서 `part` 테이블을 기준으로 `JOIN`을 진행한다.
이때, `ON` 절에서 조인 조건을 설정한다.
`part`, `crew`, `mission_record`, `mission` 테이블을 조인한다.

### 3. WHERE

```sql
WHERE m.deadline < CURRENT_DATE
```

`WHERE` 절에서 `mission` 테이블의 `deadline`이 현재 날짜보다 이전인 행을 필터링한다.
마감 기한이 지난 미션만 남게 될 것이다.

### 4. GROUP BY

```sql
GROUP BY p.name, mr.mission_id
```

`GROUP BY` 절에서 `p.name`, `mr.mission_id`를 통해 파트와 미션 조합으로 그룹을 만든다.

### 5. HAVING

```sql
HAVING COUNT(DISTINCT c.id) = (SELECT COUNT(*)
                               FROM crew
                               WHERE part_id = p.id)
```

`HAVING` 절에서는 각 파트의 미션에 대해 해당 파트 소속 크루 모두가 완료한 경우만 남긴다.
왼쪽의 `COUNT(DISTINCT c.id)`는 실제로 미션을 완료한 크루 수이며,
오른쪽의 서브쿼리는 해당 파트에 속한 전체 크루 수다.

두 수가 같다는 것은 그 파트의 모든 크루가 해당 미션을 완료했다는 의미다.

### 6. SELECT + 7. DISTINCT

```sql
SELECT DISTINCT p.name AS part_name
```

`SELECT` 절에서는 `part` 테이블의 `name`을 선택한다.
이때 `DISTINCT`를 함께 사용하여 중복된 파트 이름을 제거한다.
또한 `AS`를 통해 결과 컬럼명을 `part_name`으로 지정한다.

### 8. ORDER BY

```sql
ORDER BY p.name
```

`ORDER BY` 절에서 `part` 테이블의 `name`을 기준으로 정렬한다. 즉, 파트 이름을 기준으로 오름차순으로 정렬된다.

### 9. LIMIT

```sql
LIMIT 3;
```

`LIMIT` 절에서 결과 개수를 제한한다.
3개만 조회하도록 설정되어 있다.

즉, 정렬된 결과 중 상위 3개만 남게 된다.

## 실행 순서를 이해하면 쿼리 작성이 수월해진다.

실행 순서를 이해하면 쿼리 작성이 수월해진다. 예시로 알아보자.

### WHERE에서 별칭(alias)을 사용할 수 없다.

```sql
SELECT c.id AS crew_id
FROM crew c
WHERE crew_id = 1 -- 오류 발생
```

WHERE 절은 SELECT 절보다 먼저 실행된다.
SELECT에서 정의한 crew_id라는 별칭을 사용할 수 없다.
이런 경우에는 별칭 대신 실제 컬럼명을 사용하거나, 공통된 조건이 필요하다면 WITH절을 사용한 서브쿼리로 분리하는 것이 낫다.

### 집계 전에는 WHERE, 집계 후에는 HAVING

```sql
-- 모든 크루가 완료한 미션만 조회하고 싶은데...
-- 잘못된 예시
SELECT mission_id
FROM mission_record
WHERE COUNT(*) > 3 -- 오류 발생

-- 올바른 예시
SELECT mission_id
FROM mission_record
GROUP BY mission_id
HAVING COUNT(*) > 3
```

WHERE 절은 그룹화되기 전에 실행된다. 따라서 집계 함수(COUNT, SUM, 등)를 사용할 수 없다. 이런 조건은 HAVING 절에서 써야 한다.

### GROUP BY 없이 집계 함수를 다른 컬럼과 함께 사용하면 오류가 발생한다.

```sql
SELECT c.part_id, COUNT(*) -- 오류 발생
FROM crew c;
```

전체 집계는 단독으로는 가능하지만, 컬럼과 함께 쓰려면 반드시 `GROUP BY`가 필요하다.

```sql
-- 모든 크루의 파트별 수를 조회
SELECT c.part_id, COUNT(*)
FROM crew c
GROUP BY c.part_id;
```

### ORDER BY는 SELECT 이후라 별칭 사용 가능하다.

```sql
SELECT c.id AS crew_id
FROM crew c
ORDER BY crew_id; -- 별칭 사용 가능
```

`ORDER BY` 절은 `SELECT` 절 이후에 실행되므로, 별칭을 사용할 수 있다. 즉, `ORDER BY` 절에서 `crew_id`라는 별칭을 사용할 수 있다.

## 마무리

논리적 실행 순서는 단순한 암기가 아닌 것 같다.
쿼리를 제대로 이해하고, 오류를 줄이며, 성능까지 고려할 수 있는 중요한 개념이다.  
이제 쿼리문을 볼 때, 눈앞에 보이는 순서 뒤에 숨겨진 처리 흐름도 함께 그리는 연습을 하면 좋을 것 같다.

## 참고 자료

- [SQL Order of Execution: Understanding How Queries Run: Allan Ouko](https://www.datacamp.com/tutorial/sql-order-of-execution)
