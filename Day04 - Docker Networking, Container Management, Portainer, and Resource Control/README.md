# Docker Networking, Container Management, Portainer, and Resource Control. (2025/06/25)


### **Chapter 9: Docker Network Operations**

#### **1. Docker Network Types (`bridge`, `host`, `none`)**

By default, Docker provides three network drivers for handling container communication.

| Driver | Description | Use Case |
| :--- | :--- | :--- |
| **`bridge`** | **(Default)** Creates a private internal network. Docker manages IP addressing, routing, and a gateway for containers. | The most common driver for standalone containers that need to communicate with each other and the outside world. |
| **`host`** | The container shares the host's entire network stack. No IP address is assigned to the container. | When network performance is critical and you don't need network isolation between the container and the host. |
| **`none`** | The container is created with its own network stack but has no configured interfaces besides loopback (`lo`). | For containers that do not require any network access or for custom network setups. |

**Default Bridge Network (`docker0`) Diagram**

This diagram illustrates how a container connects to the host and the outside world through the default `docker0` bridge.

```
+------------------+     +--------------------------+     +-------------------+
|    Container     |     |        Docker Host       |     |   External Network  |
|                  |     |                          |     | (e.g., Internet)  |
|  eth0@ifXX      +<--->+  docker0 (172.17.0.1)  +<--->+    ens33 (Host IP)  |
| (172.17.0.2)     |     | (Virtual Bridge/Gateway) |     |                   |
+------------------+     +--------------------------+     +-------------------+
```

#### **2. Custom Bridge Networks**

You can create your own isolated bridge networks for better application structure and security.

**Key Commands:**

| Command | Description |
| :--- | :--- |
| `docker network create <name>` | Creates a new bridge network. |
| `docker network inspect <name>` | Shows detailed information about a network (subnet, gateway, connected containers). |
| `docker network connect <net> <cont>` | Connects a running container to an additional network. |
| `docker network disconnect <net> <cont>`| Disconnects a container from a network. |
| `docker network rm <name>` | Removes a network (must not have containers attached). |
| `docker network prune` | Removes all unused networks. |

**Communication Practice Diagram**

This diagram shows the setup used in the video to demonstrate communication between isolated networks using a multi-homed container (`alpine3`) as a router.

```
       +----------------------------------+          +----------------------------------+
       |         Docker Host (br-...)       |          |         Docker Host (docker0)      |
       |       Gateway: 172.18.0.1/16     |          |       Gateway: 172.17.0.1/16     |
       +----------------|-----------------+          +----------------|-----------------+
                        |                                          |
  +---------------------|-----------------------+    +---------------|--------------------+
  | Network: alpinenet (172.18.0.0/16)          |    | Network: bridge (172.17.0.0/16)    |
  +---------------------------------------------+    +------------------------------------+
    |         |                   |                    |                   |
.2  |      .3 |                .4 |                 .2 |                .3 |
[eth0]   [eth0]             [eth1] <----[alpine3]----> [eth0]             [eth0]
+---------+ +---------+         +----------------+         +---------+
| alpine1 | | alpine2 |         | (Multi-homed)  |         | alpine4 |
+---------+ +---------+         +----------------+         +---------+

# Communication Path:
# alpine1/2 <--> alpine3(eth1) <--> alpine3(eth0) <--> alpine4
# (Requires manual routes added inside alpine1/2 and alpine4)
```

-----

### **Chapter 10: Docker Container Operations**

#### **1. Key Container Management Commands**

This table summarizes essential commands for managing the container lifecycle.

| Command | Description |
| :--- | :--- |
| `docker container attach` | Attach to a container's main process (PID 1). Exiting can stop the container. |
| `docker container exec` | **(Recommended)** Execute a new command in a running container. |
| `docker container top` | Display the running processes of a container. |
| `docker container stats` | Display a live stream of resource usage statistics. |
| `docker container port` | List port mappings for the container. |
| `docker container rename` | Rename a container. |
| `docker container cp` | Copy files/folders between a container and the host. |
| `docker container diff` | Inspect changes (Add, Change, Delete) to a container's filesystem. |
| `docker container logs` | Fetch the logs of a container. Use `-f` to follow logs live. |

-----

### **Chapter 11: Docker Resource Control and Monitoring**

#### **1. Resource Limit Options (`docker run`)**

It is critical to set resource limits to ensure fair resource distribution and host stability.

**Memory Limit Options:**

| Flag | Description |
| :--- | :--- |
| `-m` or `--memory` | Sets a hard memory limit (e.g., `512m`, `1g`). |
| `--memory-swap` | Sets the total allowed memory + swap. Use `-1` for unlimited swap. |
| `--oom-kill-disable` | Prevents the kernel from killing the container on an Out-Of-Memory error. **(Use with caution)**. |

**CPU Limit Options:**

| Flag | Description |
| :--- | :--- |
| `--cpus` | Limits the number of CPU cores the container can use (e.g., `"1.5"`). |
| `--cpu-shares` | A relative weight for CPU time. Default is `1024`. Higher means more CPU time. |
| `--cpuset-cpus` | Pins a container to specific CPU cores (e.g., `"0,1"`). |

**Block I/O Limit Options:**

| Flag | Description |
| :--- | :--- |
| `--device-read-bps` | Limits read speed in bytes per second (e.g., `/dev/sda:1mb`). |
| `--device-write-bps` | Limits write speed in bytes per second. |
| `--device-read-iops` | Limits read rate in IO operations per second. |
| `--device-write-iops` | Limits write rate in IO operations per second. |

-----

### **GUI Management: Portainer**

Portainer is a web-based UI for managing Docker environments.

#### **1. Portainer Deployment Command Explained**

```bash
docker container run -d \
  -p 8000:8000 -p 9443:9443 \
  --name=portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

| Flag/Argument | Purpose |
| :--- | :--- |
| `-d` | Runs the container in detached (background) mode. |
| `-p 8000:8000 -p 9443:9443` | Maps the host's ports to the container's ports for the UI (9443) and edge agent (8000). |
| `--name=portainer` | Assigns a memorable name to the container. |
| `--restart=always` | Configures the container to restart automatically if it stops or the host reboots. |
| `-v /var/run/docker.sock...` | **(Crucial)** Mounts the Docker socket from the host into the container, allowing Portainer to manage Docker. |
| `-v portainer_data:/data` | Creates a named volume to persist Portainer's own settings and data. |
| `portainer/portainer-ce:latest`| Specifies the Docker image to use (Community Edition, latest version). |
