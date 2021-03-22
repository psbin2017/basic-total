# 🗂 1장 JPA 소개

## 연관 관계

```text
            Object Model
┌─ Member ────┐     ┌─ Team ──────┐
│ id          │     │ id          │
│ name        │     │ name        │
│ Team team   │ →   └─────────────┘
└─────────────┘

            Table Model
┌─ Member ────┐     ┌─ Team ──────┐
│ id(PK)      │     │ team_id(PK) │
│ name        │     │ name        │
│ team_id(FK) │ →   └─────────────┘
└─────────────┘
```

## 객체 그래프 탐색

실제 객체를 사용하는 시점에 적절한 SELECT SQL 을 수행한다. 이를 **지연 로딩**이라고 한다.

```java
member.getTeam().getName();
```

필요에 따라 지연 로딩이 아닌 함께 조회하게 설정할 수도 있다.

## 객체 비교

같은 트랜잭션일 경우 같은 객체가 조회되는 것을 보장한다.

```java
String memberId = "10";
Member member1 = jpa.find(Member.class, memberId);
Member member2 = jpa.find(Member.class, memberId);

// 같은 인스턴스를 보장하여 동일성 보장
// member1 == member2;
```

## 데이터 접근 추상화와 벤더 독립성

페이징 처리는 데이터베이스마다 SQL 이 다르다. JPA 는 특정 기술에 종속되지 않고 사용하는 데이터베이스를 알려주어 벤더 독립성을 유지한다.
