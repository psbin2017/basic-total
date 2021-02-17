# 트랜잭션

> 데이터베이스의 형태를 변화시키기 위해 수행하는 **작업의 단위**

상태를 변화시키는 것은 질의어(**SQL**) 이다. (SELECT, UPDATE, DELETE, INSERT)

## 트랜잭션 단위 예시

A 회원이 B 회원에게 계좌 이체를 진행한다고 가정, 일련의 과정은 하나의 업무로 진행되어야한다. 이 업무의 단위를 트랜잭션이라고 한다.

1. A 회원의 통장에서 10,000 원 감소.
2. B 회원의 통장에서 10,000 원 증가.

## 트랜잭션의 특징

트랜잭션의 특징은 크게 4가지로 ACID 라고 하며 안전한 트랜잭션을 제공하기 위하여 이 특징을 지켜야 한다.

### 원자성 (Atomicity)

트랜잭션이 모두 데이터베이스에 반영되거나, 반대로 모두 반영되지 말아야한다.

트랜잭션이 원자성을 잃는다면? 단위 예시로 예를 들자면 `1. A 회원의 통장에서 10,000 원 감소.` 만 진행한다면 A 회원은 잔고만 줄어들고 B 회원은 잔고가 늘어나지 않은 중대한 상황이 발생된 것이다.

### 일관성 (Consistency)

트랜잭션의 작업처리는 작업처리 결과가 항상 동일해야 한다.

트랜잭션 진행 중에 데이터베이스에 변경이 가해져도 해당 변경과는 관계없이 처음 트랜잭션을 실행한 상태로 트랜잭션을 진행해야 한다는 것이다.

### 독립성 (Isolation)

복수 개의 트랜잭션이 실행되고 있는 경우, 서로 다른 트랜잭션은 다른 트랜잭션 연산에 개입할 수 없다. 하나의 트랜잭션이 완료 될 때까지, 다른 트랜잭션의 결과를 참조할 수 없다. **단, 독립성은 성능 관련의 이유로 가장 유연한 특징 중 하나이다.**

단위 예시

1. 트랜잭션 1번 A 회원과 B 회원에게 계좌 이체. (일련의 과정은 앞 예시와 동일)
2. 트랜잭션 2번 C 회원과 B 회원에게 계좌 이체. (일련의 과정은 앞 예시와 동일)

트랜잭션 2번은 트랜잭션 1번이 끝나기 전까지 실행되지 않으며(LOCK), 트랜잭션 1번이 끝나고 하지 않기 때문에 실행된다.

### 지속성 (Durability)

트랜잭션이 모두 끝나 성공적으로 완료되었을 때, 이 결과는 영구적으로 반영되어야 한다. 일반적으로 모든 트랜잭션은 로그로 남아있게 되고 시스템 장애 발생시 장애 발생 이전 상태로 되돌릴 수 있다.

*****

## 트랜잭션 COMMIT, ROLLBACK 연산

COMMIT 은 하나의 트랜잭션이 실행되고 난 후 성공적으로 수행하여 끝마쳤음을 나타내는 상태를 의미한다.

```text
BEGIN TRAN
UPDATE accounts SET balance = balance - 10000 WHERE user = 'A 회원'
UPDATE accounts SET balance = balance + 10000 WHERE user = 'B 회원'
COMMIT TRAN
```

ROLLBACK 은 하나의 트랜잭션 처리가 비정상적으로 처리되어 원자성이 깨진 경우, 트랜잭션을 처음부터 다시 실행하거나, 부분적으로 실행된 연산을 취소하는 등의 상태를 의미한다.

ROLLBACK 은 트랜잭션 과정에서 실시한 로그를 통해서 진행된다.

```text
Redo 로그 (이후 상태를 가진 로그)
UPDATE accounts A 회원.balance = 0
UPDATE accounts B 회원.balance = 10000

Undo 로그 (이전 상태를 가진 로그)
UPDATE accounts A 회원.balance = 10000
UPDATE accounts B 회원.balance = 0
```

예기치 못한 오류로 ROLLBACK 되었다면, Redo 로그를 실행하고 Undo 로그를 실행하여 원상태인 일관성을 유지한다.

*****

## @Transactional

스프링은 `@Transactional` 어노테이션을 통하여 개발자가 직접 트랜잭션 단위를 지정할 수 있다. `@Transactional` 이 적용될 수 있는 곳은 *클래스의 메소드*와 *인터페이스* 그리고 *클래스*이다.

```java
@Transactional
public void trade() {
    // ...
}
```

## 성능 문제

