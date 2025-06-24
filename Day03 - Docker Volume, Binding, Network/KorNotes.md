
# Docker 볼륨, 바인딩, 네트워크 (2025/06/19)

#### **1. 도커 데이터 관리 소개**

  - 도커 컨테이너를 실행하면, 이는 이미지의 실행 중인 인스턴스가 됩니다. 컨테이너 내부에서 생성하는 모든 데이터는 컨테이너의 \*\*쓰기 가능한 레이어(writable layer)\*\*에 저장됩니다.
  - 이 데이터는 \*\*일시적(ephemeral)\*\*입니다. 즉, 컨테이너가 삭제되면 영구적으로 손실됩니다.
  - 이 문제를 해결하기 위해, 우리는 데이터를 컨테이너 외부에, 즉 **호스트 머신**에 저장하여 영속성을 부여해야 합니다.
  - 도커는 이를 위해 두 가지 주요 방법을 제공합니다: \*\*볼륨(Volumes)\*\*과 \*\*바인드 마운트(Bind Mounts)\*\*입니다.

#### **2. 일시적인 컨테이너 저장소 시연**

데이터 영속성이 왜 중요한지 이해하기 위해 기본 동작을 관찰해 보겠습니다.

  * **1단계: CentOS 컨테이너 실행**

    ```bash
    docker container run -it --name testos centos:8
    ```

  * **2단계: 컨테이너 내부에 파일 생성**

    ```bash
    touch testfile
    ls
    ```

    출력 결과, 컨테이너의 루트 디렉터리에 `testfile`이 생성된 것을 볼 수 있습니다.

  * **3단계: 컨테이너에서 나와서 상태 확인**

    ```bash
    exit
    docker container ls -a
    ```

    `testos` 컨테이너의 상태(STATUS)가 `Exited`로 표시됩니다. 데이터는 아직 존재하지만, 접근할 수 없는 상태입니다.

  * **4단계: 컨테이너 재시작 및 재진입**

    ```bash
    docker container start testos
    docker container exec -it testos /bin/bash
    ls
    ```

    `testfile`은 컨테이너가 삭제된 것이 아니라 멈췄다가 다시 시작되었기 때문에 여전히 존재합니다.

  * **5단계: 컨테이너 삭제 및 재생성**

    ```bash
    exit
    docker container rm -f testos
    docker container run -it --name testos centos:8
    ls
    ```

    이제 `testfile`은 **사라졌습니다**. 파일이 저장되었던 원래 컨테이너의 쓰기 가능한 레이어가 함께 삭제되었기 때문입니다. 이는 컨테이너 저장소의 일시적인 특성을 보여줍니다.

#### **3. 도커 볼륨: 영속성을 위한 권장 방식**

  - **볼륨**은 도커 컨테이너가 생성하고 사용하는 데이터를 영속화하는 데 가장 권장되는 방법입니다.
  - 볼륨은 도커에 의해 관리되며, 호스트 파일시스템의 특정 영역(리눅스의 경우通常 `/var/lib/docker/volumes/`)에 저장됩니다.
  - 볼륨을 컨테이너에 연결하면, 컨테이너의 파일시스템 내에서 일반 디렉터리나 파일처럼 보입니다.

**볼륨의 주요 장점:**

  * **도커에 의한 관리:** `docker volume create`, `docker volume ls`와 같은 도커 CLI 명령어를 사용하여 생성, 목록 확인, 삭제가 가능합니다.
  * **이식성과 안전성:** 바인드 마운트보다 백업이나 마이그레이션이 더 쉽고, 여러 컨테이너 간에 안전하게 공유할 수 있습니다.
  * **플랫폼 독립성:** 리눅스와 윈도우 컨테이너 모두에서 일관되게 작동합니다.

