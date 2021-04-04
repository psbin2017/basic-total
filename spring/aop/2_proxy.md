# Spring AOP Proxy

> 프록시? 타겟을 래핑하여 요청을 대신 받아주는 래핑 클래스이다.

스프링 AOP 에서는 이 **프록시 객체를 생성하기 위해** 2가지 프록시 메커니즘을 이용한다.

`JDK Dynamic Proxy` 와 `CGLIB Proxy` 이다.

## 비교

| | `JDK Dynamic Proxy` | ``CGLIB Proxy`` |
| --- | --- | --- |
| 프록시 객체 생성 방법 | 자바 리플렉션 | 바이트 코드 조작 |
| 명세 조건 | 인터페이스 필수 구현 | 클래스 가능 |

## Inside Spring boot

스프링 부트의 경우 트랜잭션의 대상이 인터페이스로 작성되어 구현되어 있어도 `CGLIB Proxy` 가 기본으로 적용된다.

> We've generally found cglib proxies less likely to cause unexpected cast exceptions. [출처](https://github.com/spring-projects/spring-boot/issues/8434)

스프링 부트 프로젝트 리더가 일반적으로 `CGLIB Proxy` 가 예외 발생 가능성이 낮다고 말하였다. 이를 통해 기본 전략은 `CGLIB Proxy` 이다.

## CGLIB 주의 사항

인터페이스로 작성하더라도 `CGLIB Proxy` 로 프록시 객체를 생성하도록 강제화 할 수 있다.

```java
@EnableAsync(proxyTargetClass = true)
@EnableCaching(proxyTargetClass = true)
```

- 파이널 메소드는 재정의가 불가능하기 때문에 어드바이스가 불가능하다.
- 이전에는 생성자가 2번 호출되는 문제와 기본 생성자가 반드시 있어야 하는 제약조건이 있었다.
  - 이 경우 스프링 4.0 부터 `Objenesis` 라이브러리를 통해서 인스턴스가 생성되어 해소 되었다.
  - 그러나 이는 **스프링 4.0 부터 해당사항이다.**

## 학습 내용 참고 출처

- [Spring aop Proxy](http://wonwoo.ml/index.php/post/1576)
- [Spring boot는 왜 cglib를 선택했을까?](http://wonwoo.ml/index.php/post/1708)
