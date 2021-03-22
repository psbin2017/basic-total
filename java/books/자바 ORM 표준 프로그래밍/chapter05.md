# 🗂 5장 연관관계 매핑 기초

이 장의 목표는 객체의 참조와 테이블의 외래 키를 매핑하는 것이 목표이다.

방향(Direction) : 단방향과 양방향으로 나뉘며 객체 관계에서만 존재한다.

- 회원 → 팀 또는 팀 → 회원 ... 한 쪽만을 참조하는 단방향
- 회원 → 팀, 팀 → 회원 ... 양 쪽을 서로 참조하는 양방향

다중성(Multiplicity) : 다대일(N:1), 일대다(1:N), 다대다(N:M)

- 여러 명의 회원(N)은 하나의 팀(1)에 속한다.
- 하나 팀(1)에 여러 명의 회원(N)이 속한다.

연관 관게의 주인(Owner) : 객체를 양방향 관계로 만들시 연관 관계의 주인을 정해야한다.

## 단방향 연관관계

```sql
SELECT *
FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
```

서로 참조하는 것을 양방향 연관 관계이지만 객체에서의 양방향은 서로 다른 단방향 관계의 2개이다.

```java
class A {
    B b;
}

class B {
    A a;
}
```

### 객체 관계 매핑

```java
@Entity
public class Member {

    @Id
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```

### `@JoinColumn`

| 속성 | 기능 | 기본 값 |
| --- | --- | --- |
| name | 매핑할 외래키 명 | 필드명 + _ + 참조하는 테이블의 기본 키 컬럼 명 |
| referencedColumnName | 외래 키가 참조하는 대상 테이블의 컬럼 명 | 참조하는 테이블의 기본 키 컬럼 명 |
| foreignKey (DDL) | 외래 키 제약조건 직접 지정. | |
| unique, nullable, insertable, updatable, columnDefinition, table | `@Column` 과 동일 | |

`@JoinColumn` 을 생략시 위의 예시로 team_TEAM_ID 외래 키를 사용한다.

### `@ManyToOne`

| 속성 | 기능 | 기본 값 |
| --- | --- | --- |
| optional | false 로 설정시 연관된 엔티티가 항상 있어야 한다. | true |
| fetch | 글로벌 페치 전략을 설정한다 | @ManyToOne = FetchType.EAGER, @OneToMany = FetchType.LAZY |
| cascade | 영속성 전이 기능을 사용한다. | |
| targetEntity | 연관된 엔티티 타입 정보를 설정한다. 컬렉션으로 사용해도 제네릭으로 타입 정보를 알 수 있다. | |

```java
@OneToMany
private List<Member> members; // 제네릭으로 타입 유추 가능

@OneToMany(targetEntity=Member.class)
private List members;
```

#### 객체 연관 관계 저장

```java
    Team team1 = new Team("team1", "팀1");
    em.persist(team1);

    Member member1 = new Member("member1","회원1");
    member.setTeam(team1); // 연관 관계 설정 member1 → team1
    em.persist(member1);
```

#### 객체 연관 관계 조회

- 객체 그래프 탐색
- 객체 지향 쿼리 사용 (JPQL)

객체 그래프 탐색은 간단하다.

```java
    Member member = em.find(Member.class, "member1");
    Team team = member.getTeam();
```

객체 지향 쿼리 사용은 다음과 같다

```java
private static void queryLogicJoin(EntityManager em) {
    String jpql = "select m from Member m join m.team t where" +
            "t.name=:teamName";

    List<Member> resultList = em.createQuery(jpql, Member.class)
        .setParameter("teamName", "팀1")
        .getResultList();

    for (Member member : resultList) {
        // ...
    }
}
```

#### 객체 연관 관계 수정

```java
    Team team2 = new Team("team2", "팀2");
    em.persist(team2);

    Member member = em.find(Member.class, "member1");
    member.setTeam(team2);
```

#### 객체 연관 관계 삭제

```java
    Member member = em.find(Member.class, "member1");
    member.setTeam(null);
```

연관된 엔티티를 삭제하려면 외래키 제약 조건으로 데이터베이스 오류가 발생하기 때문에 연관 관계를 먼저 끊어야한다.

```java
    member1.setTeam(null);
    member2.setTeam(null);
    em.remove(team1);
```

## 양방향 연관관계