#### **4. 실습 예제 1: Nginx 웹 서버에서 볼륨 사용하기**

  * **1단계: 이름 있는 볼륨 생성**
    이름 있는 볼륨은 참조하고 관리하기가 더 쉽습니다.

    ```bash
    docker volume create testvol
    docker volume ls
    ```

    `testvol`이 생성된 것을 확인할 수 있습니다.

  * **2단계: 볼륨을 마운트하여 Nginx 컨테이너 실행**
    `-v` 또는 `--volume` 플래그를 사용하여 볼륨을 마운트합니다. 형식은 `볼륨이름:컨테이너디렉터리` 입니다.

    ```bash
    docker container run -d --name mynginx -v testvol:/usr/share/nginx/html -p 8080:80 nginx
    ```

      * `-v testvol:/usr/share/nginx/html`: `testvol` 볼륨을 Nginx 컨테이너 내부의 기본 웹 콘텐츠 디렉터리인 `/usr/share/nginx/html`에 마운트합니다.

  * **3단계: 컨테이너의 마운트 설정 확인**

    ```bash
    docker container inspect mynginx
    ```

    JSON 출력에서 `"Mounts"` 섹션을 찾으면, 호스트에서의 `Source` 경로를 포함한 볼륨의 상세 정보를 볼 수 있습니다.

    ```json
    "Mounts": [
        {
            "Type": "volume",
            "Name": "testvol",
            "Source": "/var/lib/docker/volumes/testvol/_data",
            "Destination": "/usr/share/nginx/html",
            ...
        }
    ]
    ```

  * **4. 호스트에서 볼륨 내용 확인**
    비어있는 볼륨으로 컨테이너를 시작하면, 도커는 해당 컨테이너 디렉터리의 내용으로 볼륨을 채웁니다.

    ```bash
    ls -l /var/lib/docker/volumes/testvol/_data
    ```

    Nginx의 기본 파일인 `index.html`과 `50x.html`이 볼륨 안으로 복사된 것을 볼 수 있습니다.

  * **5. 호스트에서 웹 콘텐츠 수정**
    이제 데이터가 호스트에 있으므로 직접 편집할 수 있습니다. 이 변경 사항은 컨테이너에 반영됩니다.

    ```bash
    echo '<h1>Test Volume Web Page</h1>' > /var/lib/docker/volumes/testvol/_data/index.html
    ```

  * **6. 브라우저에서 변경 사항 확인**
    웹 브라우저를 열고 `http://<your-docker-host-ip>:8080`으로 접속하면, 기본 Nginx 페이지 대신 "Test Volume Web Page"가 표시됩니다.

#### **5. 실습 예제 2: MySQL 데이터베이스에서 볼륨 사용하기**

데이터베이스 파일은 항상 영속적이어야 하므로 이는 매우 중요한 사용 사례입니다.

  * **1단계: MySQL 데이터용 볼륨 생성**

    ```bash
    docker volume create mysqlvol
    ```

  * **2단계: 볼륨과 함께 MySQL 컨테이너 실행**

    ```bash
    docker container run -d --name mydb -v mysqlvol:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password mysql:5.7
    ```

      * `-v mysqlvol:/var/lib/mysql`: `mysqlvol` 볼륨을 MySQL의 기본 데이터 디렉터리인 `/var/lib/mysql`에 마운트합니다.
      * `-e MYSQL_ROOT_PASSWORD=password`: 환경 변수를 사용하여 MySQL 서버의 root 비밀번호를 설정합니다.

  * **3. 호스트에서 데이터 영속성 확인 (`docker1`)**

    ```bash
    tree -L 2 /var/lib/docker/volumes/mysqlvol/_data
    ```

    MySQL이 호스트 머신의 `mysqlvol` 내부에 데이터베이스 파일들을 초기화한 것을 볼 수 있습니다.

  * **4. 다른 호스트(`docker2`)에서 MySQL 서버로 접속**
    이는 클라이언트가 데이터베이스 서버에 접속하는 상황을 시뮬레이션합니다.

    ```bash
    # docker2에서 MySQL 클라이언트 설치
    yum -y install mysql

    # docker1의 MySQL 서버로 접속
    mysql -h 192.168.2.10 -u root -p
    # 비밀번호 입력: password
    ```

  * **5. 데이터베이스와 데이터 생성을 통해 영속성 테스트**

    ```sql
    CREATE DATABASE testdb;
    USE testdb;
    CREATE TABLE docker (id INT, name VARCHAR(20));
    INSERT INTO docker VALUES(1, 'docker_user');
    SELECT * FROM docker;
    exit
    ```

  * **6. 정리 작업**
    컨테이너를 삭제하더라도 볼륨에 있는 데이터는 그대로 유지됩니다. 다음 실습을 위해 컨테이너와 볼륨을 삭제하여 환경을 정리합니다.

    ```bash
    # docker1에서
    docker container rm -f mydb
    docker volume rm mysqlvol
    ```

