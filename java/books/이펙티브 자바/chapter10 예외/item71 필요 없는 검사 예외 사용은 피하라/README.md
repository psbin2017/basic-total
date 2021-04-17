# ✨ 아이템 71: 필요 없는 검사 예외 사용은 피하라

검사 예외는 발생한 문제를 프로그래머가 처리하여 안정성을 높이게끔 해준다. 물론 검사 예외를 과하게 사용하면 불편한 API 가 된다. 검사 예외를 던지는 메소드는 스트림 안에서 직접 사용할 수 없기 때문에 자바 8 부터는 부담이 더 커졌다.

API 를 제대로 사용해도 발생할 수 있는 예외이거나, 프로그래머가 의미 있는 조치를 취할 수 있는 경우라면 이 부담은 감내할 수 있을 것이다. 이 둘 중 어디에도 해당하지 않는다면 비검사 예외를 사용하는게 좋다.

```java
catch (TheChecedException e) {
    throw new AssertionError(); // 일어날 수 없다.
}

catch (TheChecedException e) {
    e.printStackTrace(); // 우리가 졌다.
    System.exit(1);
}
```

리팩토링으로 풀어보자.

```java
try {
    obj.action(args);
} catch (TheChecedException e) {
    // 예외
}

// ...

if ( obj.actionPermitted(args) ) {
    obj.action(args);
} else {
    // 예외
}
```

상태 검사 메소드와 비검사 예외를 던지는 메소드로 리펙토링하여 스트림에도 사용가능하며 예외 처리로 생기는 부담이 덜해졌다.

그러나 모든 상황에서 적용할 수 없다. 리펙토링 된 코드가 아름답다고는 말할 수 없지만 유연한 것은 맞다. 스레드를 중단하길 원한다면 `obj.action(args)` 한 줄이면 충분하다. 한편 `actionPermitted` 는 외부 동기화 없이 여러 스테르가 동시에 접근할 수 있거나 외부 요인에 의해 상태가 변할 수 있다면 해당 리팩토링은 적절하지 않다. (`actionPermitted`, `action` 호출 사이에 상태가 변할 수 있기때문에) 또한 중복으로 수행하면 성능상으로도 손해를 가져올 수 있다.

> 결론: 쉽지 않다.