트랜잭션의 동시성을 제어하기 위해 트랜잭션 격리 수준을 조정하여 성능 문제를 해소 할 수 있다. 이 때 동시성이란 **동시에 데이터베이스에 접근하는 경우 그 접근을 어떻게 제어할지에 대한 설정**을 뜻한다.

```text
트랜잭션 1 실행 중
트랜잭션 2 대기 ...
트랜잭션 3 대기 ...
트랜잭션 4 대기 ...
트랜잭션 5 대기 ...
트랜잭션 6 대기 ...
트랜잭션 7 대기 ...
...
```

*****

## 격리 레벨

언급한 트랜잭션의 성능 문제를 해소하기 위해 개발자는 트랜잭션 간의 개입을 조정할 수 있다. 사용 방법은 `@Transactional(isolation=Isolation.DEFAULT)` 과 같다.

| 옵션 명 | 설명 | 성능(상대적) | 발생 가능한 문제 |
| --- | --- | --- | --- |
| `READ_UNCOMMITED` | 커밋되지 않은 데이터에 대한 읽기 허용 | 속도 1, 정합성 5 | `Dirty Read`, `Non-Repeatable Read`, `Phantom Read` |
| `READ_COMMITED` | 커밋된 데이터에 대한 읽기 허용 | 속도 2, 정합성 4 | `Non-Repeatable Read`, `Phantom Read` |
| `DEFAULT` | DB 설정, 기본 수준의 격리 | 속도 3, 정합성 3 |
| `REPEATEABLE_READ` | 동일한 필드에 대한 다중 접근시 모두 동일한 결과를 보장 | 속도 4, 정합성 2 | `Phantom Read` |
| `SERIALIZABLE` | 가장 엄격한 수준의 격리 | 속도 5, 정합성 1 | 없음 |

### READ_UNCOMMITED

> 트랜잭션 A 가 a 라는 데이터를 b 로 변경하는 동안 다른 트랜잭션인 B 가 아직 커밋 되지 않은 데이터 b 로 읽을 수 있다.

`READ_UNCOMMITED` 은 다른 트랜잭션에서 처리하는 작업이 완료되지 않았는데도 다른 트랜잭션에서 읽을 수 있는 현상인 **Dirty Read** 가 발생할 수 있다.

#### READ_UNCOMMITED 상황 예시

```text
1. (A 트랜잭션) 회원 A 의 은행 잔고 조회: 10,000 원
2. (A 트랜잭션) 회원 A 의 은행 잔고 수정: 10,200 원
3. (B 트랜잭션) 회원 A 의 은행 잔고 조회: 10,200 원 (**Dirty Read**)
4. (A 트랜잭션) 문제가 발생하여 ROLLBACK 수행
5. (B 트랜잭션) 회원 A 의 은행 잔고 10,200 원으로 서비스를 수행함 (???)
```

### READ_COMMITED

> 트랜잭션 A 가 a 라는 데이터를 b 로 변경하고 커밋한 후 다른 트랜잭션인 B 가 커밋된 데이터 b 를 읽을 수 있다.

`READ_COMMITED` 는 하나의 트랜잭션이 같은 값을 조회하였을때 다른 결과를 볼 수 있는 **Non-Repeatable Read**가 발생할 수 있다.

#### READ_COMMITED 상황 예시

```text
1. (B 트랜잭션) 회원 A 의 은행 잔고 조회1: 10,000 원
2. (A 트랜잭션) 회원 A 의 은행 잔고 수정: 10,200 원 (커밋)
3. (B 트랜잭션) 회원 A 의 은행 잔고 조회2: 10,200 원
```

B 트랜잭션의 조회1 의 결과와 조회2의 결과가 서로 불일치하는 **Non-Repeatable Read** 문제가 발생한다.

### REPEATEABLE_READ

> 트랜잭션 A 가 완료될 때까지 SELECT 문장이 사용하는 모든 데이터에 shared LOCK 이 걸린다.

트랜잭션이 시작되고 종료되기 전까지 한 번 조회한 값은 계속 같은 값이 조회되는 격리 수준이다.

`REPEATEABLE_READ` 는 앞서 살펴본 문제는 발생하지 않는다. 그러나 INSERT/DELETE 에 대한 **Phantom Read** 가 발생할 수 있다.

#### REPEATEABLE_READ 상황 예시

