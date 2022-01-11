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

### mutate 시 주의사항

원본 요청/응답 객체를 mutate 하는 과정에서 주의

`HttpHeaders.addAll()` 시 일치하는 키가 이미 존재하는 경우 , 로 추가된다.

`HttpHeaders.setAll()` 시 일치하는 키가 이미 존재하는 경우 순서가 마지막 헤더에 더해진다. (기존 헤더 값이 유지된다.)

따라서 , 로 추가되는 것을 의도한 것이 아니라면 `HttpHeaders.setAll()` 이 정상적으로 추가가 된다.

```java
// addAll start
HttpHeaders httpHeaders = new HttpHeaders();
httpHeaders.setConnection("keep-alive");
ServerHttpRequest mutateAddAllRequest = exchange.getRequest().mutate()
        .headers(originHeaders -> originHeaders.addAll(httpHeaders))
        .build();
exchange.mutate().request(mutateAddAllRequest).build();

log.info("[addAll] :: " + exchange.getRequest().getHeaders().toString());
// addAll end

// setAll start
Map<String, String> httpHeadersV2 = new HashMap<>();
httpHeadersV2.put("Connection", "keep-alive");
ServerHttpRequest mutateSetAllRequest = exchange.getRequest().mutate()
        .headers(originHeaders -> originHeaders.setAll(httpHeadersV2))
        .build();
exchange.mutate().request(mutateSetAllRequest).build();

log.info("[setAll] :: " + exchange.getRequest().getHeaders().toString());
// setAll End
```
