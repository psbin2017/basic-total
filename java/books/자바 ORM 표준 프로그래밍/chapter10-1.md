# 🗂 10장 객체지향 쿼리 언어

- `JPQL`: Java Persistence Query Language
- `Criteria` 쿼리: `JPQL` 을 편하게 작성하도록 도와주는 API, 빌더 클래스 모음
- 네이티브 SQL: JPA 에서 `JPQL` 대신 직접 SQL 을 사용할 수 있다.
- `QueryDSL` : JPQL 을 편하게 작성하도록 도와주는 빌더 클래스 모음, 오픈소스 프레임워크.
- JDBC 직접 사용, Mybatis 같은 SQL 매퍼 프레임워크 사용

- - -

```java
@Query("select m from Member as m where m.age= :age")
List<Member> findByAge(@Param("age") Integer age);
```

- `JPQL` 은 객체지향 쿼리 언어다.
- `JPQL` 은 SQL 을 추상화해서 특정 데이터베이스 SQL 에 의존하지 않는다.
- `JPQL` 은 결국 SQL 로 변환된다.

## 기본 문법과 쿼리

```text
select_문 :: = 
    select_절
    from_절
    [where_절]
    [groupby_절]
    [having_절]
    [orderby_절]

update_문 :: = update_절 [where_절]
delete_문 :: = delete_절 [where_절]
```

- insert 문의 경우 `em.persist()` 가 있으니 없다.
- `JPQL` 에서 UPDATE, DELETE 문은 벌크 연산이다.

### SELECT

```java
SELECT m FROM Member AS m where m.username = 'Hello'
```

- 대소문자 구분: 엔티티와 속성은 대소문자를 구분한다.
- 엔티티 이름: 클래스 명이 아니라 엔티티 명이다. `@Entity(name="XXX")`
- 별칭은 필수: AS 는 생략할 수 있다.

#### TypeQuery, Query

- 반환 타입을 명확하게 지정할 수 있으면 `TypeQuery<T>` 객체
- 지정할 수 없다면 `Query` 객체

```java
TypeQuery<Member> tQuery = em.createQuery("SELECT m FROM Member m", Member.class);

Query query = em.createQuery("SELECT m.username, m.age FROM Member m");
List resultList = query.getResultList();

for (Object o : resultList) {
    Object[] result = (Object[]) o; // 결과가 둘 이상이면 Object[] 반환
    System.out.println("username = " + result[0]);
    System.out.println("age = " + result[1]);
}
```

#### 결과 조회

- `getResultList()`
- `getSingleResult()` : 결과가 정확히 하나일 때 사용하면 결과가 없으면 `NoResultException`, 있으면 `NonUniqueResultException` 이 발생한다.

- - -

## 파라미터 바인딩

이름 기준 파라미터 바인딩은 다음과 같다.

```java
TypeQuery<Member> tQuery = em.createQuery("SELECT m FROM Member m where m.username = :username", Member.class);

tQuery.setParameter("username", "User1");
```

`:username` 이 파라미터로 바인딩된다. 메소드 체인 방식으로 작성가능하다.

위치 기준 파라미터 바인딩은 다음과 같다.

```java
TypeQuery<Member> tQuery = em.createQuery("SELECT m FROM Member m where m.username = ?1", Member.class);

tQuery.setParameter(1, "User1");
```

위치 기준보다는 이름 기준 파라미터가 명확하다.

- - -

## 프로젝션

SELECT 절에 조회할 대상을 지정하는 것을 프로젝션이라 한다. 대상으로는 엔티티, 임베디드 타입, 스칼라 타입이 있다.

### 엔티티 프로젝션

```java
createQuery("SELECT m FROM Member m", Member.class)
createQuery("SELECT m.team FROM Member m", Team.class)
```

첫 번째는 회원, 두 번째는 회원과 연관된 팀을 조회하였다. 쉽게 생각하면 원하는 객체를 바로 조회한 것인데 컬럼 하나 하나 나열해서 조회해야하는 SQL 과 차이가 있다.

