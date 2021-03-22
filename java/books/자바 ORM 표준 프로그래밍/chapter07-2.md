# 🗂 7장 고급 매핑

- **상속 관계 매핑**: 객체의 상속 관계를 데이터베이스에 어떻개 매핑하는지 다룬다.
- **복합 키와 식별 관계 매핑**: 데이터베이스의 식별자가 하나 이상일 때 매핑하는 다룬다.
- **조인 테이블**: 연관관게를 관리하는 연결 테이블을 두는 방법이 있다. 이 연결 테이블을 매핑하는 방법을 다룬다.
- **엔티티 하나에 여러 테이블 매핑하기**: 보통 엔티티 하나에 테이블 하나를 매핑하지만 엔티티 하나에 여러 테이블을 매핑하는 방법을 다룬다.

## 복합 키와 식별 관계 매핑

### 식별 관계 vs 비식별 관계

테이블 사이에 관계는 외래 키가 기본 키에 포함되는지 여부에 따라 식별 관계와 비식별 관계로 구분한다.

식별 관계는 부모의 기본키를 내려 받아 자식 테이블의 기본 키 + 외래 키로 사용하는 관계이다. 비식별 관계는 부모의 기본키를 내려 받고 외래키로만 사용한다. 비식별 관계는 외래키에 NULL 을 허용하는 여부에 따라 필수적 또는 선택적 비식적 관계가 존재한다.

```text
┌─ PARENT ──────┐     ┌─ CHILD_PK ───────┐     ┌─ CHILD_FK ───────┐
│ PARENT_ID(PK) │     │ PARENT_ID(PK, FK)│     │ PARENT_ID(FK)    │
│ NAME          │     │ CHILD_ID(PK)     │     │ CHILD_ID(PK)     │
└───────────────┘     │ NAME             │     │ NAME             │
                      └──────────────────┘     └──────────────────┘
```

### 복합 키: 비식별 관계 매핑

```java
@Entity
@IdClass(ParentId.class)
public class Parent {
    @Id
    @Column(name = "PARENT_ID1")
    private String id1;

    @Id
    @Column(name = "PARENT_ID2")
    private String id2;

    // ...
}

public class ParentId implements Serializable {
    private String id1; // Parent.id1
    private String id2; // Parent.id2
}
```

- **식별자 클래스의 속성명과 엔티티에서 사용하는 식별자의 속성명이 같아야 한다.**
- `Serializable` 인터페이스를 구현해야한다.
- equals, hashCode 를 구현해야한다.
- 기본 생성자가 있어야한다.
- 식별자 클래스는 public 이어야 한다.

조회 코드는 식별자 클래스인 ParentId 클래스를 사용하여 엔티티를 조회한다.

### EmbeddedId

`@IdClass` 가 데이터베이스에 맞춘 방법이라면 `@EmbeddedId` 는 객체지향적인 방법이다.

```java
@Entity
public class Parent {
    @EmbeddedId
    private ParentId id1;
}

@Embeddable
public class ParentId implements Serializable {
    @Column(name = "PARENT_ID1")
    private String id1;

    @Column(name = "PARENT_ID2")
    private String id2;
}
```

- `@Embeddable` 어노테이션을 사용해야한다.
- `Serializable` 인터페이스를 구현해야한다.
- equals, hashCode 를 구현해야한다.
- 기본 생성자가 있어야한다.
- 식별자 클래스는 public 이어야 한다.

`@IdClass` 와 다르게 적용한 식별자 클래스를 기본키에 직접 매핑한다.

두 가지 방법에 장단점이 있는데 객체 지향적인 방법으로는 `@EmbeddedId` 지만 특정 상황에서 JPQL 의 내용이 길어지는 단점이 있다.

```java
em.createQuery("select p.id.id1, p.id.id2 from Parent p"); // @EmbeddedId
em.createQuery("select p.id1, p.id2 from Parent p"); // @IdClass
```

마지막으로 **복합 키에는 @GeneratedValue 를 사용할 수 없다. 복합 키를 구성하는 여러 컬럼 중 하나에도 사용할 수 없다.**

### 복합 키: 식별 관계 매핑

부모, 자식, 손자까지 계속 기본 키를 전달하는 식별 관계이다.

#### @IdClass 로 식별 관계 매핑

```java
@Entity
public class Parent {
    @Id @Column(name = "PARENT_ID")
    private String id;
    private String name;
}

@Entity
@IdClass(ChildId.class)
public class Child {
    @Id
    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    private Parent parent;

    @Id @Column(name = "CHILD_ID")
    private String id;
    private String name;
}

public class ChildId implements Serializable {
    private String parent; // Child.parent
    private String id; // Child.id
}

@Entity
@IdClass(GrandChildId.class)
public class GrandChild {
    @Id
    @ManyToOne
    @JoinColumns({
        @JoinColum(name = "PARENT_ID"),
        @JoinColum(name = "CHILD_ID")
    })
    private Child child;

    @Id @Column(name = "GRAND_CHILD_ID")
    private String id;
    private String name;
}

public class GrandChild implements Serializable {
    private ChildId child; // GrandChild.child;
    private String id; // GrandChild.id;
}
```

