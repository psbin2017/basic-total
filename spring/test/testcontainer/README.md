# 테스트 컨테이너

테스트 컨테이너는 개발 환경을 바라 보지 않고 로컬에 도커 컨테이너를 생성하여 테스트가 가능하다는 장점을 가지고 있다. 개발 환경을 바라보게 되면 내가 의도하지 않은 상황을 만들기도 하고 이로 인해 삽질을 하게 된다. **(내가 아닌 다른 사람이 삽질을 하게 될 수도 있다.)** 따라서 테스트 코드를 격리된 로컬 도커 컨테이너로 실행하여 삽질을 줄이고 일부 테스트 영역에서 자신있게 테스트가 가능해진다.

한계점은 실행할 테스트 컨테이너의 OS 버전이나 이런 문제들로 인해서 실제 상용 환경과 일치하는 테스트 컨테이너를 완벽하게 지원하지 않을 수 도 있다. (예: M1 AppleChip 으로 일부 이미지가 버전에 정식 지원되지 않아 버전을 변경해서 테스트를 실행하는 문제)

## 준비 내용

테스트 스코프의 의존성 추가

```gradle
testImplementation("org.testcontainers:testcontainers")
testImplementation("org.testcontainers:junit-jupiter")
```

## 예제: 코드 레벨에서 선언형으로 관리

[quickstart 예제](https://www.testcontainers.org/quickstart/junit_5_quickstart/)

```java
@Testcontainers
public class RedisBackedCacheIntTest {

    private RedisBackedCache underTest;

    // container {
    @Container
    public GenericContainer redis = new GenericContainer(DockerImageName.parse("redis:5.0.3-alpine"))
        .withExposedPorts(6379);

    // }

    @BeforeEach
    public void setUp() {
        String address = redis.getHost();
        Integer port = redis.getFirstMappedPort();

        // Now we have an address and port for Redis, no matter where it is running
        underTest = new RedisBackedCache(address, port);
    }

    @Test
    public void testSimplePutAndGet() {
        underTest.put("test", "example");

        String retrieved = underTest.get("test");
        assertThat(retrieved).isEqualTo("example");
    }
}
```

## 예제2: Docker-Compose 로 읽어서 작성하기

```java
@Container
public DockerComposeContainer composeContainer =
    new DockerComposeContainer(new File("src/test/resources/docker-compose.yml"));
```
