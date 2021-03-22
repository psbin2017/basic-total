# 🗂 8장 프록시와 연관관계 관리

- 프록시와 즉시로딩, 지연로딩: 객체는 객체 그래프로 연관된 객체들을 탐색한다. 그러나 실제 객체는 데이터베이스에 저장되어 있어 연관 객체를 마음대로 탐색하는 것은 어렵다. JPA 는 이를 해결하기 위해 프록시라는 기술을 사용한다. 실제 사용 시점에 데이터베이스에서 조회하는 기술로 효과적인 조회를 위해 즉시 로딩과 지연 로딩이라는 방법도 모두 지원한다.

- 영속성 전이와 고아 객체: 연관된 객체를 함께 저장하거나 함께 삭제할 수 있는 영속성 전이와 고아 객체 제거라는 기능을 제공한다.

## 프록시

엔티티의 값을 실제 사용하는 시점에 데이터베이스에서 조회하는 방법을 **지연 로딩**이라고 한다. 지연 로딩 기능을 사용하면서 실제 엔티티 객체 대신 데이터베이스 조회를 지연할 수 있는 가짜 객체를 프록시 객체라고 한다.

### 프록시 기초

`em.find()` 는 데이터베이스를 조회한다. 실제 사용 시점까지 조회를 미룬다면, `em.getReference()` 를 사용하면 된다. 데이터베이스 접근을 위임한 프록시 객체를 반환한다.

#### 프록시의 특징

프록시 클래스는 실제 클래스를 상속 받아 만들어지며 사용하는 입장에서는 이것이 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면 된다. 프록시 객체는 실제 객체에 대한 참조를 보관하고 프록시 객체의 메소드를 호출하면 실제 객체의 메소드를 호출한다. (delegate)

- 프록시 객체는 처음 사용할 때 한 번만 초기화된다.
- 프록시 객체가 초기화되어도 프록시 객체가 실제 엔티티가 되는 것이 아니다. 프록시 객체가 초기화되면 프록시 객체를 통해 실제 엔티티로 접근할 수 있다.
- 프록시 객체는 원본 엔티티를 상속받은 객체이므로 타입 체크 시 주의해서 사용해야 한다.
- 영속성 컨텍스트에 찾는 엔티티가 이미 있다면 `em.getReference()` 를 호출하면 프록시 객체가 아닌 실제 엔티티를 반환한다.
- 초기화는 영속성 컨텍스트의 도움을 받아야 가능하다. 따라서 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태의 프록시를 초기화하면 문제가 발생한다.

#### 프록시 객체의 초기화

프록시 객체는 데이터베이스를 조회하면 실제 엔티티 객체를 생성하는데 이것을 프록시 객체의 초기화라 한다.

1. 실제 엔티티 객체 요청
2. 영속성 컨텍스트에 실제 엔티티가 없다면 영속성 컨텍스트에 엔티티 생성을 요청 (**프록시 초기화**)
3. 영속성 컨텍스트가 데이터베이스를 조회하여 실제 엔티티 객체를 생성한다.
4. 프록시 객체는 생성된 실제 엔티티 객체의 참조를 Member target 멤버변수에 보관한다.
5. 프록시 객체는 실제 엔티티 객체의 `getName()` 을 호출하여 결과를 반환한다.

### 프록시와 식별자

프록시 객체는 식별자 값을 가지고 있으므로 식별자 값을 조회하는 `team.getId()` 를 호출해도 프록시를 초기화하지 않는다. 단 엔티티 접근 방식을 프로퍼티로 설정한 경우 (`@Access(AccessType.PROPERTY)`) 로 설정한 경우만 초기화하지 않는다.

엔티티 접근 방식을 필드로 설정한 경우 (`@Access(AccessType.FIELD)`) 로 설정하면 `getId()` 메소드가 id 만 조회하는 메소드인지 다른 필드까지 활용해서 어떤 일을 하는 메소드인지 알지 못하므로 프록시 객체를 초기화한다.

```java
Member member = em.find(Member.class, "member1");
Team team = em.getReference(Team.class, "team1");
member.setTeam(team);
```

연관관계를 설정할 때는 식별자 값만 사용하므로 프록시를 사용하면 데이터베이스 접근 횟수를 줄일 수 있다. 연관관계를 설정할 때는 엔티티 접근 방식을 필드로 설정해도 프록시를 초기화하지 않는다.

### 프록시 확인

`PersistenceUnitUtil.isLoaded(Object entity)` 메소드는 프록시 인스턴스의 초기화 여부를 알 수 있다. 조회한 엔티티가 진짜 엔티티인지 확인하는 방법은 클래스명을 직접 출력해보면 된다. 클래스 명 뒤에 ..javassis.. 라 되어 있는데 이것으로 프록시인 것을 확인할 수 있다.

