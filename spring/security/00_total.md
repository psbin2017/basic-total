# 정리

## 용어 사전

- 인증 (Authentication): 사용자가 누구인지 확인하는 절차
- 인가 (Authorization): 사용자를 식별하여 알맞은 권한을 부여
- 자격증명 (Credential): 인증에 필요한 정보, 단기는 주로 *사용자 이름과 비밀번호* 장기는 *세션과 OAuth 토큰*으로 동일한 보안 수준을 가진다.
- 적응형 단방향 함수 (Adaptive One-Way Function): 암호를 안전하게 저장하기 위한 변환 기법
- 워크 팩터 (Work Factor): 적응형 단방향 함수를 수행하는 구현체
- CSRF (Cross Site Request Forgery): 사이트 간 요청 위조 공격, 방어하기 위해선 공격하는 사이트에서 제공할 수 없는 무언가를 요청하여 두 요청을 구분해야 한다.
- 동기화 토큰 패턴 (Synchronizer Token Pattern): CSRF 방어 기법으로 세션과 별개로 HTTP 요청시 서버에서 임의로 생성한 CSRF 토큰을 비교하여 멱등성을 보장한다.
- SameSite 쿠키: CSRF 방어 기법으로 외부사이트에서 보내는 요청에서 쿠키를 사용하지 않음을 지정한다.
- 필터 (Filter): 클라이언트 요청에 대한 절차 혹은 검증 처리
- 필터 체인 (Filter Chain) : 여러 개의 필터가 연쇄적으로 묶여 있다. 이 과정에서 필터의 순서는 매우 중요하다.

## 주요 객체 정리

| 객체 | 설명 |
| --- | --- |
| `PasswordEncoder`             | 단방향 변환을 수행하는 인터페이스 |
| `DelegatingPasswordEncoder`   | 암호를 인코딩하기 위한 호환성을 지원하는 객체 |
| `DelegatingFilterProxy`       | 서블릿 컨테이너의 라이프사이클과 스프링의 `ApplicationContext` 와의 연결 |
| `FilterChainProxy`            | 1. 서블릿을 지원하는 역할 <br/> 2. `SecurityFilterChain` 를 통해 여러 `Filter` 인스턴스로 위임 <br/> 3. `DelegatingFilterProxy` 로 감싸져 있음 |
| `SecurityFilterChain`         | `Filter` 의 묶음 단위 |
| `ExceptionTranslationFilter`  | `AccessDeniedException`, `AuthenticationException` 과 같은 인증 인가에 대한 예외를 처리한다. <br/> 단, 애플리케이션에서 명시된 예외를 던지지 않으면 아무일도 하지 않는다. |

## 보안 헤더 정리

```text
// 컨텐츠 보호를 위한 캐시 비활성화, 애플리케이션에서 헤더 적용시 정적 리소스 캐시 별도 적용 가능
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0

// 컨텐츠 보호를 위한 컨텐츠 스니핑(유효한 파일을 찾아 XSS 공격) 비활성화
X-Content-Type-Options: nosniff

// https 를 이후 생략하고 요청해도 브라우저가 https 로 요청하도록 HSTS 호스트로 취급함
Strict-Transport-Security: max-age=31536000 ; includeSubDomains

// 컨텐츠 보호를 위한 클릭 재킹(웹사이트에 웹사이트를 넣는 공격) 비활성화
X-Frame-Options: DENY

// 1 차적인 XSS 공격 보호
X-XSS-Protection: 1; mode=block

// 클라이언트에게 보안 정책을 전달하여 XSS 같은 컨텐츠 인젝션 공격 취약점 개선
Content-Security-Policy
// 또는 Content-Security-Policy-Report-Only

// 사용자 마지막 방문 페이지를 가지고 있는 referrer 를 관리함, 예제는 브라우저가 도착지를 알림
Referrer-Policy: same-origin

// 개발자가 원하는 대로 특정 API 와 브라우저의 웹기능을 활성화/비활성화 하는 등의 동작 수정을 지원
Feature-Policy: geolocation 'self'

// 쿠키, 로컬 스토리지 등의 브라우저 단 데이터를 삭제
Clear-Site-Data: "cache", "cookies", "storage", "executionContexts"
```

## 스프링 시큐리티 수행 범위

- 애플리케이션의 모든 상호작용에 사용자 인증 요구
- 기본 로그인 폼 생성
- `user` 라는 이름과 콘솔에 출력한 비밀번호를 사용한 폼 기반 인증 지원 (위 예제에서 비밀번호는 `8e557245-73e2-4286-969a-ff57fe326336` 이다).
- BCrypt로 저장할 비밀번호 보호
- 사용자 로그아웃 지원
- CSRF 공격 방어
- Session Fixation 방어
- 보안 헤더 통합
- HTTP Strict Transport Security로 요청을 보호
- X-Content-Type-Options 통합
- Cache Control (애플리케이션에서 특정 정적 리소스에 캐시를 허용하도록 재정의할 수 있다.)
- X-XSS-Protection 통합
- X-Frame-Options 통합으로 클릭재킹 방어 지원
- 서블릿 API 메소드 통합:
  - `HttpServletRequest#getRemoteUser()`
  - `HttpServletRequest.html#getUserPrincipal()`
  - `HttpServletRequest.html#isUserInRole(java.lang.String)`
  - `HttpServletRequest.html#login(java.lang.String, java.lang.String)`
  - `HttpServletRequest.html#logout()`
