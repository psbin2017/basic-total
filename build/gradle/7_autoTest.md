# 테스트 자동화

학습 내용 자료 출처 : [열혈강의] 그레이들 youtube [Chapter 07 테스트 자동화](https://www.youtube.com/watch?v=vRJ42AV8V3w&list=PL7mmuO705dG2pdxCYCCJeAgOeuQN1seZz&index=24)

1. 환경 차이에 대한 제어 가능
2. 특정 조건에 대한 테스트 수행
3. 범위에 대한 테스트 수행
4. 느린 테스트에 대한 지양

## 환경 차이에 대한 제어 가능

환경 변수 예제

```properties
// EnvDir/dev/conf.properites
mode=dev
domain=devpsbin2017-dev.github.io

// EnvDir/mid/conf.properites
mode=mid
domain=devpsbin2017-mid.github.io

// EnvDir/prod/conf.properites
mode=prod
domain=devpsbin2017-prod.github.io
```

```gradle
task exeTask {
    doLast {
        def dev = new Properties()
        dev.load(new FileInputSteam("EnvDir/dev/conf.properties"))
        println "mode = ${dev.mode}"

        def mid = new Properties()
        mid.load(new FileInputSteam("EnvDir/mid/conf.properties"))
        println "mode = ${mid.mode}"

        def prod = new Properties()
        prod.load(new FileInputSteam("EnvDir/prod/conf.properties"))
        println "mode = ${prod.mode}"
    }
}
```

### ext / apply from

딱히 실무적이진 않지만 참고용

```properties
// EnvDir/dev/conf.properites
ext.mode_dev=dev
ext.domain_dev=devpsbin2017-dev.github.io

// EnvDir/mid/conf.properites
ext.mode_mid=mid
ext.domain_mid=devpsbin2017-mid.github.io

// EnvDir/prod/conf.properites
ext.mode_prod=prod
ext.domain_prod=devpsbin2017-prod.github.io
```

```gradle
apply from: "EnvDir/dev/conf.properties"
apply from: "EnvDir/mid/conf.properties"
apply from: "EnvDir/prod/conf.properties"

task exeTask {
    doLast {
        println "mode = ${mode_dev}"
        println "mode = ${mode_mid}"
        println "mode = ${mode_prod}"
    }
}
```

### environments

```gradle
environments{
    dev {
        mode = 'dev'
        domain = 'devpsbin2017-dev.github.io'
    }
    mid {
        mode = 'mid'
        domain = 'devpsbin2017-mid.github.io'
    }
    prod {
        mode = 'prod'
        domain = 'devpsbin2017-prod.github.io'
    }
}
```

```gradle
task exeTask {
    doLast {
        dev envBlock = new File('EnvDir/conf.gradle').toURL()

        def dev = new ConfigSlurper("dev").parse(envBlock)
        println "mode = " + dev.mode

        def mid = new ConfigSlurper("mid").parse(envBlock)
        println "mode = " + mid.mode

        def prod = new ConfigSlurper("prod").parse(envBlock)
        println "mode = " + prod.mode
    }
}
```

## 패턴 테스트 자동화

```gradle
apply plugin: 'java'

// JDK 버전 지정
sourceCompatibilty = 1.7
targetCompatibility = 1.7

// 컴파일시 인코딩 지정
def defaultEncoding = 'UTF-8'
[compileJava, compileTestJava]*.options*.encoding = defaultEncoding

repositories {
    mavenCentral()
}

dependencies {
    testCompile 'junit:junit:4.11'
}

// 테스트 객체를 받아 특정 패턴의 클래스 테스트
task exeTask(type: Test) {
    include '**/Gradle*.class'

    filter {
        includeTestsMathing 'Gradle'
    }
}
```

### 다른 예제

```gradle
apply plugin: 'java'

// JAVA 플러그인에서 제공
test {
    // TestNG 사용
    useTestNG()

    // 테스트 포함 및 제외 선언
    include 'org/foo/**'
    exclude 'org/boo/**'

    // 테스트 도중 오류 사항 콘솔에 표시
    testLogging.showStandardStreams = true

    // 테스트를 위한 힙 메모리 설정
    minHeapSize = "128m"
    maxHeapSize = "512m"

    // 테스트 실행 라이프사이클 표시
    beforeTest {
        descriptor -> logger.lifecycle("Running Test: " + descriptor)
    }

    // 테스트 JVM 의 오류 및 출력 내용 등을 표시
    onOutput {
        descriptor, event -> logger.lifecycle("Test: " +descriptor+ "produce standard out/err" +event.message )
    }
}
```

## JUnit

```java
package test.java;

public class GradleMsg {

    private String msg;

    public GradleMsg() {

    }

    public GradleMsg(String msg) {
        this.msg = msg
    }

    public String getMsg() {
        return this.msg;
    }
}
```

```java
package test.java;

import org.junit.Before;
import org.junit.Test;

public class GradleMsgTest {

    private GradleMsg gMsg;

    @Before
    public void printMsg() throws Exception {
        gMsg = new GradleMsg("Hello Gradle");
    }

    @Test
    public void testMsg() throws Exception {
        gMsg = new GradleMsg("Hello Gradle");
    }
}
```

```gradle
apply plugin : 'java'

repositories {
    mavenCentral()
}

dependencies {
    testCompile 'junit:junit:4.8.2'
}

test {
    // 로그 부분 처리에 대한 속성 지정
    testLogging {
        events "passed", "skipped", "failed", "standardOut", "StandardError"
    }
}

test exeTask(type: Test) {
    // 빌드 수행시 수행 결과를 html 파일 제공
    reports.html.destination = file("${reports.html.destination}/integration")
    // 빌드 수행시 수행 결과를 xml 파일 제공
    reports.junitXml.destination = file("${reports.junitXml.destination}/integration")
}
```

> gradle -b build.gradle test

로거가 실행시 나타남

> gradle -Penv=integration exeTask0

Build 디렉토리에 수행 결과가 떨어짐

## 병렬 테스트

`maxParallelForks` 를 사용하여 병렬 수행 프로세스 개수를 지정하며 병렬 처리는 시스템 사용과 함께 테스트에 가장 적합한 프로세스 개수를 파악해야 한다.

```gradle
apply plugin 'java'

repositories {
    mavenCentral()
}

dependencies {
    testCompile 'junit:junit:4.11'
}

def defaultEncoding = 'UTF-8'
[comileJava, compileTest.java]*.options.*encoding = defaultEncoding

test {
    // Gradle 의 병렬 테스트 진행
    maxParallelForks = 1
}
```

앞서 본 `GradleMsg.java` 는 그대로 사용한다.

```java
package test.java;

public class ParallelTest {

    protected void stress() {
        System.out.println("ParallelTest > Start >");
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
package test.java;

import org.junit.Before;
import org.junit.Test;

public class GradleMsgTest1 extends ParallelTest {
    private GradleMsg gMsg;

    @Before
    public void setUp() {
        stress();
    }

    @Test
    public void testMsg() {
        gMsg = new GradleMsg("Hello Gradle");
    }
}
```

> gradle clean test

처리 프로세스의 개수가 많아 질수록 시간이 늘어난다.

병렬 테스트의 목적은 스트레스 테스트가 강한 것으로 보인다.
