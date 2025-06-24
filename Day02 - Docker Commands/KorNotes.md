# **Docker 기본 명령어 및 개념 마스터 (2025/06/17)**

이 문서는 Docker의 핵심 명령어와 개념을 구조적으로 정리하여, 컨테이너 기술을 처음 접하는 분들도 쉽게 이해하고 활용할 수 있도록 돕는 것을 목표로 합니다.

## **Chapter 1: Docker 환경 및 아키텍처**

### **1. Docker 서비스 관리 (리눅스)**

Docker 엔진을 사용하기 전, 호스트 시스템에서 서비스(데몬)가 정상적으로 실행되고 있는지 확인해야 합니다.

| 명령어 | 설명 |
| :--- | :--- |
| `sudo systemctl status docker` | Docker 서비스의 현재 실행 상태(활성/비활성)를 확인합니다. |
| `sudo systemctl start docker` | Docker 서비스를 시작합니다. |
| `sudo systemctl stop docker` | Docker 서비스를 중지합니다. |
| `sudo systemctl enable --now docker` | 시스템 부팅 시 Docker가 자동 시작되도록 설정하고, 즉시 서비스를 시작합니다. |

### **2. Docker 아키텍처와 정보 확인**

Docker는 클라이언트와 서버(데몬) 구조로 동작합니다. 사용자가 `docker` 명령어를 입력하면 클라이언트가 이를 받아 Docker 데몬에 전달하여 실제 작업을 수행합니다.

**Docker 아키텍처 다이어그램**

```
+-----------------+      +-------------------------------------------+      +-----------------------+
|                 |      |               Docker 호스트                 |      |                       |
|  사용자 (CLI)    |<---->| Docker 클라이언트 <---> Docker 데몬 (서버) |<---->|  레지스트리 (Docker Hub)  |
|                 |      |                                           |      |                       |
+-----------------+      +-------------------|-----------------------+      +-----------------------+
                               |
                               +------> [ 이미지 (Images) ] ------> [ 컨테이너 (Containers) ]
```

**주요 정보 확인 명령어**

| 명령어 | 설명 | 주요 확인 항목 |
| :--- | :--- | :--- |
| `docker version` | Docker 클라이언트와 서버(Engine)의 상세 버전을 표시합니다. | `Client Version`, `Server Version`, `Go version` |
| `docker system info` | Docker 시스템의 전반적인 정보를 상세하게 보여줍니다. | 컨테이너/이미지 수, 스토리지 드라이버, **Docker Root Dir** |

-----

## **Chapter 2: Docker 이미지 관리**

이미지는 컨테이너를 생성하기 위한 읽기 전용 템플릿입니다. 필요한 애플리케이션과 라이브러리를 포함하고 있습니다.

### **주요 이미지 관리 명령어**

| 명령어 | 설명 |
| :--- | :--- |
| `docker search [이미지명]` | Docker Hub에서 이미지를 검색합니다. `--filter "is-official=true"`로 공식 이미지만 필터링할 수 있습니다. |
| `docker image pull [이미지명]:[태그]` | Docker Hub에서 이미지를 로컬로 다운로드합니다. 태그 생략 시 `latest`를 사용합니다. |
| `docker image ls` | 로컬에 저장된 모든 이미지 목록을 표시합니다. |
| `docker image inspect [이미지 ID 또는 이름]` | 이미지의 상세 메타데이터(레이어 정보, 환경변수, 기본 명령어 등)를 JSON 형식으로 보여줍니다. |
| `docker image rm [이미지 ID 또는 이름]` | 로컬에서 이미지를 삭제합니다. 컨테이너가 사용 중이면 삭제되지 않습니다. (`-f`로 강제 삭제 가능) |
| `docker image prune` | 사용되지 않는 모든 이미지(댕글링 이미지)를 한 번에 정리합니다. |

-----

## **Chapter 3: Docker 컨테이너 생명주기 관리**

컨테이너는 이미지의 실행 가능한 인스턴스입니다. 컨테이너의 생명주기(생성, 실행, 중지, 삭제)를 관리하는 방법을 알아봅니다.

### **1. 컨테이너 실행 (`docker container run`)**

