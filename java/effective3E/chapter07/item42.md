# 💎 7장: 람다와 스트림

자바 8에 추가된 함수형 인터페이스, 람다, 메소드 참조로 함수 객체를 쉽게 만들 수 있게 되었다. (함수형 프로그래밍) 추가로 스트림 API 도 지원하게 되었다.

본 장에서는 이 기능을 효과적으로 사용하는 방법을 학습한다.

## ✨ 아이템 42: 익명 클래스보다는 람다를 사용하라

[Lambda Sample](https://github.com/psbin2017/garbage-collection/blob/9686ff9911d6fc2dbb0d9e3921e29de162fffd8b/gc/src/test/java/com/collection/gc/sample/lambda/LambdaShortSample.java)

전략 패턴처럼 함수 객체를 사용하는 과거의 객체 지향 디자인 패턴에는 익명 클래스면 충분했다.

```java
Collections.sort(words, 
    (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

여기서 타입이 생략되었는데, 타입 추론 규칙은 상당히 복잡하며 다 이해하는 프로그래머가 없고 잘 알지 못한다 해도 상관 없다.

**타입을 명시해야 코드가 명확할 경우를 제외하고, 람다의 모든 매개변수는 타입을 생략하자.**

> 이전 제네릭 로 타입에서 학습한 내용으로 로 타입으로 선언하면 컴파일 오류가 났을 것이다.

## Operation

기존 챕터에서 살펴본 열거 타입의 람다식 구현방법이다.

[Lambda Operation](https://github.com/psbin2017/garbage-collection/blob/9686ff9911d6fc2dbb0d9e3921e29de162fffd8b/gc/src/test/java/com/collection/gc/sample/lambda/LambdaOperation.java)

여기에서도 보이듯이 람다는 이름이 없고 문서화가 불가능하다. 따라서 코드 자체로 동작이 명확해야하기 때문에 라인 수가 적어야한다. 너무 길거나 읽기 어려운 코드라면 리팩토링하거나 사용하지 말아야한다.

## 익명 클래스

람다의 등장으로 다수의 익명 클래스의 역할은 줄어들었지만, 익명 클래스의 사용은 추상 클래스의 인스턴스를 만들때 사용하게 된다.

람다는 자신을 참조할 수 없다. 람다에서의 this 키워드는 바깥 인스턴스를 가리킨다. 그래서 함수 객체가 자신을 참조해야한다면 익명 클래스를 사용해야 한다.

마지막으로 람다와 익명클래스는 직렬화하는 일은 삼가야 한다. 직렬화해야하는 함수 객체가 있다면 정적 중첩클래스의 인스턴스를 사용하자. [✨ 아이템 24: 멤버 클래스는 되도록 static 으로 만들라](../chapter04/item24.md)

정리하자면 익명 클래스는 타입 인스턴스를 만들때만 사용하자.