> 조회한 엔티티는 영속성 컨텍스트가 관리한다.

### 임베디드 타입 프로젝션

```java
createQuery("SELECT o.address FROM Order o", Address.class)
```

Address 는 임베디드 타입으로 조회의 시작점이 될 수 없다. 임베디드 타입은 엔티티 타입 아닌 값 타입이다.

> 조회한 임베디드 타입은 영속성 컨텍스트가 관리하지 않는다.

### 스칼라 타입 프로젝션

```java
createQuery("SELECT username FROM Member m", String.class)
createQuery("SELECT DISTINCT username FROM Member m", String.class)
createQuery("SELECT AVG(o.orderAmount) FROM Order o", Double.class)
```

숫자, 문자, 날짜와 같은 기본 데이터 타입들을 스칼라 타입이라 한다. 통계 쿼리도 주로 스칼라 타입으로 조회한다.

### 여러 값 조회

```java
List<Object[]> resultList = em.createQuery("SELECT m.username, m.age FROM Member m").getResultList();

for (Object[] row : resultList) {
    String username = (String) row[0];
    Integer age = (Integer) row[1];
}
```

스칼라 타입 뿐만 아니라 엔티티 타입도 여러 값을 함께 조회할 수 있다.

```java
List<Object[]> resultList = em.createQuery("SELECT o.member, o.product, o.orderAmount FROM Order o").getResultList();

for (Object[] row : resultList) {
    Member member = (String) row[0]; // 엔티티
    Product product = (Integer) row[1]; // 엔티티
    Integer orderAmount = (Integer) row[2]; // 스칼라
}
```

### NEW 명령어

```java
TypeQuery<UserDTO> query = em.createQuery("SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Order o", UserDTO.class);
```

1. 패키지 명을 포함한 전체 클래스 명을 입력해야 한다.
2. 순서와 타입이 일치하는 생성자가 필요하다.

반환 받을 클래스를 지정하여 JPQL 조회 결과로 넘겨준다. NEW 명령어를 사용한 클래스를 사용하여 지루한 객체 변환 작업을 줄여줄 수 있다.

- - -

## 페이징 API

```java
TypeQuery<Member> query = em.createQuery("SELECT m FROM Member m ORDER BY m.username DESC", Member.class);

query.setFirstResult(10); // 11 번째부터 시작하여
query.setMaxResults(20); // 20 건의 데이터를 조회
query.getResultList();
```

페이징 처리용 SQL 을 작성하는 일은 지루하고 반복적이다. 더 큰 문제는 데이터베이스마다 페이징 SQL 문법이 다르다는 점이다.

다른 페이징처리를 같은 API 로 처리할 수 있는 것은 데이터베이스 방언 덕분이다.

- - -

## 집합과 정렬

```sql
select
    COUNT(m), // 회원 수
    SUM(m.age), // 나이 합계
    AVG(m.age), // 나이 평균
    MAX(m.age), // 나이 최대
    MIN(m.age) // 나이 최소
from Member m
```

- COUNT: Long Type
- MAX, MIN: 문자, 숫자, 날짜 등에 사용
- AVG: 숫자 타입만 사용 가능, Double Type
- SUM: 숫자 타입만 사용 가능
  - 정수 합: Long Type
  - 소수 합: Double Type
  - BigInteger 합 BigInteger Type
  - BigDecimal 합 BigDecimal Type

### 통계 쿼리 주의사항

- NULL 은 통계에 대상이 되지 않는다.
- 값이 없는데 SUM, AVG, MAX, MIN 사용시 값은 NULL 이다. COUNT 는 0 이다.
- DISTINCT 를 집합 함수 안에 사용시 중복 값을 제거하고 집합을 구할 수 있다.
- DISTINCT 를 COUNT 에 사용시 임베디드 타입은 지원하지 않는다.

### GROUP BY, HAVING

