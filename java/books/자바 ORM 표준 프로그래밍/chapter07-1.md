# 🗂 7장 고급 매핑

- **상속 관계 매핑**: 객체의 상속 관계를 데이터베이스에 어떻개 매핑하는지 다룬다.
- **복합 키와 식별 관계 매핑**: 데이터베이스의 식별자가 하나 이상일 때 매핑하는 다룬다.
- **조인 테이블**: 연관관게를 관리하는 연결 테이블을 두는 방법이 있다. 이 연결 테이블을 매핑하는 방법을 다룬다.
- **엔티티 하나에 여러 테이블 매핑하기**: 보통 엔티티 하나에 테이블 하나를 매핑하지만 엔티티 하나에 여러 테이블을 매핑하는 방법을 다룬다.

## 상속 관계 매핑

### 조인 전략

```text
┌─ Item ─────────┐
│ ITEM_ID(PK)    │
│ NAME           │ 
│ PRICE          │ 
│ DTYPE          │ 
└────────────────┘

┌─ Album ─────────┐     ┌─ Movie ─────────┐     ┌─ Book ──────────┐
│ ITEM_ID(PK, FK) │     │ ITEM_ID(PK, FK) │     │ ITEM_ID(PK, FK) │
│ ARTIST          │     │ DIRECTOR        │     │ AUTHOR          │
└─────────────────┘     │ ACTOR           │     │ ISBN            │
                        └─────────────────┘     └─────────────────┘
```

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {
    @Id @Generatedvalue
    @Column(name = "ITEM_ID")
    private Long id;

    // ...
}

@Entity
@DiscriminatorValue(name = "A")
public class Album extends Item {
    private String artist;
}
```

1. `@Inheritance(strategy = InheritanceType.JOINED)` : 부모 클래스에 사용하여 조인 전략을 명시한다.
2. `@DiscriminatorColumn(name = "DTYPE")` : 부모 클래스에 구분 컬럼을 지정한다. (기본 값은 `DTYPE` 이다)
3. `@DiscriminatorValue(name = "A")` : 엔티티를 저장할 때 구분 컬럼에 입력할 값을 지정한다.

만약 자식 타이블에서 부모 테이블의 ID 컬럼을 그대로 사용하되 자식 테이블의 기본 키 컬럼명을 변경하고자 한다면 `@PrimaryKeyJoinColumn` 을 사용한다.

```java
@Entity
@DiscriminatorValue(name = "A")
@PrimaryKeyJoinColumn(name = "BOOK_ID")
public class Album extends Item {
    // ....
}
```

장점

- 테이블이 정규화된다.
- 외래 키 참조 무결성 제약조건을 활용할 수 있다.
- 저장공간을 효율적으로 사용한다.

단점

- 조회할 때 조인이 많이 사용되므로 성능이 저하될 수 있다.
- 조회 쿼리가 복잡하다
- 데이터를 등록할 INSERT SQL 을 두 번 실행한다.

특징

- JPA 표준 명세는 구분 컬럼을 사용하도록 하지만 하이버네이트를 포함한 몇 몇 구현체는 구분 컬럼(`@DiscriminatorColumn`) 없이도 동작한다.

### 단일 테이블 전략

```text
┌─ Item ─────────┐
│ ITEM_ID(PK)    │
│ NAME           │ 
│ PRICE          │ 
│ ARTIST         │
│ DIRECTOR       │
│ ACTOR          │
│ AUTHOR         │
│ ISBN           │
│ DTYPE          │
└────────────────┘
```

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {
    @Id @Generatedvalue
    @Column(name = "ITEM_ID")
    private Long id;

    // ...
}

@Entity
@DiscriminatorValue(name = "A")
public class Album extends Item { ... }
```

`InheritanceType.SINGLE_TABLE` 로 지정하여 단일 테이블 전략을 사용한다.

장점

- 조인이 필요없어 일반적인 조회 성능이 빠르다.
- 조회 쿼리가 단순하다.

