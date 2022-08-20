# Spring WebFlux

## Keep

### `.map()` 은 `null` 을 반환하지 않아야 한다

요약 : null 을 반환하면 실행되는 시점에 Mono(Flux) 실행 과정에서 NPE 발생

> java.lang.NullPointerException: The mapper returned a null value.

올바른 코딩 방법 : `Mono.justOrEmpty()` 또는 Optional 로 null 이 아님을 보장하도록 해야한다. 기존 MVC 와 원칙상으로는 동일한 룰이지만 에러 로그가 다르기에 기억할 것!

### PrematureCloseException

[Netty Link](https://projectreactor.io/docs/netty/release/api/reactor/netty/http/client/PrematureCloseException.html)

연결이 조기에 닫혔다는 신호 오류로 자매품 `AbortException`, `Erros$NavtiveIoException`, `ClientAbortException`, `EofException` 와 `broken pipe`, `Connect reset to peer` 등이 있다. 기존 `ClientAbortException` 스택에서 불친절? 한 에러 문구를 헤더, 바디, 응답 상태 유무 등에 따라 좀 더 상세한 메시지로 예외를 생성하여 내려준다.

다만, 문제는 현재 사용 중인 버전에 따라 auto-configure 로 등록하는 [AbstractErrorWebExceptionHandler](https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/reactive/error/AbstractErrorWebExceptionHandler.java) 가 `PrematureCloseException` 를 예외로 처리하지 않는다.

개인적인 판단으로는 `DISCONNECTED_CLIENT_EXCEPTIONS` 에 포함되는게 맞을거 같은데 빈도수를 줄이는 방법은 다음과 같다.

- WebClient (HttpClient) 를 사용할 때 풀링 방식을 사용중이라면 사용하지 않는 것을 고려한다.
- 추가로 compress 설정을 true 로 변경한다.

왜 줄어드는지는 아직 상세하게 파지 못했지만 `onInboundClose` 이벤트 이전에 `AbortException` 으로 핸들링 되거나 다른 예외로 핸들링되어서 내려가는게 아닌가 싶다. (이벤트 리스너로 전파가 되는게 아니라 그 앞에 처리되는게 아닐지 의문)

더해서 connection pool 방식에 대해 고민이 필요한데 일반적으로 DB 의 경우 커넥션 풀이 워낙 당연하다시피 구성 요소로 포함되는데 Http / TCP 통신에서 netty 는 connection pool 방식을 사용할 수 있다. 풀이 관리하는게 일반적으로 유리하겠지만 구성하고 있는 아키텍처에 따라 커넥션 풀이 유효하지 않을 수도 있다.

- - -

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

### Spring-Security 와 WebFilter + Sleuth 사용 시 주의사항

작성시점 버전

- Spring-Boot 2.4.x
- Spring-Cloud 2020.0.x

Spring-Security 가 등록하는 WebFilter 는 등록시 -100 의 순서를 가지며 나머지 후행 동작하는 필터와 Security 필터와 결합하기 위해서 Flux.iterable 로 동작하게끔 하는데, 코드에서 `flatMap()` 으로 동작하게끔 바뀌어 로깅시 `Reactor` 스레드로 동작하던 스레드가 `Parallel` 스레드로 동작이 바뀐다.

람다 내부에서 동작하는 로깅은 Sleuth 가 성능에 영향을 줄 수 있어 MANUAL 로 변경됨에 따라 `Parallel` 스레드에서 유지하려면 보정해주는 API 를 사용하거나 기본 단계를 조정해야된다.

비교적 쉬운 해결 방법은 2개 정도 있는 것으로 보임

1. 스프링 시큐리티 WebFilter 이후에 동작하는 WebFilter 로그에 대해 Sleuth API 로 로그를 TraceId / SpanId 를 보정한다. (영향도가 제일 적지만 코드 반영을 가장 많이 해야함.)
2. 용도에 따라 스프링 시큐리티 WebFilter 보다 먼저 동작해도 되는 WebFilter 라면 앞으로 조정한다. (Spring-Securiy 의 필터 순서가 변경되면 또 변경해주어야함.)

어려운 방법은..

1. 필터를 등록하는 빈을 Override 하고 스프링 시큐리티 적용시 사용하는 WebFilterProxy 객체의 순서를 조정한다.

현재 시큐리티로 등록되는 WeFilterProxy 는 순서가 하드코딩으로 박혀있다.