```java
@Entity
public class Team {

    @Id
    @Column(name = "TEAM_ID")
    private String id;

    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<Member>();

}
```

`mappedBy` 속성은 양방향 매핑일 때 사용하는 속성으로 반대쪽 매핑의 필드 명을 값으로 준다.

```java
Team team = em.find(Team.class, "team1");
List<Member> members = team.getMembers();

for (Member member : members) {
    // ...
}
```

## 연관 관계의 주인

엔티티를 양방향 연관관계로 설정하면 객체의 참조는 둘인데 외래 키는 하나로 구성된다. 따라서 테이블 관계와 객체 관계가 차이가 발생한다. 이런 차이로 인해 JPA 는 두 객체 연관 관계 중 하나를 정해서 테이블의 외래 키를 관리해야 한다. 이를 연관 관계의 주인이라고 한다.

### 양방향 매핑의 규칙: 연관 관계의 주인

연관 관계의 주인만 데이터베이스 연관관계와 매핑되고 외래 키를 관리(등록/수정/삭제)할 수 있다. 반면에 주인이 아닌 쪽은 읽기만 가능하다.

- 주인은 mappedBy 속성을 사용하지 않는다.
- 주인이 아니면 mappedBy 속성을 사용해서 속성의 값으로 연관 관계의 주인을 지정해야 한다.

**연관 관계의 주인을 정한다는 것은 외래 키의 관리자를 선택하는 것이다.**

## 양방향 연관관계의 주의점

```java
Member member1 = new Member("member1", "회원1");
em.persist(member1);

Team team1 = new Team("team1", "팀1");
// 주인이 아닌 곳에서 연관관계를 설정.
team1.getMembers().add(member1);

em.persist(team1);
```

### 순수한 객체까지 고려한 양방향 연관관계

객체 관점에서는 양쪽 방향에 모두 값을 입력하는 것이 가장 안전하다. 양방향으로 모두 값을 입력하지 않으면 JPA 를 사용하지 않은 순수 객체 상태에서 문제가 발생할 수 있다.

```java
Team team1 = new Team("team1", "팀1");
Member member1 = new Member("member1", "멤버1");
Member member2 = new Member("member2", "멤버2");

member1.setTeam(team1); // member1 → team1
member2.setTeam(team1); // member2 → team1

List<Member> members = team1.getMembers();
// members.size = 0

team1.getMembers().add(member1); // team1 → member1
team1.getMembers().add(member2); // team1 → member2

members = team1.getMembers();
// members.size = 2
```

객체까지 고려하면 이렇게 양방향으로 관계를 만들어야한다.

정리하면 다음과 같다.

- Member.team: 연관 관계의 주인, 이 값으로 외래키를 관리
- Team.members: 연관 관계의 주인이 아니다. 읽기에만 사용된다.

또한 객체의 양방향 연관 관계는 양쪽 모두 값을 입력하여 주인과 관계없이 모두 관계를 만들어주자.

### 연관관계 편의 메소드

```java
public void setTeam(Team team) {
    this.team = team;
    team.getMembers().add(this);
}
```

### 연관관계 편의 메소드 작성시 주의사항

```java
member1.setTeam(teamA);
member2.setTeam(teamB);

Member findMember = teamA.getMember(); // member1 이 여전히 조회된다.
```

teamB 로 변경시 teamA → member1 의 관계를 제거하지 않았기 때문이다. 연관 관계를 변경시 기존 팀이 있다면 회원의 연관 관계를 삭제하는 코드도 추가해야한다.

```java
public void setTeam(Team team) {
    if ( this.team != null ) {
        this.team.getMembers().remove(this);
    }
    this.team = team;
    team.getMembers().add(this);
}
```

JPA 에서는 teamA → member1 관계가 제거되지 않아도 외래 키를 변경하는 데는 문제가 없다. teamA 가 주인이 아니기 때문이다.

이후 영속성 컨텍스트에서 teamA 를 조회하면 외래 키에서는 관계가 끊어져 있으므로 아무것도 조회되지 않느다.

다만 영속성 컨텍스트가 아직 살아있는 상태에서 teamA 의 getMembers() 를 호출하면 member1 이 반환된다는 점으로 연관관계를 앞서 본 내용처럼 제거하는 것이 바람직하다.

끝으로 **연관관계의 주인을 정하는 기준**은 외래 키의 위치와 관련해서 정해야하며 비즈니스의 중요도로 접근하면 안 된다.
