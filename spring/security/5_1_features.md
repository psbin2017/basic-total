# Authentication

원글 출처 : [Spring security](https://docs.spring.io/spring-security/site/docs/5.3.2.RELEASE/reference/html5/)

번역 출처 : [토리맘의 한글라이즈 프로젝트](https://godekdls.github.io/Spring%20Security/contents/)

| 용어 | 설명 |
| --- | --- |
| 인증 (Authentication) | 사용자가 누구인지 확인하는 절차 |
| 인가 (Authorization)) | 사용자를 식별하여 알맞은 권한을 부여 |

인증은 특정 리소스에 접근하려고 하는 사용자가 누구인지 확인한다.

아이디와 패스워드를 입력하는 것이 사용자를 인증하는 것의 예이다.

인증이 완료되면 사용자를 식별하고 권한을 부여하는 인가를 실시한다.

## Password Storage

`PasswordEncoder` 인터페이스는 비밀번호를 안전하게 저장할 수 있도록 단방향 변환을 수행한다.

스프링은 단방향 변환은 적응형 단방향 함수(`Adaptive One-Way Function`)를 권장하며 이는 많은 리소스를 소모한다.

시스템에 따라 적응형 단방향 함수를 수행하는 워크 팩터(`Work Factor`)를 지정할 수 있으며 적절히 튜닝하는 것을 권고한다.

비밀번호를 매번 검증하는 것은 어플리케이션 성능에 영향을 주기 때문에 *사용자 이름과 비밀번호*인 장기 credential 은 단기 credential 인 *세션, OAuth 토큰*으로 바꾸는 것이 좋다.

단기 credential 은 동일한 보안 수준을 유지한다.

### DelegatingPasswordEncoder

- 이미 많은 애플리케이션이 마이그레이션이 어려운 이전 방식의 비밀번호를 인코딩 하고 있다.
- 비밀번호를 저장하기 위한 방법은 기술에 따라 다시 바뀔 것이다.
- 하위 호환성을 보장하지 않는 업데이트가 불가능하다.

이러힌 이슈로 `DelegatingPasswordEncoder` 를 도입하여 문제를 해소하였다.

- 비밀번호를 현재 권장하는 저장 방식으로 인코딩함을 보장한다.
- 검증은 최신과 레거시 형식을 모두 지원한다.
- 이후에 인코딩을 변경할 수 있다.

### Password Storage Format

```text
{id}encodedPassword
```

### Password Encoding

생성자에 전달한 `idForEncode` 가 비밀번호를 인코딩할 때 사용할 `PasswordEncoder` 를 결정한다.

```java
String idForEncode = "bcrypt";
Map encoders = new HashMap<>();
encoders.put(idForEncode, new BCryptPasswordEncoder());
encoders.put("noop", NoOpPasswordEncoder.getInstance());
encoders.put("pbkdf2", new Pbkdf2PasswordEncoder());
encoders.put("scrypt", new SCryptPasswordEncoder());
encoders.put("sha256", new StandardPasswordEncoder());

PasswordEncoder passwordEncoder = new DelegatingPasswordEncoder(idForEncode, encoders);
```

`DelegatingPasswordEncoder` 는 `password` 인코딩 결과를 `BCryptPasswordEncoder` 로 위임하며, 접두어(prefix) 는 `{bcrypt}` 를 사용한다.

```text
{bcrypt}$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG
```

### Password Matching

`{id}` 를 기반으로 매칭되며, 생성자에서 제공한 `PasswordEncoder` 로 매핑된다.

비밀번호와 `{id}` 가 매핑되지 않으면 예외가 발생하며, 이는 `DelegatingPasswordEncoder.setDefaultPasswordEncoderForMatches(PasswordEncoder)` 로 커스텀 할 수 있다.

### Spring Boot CLI

```text
spring encodepassword password
{bcrypt}$2a$10$X5wFBtLrL/kHcmrOGGTrGufsBX8CJ0WpQpF3pgeuxBB/H73BK1DW6
```

### BCryptPasswordEncoder

`BCryptPasswordEncoder` 구현체는 널리 사용되고 있는 알고리즘으로 비밀번호를 해싱하며 의도적으로 느리게 동작하기 떄문에 비밀번호를 해독하기 어렵다.

```java
// Create an encoder with strength 16
BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(16);
String result = encoder.encode("myPassword");
assertTrue(encoder.matches("myPassword", result));
```

### Argon2PasswordEncoder

`Argon2PasswordEncoder` 구현체는 메모리를 많이 사용하며 느리게 동작하며 최신 구현체는 `BouncyCastle` 라이브러리가 필요하다.

```java
// Create an encoder with all the defaults
Argon2PasswordEncoder encoder = new Argon2PasswordEncoder();
String result = encoder.encode("myPassword");
assertTrue(encoder.matches("myPassword", result));
```

### Pbkdf2PasswordEncoder

`Pbkdf2PasswordEncoder` 구현체는 마찬가지로 느리게 동작하며 해독하기 어렵다.

FIPS 인증이 필요하다면 이 알고리즘이 적합하다.

```java
// Create an encoder with all the defaults
Pbkdf2PasswordEncoder encoder = new Pbkdf2PasswordEncoder();
String result = encoder.encode("myPassword");
assertTrue(encoder.matches("myPassword", result));
```

### SCryptPasswordEncoder

이하 동문

```java
// Create an encoder with all the defaults
SCryptPasswordEncoder encoder = new SCryptPasswordEncoder();
String result = encoder.encode("myPassword");
assertTrue(encoder.matches("myPassword", result));
```

### Other PasswordEncoders

호환성을 가진 `PasswordEncoder` 구현체도 많다.

그러나 안전하지 않다는 것을 나타내기 위하여 `@Deprecated` 를 명시하였다.

### Password Storage Configuration

스프링 시큐리티는 기본값으로 `DelegatingPasswordEncoder` 를 사용한다.

그러나 `PasswordEncoder` 를 스프링 빈으로 정의하면 변경할 수 있다.
