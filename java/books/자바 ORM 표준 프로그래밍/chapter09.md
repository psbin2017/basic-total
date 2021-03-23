# 🗂 9장 값 타입

- 기본 값 타입 (basic value type)
  - 자바 기본 타입 (`int`, `double`)
  - 래퍼 클래스 (`Integer`)
  - String
- 임베디드 타입 (embedded type) ... 복합 값 타입
- 컬렉션 타입 (collection value type)

## 기본 값 타입

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String name;

}
```

## 임베디드 타입 (복합 값 타입)

```java
@Entity
public class Member {

    // ...

    @Embedded
    private Address address;
}
```

```java
@Embeddable
public class Address {
    private String city;
    private String street;
    private String zipcode;
}
```

- `@Embedded`: 값 타입을 정의하는 곳에서 표시
- `@Embeddable`: 값 타입을 사용하는 곳에서 표시

하이버네이트에서는 임베디드 타입을 컴포넌트라고 한다. 자바로 보았을 때는 컴포지션 관계라고 할 수 있다.

### 임베디드 타입과 연관 관계

```text
[주소:값타입] → [집코드:값타입]
[핸드폰 번호:값타입] → [핸드폰 서비스 제공자:엔티티]
```

```java
@Entity
public class Member {
    @Embedded private Address address;
    @Embedded private PhoneNumber phoneNumber;
}
```

```java
@Embeddable
public class Address {
    private String city;
    private String street;

    @Embedded private Zipcode zipcode;
}
```

```java
@Embeddable
public class ZipCode {
    private String zip;
    private String plusFour;
}
```

```java
@Embeddedable
publis class PhoneNumber {
    private String areaCode;
    private String localNumber;
    @ManyToOne PhoneServiceProvider provider; // 엔티티 참조
}
```

```java
@Entity
publis class PhoneServiceProvider {
    @Id private Long id;
}
```

### @AttributeOverride: 속성 재정의

```java
@Entity
public class Member {
    @Embedded private Address homeAddress;
    @Embedded private Address companyAddress;
}
```

위 문제는 같은 임베디드 타입을 사용하여 테이블의 컬럼명이 중복되게 된다.

```java
@Entity
public class Member {

    @Embedded
    private Address homeAddress;

    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name="city", column=@Column(name = "COMPANY_CITY")),
        @AttributeOverride(name="street", column=@Column(name = "COMPANY_STREET")),
        @AttributeOverride(name="zipcode", column=@Column(name = "COMPANY_ZIPCODE"))
    })
    @Embedded private Address companyAddress;
}
```

> `@AttributeOverrides`(`@AttributeOverride`) 는 사용하는 엔티티에서 설정해야한다. 임베디드 타입을 가지고 있어도 엔티티에서 설정해야 한다.

어노테이션이 많이 사용되어 엔티티 코드가 매우 지저분해진다. 다행히도 임베디드 타입을 중복해서 사용하는 경우는 드물다.

## 값 타입과 불변 객체

값 타입은 복잡한 객체 사상을 조금이라도 단순화하려고 만든 개념이다. 따라서 **값 타입은 단순하고 안전하게 다루어야 한다.**

### 공유 참조

임베디드 타입은 같은 값 타입을 여러 엔티티에서 공유하면 위험하다.

```java
member1.setHomeAddress(new Address("OldCity"));
Address address = member1.getHomeAddress();

address.setCity("NewCity");
member2.setHomAddress(address);
```

회원1 주소도 바뀌는 문제가 있다. 이런 버그는 잘못하고 찾아내기가 까다롭다.

```java
member1.setHomeAddress(new Address("OldCity"));
Address address = member1.getHomeAddress();

Address newAddress = address.clone();
newAddress.setCity("NewCity");
member2.setHomAddress(newAddress);
```

`clone()` 으로 부작용을 피한다. 더 간단한 방법은 수정자 메소드를 모두 제거하는 것이다.

### 불변 객체

값 타입은 가능하면 불변 객체로 설계한다.

setter 는 닫고 생성자로 초기화 한다. (getter 는 optional)

## 값 타입 비교

- 동일성 비교: 인스턴스 참조 값을 비교, `==` 사용
- 동등성 비교: 인스턴스 값을 비교, `equals()` 사용

## 값 타입 컬렉션

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    @Embedded
    private Address homeAddress;

    @ElementCollection
    @CollectionTable(name = "FAVORITE_FOODS", joinColumn = @JoinColumn(name = "MEMBER_ID"))
    @Column(name = "FOOD_NAME")
    private Set<String> favoriteFoods = new HashSet<String>();

    @ElementCollection
    @CollectionTable(name = "ADDRESS", joinColumn = @JoinColumn(name = "MEMBER_ID"))
    @Column(name = "FOOD_NAME")
    private List<Address> addressHistory = new ArrayList<Address>();

    // ...
}
```

