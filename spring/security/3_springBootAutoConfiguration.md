# Spring Boot Auto Configuration

원글 출처 : [Spring security](https://docs.spring.io/spring-security/site/docs/5.3.2.RELEASE/reference/html5/)

번역 출처 : [토리맘의 한글라이즈 프로젝트](https://godekdls.github.io/Spring%20Security/contents/)

시큐리티의 기본 설정을 활성화해서 `springSecurityFilterChain` 라는 이름의 서블릿 `Filter` 빈을 생성한다.

`user` 라는 사용자 이름과 콘솔에도 출력되는 랜덤 생성한 비밀번호를 가지는 `UserDetailsService` 빈을 생성한다.

서블릿 컨테이너에 `springSecurityFilterChain` 이란 이름의 `Filter` 빈을 등록해 모든 요청에 적용한다.

스프링 부트 설정은 간단하지만 많은 일을 수행한다.

- 어플리케이션의 모든 상호작용에 사용자 인증 요구
- 디폴트 로그인 폼 생성
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
