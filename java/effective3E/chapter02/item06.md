# 💎 2장 객체의 생성과 파괴

학습 내용 출처: [이펙티브 자바 3판](http://ebook.insightbook.co.kr/book/66)

본 챕터의 공통 주제는 다음과 같다.

1. 객체를 만들어야할 때와 만들지 않아야 할 때에 대한 구분
2. 올바른 객체 생성 방법
3. 불필요한 객체 생성을 피하는 방법
4. 객체의 파괴를 보장하며 객체 파괴 이전에 수행해야 할 작업을 관리하는 방법

## ✨ 아이템 6: 불필요한 객체 생성을 피하라

생성 비용이 비싼 객체는 캐싱하여 재사용하는 것이 좋다. (다만, 자신이 만드는 객체 생성 비용이 비싼 것인지 명확히 알 수 없다.)

```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile("" /* 생략 */);

    static boolean isRomanNumeral(String s) {
        return ROMAN.mather(s).matches();
    }
}
```

지연 초기화를 사용하여 실제 사용 이전까지 초기화를 지연하는 방법이 있지만, 코드의 복잡도가 높아진다. (아이템 67)

불필요한 객체 생성의 다른 예로는 오토박싱(auto boxing)이 있다.

```java
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i<= Integer.MAX_VALUE; i++) {
        sum += i;
    }
    return sum;
}
```

오타와 같은 선언 하나로 의도치 않은 오토박싱이 되었다. 기본 타입으로 바꿔주는 것만으로도 성능이 개선된다.
