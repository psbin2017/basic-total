# 🗂 10장 객체지향 쿼리 언어

- `JPQL`: Java Persistence Query Language
- `Criteria` 쿼리: `JPQL` 을 편하게 작성하도록 도와주는 API, 빌더 클래스 모음
- 네이티브 SQL: JPA 에서 `JPQL` 대신 직접 SQL 을 사용할 수 있다.
- `QueryDSL` : JPQL 을 편하게 작성하도록 도와주는 빌더 클래스 모음, 오픈소스 프레임워크.
- JDBC 직접 사용, Mybatis 같은 SQL 매퍼 프레임워크 사용

## `Criteria` 쿼리

- 컴파일 시점에 오류를 발견 가능하다.
- IDE 를 사용하면 코드 자동완성 지원한다.
- 동적 쿼리를 작성하기 편하다.

```java
public List<Board> findByContent(String content) {
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<Board> query = cb.createQuery(Board.class);

    // 조회를 시작할 클래스 지정
    Root<Board> b = query.from(Board.class);

    CriteriaQuery<Board> cq = query
                                .select(b)
                                .where(cb.equal(b.get("content"), content));

    return em.createQuery(cq).getResultList();
}
```

복잡하고 장황한 문제가 있다.

## `QueryDSL`

```java
public Page<MembersAndLastOrderResponse> pagingFindMembers(MemberSearchType type, String keyword, Pageable pageable) {
    // dsl 엔티티 초기화 ("별칭")
    QMember member = new QMember("m");
    QOrder order = new QOrder("q");

    // where 조건 생성
    BooleanBuilder whereBuilder = whereCondition(member, type, keyword);

    QueryResults<MembersAndLastOrderResponse> results = queryFactory
            .select(new QMembersAndLastOrderResponse(
                    member.name,
                    member.nickname,
                    member.phoneNumber,
                    member.email,
                    member.gender,
                    order.id,
                    order.productName,
                    order.paymentDate
            ))
            // from join
            .from(member)
            .leftJoin(order).on(member.id.eq(order.member.id))
            // where
            .where( whereBuilder )
            // group by
            .groupBy( member.id )
            // paging
            .offset(pageable.getOffset()).limit(pageable.getPageSize())
            .fetchResults();

    return new PageImpl<>(results.getResults(), pageable, results.getTotal());
}
```

## 네이티브 SQL

```java
public List<Board> findByTitle(String title) {
    String sql = "SELECT ID, TITLE, CONTENT FROM BOARD WHERE TITLE = 'title'";
    return em.createNamedQuery(sql, Board.class).getResultList();
}
```

## `JPQL`

```java
@Query("select m from Member as m where m.age= :age")
List<Member> findByAge(@Param("age") Integer age);
```

- `JPQL` 은 객체지향 쿼리 언어다.
- `JPQL` 은 SQL 을 추상화해서 특정 데이터베이스 SQL 에 의존하지 않는다.
- `JPQL` 은 결국 SQL 로 변환된다.

### 기본 문법과 쿼리

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

#### SELECT

```java
SELECT m FROM Member AS m where m.username = 'Hello'
```

- 대소문자 구분: 엔티티와 속성은 대소문자를 구분한다.
- 엔티티 이름: 클래스 명이 아니라 엔티티 명이다. `@Entity(name="XXX")`
- 별칭은 필수: AS 는 생략할 수 있다.

##### TypeQuery, Query

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

##### 결과 조회

- `getResultList()`
- `getSingleResult()` : 결과가 정확히 하나일 때 사용하면 결과가 없으면 `NoResultException`, 있으면 `NonUniqueResultException` 이 발생한다.

### 파라미터 바인딩

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
