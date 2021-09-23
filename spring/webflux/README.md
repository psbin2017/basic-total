# Spring WebFlux

## Tip

### Remove support `@RequestParam`

요약 : `@PostMapping` 은 `@RequestParam` 을 지원하지 않는다.
관련 링크 : [SPRING PROJECT LINK](https://github.com/spring-projects/spring-framework/issues/20067)
해결 방법 : `ServerWebExchange#getFormData()` 를 사용하거나 DTO 를 사용하여 적용한다.
