# 흐름 제어

프로세스의 앞, 중간, 뒤에 추가하여 공통 처리할 수 있는 것으로 `Filter`, `Interceptor`, `AOP` 가 있다.

## 비교

| | `Filter` | `Interceptor` | `AOP` |
| --- | --- | --- | --- |
| 단위 | `Servlet` | `Servlet` | `Method Proxy` |
| 실행 순서 (약식) | 가장 시작과 끝 (1, 5) | AOP 를 어라운드 (2, 4) | 가운데 (3) |
| 사용 인수 | `HttpServletRequest`, `HttpServletResponse` | `HttpServletRequest`, `HttpServletResponse` | `JoinPoint`, `ProceedingJoinPoint` |
| 컨텍스트 | Spring Context 외부 | Spring Context 내부 | Spring Context 내부 |
| 주 용도 | 인코딩 | 로그인 체크, 권한 체크, 요청 응답 암복호화 | 다양한 서비스 부가 기능 |

## `Filter`

`Filter` 는 `DispatcherServlet` 앞 단에 존재하여 **자원의 앞단 요청**과 **자원의 처리가 끝난 후의 응답** 의 대한 추가/공통 작업을 할 수 있다.

- `init()` 필터 인스턴스 초기화
- `doFilter()` 전/후 처리
- `destroy ()` 필터 인스턴스 종료

### `DispatcherServlet`

서블릿 컨테이너에 들어오는 요청을 가장 앞에서 받아 모든 요청을 핸들링한다. `Front Controller` Pattern 이라고 한다.

`DispatcherServlet` 그 외의 기능도 담당하지만 과감히 생략.

#### Front Controller Pattern

`Front Controller` 적용 이전에는 새로운 서블릿과 서블릿에 대응하는 URL 을 web.xml 에 일일히 작성해야 했다. 이 때문에 `web.xml` 의 양은 방대해지고 유지보수가 어려워졌다.

`Front Controller` 패턴은 모든 요청을 `Front Controller` 로 받은 후 적절한 Controller 로 요청을 전달한다.

## `Interceptor`

`Interceptor` 는 `DispatcherServlet` 가 **컨트롤러로 요청을 전달하기 전**과 **컨트롤러가 요청을 처리한 후**에 대한 추가/공통 작업을 할 수 있다.

- `preHandler()` 컨트롤러 메소드가 실행되기 전
- `postHandler()` 컨트롤러 메소드가 실행된 후 view 페이지가 렌더링 되기 전
- `afterCompletion()` view 페이지가 렌더링 된 후

## `AOP`

[AOP](./../spring/aop/1_intro.md)

`AOP` 는 `Filter` 와 `Interceptor` 보다 비즈니스 레벨의 세밀한 추가/ 공통 작업을 위해 사용한다.

- `@Before` : 메서드 실행 전
- `@After` : 메서드 실행 후
- `@AfterReturning` : 메서드 정상 반환 후
- `@AfterThrowing` : 메서드 예외 발생 후
- `@Around` : 메서드 수행 전와 메서드 수행 후

### 출처

- [Filter, Interceptor, AOP 차이 및 정리](https://goddaehee.tistory.com/154)
