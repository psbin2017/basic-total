# spring starter cache

## cache test

`ToySecondComponent` : `@PostConstruct` 로 캐시 메소드 호출해도 캐싱되지 않아 직접 호출이 이루어짐 **WORST CASE** [stackoverflow](https://stackoverflow.com/questions/28350082/spring-cache-using-cacheable-during-postconstruct-does-not-work)

요약하면 해당 시점에는 프록시 인터셉터가 완전히 로드되었음을 보장하지 못하여 직접 호출로 이어진다.

`ToyRunner` : `CommandLineRunner` 로 캐시 메소드를 호출하여 정상적으로 캐싱됨.

`ToyEventListener` : `ApplicationListener<ApplicationReadyEvent>` 로 캐시 메소드를 호출하여 정상적으로 캐싱됨.