-----

### **Part 2: 바인드 마운트와 고급 볼륨 활용**

#### **1. 볼륨의 양방향 동기화**

컨테이너 내부의 변경 사항이 호스트 볼륨에 어떻게 반영되는지 보여줍니다.

  * **1단계: 볼륨과 Apache (`httpd`) 컨테이너 생성**

    ```bash
    docker volume create wwwvol
    docker container run -d --name myweb -v wwwvol:/usr/local/apache2/htdocs -p 8080:80 httpd
    ```

  * **2단계: 호스트에서 콘텐츠 수정 및 확인**

    ```bash
    echo '<h1>Volume Test Page</h1>' > /var/lib/docker/volumes/wwwvol/_data/index.html
    curl http://192.168.2.10:8080
    # 출력: <h1>Volume Test Page</h1>
    ```

  * **3단계: 컨테이너 내부에서 콘텐츠 수정**

    ```bash
    docker container exec -it myweb /bin/bash
    echo '<h1>Modified from Container</h1>' > /usr/local/apache2/htdocs/index.html
    exit
    ```

  * **4단계: 호스트에서 변경 사항 확인**
    컨테이너 내부에서 변경한 내용이 호스트의 볼륨에 즉시 반영됩니다.

    ```bash
    cat /var/lib/docker/volumes/wwwvol/_data/index.html
    # 출력: <h1>Modified from Container</h1>
    ```

#### **2. 바인드 마운트: 호스트-컨테이너 직접 매핑**

바인드 마운트는 호스트 머신의 파일이나 디렉터리를 컨테이너로 직접 매핑합니다. 볼륨과의 가장 큰 차이점은 도커가 데이터를 관리하지 않고, 단순히 경로를 직접 연결한다는 점입니다.

  * **1단계: 호스트에 디렉터리와 파일 생성**

    ```bash
    mkdir /www
    echo '<h1>Bind Mounts Test Page</h1>' > /www/index.html
    ```

  * **2단계: 바인드 마운트를 사용하여 컨테이너 실행**
    `-v` 플래그 문법은 동일하지만, 호스트의 절대 경로를 제공합니다.

    ```bash
    docker container run -d --name mynginx -v /www:/usr/share/nginx/html -p 8080:80 nginx
    ```

  * **3단계: 결과 확인**
    `http://192.168.2.10:8080`으로 접속하면, 호스트의 `/www` 디렉터리에 있는 `index.html` 파일이 직접 제공됩니다.

  * **중요 주의사항:** 바인드 마운트의 경우, 호스트 디렉터리(`/www`)가 비어 있으면, 컨테이너의 대상 디렉터리(`/usr/share/nginx/html`)에 원래 있던 콘텐츠를 **가리거나 덮어쓰게 됩니다.**

#### **3. 사용 사례: 공유 바인드 마운트를 이용한 웹 서버 확장**

바인드 마운트는 여러 컨테이너에 걸쳐 단일 콘텐츠 소스를 공유하는 데 매우 유용합니다.

  * **1단계: 동일한 호스트 디렉터리를 공유하는 여러 Nginx 컨테이너 생성**

    ```bash
    docker container run -d --name myweb1 -v /www:/usr/share/nginx/html -p 8001:80 nginx
    docker container run -d --name myweb2 -v /www:/usr/share/nginx/html -p 8002:80 nginx
    docker container run -d --name myweb3 -v /www:/usr/share/nginx/html -p 8003:80 nginx
    ```

  * **2단계: 확인**

      * `docker container ls -a`는 세 개의 컨테이너가 모두 실행 중임을 보여줍니다.
      * `8001`, `8002`, `8003` 포트로 접속하면 모두 동일한 "Bind Mounts Test Page"가 표시됩니다.
      * 호스트의 `/www/index.html` 파일을 수정하면, 세 컨테이너가 제공하는 콘텐츠가 동시에 변경됩니다.

