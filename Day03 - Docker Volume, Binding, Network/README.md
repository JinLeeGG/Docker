
# Docker Volume, Binding, Network (2025/06/19)

### **Chapter 1: Docker Data Persistence**

By default, containers are ephemeral. To permanently save data generated within a container, Docker provides two primary solutions: **Volumes** and **Bind Mounts**.

#### **1. Data Management: Volumes vs. Bind Mounts**

This table provides a high-level comparison of the two main data persistence methods.

| Feature | Docker Volumes | Bind Mounts |
| :--- | :--- | :--- |
| **Data Location** | A dedicated area on the host, managed by Docker (`/var/lib/docker/volumes`). | Any file or directory on the host machine. |
| **Management** | Managed by **Docker** via the Docker CLI (`docker volume ...`). | Managed by the **user/host OS** via standard OS commands. |
| **Primary Use Case** | Persisting data for stateful applications like databases, config files. | Syncing source code for a development environment. |
| **Initialization** | If the volume is empty, the container's data is copied **into** the volume. | The host directory's content **overwrites** (or obscures) the container's content. |
| **Portability** | **High**. Managed by name, independent of the host's directory structure. | **Low**. Dependent on a specific path on the host. |

##### **How Volumes Work (Diagram)**

```
      [ Docker Container ]                           [ Docker Host ]
┌─────────────────────────┐                   ┌──────────────────────────────────┐
│                         │                   │                                  │
│  /usr/share/nginx/html  │ <-----(Mounts)----->│ /var/lib/docker/volumes/myvol/_data │
│      (Container Path)     │      (Data Sync)      │      (Docker Managed Area)       │
│                         │                   │                                  │
└─────────────────────────┘                   └──────────────────────────────────┘
```

##### **How Bind Mounts Work (Diagram)**

```
      [ Docker Container ]                           [ Docker Host ]
┌─────────────────────────┐                   ┌──────────────────────────────────┐
│                         │                   │                                  │
│      /usr/src/app       │ <----(Direct Map)---> │  /home/dev/my-project/src      │
│      (Container Path)     │                   │       (Any Path on Host)         │
│                         │                   │                                  │
└─────────────────────────┘                   └──────────────────────────────────┘
```

#### **2. Practical Examples**

##### **Example 1: Using a Volume with Nginx**

1.  **Create a Volume**:

    ```bash
    docker volume create nginx-vol
    ```

2.  **Run a Container and Attach the Volume**:

    ```bash
    docker run -d --name myweb -v nginx-vol:/usr/share/nginx/html -p 8080:80 nginx
    ```

3.  **Modify Data on the Host**: The change is immediately reflected in the container.

    ```bash
    echo '<h1>Hello from Docker Volume!</h1>' > /var/lib/docker/volumes/nginx-vol/_data/index.html
    ```

##### **Example 2: Using a Bind Mount for Development**

1.  **Create a Project Directory on the Host**:

    ```bash
    mkdir -p /source/my-app
    echo '<h1>Live Reload with Bind Mounts</h1>' > /source/my-app/index.html
    ```

2.  **Run a Container with a Bind Mount**:

    ```bash
    docker container run -d --name dev-server -v /source/my-app:/usr/share/nginx/html -p 8081:80 nginx
    ```

    Now, any changes you make to files in `/source/my-app` on your host are instantly available in the container, which is ideal for development.

-----

### **Chapter 2: Docker Networking**

#### **1. Docker Network Drivers**

| Driver | Description | Use Case |
| :--- | :--- | :--- |
| **`bridge`** | **(Default)** Creates a private internal network. Docker manages IP addressing, routing, and a gateway for containers. | The most common driver for standalone containers that need to communicate. |
| **`host`** | The container shares the host's network stack. There is no network isolation. | When network performance is critical and network isolation is not needed. |
| **`none`** | The container has no network interfaces besides loopback (`lo`). It's completely isolated. | For batch jobs or security-sensitive tasks that require no network access. |

##### **Default Bridge Network (`docker0`) Architecture**

```
+------------------+     +--------------------------+     +-------------------+
|   Container A    |     |      Docker Host         |     |   External Network  |
|  (172.17.0.2)    |<--->|   docker0 (172.17.0.1)   |<--->|  ens33 (Host IP)    |
+------------------+     | (Virtual Bridge/Gateway) |     |  (e.g., Internet)   |
|   Container B    |     +--------------------------+     +-------------------+
|  (172.17.0.3)    |
+------------------+
```

#### **2. User-Defined Bridge Networks**

For production, it is best practice to create custom networks to isolate applications and enable automatic service discovery using container names.

##### **Key Network Commands**

| Command | Description |
| :--- | :--- |
| `docker network create <name>` | Creates a new bridge network. |
| `docker network ls` | Lists all available networks. |
| `docker network inspect <name>` | Shows detailed information about a network. |
| `docker network connect <net> <container>`| Connects a running container to an additional network. |
| `docker network rm <name>` | Removes a network (must not have containers attached). |

##### **Practical Example: Connecting a Web App to a Database**

1.  **Create a custom network**:

    ```bash
    docker network create web-db-net
    ```

2.  **Run containers on the same network**:

    ```bash
    # Run the Database container
    docker run -d --name database --network web-db-net -e MYSQL_ROOT_PASSWORD=pass mysql:5.7

    # Run the Web App container
    docker run -d --name webapp --network web-db-net -p 8080:80 my-webapp-image
    ```

      * The `webapp` container can now connect to the database using the hostname `database` instead of a hard-coded IP address, thanks to Docker's internal DNS service.

#### **3. Additional Networking Options**

| Flag | Description | Example |
| :--- | :--- | :--- |
| **`--hostname`** | Sets the container's hostname. | `... --hostname my-server centos:8` |
| **`--dns`** | Specifies a custom DNS server for the container. | `... --dns=1.1.1.1 centos:8` |
| **`--add-host`** | Adds a custom host-to-IP mapping in the container's `/etc/hosts` file. | `... --add-host api.server:10.0.0.5 centos:8` |

-----

### **Chapter 3: Container Resource Control & Management**

#### **1. The Importance of Resource Limits**

  - By default, a container can use all of the host's hardware resources (CPU, Memory).
  - It's crucial to set limits to prevent a single container from starving other containers or crashing the host.

#### **2. Key Resource Limit Options**

| Resource | Flag | Description | Example |
| :--- | :--- | :--- | :--- |
| **Memory** | `-m` or `--memory` | Sets a hard memory limit for the container. | `-m 512m` |
| **CPU** | `--cpus` | Limits the number of CPU cores the container can use. | `--cpus="1.5"` |
| **Block I/O**| `--device-write-bps` | Limits the write speed (bytes per second) to a device. | `--device-write-bps /dev/sda:1mb`|

##### **Resource Limit Example**

```bash
# Run an Nginx container limited to 1 CPU core and 1GB of RAM
docker run -d --name limited-nginx --cpus="1" -m 1g -p 8082:80 nginx
```

#### **3. Essential Container Management Commands**

| Command | Description |
| :--- | :--- |
| `docker container stats` | Displays a live stream of resource usage statistics for running containers. |
| `docker container top <container>` | Shows the running processes inside a container. |
| `docker container logs -f <container>` | Fetches and follows the logs of a container in real-time. |
| `docker container cp <src_path> <dest_path>` | Copies files or folders between a container and the host. |
| `docker container diff <container>` | Inspects changes (Added, Changed, Deleted) to a container's filesystem. |
