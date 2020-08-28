# 퍼블리싱

학습 내용 자료 출처 : [열혈강의] 그레이들 youtube [Chapter 08 그레이들 퍼블리싱](https://www.youtube.com/watch?v=vRJ42AV8V3w&list=PL7mmuO705dG2pdxCYCCJeAgOeuQN1seZz&index=24)

## 압축하기

예제 프로젝트 구조

```text
project ─ src ─ GradleHtml.html, Gradle.js, GradleXml.xml
    └ build.gradle
```

### zip

```gradle
task exeTask(type : Zip) {
    baseName = "GradleZip"

    // script 디렉토리 생성
    into ("script") {
        // src 대상 디렉토리 지정
        from ("src") {
            // 생성 제외 파일
            exclude "*.html", "*.xml"
        }
    }

    // 빈디렉토리 포함 여부 설정
    includeEmptyDirs = true
}
```

into block 없이도 압축이 가능하고 여러개의 into block 으로도 작성 가능하다.

```gradle
task exeTask(type : Zip) {
    baseName = "GradleZip"

    from ("src") {
        // 지정한 파일명이 변경됨
        rename 'Gradle*', 'Script'
    }
}
```

baseName 이 아닌 archivename 으로도 zip 파일의 이름을 정할 수 있다.

```gradle
task exeTask(type : Zip) {
    destinationDir = file("zip")
    archiveName = "Gradle.zip"
    appendix = "file"
    classifier = "Gradle"

    from "src"

    includeEmptyDirs = true
}
```

```gradle
task exeTask(type : Zip) {
    destinationDir = file("NewZip")
    baseName = "Gradle"
    appendix = "file"
    classifier = "script"
    version = "1_0"

    from "src"

    includeEmptyDirs = true

    println 'destinationDir:' + destinationDir
    println 'baseName:' + baseName
    println 'version:' + version
    println 'archivePath:' + archivePath
}
```

ZIP 파일은 압축 형식을 지정할 수 있다.

```gradle
entryCompression = ZipEntryCompression.DEFALTED
entryCompression = ZipEntryCompression.STORED // 압축을 하지 않고 zip 파일로만 만든다.
```

### tar

```gradle
task exeTask(type : tar) {
    // ... 동일

    // 압축 방식
    copression = Compression.BZIP2
    // copression = Compression.NONE // 압축을 하지 않고 tar 파일로만 만든다.
    // copression = Compression.GZIP
}
```

### jar

zip 파일과 동일하지만 메타 정보를 처리하는 manifest 가 추가된다.

```gradle
task exeTask(type : jar) {
    // ... 동일

    manifest {
        attributes("Built-Bo","Gradle", "Implementation-Version": project.version)
    }
}
```

빌드시 하위에 `META-INF` 디렉토리가 만들어지며 `MANIFEST.MF` 파일이 생성 된 것을 볼 수 있다.

war ear 파일 압축은 강의가 잘못 되있어서 일단 스킵

## 파일 퍼블리싱

> 모듈 정의 → 아티팩트 등록 → 메타 데이터 편집 → 퍼블리싱

파일 퍼블리싱은 플러그인을 활용하는데 메이븐을 이용한 파일 퍼블리싱과 아이비를 이용한 파일 퍼블리싱 방법이 있다.

## 메이븐 퍼블리싱 플러그인

`apply plugin: 'maven-publish'`

MavenPublication 게시 및 MavenArtifactRepository 저장소에서 기능을 수행한다.

퍼블리싱 플러그인을 사용하면서 소프트웨어 컴포넌트(Software Component) 개념이 생기는데 이는 퍼블리싱을 수행하기 위해 게시하고자 하는 파일을 관리하는 개념이다.

```gradle
// publishing 설정
publishing {
    publications {
        // MavenPublication 를 사용하도록 mavenJava 모듈 식별명 사용
        mavenJava(MavenPublication) {
            // java 를 소프트웨어 컴포넌트에 등록
            from components.java
        }
    }
}
```

내용 중 `mavenJava` 는 모듈 식별명이 된다.

소프트웨어 컴포넌트를 접근하는 예제는 다음과 같다.

```gradle
apply plugin: "java"
repositories {
    mavenCentral()
}
dependencies {
    runtime "org.apache.commons:commons-lang3:3.3.1"
}
task exeTask {
    doLast {
        // components.소프트웨어 컴포넌트명 형식으로 접근
        for (d in components.java.usages.dependencies) {
            println d
        }
    }
}
```

빌드 결과 선언한 의존성인 `commons-lang3` 대한 정보가 출력된다.

```gradle
apply plugin : "java"
apply plugin : "maven-publish"

task souceJar(type : Jar) {
    // 소스 세트 입력 파일 지정
    classifier 'sources'
    from sourceSets.main.allJava

    // 등록된 아티팩트 정보 출력
    for (a in components.java.usages.artifacts) {
        println "Software Component artifacts : " + a.file
    }
}

publishing {
    publications {
        mavenJava(MavenPublication) {
            // 퍼블리싱할 압축 파일 또는 파일 지정
            artifact sourceJar {
                classifier "sources"
            }
        }
    }
}
```

빌드 결과 아티팩트에 등록되어있는 라이브러리에 대한 정보가 출력된다.

```gradle
apply plugin: "java"
apply plugin: "maven-publish"

group = "src"
version = 0.1

repositories {
    mavenCentral()
}

dependencies {
    runtime "org.apache.commons:commons-lang3:3.3.1"
}

publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
        }
    }
}
```

메이븐 로컬로 퍼블리싱하는 방법이다.

`gradle publish${식별 모듈명}PublicationToMavenLocal`

식별 모듈명은 위 예제로는 `mavenJava` 가 된다.

## 아이비 퍼블리싱 플러그인

메이븐 퍼블리싱 플러그인과 거의 유사하다. 사용하는 플러그인을 아이비 퍼블리싱 플러그인으로 교체해주어 사용하면 된다.

```gradle
publishing {
    publications {
        // IvyPublication 을 사용하도록 ivyJava 모듈 식별명 사용
        ivyJava(IvyPublication) {
            // java 를 소프트웨어 컴포넌트로 등록
            from components.java
        }
    }
}
```

```gradle
apply plugin : "java"
apply plugin : "ivy-publish" // 바뀐 부분

task souceJar(type : Jar) {

    classifier 'sources'
    from sourceSets.main.allJava

    for (a in components.java.usages.artifacts) {
        println "Software Component artifacts : " + a.file
    }
}

publishing {
    publications {
        ivyJava(IvyPublication) { // 바뀐 부분
            artifact sourceJar {
                classifier "sources"
            }
        }
    }
}
```

이전 예제에서 2줄만 변경되면 메이븐에서 아이비로 플러그인이 변경된다.

명령어는 `gradle sourceJar` 이다.

다음은 아이비 저장소로 퍼블리싱 하는 부분이다.

```gradle
apply plugin : "java"
apply plugin : "ivy-publish"

group = 'org.gradle.sample'
version = '1.0'

publishing {
    publication {
        // IvyPublication 사용
        ivyJava(IvyPublication) {
            from components.js
        }
    }
    // ivy 저장소 지정
    repositories {
        ivy {
            url "$buildDir/repo"
        }
    }
}
```

이전 메이븐과 다른점은 저장소 선언 부분이다.

`repo` 디렉토리에 설정 정보와 퍼블리싱 할때 사용한 정보 등을 확인할 수 있고 그 외 다른 경로에서 빌드된 내용을 확인 가능하다.