`run` 명령어는 `create`와 `start`를 합친 것으로, 컨테이너를 생성하고 즉시 시작합니다.

| 옵션 | 설명 |
| :--- | :--- |
| `-d` `--detach` | 컨테이너를 백그라운드(분리 모드)에서 실행합니다. |
| `-p` `--publish` | **`호스트_포트:컨테이너_포트`** 형식으로 포트를 매핑(연결)합니다. |
| `--name` | 컨테이너에 고유한 이름을 부여합니다. 지정하지 않으면 Docker가 임의의 이름을 생성합니다. |
| `-it` | `-i` (interactive)와 `-t` (tty)를 합친 옵션. 컨테이너와 상호작용하는 셸(터미널) 환경을 제공합니다. |
| `--rm` | 컨테이너가 종료될 때 자동으로 삭제됩니다. 일회성 테스트에 유용합니다. |

**예제: Nginx 웹 서버 실행**

```bash
# Nginx 컨테이너를 'webserver'라는 이름으로, 80번 포트를 연결하여 백그라운드로 실행
docker container run -d --name webserver -p 80:80 nginx
```

### **2. 컨테이너 관리 및 조작**

| 명령어 | 설명 |
| :--- | :--- |
| `docker container ls -a` | 모든 컨테이너 목록을 표시합니다. (`-a`가 없으면 실행 중인 것만) |
| `docker container stop [이름/ID]` | 실행 중인 컨테이너를 정상적으로 중지합니다. (`SIGTERM`) |
| `docker container start [이름/ID]` | 중지된 컨테이너를 다시 시작합니다. |
| `docker container restart [이름/ID]` | 컨테이너를 재시작합니다. |
| `docker container rm [이름/ID]` | 중지된 컨테이너를 삭제합니다. (`-f`로 강제 삭제 가능) |
| `docker container prune` | 중지된 모든 컨테이너를 한 번에 정리합니다. |

### **3. 실행 중인 컨테이너와 상호작용**

| 명령어 | 설명 |
| :--- | :--- |
| `docker container exec -it [이름/ID] [명령]` | **(권장)** 실행 중인 컨테이너에 새로운 프로세스를 실행하여 접속합니다. (예: `/bin/bash`) |
| `docker container attach [이름/ID]` | 컨테이너의 주 프로세스(PID 1)에 직접 연결합니다. 여기서 `exit`하면 컨테이너가 중지될 수 있습니다. |
| `docker container logs [이름/ID]` | 컨테이너의 표준 출력 로그를 확인합니다. (`-f` 옵션으로 실시간 확인 가능) |
| `docker container stats [이름/ID]` | 컨테이너의 CPU, 메모리 등 리소스 사용량을 실시간으로 모니터링합니다. |
| `docker container cp [경로1] [경로2]` | 호스트와 컨테이너 간에 파일을 복사합니다. |

-----

### **Chapter 4: Docker Hub 연동과 이미지 배포**

로컬에서 변경하거나 생성한 이미지를 Docker Hub에 올려 다른 사람과 공유하거나 다른 환경에 배포할 수 있습니다.

**이미지 배포 프로세스**

```
1. 로컬에서 이미지 변경 (Dockerfile 또는 commit)
      |
      V
2. 이미지 태그 지정 (docker image tag)
   <로컬 이미지> -> <사용자명>/<저장소명>:<태그>
      |
      V
3. Docker Hub 로그인 (docker login)
      |
      V
4. 이미지 푸시 (docker image push)
   로컬 -> Docker Hub
```

#### **주요 배포 관련 명령어**

| 명령어 | 설명 |
| :--- | :--- |
| `docker login -u [사용자명]` | Docker Hub에 로그인하여 인증 정보를 저장합니다. |
| `docker logout` | Docker Hub에서 로그아웃합니다. |
| `docker image tag [원본] [새이름:태그]` | 이미지에 새로운 이름과 태그를 부여합니다. 푸시를 위해 `사용자명/저장소명` 형식으로 태그를 지정합니다. |
| `docker image push [사용자명/저장소명:태그]` | 로컬 이미지를 Docker Hub 원격 저장소에 업로드합니다. |
