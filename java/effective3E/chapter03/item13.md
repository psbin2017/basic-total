# 💎 3장: 모든 객체의 공통 메서드

학습 내용 출처: [이펙티브 자바 3판](http://ebook.insightbook.co.kr/book/66)

모든 클래스는 재정의를 염두에 두고 설계된 것으로 재정의 시켜야 하는 일반 규약이 명확히 정의되어있다. 따라서 재정의시 이 규약을 준수하여 재정의해야한다.

## ✨ 아이템 13: clone 재정의는 주의해서 진행하라

`Cloneable`(clone() 그리고 deep copy 를 뜻한다.), 복제를 허용하는 클래스임을 나타내는 인터페이스이다.

불변 클래스는 불필요한 복사를 만들지 않지 위해 제공하지 말아야한다. clone() 메소드는 생성자와 같은 효과를 내야하며 생성된 객체의 멤버변수를 수정했을 때, 원본 객체에 영향을 주지 않으면서 동시에 복제된 객체의 불변식을 보장해야한다.

상속용 클래스는 `Cloneable` 를 구현해서는 안되며 `Cloneable` 을 구현한 스레드를 사용한 클래스를 구현할 때는 적절하게 동기화 해주어야 한다.

일반 상황과 다르게 배열을 복제할 때 배열의 clone() 메소드 사용을 권장하며 final 클래스도 `Cloneable` 을 구현해도 괜찮다.

그러나 객체는 복제 기능은 생성자, 팩토리를 이용하는 것이 바람직하기 때문에 배열만 clone() 메소드를 사용하여 복사하도록 한다.