# Docker 기본 명령어 (2025/06/17)

이 문서는 세 개의 강의 동영상에서 다룬 주요 Docker 명령어와 개념들을 구조적이고 읽기 쉽게 정리한 참고 자료입니다.

## 파트 1: Docker 기초 및 초기 이미지 작업

### 1. Docker 서비스 관리

* **Docker 서비스 시작 및 활성화:**
    ```bash
    sudo systemctl enable --now docker.service
    ```
    * `enable`: Docker가 시스템 부팅 시 자동으로 시작되도록 설정합니다.
    * `--now`: 현재 세션에서 Docker 서비스를 즉시 시작합니다.
* **Docker 서비스 상태 확인:**
    ```bash
    sudo systemctl status docker.service
    ```
    * Docker 데몬이 실행 중인지 확인합니다.

### 2. Docker 정보 및 아키텍처

* **Docker 버전 확인:**
    ```bash
    docker version
    ```
    * 클라이언트와 서버(데몬) 버전 정보를 표시합니다.
    * **개념:** Docker는 클라이언트-서버 아키텍처를 사용합니다. 클라이언트가 데몬에 명령을 보내면 데몬이 이를 실행합니다.
* **Docker 시스템 정보 확인:**
    ```bash
    docker system info
    ```
    * Docker 환경에 대한 상세 정보를 제공합니다:
        * 컨테이너 수 (실행 중, 일시정지, 중지됨).
        * 이미지 수.
        * 스토리지 드라이버 (예: `overlay2`).
        * Docker 루트 디렉토리 (기본값: `/var/lib/docker`).
        * 호스트의 운영체제 및 커널 버전.
        * 하드웨어 리소스 (CPU, 총 메모리).
    * **개념:** 컨테이너는 VM과 달리 전체 OS 설치가 필요 없이 호스트의 커널을 공유하기 때문에 가볍습니다.

### 3. 기본 컨테이너 실행 및 관리

* **`hello-world` 컨테이너 실행:**
    ```bash
    docker container run hello-world
    ```
    * 이미지가 로컬에 없으면 Docker Hub에서 가져옵니다.
    * `hello-world` 프로그램을 실행하고 출력을 인쇄한 후 종료하는 컨테이너를 생성하고 실행합니다.
* **모든 컨테이너 목록 (종료된 것 포함):**
    ```bash
    docker container ls -a
    ```
    * `-a` 또는 `--all`: 상태에 관계없이 모든 컨테이너를 표시합니다.
* **종료된 컨테이너 제거:**
    ```bash
    docker container rm <컨테이너명_또는_ID>
    ```
    * 지정된 컨테이너를 제거합니다.
    * 강제로 하지 않는 한 실행 중인 컨테이너는 제거할 수 없습니다 (파트 3 참조).

### 4. 기본 Docker 이미지 작업

* **Docker Hub에서 이미지 검색:**
    ```bash
    docker search <이미지명>
    ```
    * Docker Hub에서 이미지를 검색합니다.
    * 출력에는 `NAME`, `DESCRIPTION`, `STARS`, `OFFICIAL` 상태가 포함됩니다.
* **Docker 이미지 가져오기:**
    ```bash
    docker image pull <이미지명>[:태그]
    ```
    * 이미지를 로컬 머신에 다운로드합니다. 태그를 지정하지 않으면 `latest`가 기본값입니다.
* **로컬 Docker 이미지 목록:**
    ```bash
    docker image ls
    ```
    * 로컬 Docker 호스트에 저장된 모든 이미지를 표시합니다.
* **Docker 이미지 제거:**
    ```bash
    docker image rm <이미지명_또는_ID>
    ```
    * 로컬 저장소에서 지정된 이미지를 삭제합니다.
    * 컨테이너(종료된 것 포함)가 이미지를 사용하고 있으면 제거할 수 없습니다. 먼저 관련 컨테이너를 제거해야 합니다.

### 5. 웹 서버 실행 (Nginx 예제)