식별 관계는 기본 키와 외래 키를 같이 매핑해야 한다. 따라서 식별자 매핑인 `@Id` 와 `@ManyToOne` 을 같이 사용한다.

#### @EmbeddedId 로 식별 관계 매핑

```java
@Entity
public class Parent {
    @Id @Column(name = "PARENT_ID")
    private String id;
    prvate String name;
}

@Entity
public class Child {
    @EmbeddedId
    private ChildId id;

    @MapsId("parentId") // ChildId.parentId 매핑
    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    public Parent parent;
    private String name;
}

@Embeddable
public class ChildId implements Serializable {
    private String parentId; // @MapsId("parentId") 로 매핑
    
    @Column(name = "CHILD_ID")
    private String id;
}

@Entity
public class GrandChild {
    @EmbeddedId
    private GrandChildId id;

    @MapsId("childId") // GrandChildId.childId 와 매핑
    @ManyToOne
    @JoinColumns({
        @JoinColumn(name = "PARENT_ID"),
        @JoinColumn(name = "CHILD_ID")
    })
    private Child child;
    private String name;
}

@Embeddable
public class GrandChildId implements Serializable {
    private ChildId childId; // @MapsId("childId") 로 매핑
    
    @Column(name = "GRAND_CHILD_ID")
    private String id;
}
```

식별 관계로 사용할 연관관계의 속성에 `@MapsId` 를 사용한다. 외래 키와 매핑한 연관 관계를 기본 키에도 매핑하겠다는 뜻이다.

### 비식별 관계로의 구현

```java
@Entity
public class Parent {
    @Id @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;
    private String name;
}

@Entity
public class Child {
    @Id @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;
    private String name;

    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    private Parent parent;
}

@Entity
public class GrandChild {
    @Id @GenratedValue
    @Column(name = "GRAND_CHILD_ID")
    private Long id;
    private String name;

    @ManyToOne
    @JoinColumn(name = "CHILD_ID")
    private Child child;
}
```

식별 관계가 없기 때문에 매핑이 쉽고 복합 키를 사용하지 않기 때문에 복합 키 클래스를 만들 필요가 없다.

### 일대일 식별 관계

일대일은 자식이 따로 기본 키를 가지지 않고 부모키를 사용한다.

```java
@Entity
public class Board {
    @Id @Generatedvalue
    @Column(name = "BOARD_ID")
    private Long id;

    private String title;

    @OneToOne(mappedBy = "board")
    private BoardDetail boardDetail;
}

@Entity
public class BoardDetail {
    @Id
    private Long boardId;

    @MapsId // BoardDetail.boardId;
    @OneToOne
    @JoinColumn(name = "BOARD_ID")
    private Board board;

    private String content;
}
```

식별자가 단순히 하나의 컬럼이라면 `@MapsId` 를 사용하고 속성값을 비워둔다. `@MapsId` 는 `@Id` 를 사용한 식별자로 지정된 boardId 와 매핑된다.

### 식별, 비식별 관계의 장단점

- 식별 관계는 부모 테이블의 기본 키를 자식 테이블로 전파하기에 자식 테이블의 기본 키 컬럼이 늘어간다. 조인할 때 SQL 이 복잡해지며, 기본키 인덱스가 불필요하게 커질 수 있다.
- 식별 관계는 2개 이상의 컬럼을 합하여 복합 기본 키를 만들어야 하는 경우가 많다.
- 식별 관계를 사용할 때 기본 키가 비즈니스 의미가 있는 자연 키 컬럼을 조합하는 경우가 많다. 비즈니스와 연관도가 없다면 대리 키를 주로 사용한다. 요구사항에 따라 비즈니스가 바뀌는데 식별 관계의 자연 키 컬럼들이 자식에 손자까지 전파되면 변경이 힘들어진다.
- 식별 관계는 부모 테이블의 기본 키를 자식 테이블의 기본 키로 사용하기 때문에 비식별 관계 보다 테이블 구조가 딱딱하다.

물론 식별 관계가 가지는 장점도 있다. 기본 키 인덱스를 활용하기 좋고 특정 상황에 조인 없이 하위 테이블을 검색할 수 있다.

적절하게 식별 관계 테이블 구조를 활용하는 것도 설계의 묘미일 것이다. 그러나 신규 프로젝트 진행시는 **비식별 관계를 사용하며 기본 키는 Long 타입의 대리키**를 사용하는 것이 바람직하다. 비즈니스와 관계 없는 대리키는 유연하다. 그리고 선택적 비식별 관계보다는 필수적 비식별 관계를 사용하는 것이 좋은데 NULL 을 허용하므로 조인할 때 외부 조인을 사용해야 하기 때문이다. 반대로 필수 관계는 NOT NULL 로 항상 관계가 있어 내부 조인만 사용해도 된다.
