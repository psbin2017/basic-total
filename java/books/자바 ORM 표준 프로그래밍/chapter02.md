# 🗂 2장 JPA 시작

## 객체 매핑 시작

### `@Entity`

클래스에 테이블을 매핑할 것을 JPA 에게 알린다.

### `@Table`

엔티티 클래스에 매핑할 테이블 정보를 작성한다. name 속성으로 테이블 매핑을 알려준다.

### `@Id`

엔티티 클래스의 필드에 기본 키에 해당한다.

### `@Column`

엔티티 클래스의 필드를 컬럼에 매핑한다.

## JPA 표준 속성

프로젝트 기준으로 작성한다.

| 프로젝트 기준 | 책 기준 | 설명 |
| --- | --- | --- |
| `spring.datasource.url` | `javax.persistence.jdbc.url` | 접속 URL |
| `spring.datasource.username` | `javax.persistence.jdbc.user` | 접속 아이디 |
| `spring.datasource.password` | `javax.persistence.jdbc.password` | 접속 비밀번호 |
| `spring.datasource.driver-class-name` | `javax.persistence.jdbc.driver` | 드라이버 명 |

## 방언

특정 데이터베이스만 고유한 기능을 JPA 에서는 방언이라고 하며 애플리케이션 개발자가 특정 데이터베이스에 종솓되는 기능을 많이 사용하면 나중에 데이터베이스를 교체하기 어렵기에 방언을 설정하여 이를 해결한다.

데이터베이스 방언에 대한 설정은 다음과 같다.

| 프로젝트 기준 | 책 기준 | 설명 |
| --- | --- | --- |
| `spring.jpa.database-platform` | `hibernate.dialect` | 데이터베이스 방언 설정 |

## 애플리케이션 개발

실습 생략 부분

1. 엔티티 매니저 설정
   - **애플리케이션 전체에서 한번 생성하고 공유하여 사용해야 한다.**
   - **데이터베이스 커넥션과 관계가 있으므로 스레드간 공유나 재사용하면 안된다.**
2. 트랜잭션 관리
3. 비즈니스 로직

## JPQL

JPQL 은 엔티티 객체를 대상으로 쿼리한다.