* **포트 매핑과 함께 Nginx를 분리 모드로 실행:**
    ```bash
    docker container run -d --name webserver -p 80:80 nginx
    ```
    * `-d` 또는 `--detach`: 컨테이너를 백그라운드에서 실행합니다.
    * `--name webserver`: 컨테이너에 사람이 읽기 쉬운 이름을 할당합니다.
    * `-p 80:80`: 호스트 포트 `80`을 컨테이너 포트 `80`에 매핑합니다. (구문: `호스트_포트:컨테이너_포트`)
    * `nginx`: 사용할 Docker 이미지입니다.
* **실행 중인 컨테이너 확인:**
    ```bash
    docker container ls
    ```
    * 현재 실행 중인 컨테이너만 표시합니다.
* **컨테이너 리소스 사용량 모니터링:**
    ```bash
    docker container stats webserver
    ```
    * 지정된 컨테이너의 실시간 CPU, 메모리, 네트워크 I/O, 디스크 I/O를 표시합니다. `Ctrl+C`를 눌러 종료합니다.
* **실행 중인 컨테이너 내에서 명령 실행:**
    ```bash
    docker container exec -it <컨테이너명_또는_ID> /bin/bash
    ```
    * `-it`: `-i` (인터랙티브)와 `-t` (의사 TTY)를 결합하여 인터랙티브 셸을 제공합니다.
    * `/bin/bash`: 실행할 명령 (Bash 셸을 시작).
    * **예제:** `docker container exec -it webserver /bin/bash`
    * 컨테이너 내에서 `cat /etc/os-release`나 도구 설치 (`apt update`, `apt install htop`) 같은 명령들이 상호작용을 보여줍니다. `exit`로 컨테이너 셸을 나갑니다.
* **실행 중인 컨테이너 중지:**
    ```bash
    docker container stop webserver
    ```
    * 컨테이너를 정상적으로 중지합니다. 상태가 `Exited (0)`로 변경됩니다.
* **중지된 컨테이너 시작:**
    ```bash
    docker container start webserver
    ```
    * 이전에 중지된 컨테이너를 시작합니다. 상태가 `Up`으로 변경됩니다.
* **호스트와 컨테이너 간 파일 복사:**
    ```bash
    docker container cp <소스_경로> <대상_경로>
    ```
    * **호스트에서 컨테이너로:** `docker container cp /host/path/file.html webserver:/usr/share/nginx/html/`
    * **컨테이너에서 호스트로:** `docker container cp webserver:/container/path/file.html /host/path/`

---

## 파트 2: 고급 이미지 관리 및 Docker Hub 연동

### 1. 향상된 Docker 검색

* **검색 결과 필터링:**
    ```bash
    docker search nginx --filter "stars=100"
    ```
    * 지정된 기준으로 필터링합니다 (예: 100개 이상의 별점을 받은 이미지).
* **검색 결과 제한:**
    ```bash
    docker search nginx --limit 5
    ```
    * 표시되는 결과 수를 제한합니다.
* **전체 설명 표시:**
    ```bash
    docker search nginx --no-trunc
    ```
* **Go 템플릿으로 출력 형식 지정:**
    ```bash
    docker search nginx --format "{{.Name}}\t{{.Description}}\t{{.Stars}}\t{{.Official}}"
    ```
    * 출력의 사용자 정의 형식을 허용합니다.
* **개념: 사용 중단된 이미지:**
    * `DEPRECATED`로 표시된 이미지는 더 이상 공식적으로 유지관리되지 않으며 보안 업데이트가 부족할 수 있습니다. 새로운 배포에서는 피하는 것이 좋습니다. 예를 들어, `centos`는 `centos:8`과 같은 특정 버전 태그가 필요할 수 있습니다.

### 2. Docker 이미지 검사

* **상세한 이미지 정보 가져오기:**
    ```bash
    docker image inspect <이미지명>[:태그]
    ```
    * 다음을 포함한 포괄적인 JSON 데이터를 출력합니다:
        * `Id`, `RepoTags`, `RepoDigests`.
        * `Created`, `DockerVersion`, `Author`.
        * `Config`: `ExposedPorts`, `Env` (환경 변수), `Cmd` (기본 명령), `Entrypoint` (항상 실행되는 명령).
        * `Os`, `Architecture`.
        * `GraphDriver` (스토리지 드라이버 세부사항).