#### 프록시 강제 초기화

하이버네이트 기준 `initalize()` 메소드로 프록시를 강제 초기화할 수 있다. JPA 표준에서는 강제 표준 방법은 없고 `member.getName()` 처럼 직접 호출하면 된다. JPA 표준은 단지 초기화 여부만 확인할 수 있다.

## 즉시 로딩과 지연 로딩

프록시 객체는 주로 연관된 엔티티를 지연 로딩할 때 사용한다.

- 즉시 로딩 : `@ManyToOne(fetch = FetchType.EAGER)`
- 지연 로딩 : `@ManyToOne(fetch = FetchType.LAZY)`

### NULL 제약 조건과 JPA 조인 전략

- `@JoinColumn(nullable = true)` : NULL 허용(기본값), 외부 조인 사용 (LEFT OUTER JOIN)
- `@JoinColumn(nullable = false)` : NULL 비허용, 내부 조인 사용 (INNER JOIN)

혹은 다음 처럼 `@ManyToOne(optional = false)` 로 설정해도 내부 조인을 사용한다.

### 즉시 로딩

대부분의 JPA 구현체는 즉시 로딩을 최적화하기위해 가능하면 조인 쿼리를 사용한다.

### 지연 로딩

지연 로딩에서 프록시 객체는 실제 사용될 때까지 데이터 로딩을 미룬다.

## 즉시/지연 로딩 활용

객체의 연관 빈도 수가 중요하다.

- Member 와 연관된 Team 은 자주 함꼐 사용되었다. (즉시 로딩)
- Member 와 연관된 Order 는 가끔 사용되었다. (지연 로딩)

### 프록시와 컬렉션 래퍼

```java
Member member = em.find(Member.class, "member1")
List<Order> orders = member.getOrders();
```

하이버네이트는 엔티티를 영속 상태로 만들 때 엔티티에 컬렉션이 있으면 컬렉션을 추적하고 관리할 목적으로 원본 컬렉션을 하이버네이트가 제공하는 내장 컬렉션으로 변경하는데 이것을 컬렉션 래퍼라고 한다.

엔티티를 지연 로딩하면 프록시 객체를 사용하여 지연 로딩을 수행하지만 주문 내역 같은 컬렉션은 컬렉션 래퍼가 지연 로딩을 처리한다. 그리고 `member.getOrders()` 를 호출해도 컬렉션은 초기화 되지 않는다. `member.getOrders().get(0)` 처럼 컬렉션에서 실제 데이터를 조회할 떄 데이터베이스를 조회해서 초기화한다.

### JPA 기본 페치 전략

추천하는 방법은 모든 연관관계에 지연 로딩을 사용하는 것이다. 그리고 애플리케이션 개발이 어느 정도 완료되었을때 실제 사용 상황을 보고 꼭 필요한 곳에만 즉시 로딩을 사용하도록 최적화 하는 것이다. SQL 을 직접 사용하면 이 것이 어렵지만 JPA 는 자연스럽게 가능하다.

### 컬렉션에 FetchType.EAGER 사용 시 주의점

- 컬렉션을 하나 이상 즉시 로딩하는 것은 권장하지 않는다. 두 테이블과 일대다 조인시 N 곱하기 M 이 되면서 너무 많은 데이터를 반환하여 결과적으로 애플리케이션 성능 저하가 발생한다.
- 컬렉션 즉시 로딩은 항상 외부 조인을 사용한다. (OUTER JOIN) 회원 테이블의 외래 키에 nout null 제약을 걸면 모든 회원은 팀에 소속되므로 항상 내부조인을 사용해도 된다. 반대로 팀 테이블에서 회원 테이블로 일대다 관계를 조인할 때 회원이 한 명도 없는 팀을 내부 조인하면 팀까지 조회되지 않는 문제가 발생한다. 제약조건으로 이 상황을 막을 수 없기 때문에, JPA 는 일대다 관계를 즉시 로딩할 때 항상 외부조인을 사용한다.

`FetchType.EAGER` 의 설정과 조인전략은 다음과 같다.

- `@ManyToOne`, `@OneToOne`
  - `(optional = false)` : 내부 조인
  - `(optional = true)` : 외부 조인
- `@OneToMany`, `@ManyToMany`
  - `(optional = false)` : 외부 조인
  - `(optional = true)` : 외부 조인

## 영속성 전이: CASCADE

