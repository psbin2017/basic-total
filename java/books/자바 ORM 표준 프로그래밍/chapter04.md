# 🗂 4장 엔티티 매핑

- 객체와 테이블 매핑: `@Entity`, `@Table`
- 기본 키 매핑: `@Id`
- 필드와 컬럼 매핑: `@Column`
- 연관 관계 매핑: `@ManyToOne`, `@JoinColumn`

매핑 정보는 XML 방식도 있지만 어노테이션을 사용하는 쪽이 직관적이다.

## `@Entity`

테이블과 매핑할 클래스로 JPA 의 관리 대상이 된다. 기본값을 설정하지 않으면 클래스 이름을 사용한다.

- 기본 생성자는 필수다. (`protected` 또는 `public`)
- `final` 클래스, `enum`, `interface`, `inner` 클래스에는 사용할 수 없다.
- 저잘할 필드는 `final` 을 사용하면 안된다.

## `@Table`

엔티티와 매핑할 테이블을 지정한다. 생략시 매핑한 엔티티의 이름을 테이블 이름으로 사용한다.

| 속성 | 기능 |
| --- | --- |
| catalog | catalog 기능이 있는 데이터베이스에서 catalog 를 매핑한다. |
| schema | schema 기능이 있는 데이터베이스에서 schema 를 매핑한다. |
| uniqueConstraints | DDL 생성 시 유니크 제약 조건을 만든다. 2개 이상의 복합 유니크 제약조건도 만들 수 있다. 이 기능은 스키마 자동 생성 기능을 사용하여 DDL 생성시만 사용된다. |

## `@Id`

기본 키 생성 전략

- 직접 할당: 기본 키를 애플리케이션에서 직접 할당한다.
- 자동 생성: 대리 키 사용 방식
  - IDENTITY: 기본 키 생성을 데이터베이스에 위임한다.
  - SEQUENCE: 시퀀스를 사용하여 기본 키를 할당한다.
  - TABLE: 키 생성 테이블을 사용한다.

### 직접 할당

- 자바 기본형
- 자바 Wrapper 형
- String
- java.util.Date
- java.sql.Date
- java.math.BigDecimal
- java.math.BigInteger

기본 키 직접 할당 전략은 `em.psersist()` 로 엔티티를 저장하기 전에 애플리케이션에서 기본 키를 직접 할당하는 방법이다.

### IDENTITY

MySQL, PostgreSQL, SQL Server, DB2 에서 사용하는 전략으로 AUTO_INCREMENT 기능으로 순서대로 값을 채워준다.

지정 방법은 추가로 `@GeneratedValue(strategy = GenerationType.IDENTITY)` 어노테이션을 지정하면 된다.

이 전략은 트랜잭션을 지원하는 쓰기 지연이 동작하지 않는다. (생성 이후에 식별자를 구할수 있기 때문에)

### SEQUENCE

Oracle, PostgreSQL, DB2, H2 에서 사용하는 전략이다.

```java
@Entity
@SequenceGenerator(
    name = "BOARD_SEQ_GENERATOR",
    sequenceName = "BOARD_SEQ", // 매핑할 데이터베이스 시퀀스 명
    initialValue = 1,
    allocationSize = 1
)
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
                    generator = "BOARD_SEQ_GENERATOR")
    private Long id;
}
```

IDENTITY 전략과 같지만 내부 동작 방식은 다르다. 시퀀스 식별자를 먼저 조회후 엔티티에 할당하고 영속성 컨텍스트를 저장한다. 이후 트랜잭션 커밋하여 플러시가 일어나면 엔티티를 데이터베이스에 저장한다.

| 속성 | 기능 | 기본 값 |
| --- | --- | --- |
| name | 시퀀스 식별자 생성기 명 | 필수 |
| sequenceName | 시퀀스 명 | hibernate_sequence |
| initialValue | DDL 생성시 사용되는 처음 시작 수 | 1 |
| allocationSize | 시퀀스 한 번 호출에 증가하는 수 (성능 최적화에 사용) | 50 |
| catalog, schema | catalog, schema 이름에 해당 |  |

```sql
CREATE sequence [sequenceName]
start with [initialValue] increment by [allocationSize]
```

`@SequenceGenerator` 는 대상 필드에 사용해도 된다.

### TABLE

```java
@Entity
@TableGenerator(
    name = "BOARD_SEQ_GENERATOR",
    table = "MY_SEQUENCES",
    pkColumnValue = "BOARD_SEQ", 
    allocationSize = 1
)
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE,
                    generator = "BOARD_SEQ_GENERATOR")
    private Long id;
}
```

| 속성 | 기능 | 기본 값 |
| --- | --- | --- |
| name | 테이블 식별자 생성기 명 | 필수 |
| talbe | 키 생성 테이블 명 | hibernate_sequence |
| pkColumnName | 시퀀스 컬럼 명 | sequence_name |
| valueColumnName | 시퀀스 값 컬럼 명 | next_val |
| pkColumnValue | 키로 사용할 값 이름 | 엔티티 이름 |
| initialValue | 초기 값, 마지막으로 생성된 값 기준 | 0 |
| allocationSize | 시퀀스 한 번 호출에 증가하는 수 (성능 최적화에 사용) | 50 |
| catalog, schema | catalog, schema 이름에 해당 |  |
| uniqueConstraints (DDL) | 유니크 제약 조건 지정 | |

```text
{table}

{pkColumnName} {valueColumnName}
{pkColumnValue} {initialValue}
```

시퀀스 전략과 비교하여 SELECT - UPDATE 라는 2번 통신이라는 단점이 있다. 최적화 방법은 SEQUENCE 와 동일하다.

### AUTO

```java
@Enitity
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
}
```

