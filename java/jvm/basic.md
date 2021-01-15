# 개요

자바 개발자는 일반적으로 메모리 관리를 하지 않는다. 가비지 콜렉션이 다 알아서 해주기 때문이다.

> 저수준의 세부를 일일이 신경쓰지 않는 대가로 저수준 제어권을 포기하는 사상이 자바의 핵심이다.
> 
메모리 관리는 저수준에 가까운데 우리는 이를 포기하고, 비즈니스 로직이나 짜고 있다는 것이다. GC 는 두가지 원칙을 준수한다.

- 메모리가 부족할 때만 GC 를 수행한다.
- 살아있는 객체는 수집하지 않는다.

## Mark and Sweep Algorithm

자바의 GC 알고리즘은 `Mark and Sweep Algorithm` 이다. 이해하기도 정말 쉬운데, 살아있는 객체를 표시하고 남은 죽은 객체를 가비지 컬렉터가 수거해가는 것이다.

## Weak Generational 가설

`Weak Generational` 은 객체를 이원적 분포 양상을 보인다고 해석하였고, 실제로 경험으로 검증하였다.

**젋은 객체와 늙은 객체.**

1. 새로운 객체는 `Young Generation` 공간에 생성된다.
2. 객체마다 `Generation Count` (수집 대상에 무사통과한 횟수) 를 센다.
3. 오래 살아남은 객체는 `Old Generation` 공간에 보관된다.

이 가설에서는 늙은 객체는 젋은 객체를 참조하는 일은 거의 없다고하지만, 카드 테이블이라는 늙은 객체가 젊은 객체를 참조하는 정보를 기록하여 확인한다. 늙은 객체를 전부 확인하는 것이 아니라 카드 테이블만 확인하여 성능상 이점을 가진다.

## Young Generation

젋은 객체가 모여있는 공간이다. 대다수의 객체는 이 영역에서 수집되기 때문에 GC 가 자주 일어나는 영역이다. 이에 따라 `Young Generation` 의 GC 는 수행시간이 짧아야 한다.

```text
-XX:NewSize
-XX:MaxNewSize
-XX:NewRatio
```

튜닝에 있어 `-XX:NewRatio` 를 튜닝하는 것보다는 `-XX:NewSize` 와 `-XX:MaxNewSize` 의 사이즈를 동일하게 설정하여 튜닝하는게 좋다고한다.

길이를 달리하여 동적으로 사이즈가 줄어들고 늘어나면 `-XX:NewRatio` 를 계산해야하기 때문에 유리한 것으로 보인다.

### Eden Space

`Young Generation` 의 일부분으로 새로운 객체가 `Eden Space` 의 용량보다 큰 경우를 제외하여 `Eden Space` 에 할당된다. 그러다 할당 영역을 확보하지 못하면 **Minor GC** 를 수행한다.

수행된 GC 로 인해 메모리 단편화가 발생하여 연속된 메모리 공간이 없게 된다.
**bump-the-pointer** 라는 기술로 중간 중간 메모리를 할당하는 일은 없다. 이런 단편화를 해결하기 위해 **Compaction** 을 수행한는데 이 과정에서 오버헤드가 발생할 수 밖에 없다.

### Survivor Space

`Eden Space` 의 오버헤드를 줄이고자 생긴 영역이다. `Survivor Space` 는 동일한 사이즈로 `from` 과 `to` 로 나뉜다. 보통 **VisualVm** 같은 모니터링 툴에는 `S0`, `S1` 로 표시되곤 한다.

**Minor GC** 이후 `Eden Space` 에 생존한 젊은 객체를 `Survivor Space` 에 넘어오고 `Eden Space` 를 비우게 되어 단편화를 해결하게 된다.

```text
-XX:SurvivorRatio
```

`-XX:SurvivorRatio` 속성을 통해 `Eden Space` 과 `Survivor Space` 의 비율을 정할 수 있다. `Survivor Space` 가 두 개로 나뉘는 이유는 마찬가지인 **Minor GC** 대상으로 단편화가 발생하기 때문이다.

그러면 `from` 과 `Eden Space` 의 남은 객체를 `to` 로 옮겨서 해결한다.

### Minor GC

**Minor GC** 가 발생하면 생존한 객체는 `Generation Count` 가 증가한다. 이 프로세스를 **Aging** 이라고 한다.

방금 내용처럼 from 에서 연속된 메모리 공간을 확보하지 못해 메모리 할당이 불가능한 경우

1. `from` 에 생존객체를 `to` 로 옮기고
2. `Eden Space` 생존객체를 `from` 으로 옮긴다.

이 프로세스 또한 `Weak Generation` 가설에 입각한 행동인데 수명 가능성이 긴 녀석을 먼저 배치하여 오버헤드를 줄이는 것이다.

### Promotion

여러 번의 **Aging** 으로 젊은 객체는 `Old Generation` 에 승진할 기회를 가진다. 최근에 승진을 마친 중년 객체는 `Survivor Space` 에 존재할 수 있는데 이 경우 GC 가 복잡하게 보이는데, 앞서 말한 카드 테이블에서만 검사하여 중년 객체를 쉽고 빠르게 찾을 수 있다.