* **`--format` (Go 템플릿)을 사용하여 특정 정보 추출:**
    ```bash
    docker image inspect --format "{{.Id}}" ubuntu
    docker image inspect --format "{{.Config.Cmd}}" nginx
    docker image inspect --format "{{.Config.ExposedPorts}}" nginx
    ```

### 3. Docker 컨테이너 생명주기 명령

* **컨테이너 생성 (시작하지 않음):**
    ```bash
    docker container create --name webserver nginx
    ```
    * 컨테이너 파일시스템을 생성하지만 `Created` 상태로 유지합니다.
* **명령 직접 실행 (컨테이너 실행 후 종료):**
    ```bash
    docker container run --name test1 centos:8 /bin/cal
    ```
    * 지정된 명령을 실행한 후 컨테이너가 즉시 중지됩니다.
* **인터랙티브 셸로 실행:**
    ```bash
    docker container run -it --name test2 centos:8 /bin/bash
    ```
    * 컨테이너 내에서 인터랙티브 터미널 세션을 제공합니다.
* **분리(백그라운드) 모드로 실행:**
    ```bash
    docker container run -d --name test3 centos:8 /bin/bash
    ```
    * 컨테이너를 백그라운드에서 실행하여 터미널 제어를 반환합니다.
* **컨테이너 강제 제거 (중지 후 제거):**
    ```bash
    docker container rm -f test2
    ```
    * 일반적으로 `stop` 후 `rm`하는 것이 좋지만, `-f`는 빠른 정리에 사용할 수 있습니다.

### 4. Docker Hub 인증

* **Docker Hub 로그인:**
    ```bash
    docker login -u <Docker_Hub_사용자명>
    ```
    * Docker Hub 비밀번호를 입력하라고 메시지가 표시됩니다. 자격 증명을 로컬에 저장합니다.
* **Docker Hub 로그아웃:**
    ```bash
    docker logout
    ```
    * 로컬 로그인 자격 증명을 제거합니다.
* **Docker Hub 풀/푸시 제한:**
    * 익명: 100회 풀/6시간 (IP별).
    * 인증됨: 200회 풀/6시간.
    * 유료 계층: 무제한.

### 5. 이미지 태그 지정 및 푸시

* **Docker Hub용 로컬 이미지 태그 지정:**
    ```bash
    docker image tag <로컬_이미지명>[:로컬_태그] <Docker_Hub_사용자명>/<저장소명>[:태그]
    ```
    * Docker Hub 호환 태그로 로컬 이미지의 별칭을 생성합니다.
    * **예제:** `docker image tag nginx kim10322/webserver:1.0`
* **Docker Hub에 이미지 푸시:**
    ```bash
    docker image push <Docker_Hub_사용자명>/<저장소명>[:태그]
    ```
    * 태그가 지정된 이미지를 Docker Hub 저장소에 업로드합니다. 사전에 `docker login`이 필요합니다.

---

## 파트 3: 고급 컨테이너 작업 및 정리

### 1. `docker container run` 옵션 심화

* **`-d` (분리됨):** 백그라운드에서 실행합니다. 출력이 콘솔로 스트리밍되지 않습니다.
* **`-i` (인터랙티브):** 첨부되지 않아도 STDIN을 열어둡니다.
* **`-t` (TTY):** 의사 TTY(터미널)를 할당합니다. 인터랙티브 셸을 위해 `-i`와 함께 자주 사용됩니다.
* **`-it` (인터랙티브 + TTY):** 인터랙티브 세션(예: `bash` 셸)을 위한 일반적인 조합입니다. 셸 내 명령의 출력이 표시됩니다.
    * `-it`로 실행된 프로세스(예: `/bin/bash`)가 종료되면 컨테이너가 중지됩니다.
* **`--rm` (종료 시 제거):** 컨테이너가 종료될 때 자동으로 제거합니다. 일회성 작업에 유용합니다.
    * **예제:** `docker container run --rm --name test4 centos:8 /bin/cal`
        * 이 명령은 `test4` 컨테이너에서 `/bin/cal`을 실행합니다. `/bin/cal`이 완료되면 `test4` 컨테이너가 자동으로 제거됩니다. `docker container ls -a`로 찾을 수 없습니다.
