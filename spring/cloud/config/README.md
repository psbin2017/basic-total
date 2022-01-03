# spring-cloud-config

## 주의 사항

### actuator refresh 시 `@PostConstruct` 초기화 되지 않음

- `ApplicationListener<EnvironmentChangeEvent>` 를 구현하여 `@PostConstruct` 의 메소드를 호출한다.
- 혹은 spring-cloud-bus, spring-cloud-config-monitor 를 구현하여 MQ 에 hook 을 등록하고 수행한다.