```sql
select
    t.name,
    COUNT(m), // 회원 수
    SUM(m.age), // 나이 합계
    AVG(m.age), // 나이 평균
    MAX(m.age), // 나이 최대
    MIN(m.age) // 나이 최소
from Member m
LEFT JOIN m.team t
GROUP BY t.name
HAVING AVG(m.age) >= 10

groupby_절 ::= GROUPBY {단일값 경로 | 별칭}+
having_절 ::= HAVING 조건식
```

1. 팀 이름으로 그룹화
2. 평균 나이가 10살 이상 조회

통계 쿼리는 보통 전체 데이터를 기준으로 처리하므로 실시간으로는 부담이 있다. 따라서 통계 결과만 저장하는 테이블을 별도로 만들어 사용자가 적은 새벽에 통계 쿼리를 실행하여 결과를 보관하는 것이 바람직하다.

### ORDER BY

```sql
select
    t.name,
    COUNT(m.age) as cnt
from Member m
LEFT JOIN m.team t
GROUP BY t.name
ORDER BY cnt

orderby_절 ::= ORDER BY {상대필드 경로 | 결과 변수 [ASC | DESC]}+
```

1. ASC 오름차순 (기본값)
2. DESC 내림차순

- - -

## JPQL 조인

### INNER

```java
String teamName = "TeamA";
String query = "SELECT m FROM Member m INNER JOIN m.team t WHERE t.name = :teamName";

em.createQuery(query, Member.class).setParameter("teamName", teamName).getResultList();
```

실제 쿼리 결과는 다음과 같다.

```sql
SELECT
    // ...
FROM MEMBER M
INNER JOIN TEAM T
    ON M.TEAM_ID = T.ID
WHERE
    T.NAME = 'TeamA'
```

`m.team t` 는 회원이 가지고 있는 연관 필드로 팀과 조인한다는 의미이다. 그리고 조인한 Team 의 별칭은 t 이다.

```java
em.createQuery("SELECT m, t FROM Member m INNER JOIN m.team t");
```

조인한 두 개의 엔티티를 조회하려면 다음과 같으며 앞서 본 **여러 값 조회** 나 **NEW 명령어** 를 사용하여 결과를 처리하면 된다.

### OUTER

```java
String teamName = "TeamA";
String query = "SELECT m FROM Member m LEFT OUTER JOIN m.team t WHERE t.name = :teamName";

em.createQuery(query, Member.class).setParameter("teamName", teamName).getResultList();
```

실제 쿼리 결과는 다음과 같다.

```sql
SELECT
    // ...
FROM MEMBER M
LEFT OUTER JOIN TEAM T
    ON M.TEAM_ID = T.ID
WHERE
    T.NAME = 'TeamA'
```

### 컬렉션 조인

일대다 관계나 다대다 관계 처럼 컬렉션을 사용하는 곳에서 조인하는 것을 컬렉션 조인이라 한다.

```java
public class Order {
    // ...

    // 주문 → 회원
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;
}

public class Member {
    // ...

    // 회원 → 주문
    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}
```

1. [주문 → 회원]으로의 조인은 다대일 조인이면서 단일 값 연관 필드 (m.member) 를 사용한다.
2. [회원 → 주문]은 일대다 조인이면서 컬렉션 값 연관 필드 (m.orders) 를 사용한다.

```sql
SELECT m, o FROM Order o LEFT OUTER JOIN o.member m
SELECT m, o FROM Member m LEFT OUTER JOIN m.orders o
```

여기서 m LEFT OUTER JOIN m.orders 는 회원과 회원이 보유한 주문 목록을 컬렉션 값 연관 필드로 외부 조인 하였다.

> JOIN 절 대신 IN 을 사용할 수 있는데 이점이 없으니 그냥 JOIN 쓰자

### 세타 조인

WHERE 절을 사용하여 세타 조인을 할 수 있다. 내부 조인만 지원한다.

```sql
-- JPQL
SELECT count(m) FROM Member m, Team t WHERE m.username = t.name;

-- SQL
SELECT
    COUNT(M.ID)
FROM MEMBER M
CROSS JOIN TEAM T
WHERE
    M.USERNAME = T.NAME
```

> 전혀 관계 없는 엔티티도 조인할 수 있다.

#### JOIN ON

