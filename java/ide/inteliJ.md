# 인텔리제이

[IntelliJ를 시작하시는 분들을 위한 IntelliJ 가이드](https://www.inflearn.com/course/intellij-guide/dashboard)

## Mac

`command, ⌘` ... alt 옆 위치

`option, ⌥` ... alt 위치

`shift, ⇧` ... 동일

| 단축키 | 설명 |
| --- | --- |
| `Command(⌘) + N` | 디렉토리, 패키지, 파일 생성하기 |
| `Ctrl + R` | 현재 포커스되어있는 Class Run, 단축키 외에도 좌측 버튼으로 실행가능. |
| `Ctrl + Shift + R` | configuration 기준 이전 포커스되었던 Class Run, 단축키 외에도 좌측 버튼으로 실행가능. |
| `Command(⌘) + D` | 라인 복사 |
| `Command(⌘) + 백스페이스` | 라인 삭제 |
| `Ctrl + Shift + J` | 라인 합치기 |
| `Ctrl + Shift + J` | 줄바꿈 합치기 |
| `Shift + Option(⌥) + ↑/↓` | 라인 이동 ... 문법과 관계 없는 이동 |
| `Shift + Command(⌘) + ↑/↓` | 구문 이동 ... 문법과 관계 있는 이동 (메소드) |
| `Option(⌥) + Shift + Command(⌘) ←/→` | html 엘리먼트의 속성 값의 순서를 변경 |
| `Command(⌘) + P` | 인자값 즉시보기 |
| `Option(⌥) + Space` | 코드 구현부 즉시보기 |
| `F1` | Doc 즉시보기 |
| `Option(⌥) + ←/→` | 단어별 이동 |
| `Shift + Option(⌥) + ←/→` | 선택하면서 이동 |
| `Fn + ←/→` | 라인 끝으로 이동 ... Home/End |
| `Shift + Command(⌘) + ←/→` | 선택하면서 라인 끝으로 이동 |
| `Fn + ↑/↓` | 페이지 업, 다운 |
| `Option(⌥) + ↑/↓` | 포커스 범위 한 단계씩 늘리기 선택 확장 |
| `Command(⌘) + [,]` | 포커스 뒤로/앞으로 가기 ... 작업한 포커스로 이동 (클래스 이동까지 가능) |
| `Option(⌥) + Option(⌥) + ↓` | 멀티 포커스 |
| `F2` | 오류 라인 자동 포커스 |
| `Command(⌘) + F` | 에디터 내 검색 |
| `Command(⌘) + R` | 에디터 내 변경 |
| `Command(⌘) + Shift + F` | 프로젝트 내 검색 |
| `Command(⌘) + Shift + R` | 프로젝트 내 변경 |
| `Shift + Command(⌘) + O` | 파일 검색 ... 패키지명/찾을 클래스 명 검색 |
| `Option(⌥) + Command(⌘) + O` | 메소드 명 검색 |
| `Shift + Command(⌘) + A` | Action 검색 ... 인텔리제이 실행 이벤트 |
| `Command(⌘) + E` | 최근 파일 나타내기 |
| `Command(⌘) + Shift + E` | 최근 수정한 파일 위치 나타내기 |
| `Ctrl + Shift + SPACE` | 스마트 검색 (arguments 에도 사용 가능) |
| `Ctrl + SPACE * 2` | 정적 메소드 자동 완성 |
| `Command(⌘) + N` | 클래스 에디터 내 getter / setter / hashCode / constructor 등 완성 |
| `Ctrl + I` | implements 의 구현 메소드 대상 객체를 @Override 를 선언하여 자동 생성 |
| `Command(⌘) + J` | 라이브 템플릿 목록 조회 |
| `Command(⌘) + Option(⌥) + V` | 동일한 내용을 뽑아내어 리팩토링 (선택가능) |
| `Command(⌘) + Option(⌥) + P` | 파라미터 추출 |
| `Command(⌘) + Option(⌥) + M` | 메소드 추출 |
| `F6` | 이너클래스 추출 |
| `Shift + F6` | 변수/메소드/클래스 명 일괄 변경 |
| `Shift + Command(⌘) + F6` | 타입을 일괄 변경해줌, 단 return ignore 후 수동으로 변경해야함. |
| `Ctrl + Option(⌥) + O` | 사용하지 않는 imports 제거, Actions optimize imports on 을 사용하여 커맨드 없이 사용도 가능 |
| `Ctrl + Option(⌥) + L` | 인텔리제이 기준 인덴트로 변경 |

- - -

| 단축키 | Git 설명 |
| --- | --- |
| `Command(⌘) + 9` | View On |
| `Ctrl + V` ... | Option Popup |
| `Ctrl + V => 4` ... | History |
| `Ctrl + V => 7` ... | Branch |
| `Command(⌘) + K` | Commit |
| `Command(⌘) + Shift + K` | Push |
| `Command(⌘) + Shift + A => git pull` .. git pull | Pull |

- - -

| 단축키 | Debug 명령어 명 | 설명 |
| --- | --- | --- |
| `Command(⌘) + Shift + D` | Debug Mode 실행 (현재 위치의 메소드) | |
| `Command(⌘) + D` | Debug Mode 실행 (이전에 실행한 메소드) | |
| `Command(⌘) + Option(⌥) + R` | RESUME | 다음 브레이크 포인트로 넘어감 |
| `F8` | STEP OVER | 한줄을 실행한다. 함수가 있어도 실행 후 다음으로 넘어간다. |
| `F7` | STEP INTO | 함수 내부로 들어간다. |
| `Shift + F8` | STEP OUT | 함수를 끝까지 실행시키고 호출시킨 곳으로 되돌아간다. |
| `Option(⌥) + F8` | Evaluate Expression | 브레이크된 상태에서 코드 사용하기 |
|  | Watch | 브레이크 이후의 코드 변경 확인 |

## 단축키

들어가기 앞서 플러그인 어시스턴트를 설치하여 운영체제 호환성 적용한다.

- `CTRL + SHIFT + A` : Find Actions
- `ALT + INSERT` : 디렉토리, 패키지, 파일 생성하기

## 파일 실행

- `CTRL + SHIFT + F10` : 현재 포커스되어있는 Class Run, 단축키 외에도 좌측 버튼으로 실행가능.
- `SHFT + F10` : configuration 기준 이전 포커스되었던 Class Run, 단축키 외에도 좌측 버튼으로 실행가능.

## 코드 템플릿

- `main` or `psvm` : main 메소드 생성
- `sout` : System.out.println("hello intelij");

## 라인 수정하기

- `CTRL + D` : 라인 복사
- `CTRL + Y` : 라인 삭제
- `CTRL + SHIFT + J` : 줄바꿈 합치기
- `SHIFT + ALT + ↑/↓` : 문법과 관계 없는 이동
- `SHIFT + CTRL + ↑/↓` : 문법과 관계 있는 이동 (메소드)
- `SHIFT + CTRL + ALT ←/→` : html 엘리먼트의 속성 값의 순서를 변경

## 즉시보기

- `CTRL + P` : 인자값 즉시보기
- `SHIFT + CTRL + I` : 코드 구현부 즉시보기
- `CTRL + Q` : Doc 즉시보기

## 에디터 포커스

- `CTRL + ←/→` : 이동
- `SHIFT + CTRL + ←/→` : 선택하면서 이동
- `HOME / END` : 라인 끝으로 이동
- `SHIFT + HOME / SHIFT + END` : 선택하면서 라인 끝으로 이동
- `CTRL + W` : 선택 확장
- `CTRL + ALT + ←/→` : 작업한 포커스로 이동 (클래스 이동까지 가능)
- `CTRL + CTRL + ↑/↓` : 다중 선택
- `F2` : 오류 라인 자동 포커스

## 검색하기

- `CTRL + F` : 에디터 내 검색
- `CTRL + R` : 에디터 내 변경
- `CTRL + SHIFT + F` : 프로젝트 내 검색
- `CTRL + SHIFT + R` : 프로젝트 내 변경

- `SHIFT + CTRL + N` : 패키지명/찾을 클래스 명 검색
- `SHIFT + CTRL + ALT + N` : 메소드 명 검색
- `SHIFT + CTRL + ALT + A` : 인텔리제이 실행 이벤트

- `CTRL + E` : 최근 파일 나타내기
- `SHIFT + CTRL + E` : 최근 수정한 파일 위치 나타내기

정규 표현식으로 검색또한 가능함.

## 자동완성

- `CTRL + SHIFT + SPACE` : 스마트 검색 (arguments 에도 사용 가능)
- `CTRL + SPACE * 2` : 정적 메소드 자동 완성
- `ALT + INSERT` : 클래스 에디터 내 getter / setter / hashCode / constructor 등 완성
- `CTRL + I` : implements 의 구현 메소드 대상 객체를 @Override 를 선언하여 자동 생성
- `CTRL + J` : 라이브 템플릿 목록 조회

## 리팩토링

- `CTRL + ALT + V` : 동일한 내용을 뽑아내어 리팩토링 (선택가능)
- `CTRL + ALT + P` : 파라미터 추출
- `CTRL + ALT + M` : 메소드 추출
- `F6` : 이너클래스 추출
- `SHIFT + F6` : 변수/메소드/클래스 명 일괄 변경
- `CTRL + SHIFT + F6` : 타입을 일괄 변경해줌, 단 return ignore 후 수동으로 변경해야함.
- `CTRL + ALT + O` : 사용하지 않는 imports 제거, Actions optimize imports on 을 사용하여 커맨드 없이 사용도 가능
- `CTRL + ALT + L` : 인텔리제이 기준 인덴트로 변경

## 디버깅

| 단축키 | 명령어 명 | 설명 |
| --- | --- | --- |
| `CTRL + F9` | RESUME | 다음 브레이크 포인트로 넘어감 |
| `F8` | STEP OVER | 한줄을 실행한다. 함수가 있어도 실행 후 다음으로 넘어간다. |
| `F7` | STEP INTO | 함수 내부로 들어간다. |
| `SHIFT + F8` | STEP OUT | 함수를 끝까지 실행시키고 호출시킨 곳으로 되돌아간다. |
| `ALT + F8` | Evaluate Expression | 브레이크된 상태에서 코드 사용하기 |
|  | Watch | 브레이크 이후의 코드 변경 확인 |

- `ALT + F9` : 커서 위치에서 디버깅을 실행한다.

## Git

| 단축 키| 명령어 명 |
| --- | --- |
| `ALT + 9` | View On |
| ``ALT + ` `` ... | Option Popup |
| ``ALT + ` `` ... | History |
| ``ALT + ` `` ... | Branch |
| `CTRL + K` | Commit |
| `CTRL + SHIFT + K` | Push |
| `CTRL + SHIFT + A` .. git pull | Pull |

## 인텔리제이 에러 수정

windows 기준 한글 기본설정이 ms949 설정으로 되어있음. fileEncoding 을 UTF-8 로 변경해주면 해결 가능.

```java
-Dfile.encoding=UTF-8
-Dconsole.encoding=UTF-8
```

혹은 inteliJ JVM 설정 idea64.exe.vmoptios 파일에 작성 후 다시 빌드한다.
