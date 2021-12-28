# spring cloud sleuth

> 마이크로 서비스의 분산된 트래픽에 대한 로그를 한 곳으로 모아 이슈 트래킹에 용이성을 확보

## 상황 예시

spring cloud gateway 로 라우팅 되는 아키텍쳐 구조라고 가정한다.

내 주문 목록 조회

1. gateway server
2. order api

와 같은 동선이 된다. 또한 요구 사항에 따라, 마이크로 서비스가 다른 마이크로 서비스가 호출됨에 따라 하나의 클라이언트 트랜잭션에 여러 곳에서 로그가 발생되게 될 것이다.

## 구조

설정에 따라 구조가 다를 수 있지만 일반적인 로그 포맷은 다음과 같이 구성된다.

`INFO [${serviceName}, ${traceId}, ${spanId}]`

마이크로 서비스는 같은 Trace ID 를 공유하게 되면서 다른 Span ID 를 가지게 된다.

## 주의 사항

- 2.2.X 버전대는 EOS 직전이라 신규 프로젝트라면 3.1.X 를 고려할 것
- zipkin 말고도 kibana 또는 splunk 를 사용할 수 있다.
- RabbitMQ 나 kafka 와도 함께 사용할 수 있다. (당연히 로깅 공급자)

### WebFlux / Reactive

리액티브 형태로 논블록킹으로 변경하게 되면 traceId 와 spanId 과 올바르게 찍히지 않는 이슈가 있다.

정확한 케이스는 아니지만 논블록킹으로 바뀌게 되면서

- 요청을 받는 시점의 스레드의 traceId, spanId
- 응답을 처리하는 시점의 스레드의 traceId, spanId

양쪽 스레드의 대상이 달라 발생하는 사이드 이펙트 인것으로 추정된다.

이 문제로 같은 논블록킹에 해당하는 경우에는 traceId, spanId 을 유지하여 트래킹 가능하지만, 범위를 벗어나 다른 논블록킹 동작을 수행하는 경우 (Mono/Flux 한 단위로 생각하는게 좋을 듯) 기존 값을 유지할 수 없게 된다.

아직 명확한 해결책은 없지만 성능을 감수하고 적용하는 방법은 `WebFluxSleuthOperators.withSpanInScope` API 로 적용하는 것이다.

- [document]<https://docs.spring.io/spring-cloud-sleuth/docs/current-SNAPSHOT/reference/html/integrations.html#sleuth-reactor-integration>

아직 비 코드레벨로 비동기 기능을 수행할 수 없다. (성능 이슈로 이야기되고 있으며, 코드레벨로 심어 컨트롤 할 것을 권고하고 있다.)

```java
return someService.doSomeThing()
    .map(someData -> {
        // 호출된 컨텍스트와의 traceId, spandId 를 보장 O
        WebFluxSleuthOperators.withSpanInScope(tracer, currentTraceContext, exchange, () -> log.info("some log"));

        // 호출된 컨텍스트와 traceId, spanId 를 보장 X
        log.info("some log v2");

        // ...
    });
```

동작은 의도대로 가져오지만, 성능 저하를 의심해야될지도 모르겠다. (블록킹으로 바뀌면 의미가 없어져 버리는데...)

## 참고 링크

[LINK]<https://github.com/openzipkin/brave-example>