```java
@Embeddable
public class Address {
    private String city;
    private String street;
    private String zipcode;
}
```

```text
┌ FAVORITE_FOODS ───────┐
│ MEMBER_ID (PK, FK)    │
│ FOOD_NAME             │
└───────────────────────┘

┌ ADDRESS ──────────────┐
│ MEMBER_ID (PK, FK)    │
│ CITY (PK)             │
| STREET (PK)           │
| ZIPCODE (PK)          │
│ FOOD_NAME             │
└───────────────────────┘
```

관계형 데이터베이스에서 테이블은은 컬럼안에 컬렉션을 포함할 수 없기 때문에 별도의 관계형 테이블이 생성된다.

> `@CollectionTable` 을 생략하면 기본 값을 사용하여 매핑한다. 기본값: {엔티티이름}_{컬렉션 속성 이름}. 예를 들어 Member 엔티티의 addressHistory 는 Member_addressHistory 테이블과 매핑한다.

값 타입 컬렉션은 영속선 전이(Cascade) + 고아 객체 제거(ORPHAN REMOVE) 기능을 필수로 가진다고 볼 수 있다. 더하여 값 타입 컬렉션도 조회시 페치 전략을 선택할 수 있는데 기본 값은 LAZY 가 기본이다.

`@ElementCollection(fetch = FetchType.LAZY)`

```java
Member member = em.find(Member.class, 1L);

Address homeAddress = member.getHomeAddress(); // 이미 회원 조회시 같이 조회됨 (컬럼)

Set<String> favoriteFoods = member.getFavoriteFoods(); // LAZY

// 실제 컬렉션이 사용되는 시점에 쿼리가 실행됨
for (String favoriteFood : favoriteFoods) {
    System.out.println("favoriteFood : " + favoriteFood);
}

List<Address> addressHistory = member.getAddressHistory(); // LAZY

// 실제 컬렉션이 사용되는 시점에 쿼리가 실행됨
for (Address address : addressHistory) {
    System.out.println("address : " + address);
}
```

### 제약 사항

1. 값 타입은 식별자라는 개념이 없고 단순한 값들의 모음이기 때문에 값을 변경해버리면 데이터베이스에 저장된 원본 데이터를 찾기 어렵다.

#### Case

식별자가 100번인 회원이 관리하는 주소 값 타입 컬렉션을 변경하면 테이블에서 회원 100번과 관련된 모든 주소 데이터를 삭제하고 현재 값 타입 컬렉션에 있는 값을 다시 저장한ㄷ.

```sql
DELETE FROM ADDRESS FROM MEMBER_ID = 100
ISNERT INTO ADDRESS ...
// ...
```

실무에서는 값 타입 컬렉션 대신 매핑 테이블에 데이터가 많다면 값 타입 컬렉션 대신 일대다로 엔티티 매핑을 고려해야 한다.

여지껏 만든 값 타입 컬렉션을 만든다면 다음과 같다.

1. 새로운 엔티티를 만들고
2. 일대다 관계로 설정하고
3. 영속성 전이 + 고아 객체 제거

```java
@Entity
public class AddressHistory {
    @Id @GeneratedValue
    private Long id;

    @Emebedded
    private Address address;
}
```

```java
@Entity
public class Member {
    // ...

    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinColumn(name = "MEMBER_ID")
    private List<AddressHistory> addressHistory;
}
```

## 정리

- 엔티티 타입의 특징
  - 식별자가 있다. (`@Id`)
  - 생명주기가 있다. 생성하고, 영속화되고, 소멸되는 주기가 있다.
    - `em.persis(entity)`
    - `em.remove(entity)`
  - 공유할 수 있다. 회원 엔티티가 있다면 다른 엔티티에서 언제든지 회원 엔티티를 참조할 수 있다. (엔티티와 연관관계를 가질 수 있다.)

- 값 타입의 특징
  - 식별자가 없다.
  - 엔티티가 생명주기를 관리한다.
  - 공유하지 않는 것이 안전하다.
    - 공유해야 한다면 객체를 복사해야한다.
    - 하나의 주인만이 관리해야 한다.
    - 불변 객체로 만드는 것이 안전하다. (이건 엔티티도 포함)
