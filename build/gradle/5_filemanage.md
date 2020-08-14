# 파일 관리

## ant pattern

북마크 용도로 영어로 헤딩함

| 표현식 | 설명 |
| --- | --- |
| * | 임의 문자열로 길이에 제한 없이 패턴으로 처리 |
| ? | 임의 문자에 대하여 패턴으로 처리 |
| ** | 임의 계층의 디렉토리 |

## 파일 참조

### File 객체

```gradle
File refFile = file('src/main/java/actJava.java')
File refFile2 = file(new File('src/refLib.txt'))

task exeTask{
    doLast{
        println 'Java File Path : ' + refFile.absolutePath
        println 'Txt File Path : ' + refFile2.path
    }
}
```

### URL 객체

```gradle
File urlRef = null
URL url = new URL('file:/urlRef.html')
urlRef = file(url)

task exeTask{
    doLast{
        println 'URL Path : ' + urlRef.path
    }
}
```

### URI 객체

```gradle
File uriRef = null
URI uri = new URI('file:/uriRef.html')
uriRef = file(uri)
```

### 클로저

`File` 객체와 비슷하지만 중괄호를 사용한 것을 유의한다.

```gradle
File closer = file{'/ref.txt'}
```

### Callable 인터페이스

```gradle
import java.util.concurrent.Callable

File callRef = file(new Callable<String>() {
    String call(){
        '/refExe.java'
    }
})
```

### 파일 검증

- PathValidation.DIRECTORY : 디렉토리 경로가 올바른지 검증 (디렉토리 유무 확인)
- PathValidation.FILE :  파일 경로가 올바른지 검증 (파일 유무 확인)
- PathValidation.EXISTS : 파일 또는 디렉토리 존재 여부 확인
- PathValidation.NONE : 파일 검증하지 않음

```gradle
File checkFile = file('/index.html', PathValidation.FILE)
File checkDirectory = file('src', PathValidation.DIRECTORY)
```

## 다중 파일 참조

[FileCollection](https://docs.gradle.org/current/javadoc/org/gradle/api/file/FileCollection.html) 객체를 사용한다.

### 기본

```gradle
FileCollection fileCollection = files('index.txt', 'intro.txt')

println 'collection file[0] : ' + fileCollection[0].path
println 'collection file[1] : ' + fileCollection[1].path

fileCollection.each{
    println it.name + " : " + it.path
}
```

### FileCollection 다른 객체

```gradle
FileCollection fileCollection = files(new File('intro.txt'), new URL('file:/index.html'), new URI('file:/web.xml'))

// ...
```

리스트형과 배열형으로도 지정가능하다.

```gradle
// 리스트
List fileList = [new File('intro.txt'), new File('index.txt'), new File('log.txt')]
FilCollection fileCollection = files(fileList)

fileCollection.each{
    println 'fileList : ' + it.name + " : " + it.path
}

// 배열
FilCollection fileCollection2 = files(filesList as File[])

fileCollection2.each{
    println 'fileList : ' + it.name + " : " + it.path
}
```

### 재변환

```gradle
List fileList = fileCollection as List

Set fileSet = fileCollection as Set

File[] fileArray = fileCollection as File[]
```

### 연산자

```gradle
FileCollection fc1 = files('log.txt')
FileCollection fc2 = fc1 + files('backup.txt')

FileCollection fc3 = files('hello.txt', 'world.txt')
FileCollection fc4 = fc3 - files('world.txt')
```

### 필터링

```gradle
FileCollection fc1 = files('a.txt', 'b.xml', 'c.java', 'd.properties')

fc1.filter{collectionFile ->
    collectionFile.name.endsWith '.txt'
}
```

## 파일 트리

[FileTree](https://docs.gradle.org/current/javadoc/org/gradle/api/file/FileTree.html) 객체를 사용한다.

```gradle
FileTree fileTree = fileTree('settings')

fileTree.each{
    println 'File Name : ' + it.name + ', Path : ' + it.path
}
```

특정 패턴으로도 포함/제외 하여 `FileTree` 를 만들 수 있다.

[ant 패턴 표현식](##ant-pattern) 참고

```gradle
// ** 하위 디렉터리에 모든 포함
FileTree ft = fileTree('src'){
    include '**/*.java'
}

// 여러 문자에 대하여 (?) 지정할 경우 문자의 개수만큼 사용
ft = fileTree('src'){
    exclude '**/action????.java'
}
```

### FileTree 다른 객체

```gradle
FileTree fileMap = fileTree(dir:'src',include:'**/*.java',exclude:'**/action????.java')

FileTree filaMatch = fileTree('src')

FileTree fm1 = fileMatch.matching{
    include '**/*.java'
}

FileTree fm2 = fileMatch.matching{
    exclude '**/action????.java'
}
```

### visit, destroy

```gradle
FileTree fileVisit = fileTree('src')

fileVisit.visit(fileDetails ->
    println 'fileVisit File Name : ' + fileDetails.getNames()

    if ( fileDetails.isDirectory() ) {
        println ' =====[D]===== ' +
            fileDetails.getPath() +
            fileDetails.getSize()
    } else {
        println ' =====[F]===== ' +
            fileDetails.getPath() +
            fileDetails.getSize()
    }
)
```

## 파일 복사

[ant 패턴 표현식](##ant-pattern) 참고

```gradle
copy{
    from 'src/com/from/file' // 대상 파일 경로
    into 'src/com/to/file' // 목적지 경로

    include '**/*.java'
    exclude '**/*Dao.java'

    includeEmptyDirs = true // 빈 디렉토리 복사 여부
}
```

### rename

```gradle
copy{
    rename 'hello.java', 'world.java'
    rename '(.*)Test.java', 'Mock.java' // 특정 부분으로 명명된 내용을 변경

    from('src/com/before/file'){
        rename 'before.java', 'after.java'
    }
    into 'src/com/after/file'
}
```

// TODO 3분 59초