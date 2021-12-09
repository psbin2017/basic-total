# spring starter cache

## cache test

`ToySecondComponent` : `@PostConstruct` 로 캐시 메소드 호출해도 캐싱되지 않아 직접 호출이 이루어짐 **WORST CASE** [stackoverflow](https://stackoverflow.com/questions/28350082/spring-cache-using-cacheable-during-postconstruct-does-not-work)

요약하면 해당 시점에는 프록시 인터셉터가 완전히 로드되었음을 보장하지 못하여 직접 호출로 이어진다.

`ToyRunner` : `CommandLineRunner` 로 캐시 메소드를 호출하여 정상적으로 캐싱됨.

`ToyEventListener` : `ApplicationListener<ApplicationReadyEvent>` 로 캐시 메소드를 호출하여 정상적으로 캐싱됨.

## fail over

캐시 관련 어노테이션은 `CacheErrorHandler` 구현하여 등록하여 `ConnectionException` 등과 같은 예외 발생시에 대한 핸들링이 가능하다.

### in Redis

레디스의 경우 클러스터링으로 구현된 경우 `ConnectionException` 등과 같은 예외 상황에 대해서 다음과 같은 설정으로 유연하게 fail over 에 대응이 가능하다.

```java
@Bean
public ClusterTopologyRefreshOptions clusterTopologyRefreshOptions() {
    return ClusterTopologyRefreshOptions
            .builder()
            // 1.
            .enablePeriodicRefresh(Duration.ofSeconds(30L))
            .enableAllAdaptiveRefreshTriggers()
            .build();
}

@Bean
public RedisConnectionFactory redisConnectionFactory() {
    // server configuration
    RedisClusterConfiguration serverConfig = new RedisClusterConfiguration();
    serverConfig.setClusterNodes(redisClusterProperties.getNodes().stream()
            .map(node -> {
                HostAndPort hostAndPort = HostAndPort.parse(node);
                log.info("node : {}", hostAndPort.toString());
                return new RedisNode(hostAndPort.getHostText(), hostAndPort.getPort());
            })
            .collect(Collectors.toSet()));
    serverConfig.setMaxRedirects(3);

    // client configuration
    LettuceClientConfiguration clientConfig = LettuceClientConfiguration.builder()
            .clientOptions(ClusterClientOptions.builder()
                    .topologyRefreshOptions(clusterTopologyRefreshOptions())
                    .build())
            .commandTimeout(Duration.ofSeconds(1L))
            // 2.
            .readFrom(ReadFrom.REPLICA)
            .build();

    return new LettuceConnectionFactory(serverConfig, clientConfig);
}

@Bean
public RedisTemplate<String, Object> redisTemplate(
    RedisConnectionFactory redisConnectionFactory
) {
    RedisTemplate<String, Object> redisTemplate = new RedisTemplate();
    redisTemplate.setConnectionFactory(redisConnectionFactory);
    return redisTemplate;
}
```

1. 30초마다 한번씩 Master / Slave 에 대한 레디스 토폴로지 구성을 갱신한다.
2. `io.lettuce.core.ReadFrom` 색인 조건의 우선순위를 정할 수 있다.

#### redis cluster

> TODO 도커로 클러스터 구현 실습 필요함

Redis 공식 릴리즈에서 `/utils/create-cluster` 를 통하면 로컬에 정말 쉽게 클러스터 구성이 가능하다.

#### 시나리오

1. 로컬 클러스터를 활성화한다.
2. 애플리케이션을 캐싱한다.
3. POSTMAN 등을 통해 캐싱된 곳에 연속된 요청을 전달한다.
4. 색인 되었던 노드를 임의로 중단시킨다.
5. 애플리케이션에서 `CLUSTERDOWN` 을 확인, 이후에 다른 클러스터로 색인 하는 것을 확인 가능하다.
