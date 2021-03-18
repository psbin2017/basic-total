# ✨ 아이템 39: 명명 패턴보다 애너테이션을 사용하라

명명 패턴의 단점

- 오타가 나면 안된다. (test : tset)
- 올바른 프로그램 요소에만 사용되리란 보증이 없다. (명명 의도자와는 다르게 적용 등)
- 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.

## `@Retention`

[API DOC](https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/RetentionPolicy.html)

특정 시점에 어노테이션의 메모리를 가져갈지 범위를 지정한다.

| `RetentionPolicy` | 설명 | 컴파일 결과 |
| --- | --- | --- |
| `SOURCE` | 컴파일 전에 참조가 가능하다. | 어노테이션이 class 파일에 제거됨 |
| `CLASS` | 컴파일러가 클래스를 참조할때 가능하다. | 어노테이션이 class 파일에 포함됨. 단 런타임에서 사용할 수 없다. (기본값) |
| `RUNTIME` | 컴파일이후에도 참조가 가능하다. | 어노테이션이 class 파일에 포함됨 |

## `@Target`

[API DOC](https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/Target.html)

어노테이션의 위치를 정할 수 있다.

| `ElementType` | 설명 |
| --- | --- |
| `ANNONTATION_TYPE` | 어노테이션 |
| `CONSTRUCTOR` | 생성자 |
| `FIELD` | 멤버 변수 |
| `LOCAL_VARIABLE` | 지역 변수 |
| `METHOD` | 메소드 |
| `PACKAGE` | 패키지 |
| `TYPE` | 타입 |
| `TYPE_PARAMETER` | 매개변수 |
| `TYPE_USE` | 타입 사용시 |

## 예제

[ExceptionTestSample.java](https://github.com/psbin2017/garbage-collection/blob/master/gc/src/test/java/com/collection/gc/sample/enums/annontation/ExceptionTestSample.java)

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends  Throwable>[] value();
}
```

어노테이션을 선언하여 발생한 예외를 받아 처리할 수 있다.

## `@Repeatable`

배열 매개변수를 사용하는 대신 `@Repeatable` 메타 어노테이션을 사용할 수 있다. 단 주의할 점이 있다.

1. `@Repeatable` 어노테이션을 반환하는 컨테이너 어노테이션을 정의하고 `@Repeatable` 에 이 컨테이너 어노테이션의 class 객체를 매개변수로 전달해야한다.
2. 컨테이너 어노테이션은 내부 어노테이션 타입의 배열을 반환하는 value 메소드를 정의해야한다.
3. 컨테이너 어노테이션 타입에는 적절한 보존 정책(`@Retention`)과 적용 대상(`@Target`)을 명시해야 한다.

어노테이션으로 명명 패턴보다 나은점을 확실히 보여준다. 일반 프로그래머가 어노테이션 타입을 직접 정의할일은 거의 없지만 자바 프로그래머라면 자바가 제공하는 어노테이션 타입들은 사용해야한다. 이는 IDE 가 코드 진단 정보의 품질 개선에 도움이 된다.
