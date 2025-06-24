# Docker Networking, Container Management, Portainer, and Resource Control. (2025/06/25)

## **Consolidated Lecture Notes: Docker Operations**

Here are the complete notes from the lectures covering Docker Networking, Container Management, Portainer, and Resource Control.

### **Chapter 9: Docker Network Operations**

#### **1. Docker Network Types (`bridge`, `host`, `none`)**

  - **`bridge`**: The default network. It creates a private internal network (`docker0`) for containers, assigning them IP addresses (e.g., `172.17.0.x`).
  - **`host`**: The container shares the host's network, using the host's IP address directly without getting its own.
  - **`none`**: Isolates the container from the network, providing only a loopback interface.

#### **2. Custom Bridge Networks**

  - **Creating a Network:**
    ```bash
    # Create a default bridge network
    docker network create br0

    # Create a bridge network with a specific subnet and gateway
    docker network create -d bridge --subnet 192.168.100.0/24 --gateway 192.168.100.254 br1
    ```
  - **Connecting Containers:**
    ```bash
    # Connect at creation
    docker container run -itd --name testserver --network=br1 centos:8

    # Connect a running container
    docker network connect br1 myweb

    # Disconnect a running container
    docker network disconnect br0 myweb
    ```
  - **Communication:**
      - Containers on the **same custom network** can communicate by name.
      - Containers on **different networks** are isolated.
      - A container can be connected to multiple networks to act as a router between them. Manual route configuration (`route add...`) is required inside the containers for cross-network communication.
  - **Cleanup:**
    ```bash
    # Remove all unused networks
    docker network prune
    ```

-----

### **Chapter 10: Docker Container Operations**

#### **1. Legacy Container Linking (`--link`)**

> **Note:** This is a legacy feature. User-defined bridge networks are the modern standard for container communication.

  - The `--link` flag automatically adds a host entry (`/etc/hosts`) in the receiving container, allowing name resolution.

**Example:**

```bash
# Create a MySQL database container
docker container run -d --name mysql -e MYSQL_ROOT_PASSWORD=password mysql:5.7

# Create a WordPress container and link it to the MySQL container
docker container run -d --name wordpress --link mysql -p 80:80 wordpress:5
```

  - The `wordpress` container can now resolve the hostname `mysql` to the IP of the `mysql` container.

#### **2. Container Management Commands**

  - **`docker container attach <container>`**: Connects to the main process (PID 1) of a container. **Exiting will stop the container if the attached process is PID 1.**
  - **`docker container exec -it <container> <command>`**: **Preferred method.** Starts a *new* process (e.g., `/bin/bash`) inside a running container. Exiting this new shell does not stop the container.
  - **`docker container top <container>`**: Shows the processes running inside a container.
  - **`docker container stats`**: Live-streams resource usage statistics (CPU, Memory, Network I/O) for all or specified containers.
  - **`docker container logs -f <container>`**: Fetches container logs. The `-f` flag follows the log output in real-time.
  - **`docker container cp <src_path> <dest_path>`**: Copies files between the host and a container.
  - **`docker container diff <container>`**: Shows filesystem changes (Added, Changed, Deleted) in a container since it was created.

-----

### **Chapter 11: Docker Resource Control and Monitoring**

#### **1. Why Limit Resources?**

  - By default, a container can use all of the host's hardware resources (CPU, Memory, I/O).
  - If one container consumes excessive resources, it can negatively impact other containers on the same host. It's crucial to set limits.

#### **2. Resource Limit Options (`docker run ...`)**

  - **Memory Limits:**

      - `-m` or `--memory=`: Sets a hard memory limit (e.g., `-m 512m`). If the container exceeds this, it will be killed by an Out-of-Memory (OOM) error.
      - `--memory-swap=`: Sets the total memory plus swap the container can use. A value of `-1` allows unlimited swap.
      - `--oom-kill-disable`: Prevents the container from being killed when it hits an OOM error. **Use with caution**, as it can destabilize the host.

    <!-- end list -->

    ```bash
    # Limit memory to 200MB and total memory+swap to 300MB (100MB swap)
    docker run -d -m 200m --memory-swap 300m nginx
    ```

  - **CPU Limits:**

      - `--cpus=`: Sets the number of CPUs a container can use (e.g., `--cpus="1.5"` for one and a half CPUs).
      - `--cpu-shares=`: Sets a relative weight for CPU time. The default is `1024`. A container with a share of `2048` will get twice the CPU time as a container with `1024`.
      - `--cpuset-cpus=`: Pins a container to specific CPU cores (e.g., `--cpuset-cpus="0,1"` to use core 0 and 1).

    <!-- end list -->

    ```bash
    # Give a container 2x the CPU share relative to others
    docker run -d --name c1 --cpu-shares 2048 stress

    # Pin a container to CPU cores 0 and 1
    docker run -d --name c2 --cpuset-cpus="0,1" stress
    ```

  - **Block I/O Limits:**

      - `--device-read-bps=`, `--device-write-bps=`: Limits the read/write speed (bytes per second) for a specific device.
      - `--device-read-iops=`, `--device-write-iops=`: Limits the read/write rate (IO operations per second).

    <!-- end list -->

    ```bash
    # Limit write speed to 1MB/s on the /dev/sda device
    docker run -it --rm --device-write-bps /dev/sda:1mb ubuntu
    ```

-----

### **GUI Management: Portainer**

  - Portainer provides a powerful, web-based UI for managing Docker environments.

#### **1. Deployment**

  - Portainer itself runs as a Docker container.
  - It requires access to the Docker socket (`/var/run/docker.sock`) to manage the Docker daemon.

<!-- end list -->

```bash
# Command to run Portainer Community Edition
docker container run -d -p 8000:8000 -p 9443:9443 \
--name=portainer \
--restart=always \
-v /var/run/docker.sock:/var/run/docker.sock \
-v portainer_data:/data \
portainer/portainer-ce:latest
```

  - **Explanation:**
      - `-p 9443:9443`: Exposes the HTTPS port for the Portainer UI.
      - `--restart=always`: Ensures the Portainer container restarts automatically with the Docker daemon.
      - `-v /var/run/docker.sock...`: Mounts the host's Docker socket into the container.
      - `-v portainer_data...`: Creates a named volume to persist Portainer's own data.

#### **2. Using Portainer**

  - Access the web UI at `https://<your-docker-host-ip>:9443`.
  - On first launch, you will set up an `admin` user and password.
  - You can then connect to the local Docker environment to manage:
      - **Containers**: Start, stop, kill, inspect, view logs, and open a console (`exec`).
      - **Images**: Pull images from Docker Hub, view existing images, and remove them.
      - **Networks**: View, create, and remove networks.
      - **Volumes**: View, create, and remove volumes.
      - **Dashboard**: Get an at-a-glance overview of your Docker environment.
  - Portainer simplifies many of the command-line operations into a user-friendly graphical interface.