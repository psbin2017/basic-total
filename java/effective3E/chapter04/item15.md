# 💎 4장: 클래스와 인터페이스

학습 내용 출처: [이펙티브 자바 3판](http://ebook.insightbook.co.kr/book/66)

추상화의 기본 단위인 클래스와 인터페이스는 자바 언어의 심장과도 같다. 이 요소를 적절히 활용하기 위해 견고하고 유연한 클래스와 인터페이스를 만드는 방법에 대해 알아본다.

## ✨ 아이템 15: 클래스와 멤버의 접근 권한을 최소화하라

패키지 외부에서 쓸 이유가 없다면 package-private 로 선언한다. 이들은 API 가 아닌 내부 구현이 되어 언제든지 수정할 수 있다. 즉, 클라이언트에 아무런 피해 없이 수정, 교체, 제거할 수 있다. 반면, public 으로 선언하면 API 가 되므로 하위 호환을 위해 영원히 관리해야 한다.

한 클래스에서만 사용하는 클래스는 private static 으로 중첩시켜 사용하면 바깥 클래스 하나만 접근할 수 있다.

- private : 멤버를 선언한 톱레벨 클래스에서만 접근 할 수 있다.
- package-private : 멤버가 소속된 패키지 안의 모든 클래스에 접근 할 수 있다. 접근 제한자를 명시하지 않았을때 적용되는 접근 수준이다. (default)
- protected : package-private 를 포함하며, 이 멤버를 선언한 클래스의 하위 클래스에도 접근할 수 있다.
- public :  모든 곳에 접근할 수 있다.

1. 공개 API 클래스를 세심히 설계한 후,
2. 그 외의 모든 멤버는 private 로 만든다.
3. 그리고 같은 패키지의 다른 클래스가 접근해야 하는 멤버에 한하여 package-private 로 풀어주자.

### public 클래스의 인스턴스 필드는 되도록 public 이 아니어야 한다

필드가 가변 객체를 참조하거나 final 이 아닌 인스턴스 필드를 선언하면 값을 제한할 힘을 잃게된다. 여기에 더해 public 가변 필드를 가지는 클래스는 일반적으로 스레드에 안전하지 않다.

### 길이가 0 이아닌 배열은 클래스에서 public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메소드를 제공해서는 안된다

해결책은 2가지이다.

```java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES = 
    Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));

private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
    return PRIVATE_VALUES.clone();
}
```

첫번째는 불변 리스트로 만드는 것이고 방어적 복사를 통한 public 메서드로 변경이다.

### Java 9 Add

모듈 시스템이 도입되면서 암묵적 수준이 추가되었다. 모듈은 자신에 속하는 패키지 중 공개(export)할 것들을 (관례상 module-info.java 파일) 선언한다. protected 혹은 public 멤버라도 해당 패키지를 공개하지 않았다면 모듈 외부에서 접근할 수 없다. 모듈안에서는 선언과 영향이 없다.

이 접근 수준은 각각 public 과 protected 수준과 같으나, 그 효과가 모듈 내부로 한정되는 기능인 것이다. 이런 형태의 공유 상황은 흔한 상황은 아니지만 이 또한 패키지들 사이에서 클래스를 재배치하면 대부분 해결된다.

접근 제한자와 다르게 모듈은 상당히 주의해서 사용해야한다. 자신의 모듈 경로가 아닌 애플리케이션의 클래스패스에 두면 그 모듈 안의 모든 패키지는 마치 모듈이 없는 것처럼 행동하게 된다.

모듈은 여러 면에서 자바 프로그래밍에 영향을 주는데, 패키지를 모듈 단위로 묶고 재배치하며 패키지들의 모든 의존성을 명시한다. 마지막으로 일반 패키지로의 모든 접근에 조치를 적용해야한다.

## 정리

프로그램 요소의 접근성은 가능한 최소한으로 한다. 필요한 것만 골라 최소한의 public API 를 설계하자. public 클래스는 public static final 필드 외에는 어떠한 public 필드도 가져서는 안 된다. 또한 public static final 필드가 참조하는 객체가 불변임을 확인하라.