JPA 2.1 부터 조인시 ON 절을 지원한다.

1. ON 절 사용시 조인 대상을 필터링한 후에 조인 할 수 있다.
2. 내부 조인의 ON 절은 WHERE 절을 사용할 때와 결과가 같으므로 보통 ON 절은 외부 조인에서만 사용한다.

```sql
-- JPQL
SELECT m, t FROM Member m LEFT OUTER JOIN m.team t on t.name = 'A'

-- SQL
SELECT
    m.*,
    t.*
FROM Member m
LEFT JOIN Team t
    ON m.TEAM_ID = t.ID
    AND t.NAME = 'A'
```

조인 시점에 조인 대상을 필터링한다.

- - -

## 페치 조인

JPQL 성능 최적화를 위해 제공하는 기능이다.

`페치 조인 ::= [ LEFT [OUTER] | INNER ] JOIN FETCH 조인 경로`

**페치 조인은 객체 그래프를 유지할 때 사용하면 효과적이다.**  여러 테이블을 조인하여 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야한다면 페치 조인을 사용하느 것보다는 여러 테이블에서 필요한 필드들만 조회하여 DTO 로 반환하는 것이 더 효과적일 수 있다.

### 엔티티 페치 조인

```sql
-- JPQL
select m from Member m join fetch m.team

-- SQL
SELECT
    M.*,
    T.*
FROM MEMBER M
INNER JOIN TEAM T ON M.TEAM_ID=T.ID
```

회원과 팀을 함께 조인한다. 일반적인 `JPQL` 조인과 다르게 별칭이 없는데 **페치 조인은 별칭을 사용할 수 없다.**

> 하이버네이트의 경우 페치 조인에도 별칭을 허용한다.

**Member** 테이블

| `ID` (PK) | `NAME` | `TEAM_ID` (FK) |
| --- | --- | --- |
| 1 | 회원 1 | 1 |
| 2 | 회원 2 | 1 |
| 3 | 회원 3 | 2 |
| 4 | 회원 4 | null |

**Team** 테이블

| `ID` (PK) | `NAME` |
| --- | --- |
| 1 | 팀A |
| 2 | 팀B |
| 3 | 팀C |

**Fetch Join** 테이블

| `ID` (PK) | `NAME` | `TEAM_ID` (FK) | `ID` (PK) | `NAME` |
| --- | --- | --- | --- | --- |
| 1 | 회원 1 | 1 | 1 | 팀A |
| 2 | 회원 2 | 1 | 1 | 팀A |
| 3 | 회원 3 | 2 | 2 | 팀B |

> 회원과 팀을 지연 로딩으로 설정해도, 회원을 조회할 때 페치 조인을 사용하여 팀도 조회했으므로 연관된 팀 엔티티는 프록시가 아닌 실제 엔티티다.

지연로딩이 일어나지 않고, 프록시가 아닌 실제 엔티티로 회원 엔티티가 영속성 컨텍스트에 밀어나 준영속 상태가 되어도 연관된 팀을 조회할 수 있다.

### 컬렉션 페치 조인

```sql
-- JPQL
select t from Team t join fetch t.members where t.name = '팀A'

-- SQL
SELECT
    T.*,
    M.*
FROM TEAM T
INNER JOIN MEMBER M
    ON T.ID = M.TEAM_ID
```

이전 예제는 다대일 조인이었지만 이번에는 일대다 조인이다. `select t` 로 팀만 선택했는데 `T.*, M.*` 로 조회되었다.

**Team** 테이블

| `ID` (PK) | `NAME` |
| --- | --- |
| 1 | 팀A |
| 2 | 팀B |
| 3 | 팀C |

**Member** 테이블

| `ID` (PK) | `NAME` | `TEAM_ID` (FK) |
| --- | --- | --- |
| 1 | 회원 1 | 1 |
| 2 | 회원 2 | 1 |
| 3 | 회원 3 | 2 |
| 4 | 회원 4 | null |

**Fetch Join** 테이블