-----

### **Part 3: 도커 네트워킹**

#### **1. 도커 네트워크 기초**

  * **기본 브리지 네트워크 (`docker0`):** 기본적으로 모든 컨테이너는 이 브리지 네트워크에 연결됩니다. 이는 컨테이너 간, 그리고 컨테이너와 호스트 간의 통신을 가능하게 합니다. 호스트는 보통 `172.17.0.1` 주소를 가진 게이트웨이 역할을 합니다.
  * **사용자 정의 네트워크:** 운영 환경에서는 애플리케이션을 위해 사용자 정의 브리지 네트워크를 생성하는 것이 좋습니다. 더 나은 격리 환경을 제공하고, 컨테이너 이름으로 서로를 찾을 수 있는 자동 DNS 확인 기능을 활성화합니다.

#### **2. 사용자 정의 브리지 네트워크 시연**

  * **1단계: 사용자 정의 브리지 네트워크 생성**

    ```bash
    docker network create -d bridge testnet
    docker network ls
    ```

  * **2단계: 서로 다른 네트워크에 컨테이너 실행**

    ```bash
    # 이 컨테이너는 기본 브리지 네트워크에 연결됨
    docker container run -itd --name myweb1 centos:8

    # 이 컨테이너는 사용자 정의 'testnet' 네트워크에 연결됨
    docker container run -itd --name myweb3 --network=testnet centos:8
    ```

  * **3단계: 네트워크 격리 확인**

      * `myweb1`과 `myweb3`를 `inspect` 해보면 서로 다른 서브넷(예: `172.17.x.x` 대 `172.18.x.x`)에 있음을 알 수 있습니다.
      * `myweb3` 컨테이너 내부에서는 `myweb1`의 IP 주소로 핑을 보낼 수 없습니다. 서로 다른 네트워크에 있기 때문입니다.

#### **3. 기타 네트워크 유형**

  * **호스트 네트워크 (`--network=host`)**

      * 컨테이너가 호스트의 네트워크 스택을 공유합니다. 네트워크 격리가 없습니다.
      * 컨테이너는 자신만의 IP 주소를 갖지 않고 호스트의 IP와 포트를 직접 사용합니다.
      * `-p`를 이용한 포트 매핑이 필요 없으며 무시됩니다.
      * **명령어:** `docker container run --rm --network=host centos:8 ip address` (호스트의 네트워크 인터페이스가 표시됨).

  * **네트워크 없음 (`--network=none`)**

      * 컨테이너의 모든 네트워킹을 완전히 비활성화합니다.
      * 컨테이너는 루프백 인터페이스(`lo`)만 갖게 됩니다.
      * 보안을 강화하거나 네트워크 접근이 필요 없는 작업을 수행하는 컨테이너에 유용합니다.
      * **명령어:** `docker container run --rm --network=none centos:8 ip address` (`lo` 인터페이스만 표시됨).

#### **4. 추가 네트워킹 옵션**

다음 플래그를 사용하여 컨테이너의 네트워크를 추가적으로 설정할 수 있습니다.

  * **`--dns=<ip>`**: 컨테이너를 위한 사용자 정의 DNS 서버를 설정합니다.
    ```bash
    docker container run -it --name testos --dns=8.8.8.8 centos:8
    ```
  * **`--mac-address="<address>"`**: 특정 MAC 주소를 할당합니다.
    ```bash
    docker container run -it --mac-address="00:00:00:11:11:11" centos
    ```
  * **`--add-host <host:ip>`**: 컨테이너의 `/etc/hosts` 파일에 사용자 정의 항목을 추가합니다.
    ```bash
    docker container run -it --add-host docker1:192.168.2.10 centos
    ```
  * **`--hostname <name>`**: 컨테이너의 호스트 이름을 설정합니다.
    ```bash
    docker container run -it --hostname webserver centos:8
    ```