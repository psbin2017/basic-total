# ✨ 아이템 41: 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

마커 인터페이스(marker interface) : 아무런 메소드도 없으며 특정 속성을 가짐을 표시해주는 인터페이스.

대표적인 마커 인터페이스는 `Serializable` 인터페이스다. `Serializable` 를 구현한 인터페이스는 `ObjectOutputStream` 을 통해 쓸 수 있고, 직렬화 할 수 있음을 표시해준다.

마커 인터페이스는 마커 어노테이션의 등장으로 2 가지 다른점이 있다.

1. 마커 인터페이스는 이를 구현한 클래스의 인스턴스를 구분하는 타입으로 사용할 수 있으나, 마커 어노테이션은 아니다.
2. 마커 인터페이스는 적용 대상을 더 정밀하게 지정할 수 있다.

## 타입으로서 가지는 장점

마커 인터페이스는 어엿한 타입으로 마커 어노테이션을 사용한다면 런타임에 발견할 오류를 컴파일 타임에 잡을 수 있다.

## 적용 대상 장점

앞서 마커 어노테이션은 적용 대상(`@Target`)과 정책(`Retention`)을 지정해주었다. 이는 마커 인터페이스보다 세밀한 제한이 불가능하다.

> 예: 특정 인터페이스를 구현한 클래스만 적용하고 싶은 마커

마커 인터페이스로 정의한다면 마킹하고 싶은 클래스에서 그 인터페이스를 구현하면 된다. (하위 타입임을 보장)

다만 어노테이션을 적극 활용하는 프레임워크라면 마커 어노테이션이 가지는 이점이 있을 것이다. 이 경우 어노테이션을 사용하는 것이 프레임워크에서의 일관성을 지키는데 유리할 것이다.

## 선택 방법

마커 어노테이션: 클래스와 인터페이스 외의 프로그램 요소(모듈, 패키지, 필드, 지역변수)를 포함해야하는 경우

마커 인터페이스: 마킹 된 객체를 매개변수로 받는 메소드를 작성해야하는 경우 (컴파일 타임에 오류 해결 가능)

나머지 경우 마커 어노테이션이며 프레임워크가 어노테이션을 지향한다면  마커 어노테이션이다.
