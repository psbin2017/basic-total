# ✨ 아이템 82: 스레드 안전성 수준을 문서화하라

> 멀티스레드 환경에서도 API 를 안전하게 사용하게 하려면 클래스가 지원하는 스레드 안전성 수준을 정확히 명시해야 한다.

- 불변(immutable), `@Immutable` : 상수와 같아 외부 동기화가 필요 없다. (`String`/`Long`/`BigInteger`)
- 무조건적 스레드 안전(unconditionally thread-safe), `@ThreadSafe` : 내부에서 충실하게 동기화하여 외부에서 동기화해줄 필요가 없다. (`AtomicLong`/`ConcurrentHashMap`)
- 조건부 스레드 안전(conditionally thread-safe), `@ThreadSafe` : 일부 메소드는 동시에 사용하려면 외부 동기화가 필요하다. (`Collections.synchronized` 래퍼 메소드 ... 반환한 반복자는 외부에서 동기화해야한다.)
- 스레드 안전하지 않음(non thread-safe), `@NotThreadSafe` : 동시에 사용하려면 외부의 동기화 메커니즘으로 감싸야한다. (`ArrayList`/`HashMap`)
- 스레드 적대적 (thread-hostile), 없음 : 외부 동기화 메커니즘으로 감싸도 안전하지 않다. deprecated API 로 지정한다. `generateSerialNumber` 메소드에서 내부 동기화를 생략하면 스레드 적대적이게 된다.

**여기에서 조건부 스레드 안전 클래스의 경우, 주의해서 문서화해야한다.**

- 어떤 순서로 호출할 때 외부 동기화가 필요한가?
- 해당 순서로 호출하려면 어떻게 락을 얻어야하는가?

정적 팩토리라면 자신이 반환하는 객체의 스레드 안전성을 반드시 문서화해야한다. `Collections.synchronizedMap` 이 훌륭한 예이다.

## 서비스 거부 공격

공개된 락을 오래 쥐고 놓지 않는 방식을 서비스 거부 공격(denial-of-service attack)이라 한다.

이를 막으려면 `synchronized` 에서 사용하는 `lock` 객체를 클라이언트 바깥에서 볼 수 없는 객체여야 한다. 또한 `final` 로 선언하여 락 필드 객체의 변경 가능성을 최소화하라.
