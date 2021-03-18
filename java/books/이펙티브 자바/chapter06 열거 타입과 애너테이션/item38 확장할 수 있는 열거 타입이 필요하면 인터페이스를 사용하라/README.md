# ✨ 아이템 38: 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

대부분의 상황에서는 열거타입을 확장하는건 좋지 않다. 확장한 타입의 원소는 기반 타입의 원소로 취급되지만 반대는 성립되지 않기 때문이다. 또한 고려할 요소가 들어나 설계와 구현이 복잡해진다.

그러나 확장에 어울리는 열거 타입이 있는데 연산 코드이다. 이따금 API 가 제공하는 기본 연산 외 사용자 확장 연산을 추가할 수 있도록 제공해야하는 경우가 필요하다고 생각해보자.

## 인터페이스

[Operation.java](https://github.com/psbin2017/garbage-collection/blob/master/gc/src/test/java/com/collection/gc/sample/enums/operation/Operation.java)

기존 추상 메소드를 인터페이스 메소드로 추가한다. 추상 메소드를 제거하고 Operation 인터페이스를 상속받으면 끝.

[OperationTest.java](https://github.com/psbin2017/garbage-collection/blob/master/gc/src/test/java/com/collection/gc/sample/enums/operation/OperationTest.java)

인터페이스를 이용하여 확장 가능한 열거 타입을 만들어도 단점이 있는데 열거 타입끼리 구현을 상속할수 없다는 것에 있다. 이는 인터페이스에 디폴트 메소드로 두어 추가하는 방법이 있다.

Opertaion 에 예제는 모든 연산기호에 대해 apply 메소드를 가지고 있어야 한다는 것이다. 이 경우 공유하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 클래스로 분리하는 방법으로 중복을 해소할 수 있다.

## 정리

열거 타입 자체는 확장이 불가능하지만 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용하여 같은 효과를 지닐수 있다.
