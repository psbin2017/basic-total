# ✨ 아이템 11: equals 를 재정의하려거든 hashCode 도 재정의하라

- equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메소드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야한다. (단, 애플리케이션을 재실행시 값이 달라져도 상관없다.)
- equals(Object) 가 두 객체를 같다고 판단하였다면, 두 객체의 hashCode 는 똑같은 값을 반환해야 한다.
- equals(Object) 가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode 가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

논리적으로 같은 객체는 같은 해시코드를 반환해야한다.

## 좋은 hashCode 를 작성하기 위한 요령

1. int 변수 result 를 선언한 후 값 c로 초기화한다. 이때 c 는 해당 객체의 첫 번째 핵심 필드를 단계 2.a 방식으로 계산한 해시코드다. (핵심 필드는 equals 비교에 사용되는 필드)
2. 해당 객체의 나머지 핵심 필드 f 에 대하여 다음을 수행한다.
   - 해당 필드의 해시코드 c 를 계산한다.
     - 기본 타입 필드는, Type.hashCode(f) 를 수행
     - 참조 타입 필드이면서 이 클래스의 equals 메소드가 이 필드의 equals 를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode 를 재귀적으로 호출한다. (복잡한 계산의 경우 표준형을 만들어 표준형의 hashCode 를 호출한다.)
     - 필드 값이 null 이면 0 을 사용한다. (다른 상수도 상관없지만 전통적으로 0 을 사용한다.)
     - 필드가 배열이라면, 핵심 원소 각각을 별도의 필드로 간주하여 다룬다. 앞서 본 규직을 재귀적으로 적용하여 다룬다.
   - 선행 단계에서 계산한 해시코드 c 로 result 를 갱신한다. (result = 31 * result + c)
3. result 를 반환한다.

```java
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

31 인 이유는 홀수이면서 소수이기 때문이며 소수를 곱하는 이유는 전통적으로 그래왔다.

## Objects

```java
@Override
public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```

단 한줄로 작성한 이 코드는 앞서본 요령보다는 느리다. 타입에 따라 박싱/언박싱 을 거쳐야 하기 때문이다.

## 캐싱

클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 캐싱 방식을 고려해야한다.

```java
@Override
public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
```

동일한 중요포인트는 성능을 높인답시고 중요 필드를 계산할때 생략해서는 안된다. 속도는 빨라져도 해시의 품질이 나빠지며, 수많은 인스턴스가 몇 개의 해시코드로 집중되어 해시테이블 속도가 느려질 것이다. 마지막으로 hashCode 가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자 그래야 hashCode 에 의존하지 않고, 계산방식을 추후에 변경해도 지장이 없어진다.
