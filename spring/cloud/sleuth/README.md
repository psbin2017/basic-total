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

## 참고 링크

[LINK]<https://github.com/openzipkin/brave-example>
