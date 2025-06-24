
# Docker Volume, Binding, Network (2025/06/19)

### **Part 1: Docker Data Management - Volumes and Persistence**

#### **1. Introduction to Docker Data Management**

  - When a Docker container runs, it's an instance of an image. Any data created inside the container is stored on the container's **writable layer**.
  - This data is **ephemeral**, meaning it is permanently lost when the container is deleted.
  - To solve this, we need to persist data by storing it outside the container, on the **host machine**.
  - Docker offers two primary methods for this: **Volumes** and **Bind Mounts**.

#### **2. Demonstrating Ephemeral Container Storage**

To understand why data persistence is crucial, let's observe the default behavior.

  * **Step 1: Run a CentOS container.**

    ```bash
    docker container run -it --name testos centos:8
    ```

  * **Step 2: Create a file inside the container.**

    ```bash
    touch testfile
    ls 
    ```

    Output shows `testfile` is created in the container's root directory.

  * **Step 3: Exit and inspect the container.**

    ```bash
    exit
    docker container ls -a
    ```

    The `testos` container will have an `Exited` status. The data is still there, just not accessible.

  * **Step 4: Restart and re-enter the container.**

    ```bash
    docker container start testos
    docker container exec -it testos /bin/bash
    ls
    ```

    The `testfile` is still present because the container was only stopped and restarted.

  * **Step 5: Delete and recreate the container.**

    ```bash
    exit
    docker container rm -f testos
    docker container run -it --name testos centos:8
    ls
    ```

    The `testfile` is now **gone**. This is because the original container, along with its writable layer where the file was stored, was deleted. This demonstrates the ephemeral nature of container storage.

#### **3. Docker Volumes: The Preferred Method for Persistence**

  - **Volumes** are the recommended mechanism for persisting data in Docker.
  - They are managed by Docker and stored in a specific directory on the host filesystem (typically `/var/lib/docker/volumes/` on Linux systems).
  - When you attach a volume to a container, it appears as a standard directory or file within the container's filesystem.

**Key Advantages of Volumes:**

  * **Managed by Docker:** Can be created, listed, and removed using Docker CLI commands (e.g., `docker volume create`).
  * **Portability and Safety:** Easier to back up, migrate, and safely share among multiple containers.
  * **Platform Independent:** Works consistently across both Linux and Windows containers.

#### **4. Practical Example 1: Using Volumes with an Nginx Web Server**

  * **Step 1: Create a named volume.**
    A named volume is easier to reference and manage.

    ```bash
    docker volume create testvol
    docker volume ls
    ```

    This confirms that `testvol` has been created.

  * **Step 2: Run an Nginx container with the volume mounted.**
    The `-v` or `--volume` flag is used to mount a volume. The format is `volume-name:container-directory`.

    ```bash
    docker container run -d --name mynginx -v testvol:/usr/share/nginx/html -p 8080:80 nginx
    ```

      * `-v testvol:/usr/share/nginx/html`: Mounts our `testvol` volume to `/usr/share/nginx/html`, which is the default web content directory inside the Nginx container.

  * **Step 3: Inspect the container's mount configuration.**

    ```bash
    docker container inspect mynginx
    ```

    In the JSON output, find the `"Mounts"` section. It will show the details of the volume, including its `Source` path on the host.

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

  * **Step 4: Check the volume's content on the host.**
    When a container is started with an empty volume, Docker populates the volume with the content from the specified container directory.

    ```bash
    ls -l /var/lib/docker/volumes/testvol/_data
    ```

    You will see `index.html` and `50x.html`, the default Nginx files, have been copied into the volume.

  * **Step 5: Modify the web content from the host.**
    Now that the data is on the host, we can edit it directly. This change will be reflected in the container.

    ```bash
    echo '<h1>Test Volume Web Page</h1>' > /var/lib/docker/volumes/testvol/_data/index.html
    ```

  * **Step 6: Verify the change in the browser.**
    Open a web browser and navigate to `http://<your-docker-host-ip>:8080`. You will see "Test Volume Web Page" instead of the default Nginx page.

#### **5. Practical Example 2: Using Volumes with a MySQL Database**

This is a critical use case, as database files must always be persistent.

  * **Step 1: Create a volume for MySQL data.**

    ```bash
    docker volume create mysqlvol
    ```

  * **Step 2: Run the MySQL container with the volume.**

    ```bash
    docker container run -d --name mydb -v mysqlvol:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password mysql:5.7
    ```

      * `-v mysqlvol:/var/lib/mysql`: Mounts `mysqlvol` to `/var/lib/mysql`, which is MySQL's default data directory.
      * `-e MYSQL_ROOT_PASSWORD=password`: Sets the database root password using an environment variable.

  * **Step 3: Verify data persistence on the host.**

    ```bash
    tree -L 2 /var/lib/docker/volumes/mysqlvol/_data
    ```

    This shows that MySQL has initialized its database files inside the `mysqlvol` on the host machine.

  * **Step 4: Connect to the MySQL server from another host (`docker2`).**
    This simulates a client connecting to the database server.

    ```bash
    # On docker2, install the MySQL client
    yum -y install mysql

    # Connect to the MySQL server on docker1
    mysql -h 192.168.2.10 -u root -p
    # Enter password: password
    ```

  * **Step 5: Create a database and data to test persistence.**

    ```sql
    CREATE DATABASE testdb;
    USE testdb;
    CREATE TABLE docker (id INT, name VARCHAR(20));
    INSERT INTO docker VALUES(1, 'docker_user');
    SELECT * FROM docker;
    exit
    ```

  * **Step 6: Clean up and verify persistence.**
    Even if we remove the container, the data in the volume will remain.

    ```bash
    # On docker1
    docker container rm -f mydb
    docker volume rm mysqlvol
    ```

    This ensures our environment is clean for the next steps.

