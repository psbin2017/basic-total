# 🗂 6장 다양한 연관관계 매핑

엔티티의 연관관계를 매핑할 때는 3가지를 고려해야한다.

1. 다중성
2. 단방향, 양바향
3. 연관관계의 주인

## 다대일

### 다대일 단반향

회원과 팀 관계

회원은 팀 엔티티를 참조할 수 있지만 반대로 팀은 회원을 참조할 필드가 없다.

### 다대일 양방향

회원과 팀 관계 (참조 필드 `List<Member>` 추가)

양방향의 경우 외래 키가 있는 쪽이 연관관계의 주인이다. 따라서 `Member.team` 가 연관관계의 주인이다. 주인이 아닌 `Team.members` 는 조회를 위한 JPQL 이나 객체 그래프를 탐색할 때 사용한다.

양방향 연관관계는 항상 서로를 참조해야한다. 양쪽에서 메소드를 작성시에 무한루프에 빠지지 않도록 주의해야 한다.

```java
public void setTeam(Team team) {
    this.team = team;

    if ( ! team.getMembers().contains(this) ) {
        team.getMembers().add(this);
    }
}

public void addMember(Member member) {
    this.members.add(member);
    if ( member.getTeam() != this ) {
        member.setTeam(this);
    }
}
```

## 일대다

## 일대다 단방향

회원과 팀 관계 (팀 객체가 외래키를 직접 관리)

```java
@OneToMany
@JoinColumn(name = "TEAM_ID") // MEMBER 테이블의 TEAM_ID (FK)
private List<Member> members = new ArrayList<Member>();
```

일대다의 매핑 단점은 엔티티의 저장과 연관관계 처리를 INSERT SQL 한번에 끝낼 수 있지만, 다른 테이블에 외래 키가 있다보니 연관관계 처리를 위해서 UPDATE SQL 을 추가로 실행해야한다.

```java
em.persist(member1); // INSERT - member1
em.persist(team1); // INSERT - team1, UPDATE - member1.fk
```

일대다 단방향보다는 다대일 양방향을 사용하는 것이 좋다.

**다대일 양방향은 없다. 다대일 관계 특성상 외래키는 항상 다쪽에 있기 때문이다.**

## 일대일

회원은 하나의 사물함을 가진다.

일대일 관계는 주 테이블이나 대상 테이블 둘 중 어느 곳이나 외래 키를 가질 수 있다.

- 주 테이블에 외래키 : 객체지향 개발자들이 선호하는 방법으로 주 테이블만 확인해도 대상 테이블과의 연관관계가 있는 것을 알 수 있다.
- 대상 테이블에 외래키 : 데이터베이스 개발자들이 선호하는 방법으로 일대일 관계에서 일대다로 변경할 때 테이블 구조를 그대로 유지할 수 있다는 장점이 있다.

### 주 테이블 외래키

```java
public class Member {

    // ...

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
}

// 양방향은 아래를 추가

public class Locker {

    // ...

    @OneToOne(mappedBy = "locker")
    private Member Member;
}
```

### 대상 테이블에 외래키

단방향은 매핑할 수 있는 방법이 없다. 양방향으로 Member 객체에 Locker 를 주인으로 설정해야 한다.

```java
public class Member {

    // ...

    @OneToOne(mappedBy = "member")
    private Locker locker;
}

public class Locker {

    // ...

    @OneToOne
    @JoinColmn(name = "MEMBER_ID")
    private Member member;
}
```

프록시를 사용할 때 외래 키를 직접관리하지 않는 일대일 관계는 지연 로딩을 설정해도 즉시 로딩된다. (8장에서 추가로 다루기에 여기까지만 이야기)

## 다대다

관계형 데이터베이스는 정규화되어있는 2개의 테이블로는 다대다 관계를 표현할 수 가 없다. 따라서 연관관계를 가진 연결 테이블을 만들어야 한다.

