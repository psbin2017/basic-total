# 제어의 역전과 의존성 주입

자바는 기본적으로 애플리케이션 개발에 풍부한 기능을 제공하지만 개발된 요소들을 하나의 큰 구조로 만드는 **결합하는 기능은 부족**하다. 이런 단점을 극복하기 위해서 아키텍처와 개발자가 팩토리, 추상 팩토리, 빌더, 데코레이터 그리고 서비스 로케이터 등과 같은 디자인 패턴을 적용하였다. 그러나 디자인 패턴은 좋은 설계와 개발에 불과하기에 온전한 해결책으로 제시할 수 없다.

**제어의 역전(IoC : Inversion Of Control)** 은 위 문제점의 해결책을 제시한다.

서로 다른 컴포넌트들을 사용할 준비가 되었다는 의미를 공식적으로 제공하여 해결한다. 스프링은 공식화된 디자인 패턴을 자신만의 애플리케이션에 통합할 수 있는 일급 객체로 규정하여 IoC 컴포넌트 방식을 통하여 견고하고 유지보수가 쉬운 애플리케이션을 설계할 수 있다.

> 제어의 역전은 마틴 파울러를 통해 **의존성 주입(DI : Dependency Injection)** 이라는 명칭으로 제안되었다.

## 컨테이너 (IoC Container)

`ApplicationContext` 인터페이스는 IoC 컨테이너를 의미한다.

> IoC Container == ApplicationContext

1. 빈을 인스턴스화한다.
2. 빈들을 조합한다.
    - 빈의 설정과 조합은 메타데이터를 통해 알아낸다.

## 메타데이터 (Meta Data)

메타 데이터를 작성하는 방법

1. XML: 전통적인 방법
2. @Annontation: 스프링 2.5 부터 지원한 방법
3. 자바 기반: 스프링 3.0 부터 지원한 방법

## 빈 (Bean)

빈의 정의에는 다음의 메타 데이터를 필요로 한다.

1. 패키지에 최적화된 클래스 명: 정의된 빈의 실제 구현 클래스
2. 빈의 행동에 대한 설정 요소들: 컨테이너에서의 빈의 동작 방법 (스코프, 라이프사이클, 콜백 등)
3. 빈이 동작하는데 필요한 다른 빈들에 대한 참조: 협력 객체 또는 의존성이라고 부른다.
4. 새로 생성된 객체에 설정해야 하는 그 외 설정 값: 커넥션 풀을 관리하는 빈에서 사용하는 커넥션 수 등과 같은 설정 값

```text
┌ IoC Container (ApplicationContext) ┐
│             <Meta Data>            │
│ [Bean] [Bean] [Bean] [Bean] [Bean] │
│ [Bean] [Bean] [Bean] [Bean] [Bean] │
│ [Bean] [Bean] [Bean] [Bean] [Bean] │
│ [Bean] [Bean] [Bean] [Bean] [Bean] │
└────────────────────────────────────┘
```

## 의존성 주입

의존성 주입은 객체가 의존성을 제공 받을 때 낮은 결합도(decoupling)로 효과적이다.

### 생성자 주입 (Constructor Injection)

컨테이너가 여러 종속 변수를 나타내는 여러 개의 인수로 생성자를 호출하여 수행한다.

빈을 생성하기 위해 특정 인수를 가진 정적 팩토리 메소드를 호출하는 것과 유사하다.

```java
public class SimpleMovieListener {

    private MovieFinder movieFinder;

    public SimpleMovieListener(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

### 세터 주입 (Setter Injection)

빈을 인스턴스화하기 위해 기본 생성자 또는 인수가 없는 정적 팩토리 메소드를 호출한 후 빈에서 세터 메소드를 호출하는 컨테이너에 의해 수행된다.

```java
public class SimpleMovieListener {