-----

### **Part 2: Bind Mounts and Advanced Volume Usage**

#### **1. Bidirectional Sync with Volumes**

This section demonstrates how changes in the container are reflected back to the host volume.

  * **Step 1: Create a volume and an Apache (`httpd`) container.**

    ```bash
    docker volume create wwwvol
    docker container run -d --name myweb -v wwwvol:/usr/local/apache2/htdocs -p 8080:80 httpd
    ```

  * **Step 2: Modify content from the host and verify.**

    ```bash
    echo '<h1>Volume Test Page</h1>' > /var/lib/docker/volumes/wwwvol/_data/index.html
    curl http://192.168.2.10:8080 
    # Output: <h1>Volume Test Page</h1>
    ```

  * **Step 3: Modify content from inside the container.**

    ```bash
    docker container exec -it myweb /bin/bash
    echo '<h1>Modified from Container</h1>' > /usr/local/apache2/htdocs/index.html
    exit
    ```

  * **Step 4: Verify the change on the host.**
    The change made inside the container is instantly visible on the host's volume.

    ```bash
    cat /var/lib/docker/volumes/wwwvol/_data/index.html
    # Output: <h1>Modified from Container</h1>
    ```

#### **2. Bind Mounts: Direct Host-to-Container Mapping**

Bind mounts map a file or directory on the host to a file or directory in the container. The key difference from volumes is that Docker does not manage the data; it's just a direct path mapping.

  * **Step 1: Create a directory and file on the host.**

    ```bash
    mkdir /www
    echo '<h1>Bind Mounts Test Page</h1>' > /www/index.html
    ```

  * **Step 2: Run a container using a bind mount.**
    The `-v` flag syntax is the same, but you provide an absolute path from the host.

    ```bash
    docker container run -d --name mynginx -v /www:/usr/share/nginx/html -p 8080:80 nginx
    ```

  * **Step 3: Verify the result.**
    Access `http://192.168.2.10:8080`. It will serve the `index.html` file directly from the host's `/www` directory.

  * **Important Caveat:** With bind mounts, if the host directory is empty, it **hides/overwrites** any content that might have existed in the container's target directory (e.g., `/usr/share/nginx/html`).

#### **3. Use Case: Scaling Web Servers with a Shared Bind Mount**

Bind mounts are excellent for sharing a single source of content across multiple containers.

  * **Step 1: Create multiple Nginx containers pointing to the same host directory.**
    ```bash
    docker container run -d --name myweb1 -v /www:/usr/share/nginx/html -p 8001:80 nginx
    docker container run -d --name myweb2 -v /www:/usr/share/nginx/html -p 8002:80 nginx
    docker container run -d --name myweb3 -v /www:/usr/share/nginx/html -p 8003:80 nginx
    ```
  * **Step 2: Verify.**
    Accessing the web servers on ports `8001`, `8002`, and `8003` will all serve the exact same content from the `/www` directory on the host. Modifying the `index.html` file in `/www` updates all three sites instantly.

-----

### **Part 3: Docker Networking**

#### **1. Docker Network Fundamentals**

  * **Default Bridge Network (`docker0`):** By default, all containers are attached to this bridge network. It allows containers to communicate with each other and with the host. The host usually acts as the gateway at `172.17.0.1`.
  * **User-Defined Networks:** It is best practice to create custom bridge networks for applications. They provide better isolation and enable automatic DNS resolution between containers by their names.

#### **2. User-Defined Bridge Network Demonstration**

  * **Step 1: Create a custom bridge network.**

    ```bash
    docker network create -d bridge testnet
    docker network ls
    ```

  * **Step 2: Run containers on different networks.**

    ```bash
    # This container is on the default bridge network
    docker container run -itd --name myweb1 centos:8

    # This container is on our custom 'testnet' network
    docker container run -itd --name myweb3 --network=testnet centos:8
    ```

  * **Step 3: Observe network isolation.**

      * Inspect `myweb1` and `myweb3` to see they are on different subnets (e.g., `172.17.x.x` vs. `172.18.x.x`).
      * From inside `myweb3`, you **cannot** ping `myweb1` by its IP address because they are on separate networks.

#### **3. Other Network Types**

  * **Host Network (`--network=host`)**

      * The container shares the host's network stack. There is no network isolation.
      * The container uses the host's IP address and does not get its own.
      * Port mapping with `-p` is not needed and is ignored.
      * **Command:** `docker container run --rm --network=host centos:8 ip address` (This will show the host's network interfaces).

  * **None Network (`--network=none`)**

      * Completely isolates the container from all networking.
      * The container only gets a loopback interface (`lo`).
      * Useful for security or for tasks that don't require network access.
      * **Command:** `docker container run --rm --network=none centos:8 ip address` (This will only show the `lo` interface).

#### **4. Additional Networking Options**

You can further customize a container's network configuration with these flags:

  * **`--dns=<ip>`**: Sets a custom DNS server for the container.
    ```bash
    docker container run -it --name testos --dns=8.8.8.8 centos:8
    ```
  * **`--mac-address="<address>"`**: Assigns a specific MAC address.
    ```bash
    docker container run -it --mac-address="00:00:00:11:11:11" centos
    ```
  * **`--add-host <host:ip>`**: Adds a custom entry to the container's `/etc/hosts` file.
    ```bash
    docker container run -it --add-host docker1:192.168.2.10 centos
    ```
  * **`--hostname <name>`**: Sets the container's hostname.
    ```bash
    docker container run -it --hostname webserver centos:8
    ```