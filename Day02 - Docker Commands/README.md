# **Docker Basic Commands and Concepts (2025/06/17)**

This document provides a structured, easy-to-read reference for the core Docker commands and concepts covered in the three lecture videos, designed to help beginners understand and utilize container technology effectively.

## **Chapter 1: Docker Environment & Architecture**

### **1. Docker Service Management (Linux)**

Before using the Docker engine, you must ensure the service (daemon) is running correctly on the host system.

| Command | Description |
| :--- | :--- |
| `sudo systemctl status docker` | Checks the current running status (active/inactive) of the Docker service. |
| `sudo systemctl start docker` | Starts the Docker service. |
| `sudo systemctl stop docker` | Stops the Docker service. |
| `sudo systemctl enable --now docker` | Enables Docker to start automatically on system boot and immediately starts the service. |

### **2. Docker Architecture and Information**

Docker operates on a client-server architecture. When a user enters a `docker` command, the client sends it to the Docker daemon, which executes the task.

**Docker Architecture Diagram**

```
+-----------------+      +-------------------------------------------+      +-----------------------+
|                 |      |               Docker Host                 |      |                       |
|   User (CLI)    |<---->| Docker Client <---> Docker Daemon (Server)|<---->|  Registry (Docker Hub)  |
|                 |      |                                           |      |                       |
+-----------------+      +-------------------|-----------------------+      +-----------------------+
                               |
                               +------> [ Images ] ------> [ Containers ]
```

**Key Information Commands**

| Command | Description | Key Items to Check |
| :--- | :--- | :--- |
| `docker version` | Displays detailed version information for the Docker client and server (Engine). | `Client Version`, `Server Version`, `Go version` |
| `docker system info` | Shows comprehensive information about the Docker system environment. | Container/Image counts, Storage Driver, **Docker Root Dir** |

-----

## **Chapter 2: Docker Image Management**

An image is a read-only template containing the application and its libraries, used to create containers.

### **Core Image Management Commands**

| Command | Description |
| :--- | :--- |
| `docker search [image_name]` | Searches for images on Docker Hub. Filter official images with `--filter "is-official=true"`. |
| `docker image pull [image_name]:[tag]` | Downloads an image from Docker Hub to the local machine. Defaults to the `latest` tag if not specified. |
| `docker image ls` | Lists all images stored on the local Docker host. |
| `docker image inspect [image_id_or_name]` | Shows detailed metadata of an image (layers, environment variables, default command) in JSON format. |
| `docker image rm [image_id_or_name]` | Deletes an image from the local storage. Fails if the image is in use by a container. (Use `-f` to force). |
| `docker image prune` | Removes all unused (dangling) images at once. |

-----

## **Chapter 3: Docker Container Lifecycle Management**

A container is a runnable instance of an image. This section covers how to manage its lifecycle: creating, running, stopping, and removing.

### **1. Running a Container (`docker container run`)**

The `run` command combines `create` and `start` to create and immediately launch a container.

| Option | Description |
| :--- | :--- |
| `-d` `--detach` | Runs the container in the background (detached mode). |
| `-p` `--publish` | Maps a host port to a container port using the format: **`host_port:container_port`**. |
| `--name` | Assigns a unique, human-readable name to the container. If omitted, Docker generates a random name. |
| `-it` | A combination of `-i` (interactive) and `-t` (tty), providing an interactive shell environment. |
| `--rm` | Automatically removes the container when it exits. Useful for one-off tasks. |

**Example: Running an Nginx Web Server**

```bash
# Run an Nginx container named 'webserver', mapping port 80, in the background
docker container run -d --name webserver -p 80:80 nginx
```

### **2. Managing and Inspecting Containers**

| Command | Description |
| :--- | :--- |
| `docker container ls -a` | Lists all containers. (Without `-a`, lists only running containers). |
| `docker container stop [name/id]` | Gracefully stops a running container by sending a `SIGTERM` signal. |
| `docker container start [name/id]` | Starts a stopped container. |
| `docker container restart [name/id]` | Restarts a container. |
| `docker container rm [name/id]` | Removes a stopped container. Use `-f` to force-remove a running container. |
| `docker container prune` | Removes all stopped containers at once. |

### **3. Interacting with Running Containers**

| Command | Description |
| :--- | :--- |
| `docker container exec -it [name/id] [cmd]` | **(Recommended)** Executes a new process inside a running container (e.g., `/bin/bash`). |
| `docker container attach [name/id]` | Attaches directly to the main process (PID 1) of the container. Exiting can stop the container. |
| `docker container logs [name/id]` | Fetches the standard output logs of a container. Use `-f` to follow the logs in real-time. |
| `docker container stats [name/id]` | Monitors a container's resource usage (CPU, memory, etc.) in real-time. |
| `docker container cp [src_path] [dest_path]` | Copies files between the host and a container. |

-----

### **Chapter 4: Docker Hub Integration and Image Distribution**

You can upload modified or new images to Docker Hub to share them or deploy them to other environments.

**Image Distribution Workflow**

```
1. Modify an image locally (via Dockerfile or commit)
      |
      V
2. Tag the image for Docker Hub (docker image tag)
   <local_image> -> <username>/<repo_name>:<tag>
      |
      V
3. Log in to Docker Hub (docker login)
      |
      V
4. Push the image (docker image push)
   Local Host -> Docker Hub
```

#### **Core Distribution Commands**

| Command | Description |
| :--- | :--- |
| `docker login -u [username]` | Logs into Docker Hub and stores authentication credentials locally. |
| `docker logout` | Logs out of Docker Hub. |
| `docker image tag [source] [target:tag]` | Creates a new tag for an image. To push, the target must be in the `username/repository` format. |
| `docker image push [username/repo:tag]` | Uploads a local image to a Docker Hub repository. |
