# Spring WebFlux

## Keep

### `.map()` 은 `null` 을 반환하지 않아야 한다

요약 : null 을 반환하면 실행되는 시점에 Mono(Flux) 실행 과정에서 NPE 발생

> java.lang.NullPointerException: The mapper returned a null value.

올바른 코딩 방법 : `Mono.justOrEmpty()` 또는 Optional 로 null 이 아님을 보장하도록 해야한다. 기존 MVC 와 원칙상으로는 동일한 룰이지만 에러 로그가 다르기에 기억할 것!

## Tip

### Remove support `@RequestParam`

요약 : `@PostMapping` 은 `@RequestParam` 을 지원하지 않는다.
관련 링크 : [SPRING PROJECT LINK](https://github.com/spring-projects/spring-framework/issues/20067)
해결 방법 : `ServerWebExchange#getFormData()` 를 사용하거나 DTO 를 사용하여 적용한다.
