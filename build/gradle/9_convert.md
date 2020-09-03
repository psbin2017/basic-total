# 변환하기

학습 내용 자료 출처 : [열혈강의] 그레이들 youtube [Chapter 09 그레이들로 변환하기](https://www.youtube.com/watch?v=jPXyzDxou5g&list=PL7mmuO705dG2pdxCYCCJeAgOeuQN1seZz&index=32)

레거시 프로젝트의 앤트 메이븐 빌드시스템을 그레이들로 변환할 수 있다.

## 앤트에서 그레이들

앤트 프로젝트가 그레이들 프로젝트로 변환할 수 있는 방법은 크게 2가지이다.

그레이들에서 제공하는 기능을 활용하여 앤트로 구현된 내용을 그레이들로 변환하는 방법

1. Ant Task 에 해당하는 Gradle Task 확인
2. build.gradle

앤트로 구성된 빌드 내용을 그레이들에서 활용하는 방법

1. Ant build.xml import
2. Ant Task 호출
3. build.gradle

크게 2가지 이다.

다음은 그레이들에서 이미 작성된 앤트를 사용하는 방법이다.

```gradle
ant.importBuild 'build.xml'

task exeTask {
    doLast {
        ant.echo('hello from ant')
        ant.zip(descfile:'archive.zip') {
            fileset(dir:'src') {
                exclude(name:'**.xml')
                include(name:'**.java')
            }
        }
        def path = ant.path {
            fileset(dir:'src', includes:'*.java')
        }
        path.list().each {
            println it
        }
    }
}
```

앤트의 작성된 build.xml 을 읽는 방법은 다음과 같다.

```xml
<!-- build.xml -->
<project>
    <target name="antTask">
        <echo>To Gradle From build.xml of Ant</echo>
    </target>
</project>
```

```gradle
// ant 의 build.xml 파일 import
ant.importBuild 'build.xml'

task gradleTask(dependsOn: exeTask) {
    doLast {
        println 'This is Gradle User Task'
    }
}
```

ant:echo 가 먼저 실행되고 gradle 에 정의된 내용이 출력된다.

작성된 내용은 그레이들에서 앤트를 의존하는 관계인데 반대의 경우는 다음과 같다.

```xml
<project>
    <target name="exeTask" depends="gradleTask">
        <echo>To Gradle From build.xml of Ant</echo>
    </target>
</project>
```

```gradle
// ant 의 build.xml 파일 import
ant.importBuild 'build.xml'

task gradleTask {
    doLast {
        println 'This is Gradle User Task'
    }
}
```

의존관계 형성이 역전된 것 외에는 특이사항이 없다.

### 예제

```xml
<project name="9_4" default="jar" basedir=".">
    <description>
        Create jar File
    </description>

    <!-- Ant property -->
    <property name="version" value="0.1" />
    <property name="src" location="src" />
    <property name="build" location="build" />
    <property name="gradle" location="gradle" />

    <target name="compile" description="java code compile">
        <mkdir dir="${build}">
        <javac srcdir="${src}" destdir="${build}" includeantruntime="false" />
    </target>

    <target name="jar" description="create jar">
        <mkdir dir="${gradle}/lib">
        <jar jarfile="${gradle}/lib/${ant.project.name}-${version}.jar" basedir="${build}" />
    </target>

</project>
```

> gradle jar

```gradle
ant.importBuild 'build.xml'

task exeTask(dependsOn: "jar" ) {
    println "ant version:" + ant.properties.version
    println "ant src:" + ant.properties.src
    println "ant build:" + ant.properties.build
    println "ant gradle:" + ant.properties.gradle

    ant.properties.version = "2.0"

    println "ant version:" + ant.properties.version
}
```

`build.xml` 에 선언된 속성 값에 대해서 사용이 가능하다.

> gradle exeTask

## 메이븐에서 그레이들

앤트와 마찬가지로 메이븐 프로젝트를 그레이들 프로젝트로 변환 하는 방법도 동일하다.

그레이들에서 제공하는 기능을 활용하여 메이븐으로 구현된 내용을 그레이들로 변환하는 방법

1. Maven plug-in → Gradle plug-in
2. build.gradle

메이븐으로 구성된 빌드 내용을 그레이들에서 활용하는 방법

1. pom.xml → build.xml
2. build.gradle

## 배포하기

- compile : 프로젝트의 소스를 컴파일하는 데 필요한 의존 관계 지정 시 사용
- runtime : 프로젝트 클래스에 대하여 필요한 의존 관계 지정 시 사용 (compile 에 포함됨)
- testCompile : 프로젝트의 테스트 소스를 컴파일하는 데 필요한 의존 관계 지정
- testRuntime : 테스트를 실행하는 데 필요한 의존 관계 지정 시 사용
- providedCompile : 컴파일 수행 시에는 필요하지만 배포하고자 할 경우에는 제외 되어야할 때 사용
- providedRuntime : 실행 시(runtime) 에만 사용 (war plug-in 에만 사용 가능)

### gradle wrapper

> gradle wrapper