단점

- 자식 엔티티가 매핑한 컬럼은 모두 null 을 허용 해야한다. (ARTIST ~ ISBN)
- 로우가 많아져 테이블이 커져 오히려 느려질수도 있다.

특징

- 반드시 구분 컬럼을 `@DiscriminatorColumn` 설정해야한다.
- `@DiscriminatorValue` 미설정 시 엔티티 명을 따라간다.

### 구현 클래스마다 테이블 전략

```text
┌─ Album ─────┐     ┌─ Movie ─────┐     ┌─ Book ──────┐
│ ITEM_ID(PK) │     │ ITEM_ID(PK) │     │ ITEM_ID(PK) │
│ NAME        │     │ NAME        │     │ NAME        │ 
│ PRICE       │     │ PRICE       │     │ PRICE       │ 
│ ARTIST      │     │ DIRECTOR    │     │ AUTHOR      │
└─────────────┘     │ ACTOR       │     │ ISBN        │
                    └─────────────┘     └─────────────┘
```

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {
    @Id @Generatedvalue
    @Column(name = "ITEM_ID")
    private Long id;

    // ...
}

@Entity
public class Album extends Item { ... }
```

`InheritanceType.TABLE_PER_CLASS` 로 구현 클래스마다 테이블 전략을 사용한다.

장점

- 서브 타입을 구분하여 처리할 때 효과적이다.
- not null 제약조건을 사용할 수 있다.

단점

- 여러 자식 테이블을 함께 조회시 성능이 느리다. (UNION SQL)
- 자식 테이블을 통합하여 쿼리할 때 어렵다.

특징

- 구분 컬럼을 사용하지 않는다.

이 전략은 추천하지 않는 전략으로 앞서본 전략들을 사용하는 것이 바람직하다.

## @MappedSuperclass

지금까지 본 상속 관계 매핑은 부모 클래스와 자식 클래스를 모두 데이터베이스 테이블에 매핑하였다.

부모 클래스는 테이블과 매핑하지 않고, 부모 클래스를 상속 받는 자식클래스에게 매핑 정보만 제공하고자 한다면 `@MappedSuperclass` 를 사용하면 된다.

```text
┌─ BaseEntity ──┐     ┌─ Member ─────┐     ┌─ Seller ────┐
│ ID(PK)        │     │ EMAIL        │     │ SHOP_NAME   │
│ NAME          │     └──────────────┘     └─────────────┘
└───────────────┘
```

```java
@MappedSuperclass
public abstract class BastEntity {
    @Id @GeneratedValue
    private Long id;
    private String name;
}

@Entity
public class Member extends BaseEntity { ... }
```

부모로 부터 물려받은 매핑 정보를 재정의하려면 `@AttributeOverrides` 나 `@AttributeOverride` 를 사용하며, 연관관계를 재정의하려면 `@AssociationOverrides` 나 `@AssociationOverride` 를 사용한다.

```java
@Entity
@AttributeOverride(name = "id", column = @Column(name = "MEMBER_ID"))
public class Member extends BasyEntity { ... }

@Entity
@AttributeOverrides({
    @AttributeOverride(name = "id", column = @Column(name = "MEMBER_ID")),
    @AttributeOverride(name = "name", column = @Column(name = "MEMBER_NAME"))
})
public class Member extends BasyEntity { ... }
```

특징

- 테이블과 매핑되지 않고 자식 엔티티의 매핑 정보를 상속하기 위해 사용한다.
- `@MappedSuperclass` 로 지정한 클래스는 엔티티가 아니므로 em.find() 나 JPQL 에서 사용할 수 없다.
- 이 클래스를 직접 생성해서 사용할 일은 거의 없으므로 추상 클래스로 만드는 것을 권장한다.

`@MappedSuperclass` 를 사용하면 등록일자, 수정일자, 등록자, 수정자와 같은 여러 엔티티에서 공통으로 사용하는 속성을 효과적으로 사용할 수 있다.
