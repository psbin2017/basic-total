# ✨ 아이템 23: 태그 달린 클래스보다는 클래스 계층구조를 활용하라

```java
class Figure {
    // only use CIRCLE ...
    double radius;

    // only use constructor CIRCLE ...
    Figure(double radius) {
        // ...
    }

    double area() {
        switch(shape) {
            case CIRCLE:
                return Math.PI * (radius * radius);
            // ...
        }
    }
}
```

태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다. 태그 동작을 추상으로 바꾸고 이를 상속하여 달리 처리하는 방법이 있다.

1. 루트가 될 추상 클래스를 정의하여, 태그 값에 따라 동작이 달라지는 메서드를 **추상 메서드**로 선언한다.
2. 태그 값에 상관 없이 동작이 일정한 메서드들은 루트 클래스에 **일반 메서드**로 선언한다.
3. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드도 루트 클래스로 올린다.

```java
abstract class Figure {
    // type 1
    abstract double area();
}

class Circle extends Figure {
    @Override
    double area() {}
}
```

`switch` 문과 같은 태그로 구분할 필요가 없고 각 구현체가 직접 추상 메서드를 구현한다. 루트 클래스의 코드를 건드리지 않고 독립적으로 계층구조를 확장하여 사용할 수도 있다.

타입 사이의 자연스러운 계층 관계를 반영하여 유연성은 물론 컴파일타임 타입 검사 능력을 높여주는 장점도 있다.