* **백그라운드 실행 비교:**
    * `-d` 없이 명령 실행 (포어그라운드): 명령을 실행하는 터미널이 차단되고 출력이 스트리밍됩니다. 터미널을 닫거나 `Ctrl+C`를 누르면 프로세스(일반적으로 컨테이너)가 중지됩니다.
    * `-d`로 명령 실행 (분리됨): 명령이 백그라운드에서 실행됩니다. 터미널이 즉시 반환되고 출력이 표시되지 않습니다. `docker container logs <이름>`을 사용하여 출력을 봅니다.

### 2. 컨테이너 로깅 및 모니터링

* **컨테이너 로그 보기:**
    ```bash
    docker container logs <컨테이너명_또는_ID>
    ```
    * 포어그라운드 또는 분리 모드로 실행되었는지에 관계없이 컨테이너에서 로그를 검색합니다. 백그라운드 서비스 문제 해결에 유용합니다.

### 3. 컨테이너 생명주기 관리

* **`docker container stop <이름_또는_ID>`:** `SIGTERM` 신호(정상 종료)를 보냅니다. 타임아웃(기본 10초) 후 프로세스가 중지되지 않으면 `SIGKILL`을 보냅니다.
    * `Exited (0)` 상태의 컨테이너는 성공적으로 종료되었음을 의미합니다.
* **`docker container start <이름_또는_ID>`:** 중지된 컨테이너를 시작합니다.
* **`docker container restart <이름_또는_ID>`:** 컨테이너를 중지한 후 시작합니다.
* **`docker container kill <이름_또는_ID>`:** 컨테이너의 주 프로세스에 `SIGKILL` 신호(즉시, 강제 종료)를 보냅니다. 정상 종료가 없습니다.
    * `SIGKILL`(신호 9)은 `SIGTERM`(신호 15)과 다릅니다. `SIGKILL`은 프로세스가 잡거나 무시할 수 없습니다.

### 4. 효율적인 컨테이너 및 이미지 정리

* **모든 종료된 컨테이너 제거:**
    ```bash
    docker container rm -f $(docker container ls -aq)
    ```
    * `docker container ls -aq`: 모든 컨테이너 ID를 나열합니다 (`-a`는 모두, `-q`는 조용히/ID만).
    * `$()`: 명령 치환으로, `docker container ls -aq`의 출력이 `docker container rm -f`의 인수가 됩니다.
    * `-f`: 강제 제거 (실행 중인 컨테이너를 제거하기 전에 중지).
* **모든 댕글링 이미지 제거 (컨테이너와 연관되지 않은 이미지):**
    ```bash
    docker image rm -f $(docker image ls -aq)
    ```
    * `docker image ls -aq`: 모든 이미지 ID를 나열합니다.
    * **참고:** 이 명령은 강력하며 *실행 중인* 컨테이너에서 적극적으로 사용되지 않는 모든 이미지를 제거합니다. 현재 실행 중인 컨테이너와 연결되지 않은 유지하고 싶은 이미지가 있다면 더 선별적인 접근이 필요할 수 있습니다.

### 5. 편의를 위한 별칭 (선택사항이지만 권장)

자주 사용하는 명령을 단순화하려면 셸의 설정 파일(예: Bash용 `~/.bashrc`)에 별칭을 생성할 수 있습니다.

* **`~/.bashrc` 편집:**
    ```bash
    vi ~/.bashrc # 또는 nano ~/.bashrc
    ```
* **별칭 추가 (예제):**
    ```bash
    alias crm='docker container rm -f $(docker container ls -aq)'
    alias irm='docker image rm -f $(docker image ls -aq)'
    alias cls='docker container ls -a'
    ```
* **변경사항 적용 (`~/.bashrc` 저장 후):**
    ```bash
    source ~/.bashrc
    ```
    * 이는 `.bashrc` 파일을 다시 로드하여 현재 세션에서 새 별칭을 활성화합니다.
