# JPA

## 모든 연관관계는 지연로딩으로 설정

> `@ManyToOne(fetch = FetchType.LAZY)`

- 즉시로딩(`EAGER`)은 예측이 어렵고, 어떤 SQL 이 실행될지 추적하기 어렵다.
- JPQL 실행시 [N + 1](./issue/N+1%20문제/README.md) 문제가 자주 발생한다.
- 엔티티를 함께 조회해야 하면, fetch join 또는 엔티티 그래프 기능을 사용하자.
- `@XToOne` 관계는 기본 값이 즉시 로딩이므로 직접 지연 로딩으로 설정해야 한다.

![@ManyToOne](./images/1_@ManyToOne.PNG)

## 컬렉션 필드 초기화

> `private List<Category> child = new ArrayList<>();`

엔티티 영속화시 컬렉션을 감싸서 내장 컬렉션으로 변경한다. 내부 메커니즘 문제가 발생할 수 있다. 필드 레벨에서 선언하는게 깔끔하다.

## `@Transactional(readOnly=true)`

> `@Transactional(readOnly=true)`

1. 서비스 레이어에 `@Transactional(readOnly=true)`
2. 읽기 외의 기능을 사용하는 메소드는 명시적으로 `@Transactional` 사용
