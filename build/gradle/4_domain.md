# 도메인

학습 내용 자료 출처 : [열혈강의] 그레이들 youtube [Chapter 04 그레이들의 도메인 객체](https://www.youtube.com/watch?v=hbZJPhceVg4&list=PL7mmuO705dG2pdxCYCCJeAgOeuQN1seZz&index=13)

## Project

프로젝트의 환경 구성, 의존 관계, 태스크 등의 내용을 제어하거나 참조한다.

자세한 내용은 및 설명은 [DSL Proejct](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html) 에 있다.

`build.gradle` 파일과 대응된다.

`Project` 객체의 라이프 사이클은 다음과 같다.

1. 빌드 수행을 위한 `Settings` 객체 생성
2. `settings.gradle` 스크립트 파일이 있을 경우 `Settings` 객체 비교
3. 구성된 `Settings` 객체를 이용하여 `Project` 객체의 계층 구조 생성
4. 멀티 프로젝트일 경우 부모 프로젝트로부터 `Project` 객체 생성 후 자식 프로젝트의 `Project` 객체 생성

### Project 객체 구조

- TaskContainer : `create()` 를 통해 클래스 컴파일, 단위 테스트 실행, war 압축 등 `getByName()` 등으로 태스크 정보를 가져올 수 있다.
- ConfigurationContainer : 프로젝트의 구성을 관리할 수 있다.
- DependencyHandler : 의존 관계를 관리할 수 있다.
- ArtifactHandler : 프로젝트의 결과물을 관리할 수 있다.
- RepositoryHandler : 프로젝트의 저장공간을 관리한다.
- Gradle : ?

### Project 객체 속성

속성을 통하여 프로젝트의 버전 빌드 상태 경로 등을 알 수 있다.

| 속성 | 설명 |
| --- | --- |
| version | 프로젝트나 결과물의 버전 (설정 없을 시 unspecified) |
| description | 프로젝트 설명 |
| name | 프로젝트의 이름 |
| state | 프로젝트 빌드 상태 (프로젝트 **상태**의 종류 : `NOT EXECUTED`, `EXECUTING`, `EXECUTED`, `FAILED`) |
| status | 프로젝트 결과물의 상태 (프로젝트 **결과물 상태**의 종류 : `NOT EXECUTED`, `EXECUTING`, `EXECUTED`, `FAILED`) |
| path | 프로젝트 경로 (경로 구분자 ':') |
| projectDir | 프로젝트 기준 디렉터리 |
| group | 프로젝트가 속한 그룹 (특별한 경우만 지정, 루트 프로젝트는 공백문자, 하위) |프로젝트는 루트 프로젝트나 부모 프로젝트로 지정) |
| buildDir | 프로젝트 빌드 디렉터리 (모든 결과물이 생성되는 디렉터리, 기본값 : ) |proejctDir/build) |
| plugins | `Project` 객체에 적용된 플러그인의 컨테이너 |
| projectDir | 기준 프로젝트 참조 |
| rootProject | 루트 프로젝트 참조 |
| parent | 기준 프로젝트의 상위(부모) 프로젝트 참조 |
| childParents | 기준 프로젝트의 하위(자식) 프로젝트 참조 (Map 형식으로 저장) |
| allProjects | 기준 프로젝트에 포함된 모든 프로젝트 참조 (Set 형식으로 저장) |
| subprojects | 기준 프로젝트 이하의 모든 프로젝트 참조 (Set 형식으로 저장) |

### Project 객체 API

| API | 설명 |
| --- | --- |
| project(path) | 지정된 경로의 프로젝트에 대하여 설정(상대 경로로 지정 가능) |
| project(path, configureClosure) | 지정된 경로의 프로젝트에 대하여 클로저를  |사용하여 프로젝트 구성(상대 경로로 지정 가능)
| absoluteProjectPath(path) | 절대 경로를 변환하여 프로젝트 확인 |
| apply(closure) | 플러그인이나 스크립트를 적용, **자주 사용됨** |
| configure(object, configureClosure) | 클로저를 통하여 설정된 상태를 이용하여  |객체를 구성
| subproject(action) | 해당 프로젝트의 하위 프로젝트를 설정 |
| task(name) | 주어진 이름으로 태스크를 생성하고 프로젝트에 추가 |
| afterEvaluate(action) | 프로젝트가 평가된 직후 추가 |
| beforeEvaluate(action) | 프로젝트가 평가되기 바로 직전 추가 |