```text
1. (A 트랜잭션) 회원 A 의 은행 잔고를 조회1
2. (B 트랜잭션) 회원 A 의 은행 잔고를 수정
3. (A 트랜잭션) 회원 A 의 은행 잔고를 조회2 (조회1 과 결과가 동일)
4. (B 트랜잭션) 회원 C 의 은행 잔고를 생성
5. (A 트랜잭션) 회원 C 의 은행 잔고를 조회 (조회가 된다)
```

A 트랜잭션이 B 트랜잭션에서 행에 대한 수정은 영향을 받지 않았지만, 행에 대한 추가/삭제는 영향을 받게 된다.

### SERIALIZABLE

읽기 작업을 포함한 모든 작업에 해당 레코드에 대한 모든 변경을 제한한다.

## 격리 레벨에 따른 문제 현상 정리

- Dirty Read (=Uncommitted Dependency)

커밋되지 않은 수정 중인 데이터를 다른 트랜잭션에서 읽을 수 있도록 허용할 때 발생한다.

- Non-Repeatable Read (=Inconsistent Analysis)

한 트랜잭션 내에서 같은 쿼리를 두 번 수행할 때 그 사이에 다른 트랜잭션이 값을 수정 또는 삭제함으로써 두 쿼리의 결과가 상이하게 나타나는 비 일관성이 발생한다.

- Phantom Read

한 트랜잭션 안에서 일정 범위의 레코드를 두 번 이상 읽을 때, 첫 번째 쿼리에서 없던 레코드가 두 번째 쿼리에서 나타나는 현상으로, 트랜잭션 도중 새로운 레코드가 삽입되는 것을 허용하기 때문에 나타난다.

*****

## 데이터베이스 별 지원하는 격리 레벨

MYSQL 의 기본 격리 레벨은 `REPEATABLE READ` 이며 앞서 살펴 본 4 가지 격리 레벨을 사용할 수 있다.