다대다 단방향의 매핑은 다음과 같다.

```java
public class Member {
    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT",
            joinColumns = @JoinColumn(name = "MEMBER_ID"),
            inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID") )
    private List<Product> products = new ArrayList<Product>();
}
```

- @JoinTable.name : 연결 테이블을 지정한다.
- @JoinTable.joinColumns : 현재 방향인 회원과 매핑할 조인 컬럼 정보를 지정한다.
- @JoinTable.inverseJoinColumns = 반대 방향인 상품과 매핑할 조인 컬럼 정보를 지정한다.

다음은 다대다 단방향 저장 예제이다.

```java
// ...
em.persist(productA);

// ...
memberA.getProducts().add(productA);
em.persist(memberA);
```

INSERT INTO PRODUCT ... INSERT INTO MEMBER ... INSERT INTO MEMBER_PRODUCT ... 순서로 실행된다.

다음은 다대다 양방향 예제이다.

```java
public class Product {
    @ManyToMany(mappedBy = "product")
    private List<Member> members;
}
```

동일하게 @ManyToMany 를 설정하고 원하는쪽에 mappedBy 로 연관관계의 주인을 지정한다. 앞서 본 편의 메소드를 추가하여 연관관계를 설정하도록 한다.

### 다대다 매핑 한계 극복

연결 테이블을 사용하여 자동으로 처리하여 도메인 모델이 단순해지고 여러가지로 편리하다. 그러나 매핑을 실무에서 적용하면 한계가 있다. 보통은 연결테이블에 주문 수량 컬럼이나 주문한 날짜와 같은 컬럼이 필요하게된다.

컬럼을 추가하게 되면 더는 @ManyToMany 를 사용할 수 없게 된다. 연결 테이블을 엔티티로 만들어 Member 와 MemberProduct 의 일대다 관계, MemberProduct 와 Product 의 다대일 관계를 만들어야 한다.

```java
@Entity
@IdClass(MemberProductId.Class)
public class MemberProduct {
    @Id
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @Id
    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;
}

// ...

public class MemberProductId implements Serializable {
    private String member; // MemberProduct.member 와 연결
    private String product; // MemberProduct.product 와 연결

    // equals and hashCode
    // default constructor
}
```

위 예제는 `@IdClass` 로 복합 키를 만들어 연결 테이블에 사용되는 참조 외래 키 클래스를 만들었다. 복합 키를 위한 식별자 클래스는 다음 조건을 가진다.

- 복합 키는 별도의 식별자 클래스로 만들어야한다.
- Serializable 을 구현해야 한다.
- equals 와 hashCode 를 구현해야 한다.
- 기본 생성자를 가져야 한다.
- public 클래스 이어야한다.

이 외 방법으로 `@EmbededId` 를 사용하여 복합 키를 만들어 식별 관계를 만드는 방법이 있다. (7장에 추가로 다룸)

다음은 회원상품 엔티티를 포함한 저장예제이다.

```java
// 회원 저장 (생략)

// 상품 저장 (생략)

// 회원-상품 저장
memberProduct.setMember(member1); // 연관관계 설정
memberProduct.setProduct(productA); // 연관관계 설정
em.persist(memberProduct);
```

### 다대다 새로운 기본키 사용

일반적인 상황이라면 추천하는 기본 키 생성 전략은 데이터베이스에서 자동으로 생성해주는 대리 키를 사용하는 것이다. 간편하고 영구히 사용가능하며 비즈니스에 의존하지 않는다. 이번에는 회원-상품 테이블의 용도를 주문 테이블로 변경하여 적용한다.

```java
@Entity
public class Order {
    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;
}
```

이전에 보았던 식별 관계에 복합 키를 사용하는 것보다 매핑이 직관적이며 단순해져 이해하기 쉬워진다. 또한 회원 엔티티와 상품 엔티티는 큰 변동이 없다는 것이다.