| `ID` (PK) | `NAME` | `ID` (PK) | `TEAM_ID` (FK) | `NAME` |
| --- | --- | --- | --- | --- |
| 1 | 팀A | 1 | 1 | 회원 1 |
| 1 | 팀A | 1 | 1 | 회원 2 |
| 2 | 팀B | 3 | 2 | 회원 3 |

TEAM 테이블에서는 팀A 값이 하나지만 MEMBER 테이블과 조인하면서 결과과 증가해 팀A 가 2 건 조회되었다.

> 일대다 조인은 결과가 증가할 수 있지만 일대일, 다대일 조인은 결과가 증가하지 않는다.

```java
String jpql = "select t from Team t inner join t.members";
List<Team> teams = em.createQuery(jpql, Team.class).getResultList();

for ( Team team : teams ) {

    for ( Member member : team.getMembers() ) {
        // 지연 로딩 발생 안함
    }
}
```

### 페치 조인과 DISTINCT

```sql
-- JPQL
select distinct t from Team t inner join t.members
```

일대다 조인시 중복된 결과가 발생할 수 있다.

```text
teamname = 팀A
→ username = 회원 1
→ username = 회원 2
teamname = 팀A
→ username = 회원 1
→ username = 회원 2
```

이 경우 distinct 절을 사용하여 팀 엔티티에 대한 중복을 제거할 수 있다.

```text
teamname = 팀A
→ username = 회원 1
→ username = 회원 2
```

### 페치 조인과 일반 조인의 차이

```sql
-- JPQL
-- 페치 조인 미사용
select t from Team t join t.members where t.name = '팀A'

-- SQL
SELECT
    T.*
FROM TEAM T
INNER JOIN MEMBER
    ON T.ID = M.TEAM_ID
WHERE
    T.NAME = '팀A'
```

페치 조인을 사용하지 않고 사용시 SELECT 절에서 회원을 전혀 조회하지 않는다.

> JPQL 은 결과를 반환시 연관관계까지 고려하지 않는다. 단지 SELECT 절에 지정한 엔티티만 조회한다.

또한 지연로딩 설정시 컬렉션 래퍼를 반환하고, 즉시로딩으로 설정시 회원 컬렉션을 즉시 로딩하기 위해 쿼리를 한번 더 실행한다.

반면 **페치 조인시 연관된 엔티티도 함께 조회한다.**

### 페치 조인의 특징과 한계

페치 조인은 SQL 한 번으로 연관된 엔티티들을 함께 조회할수 있어 SQL 호출 횟수를 줄여 성능을 최적화할 수 있다. 페치 조인은 글로벌 로딩 전략보다 우선한다. **글로벌 로딩 전략을 지연 로딩으로 설정해도 JPQL 페치 조인시 즉시 로딩으로 실행한다.**

> 글로벌 로딩 전략 = 지연 로딩 ... 최적화가 필요한 쿼리 = 페치 조인

#### 페치 조인은 별칭을 줄 수 없다

- SELECT, WHERE 절, 서브 쿼리에 대한 페치 조인 대상을 사용할 수 없다.
- 하이버네이트를 포함한 몇 구현체는 페치 조인에 별칭을 지원한다. 그러나 별칭을 잘못 사용하면 연관된 데이터 수가 달라져 데이터 무결성이 깨질 수 있다.

> 특히 2차 캐시와 사용시 조심해야한다. 연관된 데이터 수가 달라진 상태에서 2차 캐시에 저장시 다른 곳에서 조회시 연관된 데이터수가 달라지는 문제가 발생한다.

#### 둘 이상의 컬렉션을 페치할 수 없다

컬렉션 * 컬렉션의 카테시안 곱이 발생하기 때문에 둘 이상의 컬렉션을 페치할 수없다.

#### 컬렉션을 페치 조인시 페이징 API 를 사용할 수 없다

- 컬렉션(일대다)이 아닌 단일 값 연관 필드(일대일, 다대일)들은 페치 조인 사용시 페이징 API 를 사용 가능하다.
- 컬렉션으로 페치 조인하고 페이징 API 적용시 경고 로그를 뿜으면서 메모리에서 페이징을 처리한다.