특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶다면 영속성 전이 기능을 사용한다. JPA 는 CASCADE 옵션으로 영속성 전이를 제공한다.

```java
Parent parent = new Parent();
em.persist(parent);

Child child1 = new Child();
child.setParent(parent); // 자식 -> 부모 연관관계 설정
parent.getChildren().add(child1) // 부모 -> 자식
em.persist(child1);

Child child2 = new Child();
child.setParent(parent); // 자식 -> 부모 연관관계 설정
parent.getChildren().add(child2) // 부모 -> 자식
em.persist(child2);
```

**JPA 에서 엔티티를 저장할 때 연관된 모든 엔티티는 영속 상태여야 한다.** 위 상황과 같이 부모만 영속 상태로 만들면 연관된 자식까지 한 번에 영속 상태로 만들 수 있다.

영속성 전이는 연관관계를 매핑하는 것과는 아무 관련이 없다. 단지 **엔티티를 영속화할 때 연관 엔티티도 같이 영속화하는 편리함을 제공하는 것이다.**

### 영속성 전이: 저장

```java
@Entity
public class Parent {

    @OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)
    private List<Child> children = new ArrayList<Child>();
}

// ...

Child child1 = new Child();
Child child2 = new Child();

Parent parent = new Parent();

child1.setParent(parent); // 연관 관계 추가
child2.setParent(parent); // 연관 관계 추가
parent.getChildren().add(child1);
parent.getChildren().add(child2);

em.persist(parent);
```

### 영속성 전이: 삭제

`CascadeType.REMOVE` 로 설정하고 부모 엔티티만 삭제하면 연관 자식 엔티티도 함께 삭제된다.

```java
Parent findParent = em.find(Parent.class, 1L);
Child findChild1 = em.find(Child.class, 1L);
Child findChild2 = em.find(Child.class, 2L);

em.remove(findParent);
em.remove(findChild1);
em.remove(findChild2);

// ...

Parent findParent = em.find(Parent.class, 1L);
em.remove(findParent);
```

실행시 DELETE SQL 을 3번 실행하고 부모는 물론 연관 자식도 모두 삭제한다.

### CASCADE 종류

```java
public enum CascadeType {
    ALL, // 모두 적용
    PERSIST, // 영속
    MERGE, // 병합
    REMOVE, // 삭제
    REFRESH, // REFRESH
    DETACH // DETACH
}
```

`casecade = {CascadeType.PERSIST, CascadeType.REMOVE}` 와 같이 여러 속성을 사용할 수 있다.

`CascadeType.PERSIST`, 와 `CascadeType.REMOVE` 는 `em.persist()`, `em.remove()` 실행 시 전이가 바로 발생하지 않고 플러시 호출시 전이가 발생한다.

## 고아 객체

부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능을 제공하는데 이를 고아 객체(ORPHAN) 제거라 한다. **부모 엔티티의 컬렉션에서 자식 엔티티 참조만 제거하면 자식 엔티티가 자동으로 삭제된다.**

```java
@OneToMany(mappedBy = "parent", orphanRemoval = true)
private List<Child> children = new ArrayList<Child>();

// ...

Parent parent1 = em.find(Parent.class, id);
parent1.getChildren().remove(0); // 자식 엔티티를 컬렉션에서 제거
```

컬렉션을 제거하는 것으로 데이터베이스의 데이터도 제거된다. 고아 객체 제거 기능은 영속성 컨텍스트가 플러시할 때 적용된다. 모든 자식 엔티티를 제거하는 것은 `clear()` 를 호출하면 된다.

**참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 것이다.** 따라서 이 기능을 참조하는 곳이 하나일 때만 사용해야 한다. 이런 이유로 orphanRemoval 은 `@OneToOne`, `@OneToMany` 에서만 사용할 수 있다.

부가적으로 고아 객체 제거에는 부모가 제거되면 자식도 같이 제거된다. 이 것은 `CascadeType.REMOVE` 를 설정한 것과 동일하다.

## 영속성 전이 + 고아 객체, 생명주기

`CascadeType.ALL` + `orphanRemoval = true` 동시 사용시 부모 엔티티를 통해서 자식 엔티티의 생명주기를 관리할 수 있다.

자식을 저장하려면 부모에서 등록하면 된다. (CASCADE)

```java
Parent parent = em.find(Parent.class, parentId);
parent.addChild(child1);
```

자식을 삭제하려면 부모에서 삭제하면 된다. (orphanRemoval)

```java
Parent parent = em.find(Parent.class, parentId);
parent.getChildren().remove(removeObject);
```

> 영속성 전이는 DDD 의 Aggregate Root 개념을 구현할 때 사용하면 편리하다.
