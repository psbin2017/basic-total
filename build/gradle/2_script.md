# 스크립트 파일

학습 내용 자료 출처 : [열혈강의] 그레이들 youtube [Chapter 02 그레이들과 빌드](https://www.youtube.com/watch?v=_yLS067GEqY&list=PL7mmuO705dG2pdxCYCCJeAgOeuQN1seZz&index=4)

스크립트 파일 구조 설명에 앞서 사용하고 있는 언어인 그루비에 대해 알아본다.

## 그루비

JVM 에 의해 동작하며 자바와 거의 비슷하다.

```groovy
// def 형 사용

// 자바와 같은 형 지정
String id = 'gradle'
// 형 지정 생략
def id = 'gradle'

// 문자열 사용
// 변수/프로퍼티는 큰따옴표
def id = "ID : ${project.id}"
// {} 생략 가능
String id = "ID : $id"

// 클로저 사용
task hello<< {
 println "Hello Gradle"
}

def id = { closer -> println "id , $closer" }

// call()
id.call('gradle')

// 일반 메소드 호출방식 사용
id('gradle')

// 괄호 생략 사용
id 'gradle'

// 리스트와 맵
def id = [ 'gradle', 'groovy' ]
id[1] = 'script'

def id = [ 'a':'gradle', 'b':'gradle' ]
assert id['a'] == 'gradle'
```

## 구성

스크립트는 크게 처리문과 스크립트 블록으로 이루어져 있다.

## 주요 스크립트 블록

| 스크립트 블록 | 설명 |
| --- | --- |
| repositories | 저장소 설정 |
| dependencies | 의존 관계 설정 |
| buildscript | 빌드 스크립트 클래스 패스 설정 |
| initscript | 초기화 스크립트 설정 |
| configurations | 환경 구성 설정 |
| allproject | 서브 프로젝트 포함 전체 프로젝트 설정 |
| subproject | 서브 프로젝트 설정 |
| artifacts | 빌드 결과에 대한 설정 |

## 변수

| 변수 | 설명 | 사용범위 |
| --- | --- | --- |
| 지역 변수 | 선언된 부분에서 영향력 있는 변수 | Gradle 에 모든 스크립트 |
| 시스템 프로퍼티 | 시스템 관련 정보 변수 | Gradle 에 모든 스크립트 |
| 확장 프로퍼티 | 도메인 객체 확장 용도로 사용하는 변수 | Gradle 에 모든 스크립트 |
| 프로젝트 프로퍼티 | 프로젝트에서 사용하는 변수 | 빌드 스크립트 |

```groovy
// gradle.properties
msg=Hi, Gradle!

// build.gradle
task hello{
    println msg
}
```

## 명령 인수로 사용

| 우선순위 | 지정 방법 |
| --- | --- |
| 1 | 프로젝트 디렉토리의 gradle.properties 지정 |
| 2 | 홈 디렉터리의 gradle.properties 지정 |
| 3 | 환경 변수 지정 |
| 4 | 명령어 옵션 -D 옵션 |
| 5 | 명령어 옵션 -P 옵션 |

동일한 속성명으로 지정된 경우에는 우선순위가 늦은 것으로 지정된다.
