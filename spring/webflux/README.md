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
