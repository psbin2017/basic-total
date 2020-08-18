# 의존성 관리

학습 내용 자료 출처 : [열혈강의] 그레이들 youtube [Chapter 06 의존 관계](https://www.youtube.com/watch?v=hAtN9pd9xGg&list=PL7mmuO705dG2pdxCYCCJeAgOeuQN1seZz&index=21)

의존성의 관리 목적은 크게 다음과 같다.

- 의존 관계 해결 자동화: 설정 지점을 통해 특정 위치에서 필요한 라이브러리를 내려받거나 복사하여 추가
- 전이적 의존성 관리: 외부 라이브러리나 프로젝트 등이 추가로 다른 외부의 라이브러리나 프로젝트를 필요한지를 파악하거나 관리
- 의존성 관계 표시: 프로젝트가 특정 외부 라이브러리의 어떤 버전에 의존하는지 여부 표시

```gradle
// build 를 위한 환경 구성/환경 설정
configurations {
    conf1
}

// 의존 관계 정의
dependencies {
    conf1 gradleApi() // 의존 관계를 지정한다
}

task exeTask {
    // 설정된 환경 구성을 사용
    doLast {
        configuration.cof1.each {
            println it.absolutePath
        }
    }
}
```

1. 환경 구성을 정의한다.
   - 필요 요소에 대한 의존 관계를 설정
2. 의존 관계를 정의한다.
   - 필요 요소를 참조하여 관계를 정의 (외부/내부 라이브러리 및 프로젝트)
3. 환경 구성을 사용한다.

## 의존성 지정 방법

- 외부 모듈 의존 관계 지정: 인터넷 저장소에 대하여 의존 관계 지정(저장소 정의 필요)
- 파일 의존 관계 지정: 파일 시스템의 파일에 대해 의존 관계 지정
- 프로젝트 의존 관계 지정: 특정 프로젝트에서 다른 프로젝트에 대하여 의존 관계 지정
- Gradle API 의존 관계 지정: 사용 중인 API가 포함된 라이브러리 파일에 대해 의존 관계 지정
- 로컬 그루비 의존 관계 지정: 현재 사용 중인 그루비에 대해 내장된 라이브러리를 의존 관계 지정

### 외부 모듈 의존 관계 지정

> '그룹명', '모듈명', '버전 번호'

```gradle
dependencies {
    conf1 'ch.qos.logback:logback-classic:1.0.13'
    conf1 'org.springframework:spring-orm:4.0.2.RELEASE'
    conf1 (group: 'org.gradle.api.plugins', name: 'gradle-cargo-plugin', version: '0.6.1')
}
```

위에서 작성한 예제를 종합하면 다음과 같다.

```gradle
configurations{
    conf1
}

repositories{
    mavenCentral()
}

dependencies{
    conf1 'ch.qos.logback:logback-classic:1.0.13'
    conf1 'org.springframework:spring-orm:4.0.2.RELEASE'
    conf1 (group: 'org.gradle.api.plugins', name: 'gradle-cargo-plugin', version: '0.6.1')
}

task exeTask{
    doLast{
        configurations.conf1.each{
            println it.absolutePath
        }
    }
}
```

### 파일 의존 관계 지정

```gradle
dependencies{
    // 1. 파일 의존성 - 특정 파일이나 라이브러리 지정
    conf1 files("lib/commonLib.jar")
    // 2. 파일 의존성 - 디렉토리 지정
    conf1 fileTree(dir:"lib", include:"**/*.jar")
}
```

### 프로젝트 의존 관계 지정

```gradle
dependencies{
    compile project(':subProject')
    conf1 'ch.qos.logback:logback-classic:1.0.13'
}
```

### Gradle API + 로컬 그루비 의존 관계 지정

```gradle
apply plugin: 'groovy'

dependencies{
    // gradle api 의존관계
    conf1 gradleApi()

    // local groovy 의존관계
    conf1 localGroovy()
}
```

## 동적 버전 지정

```gradle
configurations.confDef.resolutionStrategy.cacheDynamicVerionsFor 10, 'minutes'
configurations.confDef.resolutionStrategy.cacheChangingModulesFor 10, 'hours'

dependencies{
    // 버전 중 최신 버전 지정
    confDef 'org.slf4j:slf4j-api:1.+'

    // 출시 버전 중 최신 버전에 대한 지정
    confDef 'commons-cli:commons-cli:latest.integration'

    // 배포 버전 중 최신 버전에 대한 지정
    confDef 'junit:junit:latest.release'
}
```

## 버전 충돌 해소

기본적으로 최신 버전을 사용하는 것을 정책으로 함

- Newest: 버전이 충돌할 경우 최신 버전을 사용한다.
- Fail: 버전이 충돌할 경우 오류를 발생시켜 빌드가 실패하도록 한다.

```gradle
configurations{
    confDef
    // 환경 구성 상속
    newConfDef.extendsFrom confDef
}

configurations.newConfDef{
    resolutionStrategy{
        failOnVersionConflict()
    }
}
```

## 모듈 제외

```gradle
dependencies{
    testCof1 (group: 'org.spockframework', name:'spock-core', version:'0.7-Groovy-2.0'){
        exclude module: 'Groovy-all'
    }
}
```

## 버전 강제 지정

```gradle
configurations.newConfDef{
    resolutionStrategy{
        failOnVersionConflict()
        // 버전 사용을 강제적으로 지정한다.
        force 'org.codehaus.Groovy:Groovy-all:2.3.1'
    }
}
```

// TODO 의존 관계 - 2. 환경 구성 정의 ..