### Project 객체 실습

```text
./eample/4_1_project (Project)
    └ build.gradle
    └ settings.gradle
```

`settings.gradle` 있는 내용을 (gradle. 지우고) `build.gradle` 에 옮겨서 실행하면 정상적으로 실행은 된다.

그러나 `beforeEvaluate` 가 동작하지 않는 것을 볼 수 있는데 이는 프로젝트 평가를 시행하지 않고 바로 태스크를 실행하였기 때문이다.

## Task

`Action` 객체를 생성하여 구성

내부적으로 `execute()` API 를 호출하여 실행

예외 처리를 통한 빌드 수행 제어

자세한 내용은 및 설명은 [DSL Task](https://docs.gradle.org/current/dsl/org.gradle.api.Task.html) 에 있다.

### Task 객체 속성

| 속성 | 설명 |
| --- | --- |
| name | 태스크 이름 (프로젝트 내의 고유 명) |
| description | 태스크 설명 |
| group | 태스크가 속한 그룹 (태스크 목록을 사용자에게 표시 할 때 관련 태스크를 그룹화 하는 데 사용) |
| path | 태스크의 경로 (태스크의 정규화된 이름, : 로 구분 ) |
| action | 태스크에 의해 실행되는 순서 지정 (action 속성으로 Action 객체에 지정 후 지정된 순서대로 실행) |
| dependsOn | 태스크의 의존 관계 지정 표시 |
| enabled | 태스크 실행 여부 |
| finalizedBy | 해당 태스크 실행 후 최종 수행할 태스크 지정 |
| inputs | 태스크 입력 정보 |
| mustRunAfter | 태스크 실행 순서 제어 (강제) |
| shouldRunAfter | 태스크 실행 순서 제어 (비강제) |
| state | 태스크 실행 상태 |

### Task 객체 API

| API | 설명 |
| --- | --- |
| doFirst(action) | 해당 태스크의 실행을 위한 Action 객체 리스트의 처음 부분에 위치하여 태스크가 실행될 때 먼저 처리 |
| doLast(action) | 해당 태스크의 실행을 위한 Action 객체 리스트의 마지막 부분에 위치하여 실행 |
| leftShift(action) | << 연산자를 사용하여 leftShift() 호출, Action 객체 리스트의 마지막 부분에 추가 |
| Property(propertyName) | 태스크에 지정된 속성 값을 출력 |
| setProperty(name, value) | 태스크의 속성 설정 |
| hasProperty(propertyName) | 지정된 속성 값을 가졌는지 확인 |
| dependsOn(paths) | 태스크의 의존 관계 |
| onlyIf(onlyIfSpec) | 지정된 조건을 만족할 경우 태스크 진행 |

## Gradle 객체

### Gradle 객체 속성

| 속성 | 설명 |
| --- | --- |
| gradle | Gradle 객체 반환, 초기화 스크립트에서 Gradle 속성과 메소드에 명시적으로 접근할 때 사용 |
| gradleHomeDir | Gradle 홈 디렉토리 |
| gradleUserHomeDir | 사용자의 홈 디렉토리 |
| gradleVersion | 현재 사용 중인 Gradle 버전 |
| includeBuilds | 포함된 빌드 관련 정보 |
| plugins | Gradle 에 사용된 플러그인 컨테이너 |
| rootProejct | Root 프로젝트 위치 |
| startParameter | 빌드 수행 관련 파라미터 정보 |
| taskGraph | 태스크 그래프 정보 표시 |

### Gradle 객체 API

| API | 설명 |
| --- | --- |
| addBuildListener(buildListener) | BuildListener 추가, 빌드 실행 중 발생되는 이벤트를 전달한다 |
| addListener(listener) | 지정된 리스너 추가, 지정된 인터페이스 구현 가능 |
| removeListener(listener) | 빌드로부터 지정된 리스너 제거 |
| addProjectEvaluationListener(listener) | 프로젝트 평가 리스너, 프로젝트 평가 시 이벤트 전달 |
| afterProject() | 프로젝트 평가 후 바로 호출할 작업 추가 |
| apply() | 플러그인 또는 스크립트 적용 |
| beforeProject() | 프로젝트 평가되기 바로 전 호출할 작업 추가 |
| buildFinished() | 빌드가 완료될 떄 호출할 작업 추가 |
| projectEvaluated() | 모든 프로젝트를 평가된 후 호출할 작업 추가 |
| projectLoaded() | 프로젝트가 빌드를 위해 로드된 후 호출할 작업 추가 |
| settingsEvaluated() | 프로젝트 관련 빌드 설정이 로드 및 평가될 때 호출할 작업 추가 |
| userLogger() | Logger 를 사용할 수 있도록 제공 |

### Gradle 실습

`addBuildListener` 실습 `build.gradle` 작성이 아닌 `init.gradle` 로 작성한다.

`TaskExecutionListener` 인터페이스를 구현하여 빌드 리스너에 추가한다.

실행 명령어는 이전 실습과 다르게 `init.gradle` 을 사용하기 때문에 `gradle -I init.gradle help` 로 실행한다.

```txt
> Task :help
before execute : help
Caching disabled for task ':help' because:
  Build cache is disabled
Task ':help' is not up-to-date because:
  Task has not declared any outputs despite executing actions.

Welcome to Gradle 6.5.1.

To run a build, run gradle <task> ...

To see a list of available tasks, run gradle tasks

To see a list of command-line options, run gradle --help

To see more detail about a task, run gradle help --task <task>

For troubleshooting, visit https://help.gradle.org
after execute : help
:help (Thread[Execution worker for ':',5,main]) completed. Took 0.007 secs.
```

`help` 키워드는 그레이들의 내장 키워드이다. 대신하여 `build.gradle` 에 작성한 exeTask 를 명시하여 실행할 수 있다.

## Settings 객체

설정 스크립트로서 멀티 프로젝트 설정 등 프로젝트 빌드 수행 전 Settings 객체를 생성한다.

### Settings 객체 속성

| 속성 | 설명 |
| --- | --- |
| gradle | 현재 빌드를 위한 Gradle 객체 |
| plugins | Settings 객체에 적용한 플러그인 컨테이너 |
| rootDir | 빌드의 루트 디렉토리 (루트 디렉토리의 프로젝트 디렉토리) |
| rootProject | 빌드의 루트 프로젝트 |
| settings | Settings 객체 참조 (자기 자신) |
| settingDir | 빌드의 설정 디렉토리 (설정 디렉토리는 설정 파일을 포함한 디렉토리) |
| startParameter | Gradle 이 빌드 수행시점에 사용된 명령어 인수 |

### Settings 객체 API

| API | 설명 |
| --- | --- |
| findProject() | 지정된 디렉터리 혹은 파일 경로와 일치하는 Project 객체 반환 (일치하는 디렉터리 또는 파일 경로가 없을 시 null 반환) |
| Project() | 위와 동일하나 없을 시 에러 발생 |
| include() | 계층형 멀티 프로젝트 추가 시 사용 |
| includeFlat() | 단층형 멀티 프로젝트 추가 시 사용 |

## 기타 객체

| 객체 | 설명 |
| --- | --- |
| Script | Gradle 의 특정 메소드를 추가하기 위해 사용. Gradle 의 스크립트에서 Script 객체의 인터페이스를 구현하여 메소드와 속성을 사용 |
| SourceSet | 자바 소스 및 자원에 대해 그룹을 형성하여 사용 |
| ExtensionAware | Runtime 에 다른 객체와 함께 확장하여 사용 extensions 이라는 확장 속성을 저장하는 컨테이너 이용 |
| ExtraPropertiesExtension | ext 로 정의된 확장 속성 으로 has(), set(), get() 3가지 API 로 특정 키 값 제어 |
