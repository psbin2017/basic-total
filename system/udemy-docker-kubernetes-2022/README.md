# 정리

## Tips

- 매번 수동으로 컨테이너를 제거하기 귀찮으니 `docker run` 시에 `--rm` 옵션을 주자.

## 컨테이너/이미지 명령어

### `docker build`

> 특정 상태의 파일을 이미지로 빌드한다.

`-t` : 빌드시 이름과 태그명을 기입할 수 있다.

### `docker ps`

> 실행중인 컨테이너를 확인한다.

`-a` : 실행했던 컨테이너도 확인한다.

### `docker run`

> 이미지를 새 컨테이너로 실행한다.

- `-p ${노출 포트} : ${연결 포트}` : 네트워크 모드로 포트를 노출시킨다.
- `-i` : 인터렉티브 모드, 표준 입력이 열린 상태로 유지한다. 컨테이너에 무언가를 입력할 수 있다.
- `-t` : 터미널을 생성한다.
- `-it` : ⭐⭐⭐ 터미널을 생성하고 컨테이너에 입력 가능한 인터렉티브 모드로 접속한다.
- `--rm` : ⭐⭐⭐ 중지되면 자동으로 컨테이너를 자동으로 제거한다.
- `-v ${named voulme} ${directory}` 옵션은 컨테이너에 볼륨을 추가할 수 있다. 용도에 따라 볼륨이 읽기 권한만 부여하고자 하는경우 `:ro` 를 선언하여 사용할 수 있다.
- `-e ${key} ${value}` 는 환경 변수를 추가할 수 있다.

기본적으로 attached 모드로 실행된다. 이미지 아이디로도 실행 가능하고 빌드 이름:태그 등으로도 실행 가능하다.

### `docker start`

> 컨테이너 아이디 또는 이름으로 이전에 실행했었던 컨테이너를 실행한다.

기본적으로 detached (백그라운드 실행) 모드로 실행된다.

- `-a` : attached 모드로 실행한다.
- `-i` : 인터렉티브 모드

### `docker stop`

> 실행되고 있는 컨테이너를 중지한다.

### `docker attach`

> 실행 중인 컨테이너에 접근한다. start 상태로 되는 것과 동일

### `docker logs`

> 컨테이너에서 출력된 과거의 로그를 확인할 수 있다.

- `-f` : 연결된 상태로 계속 확인할 수 있다.

### `docker rm`

> 중지된 상태의 컨테이너를 삭제한다.

`${id1} ${id2}` 처럼 나열하여 삭제할 수 있다.

### `docker images`

> 가지고 있는 모든 이미지를 나타낸다.

### `docker rmi`

> 이미지를 삭제한다. 단, 삭제할 이미지로 사용되고 있는 컨테이너가 모두 중지되어 있어야 한다. 중지되어 있지 않다면 컨테이너를 중지해야 한다.

`${id1} ${id2}` 처럼 나열하여 삭제할 수 있다.

### `docker image`

> 이미지와 관련된 명령어를 수행한다.

- `prune` : 사용되지 않는 모든 이미지를 제거한다.
- `inspect` : 이미지에 대한 정보가 출력된다.

### `docker cp`

> 로컬에 있는 이미지가 아닌 파일을 특정 컨테이너로 복사해서 넣는다. 반대로 컨테이너에 있는 파일을 로컬로 이동 가능하다.

`docker cp dummy/. boring_vaughan:/test` : 로컬에 있는 dummy 디렉토리 하위 파일을 컨테이너 test 경로로 복사한다.

`docker cp boring_vaughan:/test dummy` :  컨테이너의 test 디렉토리를 로컬에 있는 dummy 디렉토리로 복사한다.

### `docker push`

> 도커 허브에 이미지를 공유한다.

도커는 이미지를 푸시할 때 의존하는 파일이 이미 도커 허브에 있다면 추가 정보만 푸시하여 이미지를 경량화 한다.

### `docker pull`

> 도커 허브에서 **최신 업데이트** 이미지를 가져온다.

`docker run` 은 로컬에 히스토리를 검색해 없으면 이미지를 자동으로 풀링하지만, 로컬에 있다면 최신 버전인지 확인하지 않는다. 따라서 항상 최신 버전을 가져와야 한다면 `docker pull` 을 사용하고 최신 버전을 명시적으로 가져온 후 `docker run` 으로 실행해야한다.

### `docker login`

> 도커 허브에 로그인한다.

### `docker logout`

> 도커 허브에 로그아웃한다.

### `docker volume`

> 볼륨에 접근할 수 있다.

- `prune` : 사용하지 않는 볼륨을 삭제한다.
- `inspect` : 볼륨의 상태를 확인한다.
- `delete` : 볼륨을 삭제한다.
- `create` : 볼륨을 생성한다.

바인드 마운트 볼륨의 경우 로컬 절대 경로를 사용할 수 도 있고 아래처럼 바로가기를 사용할 수 있다.

- macOS / Linux: `-v $(pwd):/app`
- Windows: `-v "%cd%":/app`

- - -

## 네트워크 명령어

### `docker network`

- `create ${name}` : 네트워크를 생성한다
- `ls` : 네트워크를 조회한다.
- `--driver OPTIONS` : 드라이버 옵션을 설정한다.

네트워크에 참여하기위해서는 컨테이너 실행시 `--network ${name}` 으로 네트워크를 부여한다.