- [MYSQL DOCS](https://dev.mysql.com/doc/refman/8.0/en/set-transaction.html)

ORACLE 의 기본 격리 레벨은 `READ_COMMITED` 이며 `READ_COMMITED`, `SERIALIZABLE` 2 가지 격리 레벨을 지원한다. 단, `REPEATEABLE_READ` 의 경우 **SELECT ... FROM ... FOR UPDATE** 문으로 `REPEATEABLE_READ` 기능을 사용할 수 있다.

ORACLE 의 경우는 특이사항으로 고립화 수준을 높이더라도 LOCK 을 사용하지 않기 때문에 동시성이 저하 되지 않는다.

- [ORACLE DOCS](https://docs.oracle.com/cd/E25054_01/server.1111/e25789/consist.htm)

### ORACLE LOCK

> 내용이 어려워서 간략하게만 요약한다.

- **행은 작성자(writer/수정자)가 수정한 경우에만 잠긴다.**
- 질의어(QUERY) 가 행을 업데이트 한다면 이 행에 대해서만 잠김을 획득한다. 행 수준에 대한 테이블 데이터를 잠그게되면 데이터베이스는 동일한 데이터에 대한 경합(contention)을 최소화한다.
- 정상적인 상황이라면 데이터베이스가 행 잠금 수준을 블록 또는 테이블 단위로 확대(escalate)하지 않는다. (**다른 행은 잠기지 않는다는 뜻**)

- 독자(reader)는 작성자를 잠그지 않는다.
  - 행에 대한 독자를 잠그지 않기 때문에 작성자는 이를 수정할 수 있다. (단, `SELECT ... FOR UPDATE` 는 예외로 읽고 있는 대상이 잠긴다.)

- 작성자(writer)는 독자를 잠그지 않는다.
  - 작성자가 행을 변경하는 경우 데이터베이스는 undo 데이터를 사용하여 독자에게 행에 대한 일관된 보기를 제공한다.

*****

## 전파 레벨

```java

public class AService {
    @Autowired
    private BService bService;

    @Autowired
    private CService cService;

    // ...

    public void doATask() {
        bService.doBTask();
        cService.doCTask();
        // ...
    }
}

// ...

public class BService {
    @Transaction
    public void doBTask() {
        // ...
    }
}

public class CService {
    @Transaction
    public void doCTask() {
        // ...
    }
}
```

`@Transactional` 는 해당 메소드를 하나의 트랜잭션으로 진행하도록 한다는 뜻이다. 이 때 트랜잭션 내부에서 다른 `@Transactional` 이 선언된 메소드를 선언하면 어떻게 처리될까?

새로운 트랜잭션을 생성할 수도 있고, 이미 실행되고 있는 부모 트랜잭션에 합류할 수도 있을 것이다. 스프링은 이와 같은 전파를 옵션으로 설정할 수 있다.

### REQUIRED (기본 값)

`doATask()` 는 하나의 트랜잭션으로 묶여 처리된다. 즉, `doBTask()` 과 `doCTask()` 는 같은 A 트랜잭션으로 처리된다.

### MANDATORY

```java
public class CService {
    @Transaction(propagation = Propagation.MANDATORY)
    public void doCTask() {
        // ...
    }
}
```

`doATask()` 의 경우 `REQUIRED` 과 동일하게 처리된다. 그러나 `doCTask()` 만 단독 실행할 경우 부모 트랜잭션이 없기 때문에 예외가 발생한다.

### REQUIRES_NEW

```java
public class CService {
    @Transaction(propagation = Propagation.REQUIRES_NEW)
    public void doCTask() {
        // ...
    }
}
```

`doATask()` 에서 `doBTask()` 과 `doCTask()` 각각의 트랜잭션이 생성되어 처리된다. 즉, 각 기능이 끝날 때 커밋된다.

### SUPPORTS

```java
public class CService {
    @Transaction(propagation = Propagation.SUPPORTS)
    public void doCTask() {
        // ...
    }
}
```

부모 트랜잭션이 있다면 합류한다. 진행 중인 부모 트랜잭션이 없다면 트랜잭션을 생성하지 않는다.

### NOT_SUPPORTED

```java
public class CService {
    @Transaction(propagation = Propagation.NOT_SUPPORTED)
    public void doCTask() {
        // ...
    }
}
```

부모 트랜잭션이 있다면 보류한다. 진행 중인 부모 트랜잭션이 없다면 트랜잭션을 생성하지 않는다.

### NEVER

```java
public class CService {
    @Transaction(propagation = Propagation.NEVER)
    public void doCTask() {
        // ...
    }
}
```

부모 트랜잭션이 있다면 예외가 발생한다. 위 예제라면 `doATask()` 에서 예외가 발생할 것이다.

부모 트랜잭션이 없다면 트랜잭션을 생성하지 않고 기능을 수행한다.

### NESTED

```java
public class CService {
    @Transaction(propagation = Propagation.NESTED)
    public void doCTask() {
        // ...
    }
}
```

부모 트랜잭션이 있다면 중첩된 트랜잭션을 생성한다. 위 예제라면 `doATask()` 에서 중첩 트랜잭션을 생성한다. 중첩 트랜잭션의 커밋은 부모에게 위임한다. 중첩 트랜잭션의 롤백은 부모에게 전파되지 않는다. 부모 트랜잭션에서 롤백에 발생하면 중첩 트랜잭션도 롤백된다.

부모 트랜잭션이 없다면 새로운 트랜잭션을 생성한다.

*****

### 전파 타입 정리

| | 진행 중인 트랜잭션이 있음 | 진행 중인 트랜잭션이 없음 |
| --- | --- | --- |
| `REQUIRED` | 해당 트랜잭션을 사용한다. | 새로운 트랜잭션을 생성한다. |
| `MANDATORY` | 해당 트랜잭션을 사용한다. | 예외가 발생한다. |
| `REQUIRES_NEW` | 해당 트랜잭션은 보류되고 새로운 트랜잭션을 생성한다. | 새로운 트랜잭션을 생성한다. |
| `SUPPORTS` | 해당 트랜잭션을 사용한다. | 트랜잭션 없이 진행한다. |
| `NOT_SUPPORTED` | 해당 트랜잭션을 보류한다. | 트랜잭션 없이 진행한다. |
| `NEVER` | 예외가 발생한다. | 트랜잭션 없이 진행한다. |
| `NESTED` | 중첩 트랜잭션을 생성한다. | 새로운 트랜잭션을 생성한다. |

*****

## 학습내용 참고 출처

- [[10분 테코톡] 🙊 에이든의 트랜잭션 메커니즘](https://www.youtube.com/watch?v=ImvYNlF_saE)
- [[10분 테코톡] 🌼 예지니어스의 트랜잭션](https://www.youtube.com/watch?v=e9PC0sroCzc)
- [데이터베이스 Transaction Isolation Level](https://effectivesquid.tistory.com/entry/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-Isolation-Level)
- [[db] 트랜잭션 격리 수준(isolation level)](https://joont92.github.io/db/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EA%B2%A9%EB%A6%AC-%EC%88%98%EC%A4%80-isolation-level/)
- [Isolation level (정리)](http://egloos.zum.com/ljlave/v/1530887)
- [[Spring] 트랜잭션의 전파 설정별 동작](https://deveric.tistory.com/86)