기본 값이자, 선택한 데이터베이스 방언에 따라 자동으로 선택한다.

### 권장하는 식별자 선택 전략

1. null 불가능
2. 유일해야 한다.
3. 불변

- 자연키: 비즈니스의 의미 있는 키, 주민등록번호 등 (비권장)
- 대리키: 시퀀스, auto_increment

## `@Column`

| 속성 | 기능 |
| --- | --- |
| name | 컬럼 명 지정 |
| nullable | 널 가능 여부 지정 |
| length | 컬럼 길이값 지정 |
| insertable (잘 사용안함) | 엔티티 저장 시 이 필드도 같이 저장한다. (default:`true`) |
| updatable (잘 사용안함) | 엔티티 수정 시 이 필드도 같이 수정한다. (default:`true`) |
| table (잘 사용안함) | 엔티티를 두 개 이상의 테이블에 매핑시 사용. 지정 필드를 다른 테이블에 매핑 가능함 |
| columnDefinition | 컬럼 정보를 직접 준다. |
| precision, scale | BigDecimal 타입에서 사용하며 precision 은 소수점을 포함한 전체 자리수를 scale 은 소수의 자리수다. double 이나 float 에는 적용되지 않으며, 아주 작은 소수나 아주 큰 숫자를 다룰 때만 사용한다. |

단, 이 기능은 스키마 자동 생성 기능을 사용하여 DDL 생성시만 사용된다.

| 어노테이션 | 설명 |
| --- | --- |
| `@Enumerated` | 자바의 enum 타입 매핑 |
| `Temporal` | 날짜 타입 매핑 |
| `Lob` | BLOB, CLOB 타입 매핑 |
| `Transient` | 특정 필드를 데이터베이스에 매핑하지 않음 |
| `Access` | JAP 가 엔티티에 접근하는 방식 지정 |

### `@Enumerated`

- `EnumType.ORDINAL` : enum 순서를 데이터베이스에 저장. 저장되는 데이터 크기가 작지만, 순서를 변경할 수 없다.
- `EnumType.STRING` : enum 이름을 데이터베이스에 저장. 순서가 변경되어도 상관 없지만, 데이터 크기가 상대적으로 크다.

### `Temporal`

- `TemporalType.DATE` : 날짜, 데이터베이스 date 타입과 매핑 (예: 2013-10-11)
- `TemporalType.TIME` : 날짜, 데이터베이스 time 타입과 매핑 (예: 11:11:11)
- `TemporalType.TIMESTAMP` : 날짜, 데이터베이스 timestamp 타입과 매핑 (예: 2013-10-11 11:11:11)

필수로 `TemporalType` 을 지정해야하며 생략시 자바의 Date 와 유사한 timestamp 로 정의된다. 방언에 따라 다르게 정의된다.

- datetime: MySQL
- timestamp: H2, Oracle, PostpreSQL

### `@Lob`

- CLOB: String, char[], java.sql.CLOB
- BLOB: byte[], java.sql.BLOB

### `@Transient`

데이터베이스에 저장 또는 조회 용도로 사용하지 않고 객체에 임시로 값을 보관시 사용한다.

### `@Access`

엔티티 데이터에 접근하는 방식을 지정한다.

- `AccessType.FIELD`: 필드에 직접 접근하며 필드 권한이 private 여도 접근 가능하다.
- `AccessType.PROPERTY`: 접근자를 사용한다.

설정하지 않은 경우 `@Id` 필드에 위치에 따라 접근 방식이 결정된다.

```java
@Entity
// @Acess(AcessType.FIELD)
public class Member {
    @Id
    public String id;
}

@Entity
// @Access(AcessType.PROPERTY)
public class Member {
    public String id;

    @Id
    public String getId() {
        return id;
    }

    @Transient
    private String firstName;

    @Transient
    private String lastName;

    @Access(AcessType.PROPERTY)
    public String getFullName() {
        return firstName + lastName;
    }
}
```

기본은 필드 접근 방식을 사용하고 `getFullName()` 만 프로퍼티 접근 방식을 사용한다. 저장시 회원 테이블의 FULLNAME 컬럼에 firstName + lastName 의 결과가 저장된다.

## `@ManyToOne`

## `@JoinColumn`

## 스키마 자동 생성

`hibernate.hbm2ddl.auto` : `spring.jpa.hibernate.ddl-auto` 은 애플리케이션 실행 시점에 데이터베이스 테이블을 자동으로 생성한다.

| 속성 | 설명 | 사용법 |
| --- | --- | --- |
| `create` | 실행시 DROP - CREATE | 개발 초기 단계/개발 환경 |
| `create-drop` | 실행시 DROP - CREATE - 종료시 DROP | 개발 환경/CI 서버 |
| `update` | 테이블과 엔티티 매핑정보를 비교하여 변경 사항만 수정 | 개발 초기 단계/테스트 서버 |
| `validate` | 테이블과 엔티티 매핑정보를 비교하여 경고를 남기고 애플리케이션을 실행하지 않는다. | 테스트 서버/스테이징 서버/운영 서버 |
| `none` | 자동 생성 기능을 사용하지 않는다. (유효하지 않은 옵션 값이다) | 운영 서버 |

`hibernate.show_sql` : `spring.jpa.show-sql` 은 콘솔에 실행되는 테이블 생성을 출력할 수 있다.

## 이름 매핑 전략 변경

직접 매핑을 작성하여 변경하는 방법

```java
// ...

@Column(name="role_type")
String roleType

// ...
```

설정을 통한 변경 방법

`hibernate.ejb.naming_strategy` : `spring.jpa.hibernate.naming.physical-strategy` ... `org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl` 로 전역으로 카멜케이스로 선언된 필드도 언더스코어 표기법으로 생성할 수 있다.
