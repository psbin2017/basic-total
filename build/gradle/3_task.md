# 태스크

학습 내용 자료 출처 : [열혈강의] 그레이들 youtube [Chapter 03 그레이들 기본](https://www.youtube.com/watch?v=bfZ88Omjf-E&list=PL7mmuO705dG2pdxCYCCJeAgOeuQN1seZz&index=8)

## 태스크 형태

```gradle
task hello<<{
    println 'Hello Gradle!'
}
```

```gradle
// def 형으로 선언 및 클로저 정의
def strMsg = { println 'Hello Gradle!' }

// 태스크 정의
task hello{}

hello.leftShift(strMsg)
```

두 개의 빌드 스크립트는 출력 결과가 동일하다.

## 태스크 기본

### <<

```gradle
// 라이프 사이클 중 실행 단계에서 수행됨
task lifeTask1<<{
    println 'Hello Gradle1'
}

// 라이프 사이클 중 설정 단계에서 수행됨 (먼저 수행됨)
task lifeTask2{
    println 'Hello Gradle2'
}

// > gradle lifeTask1 lifeTask2
```

### 프로퍼티

```gradle
task propTask<<{
    println description + 'Good';
}
propTask.description = 'Task Execution'

// > gradle propTask
```

아래에 프로퍼티를 작성해야 정상적으로 실행된다. (`task` 를 먼저 선언해서 해소할 수 있다.)

### String 객체 함수

```gradle
task stringTask<<{
    String strOutput = 'Good Day'
    println '1. String change:' + strOutput.toUpperCase()
    println '2. String change:' + strOutput.toLowerCase()
}

// > gradle stringTask
```

### times

```gradle
task loopTask<<{
    // 루프문 0 부터 9 까지 10번 반복
    10.times { println "$it" }
}
// > gradle loopTask
```

변수를 받아 반복 정의할 수도 있다.

```gradle
3.times{counter->
    task "counterTask$counter"<<{
        println "task counter : $counter"
    }
}

// > gradle counterTask1
```

### doFirst / doLast

`doFirst` 와 `doLast` 를 통해서 태스크의 실행 순서를 처리할수 있다.

이 경우 `doFirst` > `task` > `doLast` 로 실행된다.

```gradle
task helloTask<<{}
task helloTask.doFirst<<{}
task helloTask.doLast<<{}

// > gradle helloTask
```

### ext

`ext` 는 그레이들에서 제공하는 기본 객체이다.

```gradle
task userInfo{
    ext.userName = "John"
}

task userTask<<{
    println "Name : " + userInfo.userName
}

// > gradle userTask
```

### defaultTasks

실행 시점에 task 를 나열하지 않는 `defaultTasks` 가 있다.

```gradle
defaultTasks 'defTask1','defTask2','defTask3'
task defTask1<<{}
task defTask2<<{}
task defTask3<<{}

// > gradle (선언 없이도 defaultTasks 로 선언된 내용이 실행됨)
```

### each

`times` 와 다른 점은 each 는 해당 객체와 값의 갯수로 실행하게 된다.

```gradle
def confMap = ['imgConf':'helloWold.co.kr', 'smsConf':'byeWorld.co.kr']

confMap.each { svDomain, domainAddr->
    task "eachTask$(svDomain)"<<{
        println domainAddr
    }
}

// > gradle eachTaskimgConf
```

## 빌드 분기

### onlyIf

특정 조건일 때 빌드 수행

```gradle
task onlyTask {}

task onlyTask.onlyIf{
    buildType == 'partial-build'
}

// > gradle onlyTask (실패)
// > gradle -PbuildType=partial-build onlyTask (성공)
```

### 예외 처리

```gradle
task processTask<<{}

processTask<<{
    if ( process == 'error' ) {
        throw new StopExecutionException()
    }
}

processTask<<{}

// > gradle Pprocess=error processTask (빌드 정상, 예외 처리되어 아래 빌드 미수행)
// > gradle Pprocess=ok processTask (빌드 정상, 예외 미발생)
```

## 실행 순서 제어

### mustRunAfter

`afterTask` 는 `beforeTask` 를 수행하고난 후에 실행한다.

```gradle
task beforeTask<<{}

task afterTask<<{}

afterTask.mustRunAfter beforeTask

// > gradle (afterTask 를 우선 선언해도 beforeTask 먼저 실행)
```

### shouldRunAfter

`shouldRunAfter` 는 `mustRunAfter` 처럼 순서를 지정하지만, 순서가 무시되는 예외가 있다.

```gradle
task beforeShTask<<{}
task afterShTask<<{}

afterShTask.shouldRunAfter beforeShTask

// > gradle (afterShTask 를 우선 선언해도 beforeShTask 먼저 실행)
```

### 의존관계와 순서 지정 방법간 차이

`dependsOn` 을 통해 각 태스크간에 의존관계를 지정할 수 있다.

태스크간 지정된 흐름으로 빌드가 수행되지만, `mustRunAfter` 와 `shouldRunAfter` 는 지정된 태스크만 순서를 결정하여 수행한다.

## 태스크 그래프

방향성 비순환 그래프 : 태스크간 의존 관계를 시각적으로 표현

```gradle
task FirstTask<<{
    println 'First Task'
}

task SecondTask(dependsOn : 'FirstTask')<<{
    println 'Second Task'
}

task ThirdTask(dependsOn : 'SecondTask')<<{
    println 'Third Task'
}

// > gradle ThirdTask
```

제약사항은 다음과 같다.

1. 실행시 같은 태스크 중복 지정
2. `mustRunAfter`, `shouldRunAfter`
3. 종료 태스크(Finalizer Task) 지정

### 종료 태스크

```gradle
task FinTask<<{
    println 'Finish Task'
}

FirstTask.finlizedBy FinTask
```

만약 `FirstTask` 에서 예외 발생시 `FinTask` 로 진행된다.