더 **Aging** 과정을 가지게 되어 승진 과도기를 마친 늙은 객체는 `Old Generation` 에 이동하게 된다.

```text
-XX:InitialTenuringThreshold
-XX:MaxTenuringThreshold
```

### Premature Promotion

적당한 나이를 먹지 않았는데 어쩔수 없이 이동하는 경우를 조기 승진이라고 한다. 메모리 할당이 잦음으로 인해 `Survivor Space` 에 적당한 공간이 없는 경우 나이에 상관 없이 Old `Generation` 으로 옮겨지는 것을 말한다. 또 `Eden Space` 에 할당할 수 없을 만큼 큰 객체도 `Old Generation` 에 할당되는데 이 또한 조기 승진이다.

```text
-XX:TargetSurvivorRatio
```

`-XX:TargetSurvivorRatio` 은 **Minor GC** 이후 `from` 의 사용율을 제한하는 옵션이다. 이 경우도 조기 승진이되는데, **Major GC** 또는 **Full GC** 가 일어나기 전까지 수거해가지 않기 때문에 적절하게 `Young Generation` 의 사이즈를 튜닝해야한다.

### Old Generation

조기 승진하거나 승진한 객체들의 영역이다. 대다수가 젊은 객체로 수명을 다하기 때문에 `Old Generation` 오는 객체는 비율상 낮다.

### Major GC

`Old Generation` 이 꽉찬 경우 수행된다. GC 일어나는 빈도수가 적지만 `Old Generation` 의 공간을 `Young Generation` 보다 크게 잡기때문에 GC 소모시간이 길다.

### Full GC

**Minor GC** + **Major GC** 를 **Full GC** 라고 한다.

## Permanent Generation

`Permanent Generation` 은 자바8 시점으로 사라졌다. 클래스의 메타데이터, 메서드의 메타데이터, 상수풀, static 변수 등이 들어간다. 이름이 영구적이다보니 GC 대상이 아닌 것처럼 보이지만 GC 가 수행된다.

```text
-XX:PermSize
-XX:MaxPermSize
```

GC 를 수행하지 않는다면 OOME (`java.lang.OutOfMemoryError: PermGen space`) 가 난다.

- collection 을 static 을 만들고 요소를 계속 할당하는 경우
- 서버를 주기적으로 재시작하지 않고 계속해서 반영하는 HotDeploy 로 메타데이터가 계속 쌓이는 경우

## Metadata

자바 8 시점에서 나타난 영역으로 OS 에서 관리하는 영역이다. 상수풀이나 static 변수는 Heap 메모리로 옮겨졌고, 개발자가 실수하기 쉽거나 변경이 잦은 내용은 힙 메모리로 끌고와서 GC 대상이 되게 끔 변경하였다. 이 또한 OOME (`java.lang.OutOfMemoryError: Metadata space`) 가 발생할 수 있다.

```text
-XX:MetaspaceSize
-XX:MaxMetaspaceSize
```

## OOME

메모리가 부족한경우 힙 메모리의 GC 를 수행하는데 수행했음에도 불구하고 새로운 객체를 할당할 수 없는 경우 OOME 가 발생한다. 빠르게 해결해야하는 경우 `-Xmx` 와 `-Xms` 메모리를 늘려보는 방법이 있다.

## 상식

- Hotspot VM의 GC는 Arena라는 메모리 영역에서 작동한다.
- Hotspot VM은 시작 시 메모리를 유저 공간에 할당/관리한다.
- 따라서 힙 메모리를 관리할 때 시스템 콜을 하지 않으므로 커널 공간으로 컨텍스트 스위칭을 하지 않아서 성능 향상에 도움이 된다.
- Hotspot VM은 할당된 메모리 가장 마지막의 다음 영역을 가리켜 연속된 빈 공간에 효율적으로 빠르게 할당하는 bump-the-pointer라는 기술을 사용했다.
- Hotspot VM은 멀티 스레드 환경에서 객체를 할당할 때 스레드 간의 경합 등등의 문제를 줄이고자 TLAB(Thread Local Allocation Buffer)를 사용했다.
- Eden Space를 여러 버퍼로 나누어 각 어플리케이션 스레드에게 할당함으로써 자기 자신이 사용해야 할 버퍼를 바로 찾게되고, - 리소스를 공유해서 생기는 문제를 없애버렸다.
- 만약 본인에게 할당된 TLAB가 부족할 경우에는 크기를 동적으로 조정한다.

## 학습 내용 참고 출처

- [(JVM) Garbage Collection Basic](https://perfectacle.github.io/2019/05/07/jvm-gc-basic/?fbclid=IwAR2RrYl96TgWtzzmrQmLzWNG69EtGA_7-13M263CNk1SF6nsbaE-p8fEeYA#%EC%95%8C%EC%95%84%EB%91%90%EB%A9%B4-%EC%A2%8B%EC%9D%84-%EC%83%81%EC%8B%9D)