# N + 1

- 1 명의 회원은 N 개의 주문을 가질 수 있다.

```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}
```

```java
@Entity
@Table(name = "Orders")
public class Order {
    @Id @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "member_id")
    private Member member;

}
```

## 증상

`OrderRepository.findAll()` 수행 시 문제가 발생한다.

```sql
-- N 개의 결과가 반환됨
SELECT * FROM ORDERS;

-- N 개의 쿼리를 실행함
SELECT * FROM MEMBER WHERE member_id = 1;
SELECT * FROM MEMBER WHERE member_id = 2;
...
SELECT * FROM MEMBER WHERE member_id = N;
```

> 엔티티 관계가 더있으면 상황이 더 악화된다.

## 해결 방안

1. `Fetch Join` 또는 `@EntityGraph` 를 사용한다.
2. 글로벌 페치 전략을 LAZY 를 사용한다. (즉시 로딩이 불필요한 `@XToOne` 에 대한 모든 부분)
3. OSIV 적용.

그 외 자동으로 만들어주는 쿼리에 대한 부분을 신용할 수 없다면 직접 작성할 것, 쿼리를 모두 확인해야한다.

### Join Fetch

```java
@Query("select o from Orders join fetch o.member")
List<Order> findAllJoinFetch();
```

Join Fetch 는 Inner JOIN 으로 수행된다.

### `@EntityGraph`

```java
@EntityGraph(attributePaths = "member")
@Query("select o from Orders o")
List<Order> findAllEntityGraph();
```

Entity Graph 는 Outer Join 으로 수행된다.

**둘 다 카테시안 곱이 발생하여 `Member` 의 수 만큼 `Orders` 가 중복으로 발생하게 된다.**

#### 카테시안 곱 해결 방법

##### `Set`, `LinkedHashSet`

1. 일대다 필드의 타입 `Set` 으로 선언. 순서를 보장하기 위해 `LinkedHashSet` 사용

```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    @OneToMany(mappedBy = "member")
    private Set<Order> orders = new LinkedHashSet<>();
}
```

##### DISTINCT

```java
@Query("select DISTINCT o from Orders join fetch o.member")
List<Order> findAllJoinFetch();
```

```java
@EntityGraph(attributePaths = "member")
@Query("select DISTINCT o from Orders o")
List<Order> findAllEntityGraph();
```

### OSIV

스프링 OSIV

- 영속성 컨텍스트를 프레젠테이션 계층까지 유지한다.
- 프레젠테이션 계층에는 트랜잭션이 없으므로 엔티티를 수정할 수 없다.
- 프레젠테이션 계층에는 트랜잭션이 없지만 트랜잭션 없이 읽기를 사용하여 지연 로딩을 할 수 있다.

## 학습 내용 출처

- [Spring - Open Session In View](https://blog.kingbbode.com/27)
- [실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1/dashboard)
- [JPA N+1 쿼리 문제와 해결](https://meetup.toast.com/posts/87)
- [JPA N+1 문제 및 해결방안](https://jojoldu.tistory.com/165)
- [VOCATEST 1. JPA가 만들어주는 SQL 출력문 확인하고 리팩토링하기](https://siyoon210.tistory.com/150)
