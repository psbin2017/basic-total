# 자주 사용하는 명령어 정리

[explain shell](https://explainshell.com/explain?cmd=ps+-ef%7Cgrep+java)

## 설치 (apt-get)

```text
# 설치 상태 확인
sudo dpkg -l | grep ${파일명}

# 설치
sudo apt-get install ${파일명}
```

## 프로세스 확인

```text
ps -ef | grep java
ps -aux | grep java
```

## 서버 메모리 확인

```text
# 메모리
free

# 톰캣 메모리
ps -aux | grep java

# 자바 메모리
java -XX:+PrintFlagsFinal -version 2>&1 | grep -i -E 'heapsize|permsize|version'
```

## 시스템 체크

```text
# 시스템 메시지 확인
dmesg | tail

vmstat 1
  [r]: 동작중인 프로세스 숫자
  [free]: free memory
  [si, so]: 스왑 .. 0 이 아니면 메모리 부족
  [us, sy, id, wa, st]: user time, system time, idle time, wait I/O, stolen time

# CPU 간 상태 확인
mpstat -P ALL 1

# top 과 비슷함
pidstat 1

# ctrl + s 업데이트 중지, ctrl + q 업데이트 시작
top
```

## 커넥션 체크

```text
# 커넥션 수
netstat -antop | grep LISTEN | grep :80 wc -l

# 커넥션 유형
netstat -antop | grep LISTEN | grep :80
```

## Java

버전 정보 상세

> java -XshowSettings:all -version