    private MovieFinder movieFinder;

    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

### 생성자 주입으로 해야 할까 세터 주입으로 해야할까

두 가지 DI 방법을 적용하는 기준은 필수 의존성은 생성자 주입으로 선택 의존성은 세터 주입으로 적용하는 것이 좋다. 세터에 `@Required` 어노테이션을 사용하여 의존성이 꼭 필요한 세터를 만들 수 있다.

스프링 팀은 세터 주입을 더 선호한다. 생성자 인수가 많아진다면 다루기가 쉽지 않기 때문이다. 특히나 프로퍼티가 선택적이라면 더욱 다루기가 까다로워진다. 반면에 세터는 나중에 다시 설정하거나 다시 주입해야하는 클래스의 객체들을 만들 수 있다. JMX MBean 을 통한 관리는 대표적인 세터 주입 사례이다.

몇몇 순수주의자들은 생성자 주입을 선호하는데, 모든 객체의 의존성을 제공한다는 것이 항상 클라이언트에게 완전히 초기화된 상태를 반환하는 것을 의미한다. 단점은 위에서 언급한대로 객체의 재구성이나 재주입이 쉽지 않다는 것이다.

개별 클래스에 적절한 DI 를 사용해야 한다. 때때로 서드파티 클래스를 다루게 될 때 어떤 DI 를 사용할지 직접 선택해야 한다. 레거시 클래스는 어떤 세터 메소드도 제공하지 않을 수 있으므로 생성자 주입이 유일한 DI 가 될 수 있다.

### 필드 주입 (Field Injection)

앞선 예제와 다르게 스프링에서 제공하는 어노테이션을 통한 필드 주입이다.

```java
public class SimpleMovieListener {
    @Autowired private MovieFinder movieFinder;
}
```

### 각 주입 순서 비교

`A Bean` 의 의존성 주입 과정 예시이다. `A Bean` 은 `B Bean` 에 의존성을 가진다. 풀어서 말하자면 `A Bean` 을 인스턴스화 하기 위해선 `B Bean` 이 필요하다.

세터 주입(Setter Injection)

1. `A Bean` 의 의존성인 `B Bean` 의 생성자를 호출한다. **(이 시점에 `B Bean` 은 생성된다)**
   - `B Bean` 이 생성(인스턴스화) 되지 않은 경우 빈 팩토리를 통해 생성한다.
2. `A Bean` 에서 `B Bean` 수정자를 호출하여 주입한다.

필드 주입(Field Injection)

1. 세터 주입 1. 과 동일 **(마찬가지로 이 시점에 `B Bean` 은 생성된다)**
2. `A Bean` 에서 어노테이션이 붙은 `B Bean` 을 찾아 의존성을 주입한다.

생성자 주입(Constructor Injection)

1. `A Bean` 의 생성자를 호출한다.
2. `A Bean` 의 생성자에서 `B Bean` 의 생성자를 호출한다.
    - `B Bean` 이 인스턴스화 되지 않은 경우 빈 팩토리를 통해 인스턴스화 한다.

> `A Bean` 이 생성자로 생성되는 시점에 필요한 `B Bean` 을 주입한다.

주입 순서로 **순환 참조는 생성자 주입에서만 문제가 된다.**

#### 순환 참조 예시

`A Bean` 은 `B Bean` 에 의존성을 가지며, 반대로 `B Bean` 은 `A Bean` 에 의존성을 가진다.

세터 주입(Setter Injection)

1. `A Bean` 의 의존성인 `B Bean` 의 생성자를 호출한다.
2. `B Bean` 의 의존성인 `A Bean` 의 생성자를 호출한다.
3. 각 Bean 에서 수정자를 호출하여 주입한다.

필드 주입(Field Injection)

1. 세터 주입 1. 2. 와 동일
2. 각 Bean 에서 어노테이션이 붙은 Bean 을 찾아 의존성을 주입한다.

생성자 주입(Constructor Injection)

1. `A Bean` 의 생성자를 호출한다.
2. `A Bean` 의 생성자에서 `B Bean` 의 생성자를 호출한다.
3. `B Bean` 에서 `A Bean` 의 생성자를 호출한다.

// TODO