> 데이터가 많으면 성능 이슈와 메모리 초과 예외가 발생한다.

- - -

## 경로 표현식

경로 표현식은 쉽게 이야기하면 점을 찍어 객체 그래프를 탐색하는 것이다. `m.username`, `m.team`, `m.orders`, `t.name` 이 모두 경로 표현식을 사용한 예이다.

### 용어 정리

- 상태 필드: 값을 저장하기 위한 필드 (필드 or 프로퍼티)
- 연관 필드: 연관관계를 위한 필드, 임베디드 타입 포함 (필드 or 프로퍼티)
  - 단일 값 연관 필드: `@ManyToOne`, `@OneToOne` 대상이 엔티티
  - 컬렉션 값 연관 필드: `@OneToMany`, `@ManyToMany` 대상이 컬렉션

```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    @Column(name = "name")
    private String username; // 상태 필드

    private Integer age; // 상태 필드

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private Team team; // 연관 필드: 단일 값 연관

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>(); // 연관 필드: 컬렉션 값 연관
}
```

- 상태 필드: m.username, m.age
- 단일 값 연관 필드: m.team
- 컬렉션 값 연관 필드: m.orders

### 경로 표현식과 특징

```sql
/**
 * 상태 필드 예제
 */
-- JPQL
select m.username, m.age from Member m

-- SQL
SELECT m.name, m.age FROM Member m

/**
 * 단일 값 예제
 */
-- JPQL
select m.team from Member m

-- SQL
SELECT m.* FROM Member m
INNER JOIN Team t
    ON m.member_id = t.member_id

/**
 * 컬렉션 값 예제
 */
-- JPQL
SELECT m.orders FROM Member M

-- SQL
SELECT m.* FROM Member m
INNER JOIN Order o
    ON m.member_id = o.member_id
```

- 상대 필드 경로: 경로 탐색의 끝으로 더 탐색이 불가능하다.
- 단일 값 연관 경로: **묵시적으로 내부조인이 일어난다.** 단일 값 연관 경론느 계속 객체 탐색할 수 있다.
- 컬렉션 값 연관 경로: **묵시적으로 내부 조인이 일어난다.** 더 탐색할 수 없다. 단, FROM 절에서 조인을 통해 별칭을 얻으면 별침으로 탐색할 수 있다.

**묵시적 내부 조인**이란 JPQL 에서 명시적으로 조인을 하지 않아도 결과에서는 반영된다.

```sql
select t.members from Team t -- 성공
select t.members.username from Team t -- 실패
select m.username from Team t join t.members m -- 성공 (별칭 획득)
```

컬렉션에서 경로 탐색은 허용되지 않는다. 단, 컬렉션에 별칭을 획득하면 경로 탐색이 가능하다.

#### 예제

```sql
-- JPQL
select o.member.team
from Order o
where o.product.name = 'productA' and o.address.city = 'JINJU'

-- SQL
SELECT t.*
FROM Orders o
INNER JOIN Member m ON o.member_id = m.member_id
INNER JOIN Team t ON t.team_id = m.team_id
INNER JOIN Product p ON o.product_id = p.product_id
WHERE
    p.name = 'productA'
    AND o.city = 'JINJU'
```

`o.member`, `o.member.team`, `o.product.name` 로 총 3 번 조인하였다. address 는 임베디드 타입이다.

#### 묵시적 조인 주의사항

- 항상 내부조인이다.
- 컬렉션은 경로 탐색의 끝이다. 경로 탐색을 한다면 명시적 조인으로 별칭을 주어야 한다.
- 경로 탐색은 주로 SELECT, WHERE 절에서 사용하지만 묵시전 조인으로 SQL 의 FROM 절에도 영향을 준다.

- - -

## 서브 쿼리

// TODO

- - -

## 조건식

- - -

## 다형성 쿼리

- - -

## 사용자 정의함수 호출

- - -

## 기타 정리

- - -

## 엔티티 직접 사용

- - -

## Named 쿼리: 정적 쿼리
