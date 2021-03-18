# ✨ 아이템 3: private 생성자나 열거 타입으로 싱글턴임을 보증하라

```java
// public static final 필드 방식의 싱글턴
public class Elvis {
   public static final Elvis INSTANCE = new Elvis();
   private Elvis() { ... }

   // ...
}

// 정적 팩토리 방식
public class Elvis {
   private static final Elvis INSTANCE = new Elvis();
   private Elvis() { ... }
   public static Elvis getInstance() { return INSTANCE(); }

   // ...
}
```

위 방법은 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기 어렵다는 단점이 있다. 리플렉션 API 를 사용시 생성자를 2번 호출할 수 있기 때문에 두 번 생성시 예외를 던지게 하면 된다.

두 방식에 문제는 직렬화(Serializable)인데 모든 인스턴스 필드를 `transient` 를 선언하며 `readResolve` 메소드를 제공하여야한다. (item89 에서 다룬다.)

```java
public enum Elvis {
    INSTANCE;

    // ...
}
```

class 가 아닌 enum 으로 작성시 간결하고 쉽게 직렬화가 가능하다. 대부분의 상황에서 원소가 하나인 열거 타입으로 싱글톤으로 만드는 것이 가장 좋은 방법이다. 단, 싱글톤이 Enum 외에 다른 클래스를 상속해야 한다면 이 방법은 사용할 수 없다. (열거 타입이 아닌 다른 인터페이스를 구현하도록 선언할 수는 있다.)
