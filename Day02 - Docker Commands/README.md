
# Basic Docker Commands (2025/06/17)

## Part 1: Docker Basics & Initial Image Operations

### 1. Docker Service Management

* **Start and Enable Docker Service:**
    ```bash
    sudo systemctl enable --now docker.service
    ```
    * `enable`: Configures Docker to start automatically on system boot.
    * `--now`: Starts the Docker service immediately for the current session.
* **Check Docker Service Status:**
    ```bash
    sudo systemctl status docker.service
    ```
    * Verifies if the Docker daemon is running.

### 2. Docker Information & Architecture

* **Check Docker Version:**
    ```bash
    docker version
    ```
    * Displays client and server (daemon) version information.
    * **Concept:** Docker uses a client-server architecture. The client sends commands to the daemon, which executes them.
* **Check Docker System Information:**
    ```bash
    docker system info
    ```
    * Provides detailed information about the Docker environment, including:
        * Number of containers (running, paused, stopped).
        * Number of images.
        * Storage driver (e.g., `overlay2`).
        * Docker Root Directory (`/var/lib/docker` by default).
        * Operating System and Kernel version of the host.
        * Hardware resources (CPUs, Total Memory).
    * **Concept:** Containers are lightweight because they share the host's kernel, unlike VMs that require a full OS installation.

### 3. Running & Managing Basic Containers

* **Run a `hello-world` container:**
    ```bash
    docker container run hello-world
    ```
    * If the image is not local, Docker pulls it from Docker Hub.
    * Creates and runs a container that executes the `hello-world` program, prints output, and exits.
* **List all containers (including exited):**
    ```bash
    docker container ls -a
    ```
    * `-a` or `--all`: Shows all containers, regardless of their status.
* **Remove an exited container:**
    ```bash
    docker container rm <container_name_or_id>
    ```
    * Removes the specified container.
    * Cannot remove a running container unless forced (see Part 3).

### 4. Basic Docker Image Operations

* **Search Docker Hub for images:**
    ```bash
    docker search <image_name>
    ```
    * Searches for images on Docker Hub.
    * Output includes `NAME`, `DESCRIPTION`, `STARS`, and `OFFICIAL` status.
* **Pull a Docker image:**
    ```bash
    docker image pull <image_name>[:tag]
    ```
    * Downloads the image to your local machine. If no tag is specified, `latest` is assumed.
* **List local Docker images:**
    ```bash
    docker image ls
    ```
    * Displays all images stored on your local Docker host.
* **Remove a Docker image:**
    ```bash
    docker image rm <image_name_or_id>
    ```
    * Deletes the specified image from local storage.
    * An image cannot be removed if any containers (even exited ones) are using it. You must remove associated containers first.

### 5. Running a Web Server (Nginx Example)

* **Run Nginx in detached mode with port mapping:**
    ```bash
    docker container run -d --name webserver -p 80:80 nginx
    ```
    * `-d` or `--detach`: Runs the container in the background.
    * `--name webserver`: Assigns a human-readable name to the container.
    * `-p 80:80`: Maps host port `80` to container port `80`. (Syntax: `host_port:container_port`)
    * `nginx`: The Docker image to use.
* **Check running containers:**
    ```bash
    docker container ls
    ```
    * Displays only currently running containers.
* **Monitor container resource usage:**
    ```bash
    docker container stats webserver
    ```
    * Shows real-time CPU, memory, network I/O, and disk I/O for the specified container. Press `Ctrl+C` to exit.
* **Execute a command inside a running container:**
    ```bash
    docker container exec -it <container_name_or_id> /bin/bash
    ```
    * `-it`: Combines `-i` (interactive) and `-t` (pseudo-TTY) to provide an interactive shell.
    * `/bin/bash`: The command to execute (starts a Bash shell).
    * **Example:** `docker container exec -it webserver /bin/bash`
    * Inside the container, commands like `cat /etc/os-release` or installing tools (`apt update`, `apt install htop`) demonstrate interaction. Use `exit` to leave the container shell.
* **Stop a running container:**
    ```bash
    docker container stop webserver
    ```
    * Gracefully stops the container. Status changes to `Exited (0)`.
* **Start a stopped container:**
    ```bash
    docker container start webserver
    ```
    * Starts a previously stopped container. Status changes to `Up`.
* **Copy files between host and container:**
    ```bash
    docker container cp <source_path> <destination_path>
    ```
    * **Host to Container:** `docker container cp /host/path/file.html webserver:/usr/share/nginx/html/`
    * **Container to Host:** `docker container cp webserver:/container/path/file.html /host/path/`

---

## Part 2: Advanced Image Management & Docker Hub Interaction

### 1. Enhanced Docker Search

* **Filter search results:**
    ```bash
    docker search nginx --filter "stars=100"
    ```
    * Filters by specified criteria (e.g., images with more than 100 stars).
* **Limit search results:**
    ```bash
    docker search nginx --limit 5
    ```
    * Limits the number of results displayed.
* **Display full description:**
    ```bash
    docker search nginx --no-trunc
    ```
* **Format output with Go template:**
    ```bash
    docker search nginx --format "{{.Name}}\t{{.Description}}\t{{.Stars}}\t{{.Official}}"
    ```
    * Allows custom formatting of the output.
* **Concept: Deprecated Images:**
    * Images marked `DEPRECATED` are no longer officially maintained and might lack security updates. It's best to avoid them for new deployments. For example, `centos` might require a specific version tag like `centos:8`.

### 2. Docker Image Inspect

* **Get detailed image information:**
    ```bash
    docker image inspect <image_name>[:tag]
    ```
    * Outputs comprehensive JSON data including:
        * `Id`, `RepoTags`, `RepoDigests`.
        * `Created`, `DockerVersion`, `Author`.
        * `Config`: `ExposedPorts`, `Env` (environment variables), `Cmd` (default command), `Entrypoint` (command that always runs).
        * `Os`, `Architecture`.
        * `GraphDriver` (storage driver details).
* **Extract specific information using `--format` (Go template):**
    ```bash
    docker image inspect --format "{{.Id}}" ubuntu
    docker image inspect --format "{{.Config.Cmd}}" nginx
    docker image inspect --format "{{.Config.ExposedPorts}}" nginx
    ```

### 3. Docker Container Lifecycle Commands

* **Create a container (does not start it):**
    ```bash
    docker container create --name webserver nginx
    ```
    * Creates the container filesystem but keeps it in `Created` state.
* **Run a command directly (container runs and exits):**
    ```bash
    docker container run --name test1 centos:8 /bin/cal
    ```
    * Runs the specified command and then the container immediately stops.
* **Run with interactive shell:**
    ```bash
    docker container run -it --name test2 centos:8 /bin/bash
    ```
    * Provides an interactive terminal session inside the container.
* **Run in detached (background) mode:**
    ```bash
    docker container run -d --name test3 centos:8 /bin/bash
    ```
    * Runs the container in the background, returning control to your terminal.
* **Force remove a container (stops then removes):**
    ```bash
    docker container rm -f test2
    ```
    * Generally, it's better to `stop` then `rm`, but `-f` can be used for quick cleanup.

### 4. Docker Hub Authentication

* **Login to Docker Hub:**
    ```bash
    docker login -u <your_docker_hub_username>
    ```
    * Prompts for your Docker Hub password. Stores credentials locally.
* **Logout from Docker Hub:**
    ```bash
    docker logout
    ```
    * Removes local login credentials.
* **Docker Hub Pull/Push Rate Limits:**
    * Anonymous: 100 pulls/6 hours (by IP).
    * Authenticated: 200 pulls/6 hours.
    * Paid tiers: Unlimited.

### 5. Tagging & Pushing Images

* **Tag a local image for Docker Hub:**
    ```bash
    docker image tag <local_image_name>[:local_tag] <your_docker_hub_username>/<repository_name>[:tag]
    ```
    * Creates an alias for a local image with a Docker Hub compatible tag.
    * **Example:** `docker image tag nginx kim10322/webserver:1.0`
* **Push an image to Docker Hub:**
    ```bash
    docker image push <your_docker_hub_username>/<repository_name>[:tag]
    ```
    * Uploads the tagged image to your Docker Hub repository. Requires prior `docker login`.

---

## Part 3: Advanced Container Operations & Clean-up

### 1. `docker container run` Options Deep Dive

* **`-d` (detached):** Runs in the background. Output is not streamed to the console.
* **`-i` (interactive):** Keeps STDIN open, allowing for input even if not attached.
* **`-t` (TTY):** Allocates a pseudo-TTY (terminal). This is often used with `-i` for interactive shells.
* **`-it` (interactive + TTY):** A common combination for interactive sessions (e.g., `bash` shell). Output from commands within the shell is displayed.
    * When the process run with `-it` (e.g., `/bin/bash`) exits, the container stops.
* **`--rm` (remove on exit):** Automatically removes the container when it exits. Useful for one-off tasks.
    * **Example:** `docker container run --rm --name test4 centos:8 /bin/cal`
        * This command runs `/bin/cal` in a `test4` container. Once `/bin/cal` finishes, the `test4` container is automatically removed. You won't find it with `docker container ls -a`.
* **Background execution comparison:**
    * Running a command without `-d` (foreground): The terminal running the command will be blocked, and output will stream to it. If you close the terminal or hit `Ctrl+C`, the process (and usually the container) stops.
    * Running a command with `-d` (detached): The command runs in the background. The terminal returns immediately, and output is not shown. Use `docker container logs <name>` to view output.

### 2. Container Logging & Monitoring

* **View container logs:**
    ```bash
    docker container logs <container_name_or_id>
    ```
    * Retrieves logs from a container, regardless of whether it was run in foreground or detached mode. Useful for troubleshooting background services.

### 3. Container Lifecycle Management

* **`docker container stop <name_or_id>`:** Sends a `SIGTERM` signal (graceful shutdown). After a timeout (default 10s), if the process hasn't stopped, it sends `SIGKILL`.
    * A container in `Exited (0)` status means it terminated successfully.
* **`docker container start <name_or_id>`:** Starts a stopped container.
* **`docker container restart <name_or_id>`:** Stops and then starts a container.
* **`docker container kill <name_or_id>`:** Sends a `SIGKILL` signal (immediate, forceful shutdown) to the container's main process. No graceful shutdown.
    * `SIGKILL` (signal 9) is different from `SIGTERM` (signal 15). `SIGKILL` cannot be caught or ignored by processes.

### 4. Efficient Container & Image Cleanup

* **Remove all exited containers:**
    ```bash
    docker container rm -f $(docker container ls -aq)
    ```
    * `docker container ls -aq`: Lists all container IDs ( `-a` for all, `-q` for quiet/only IDs).
    * `$()`: Command substitution, so the output of `docker container ls -aq` becomes arguments to `docker container rm -f`.
    * `-f`: Forces removal (stops running containers before removing).
* **Remove all dangling images (images not associated with any containers):**
    ```bash
    docker image rm -f $(docker image ls -aq)
    ```
    * `docker image ls -aq`: Lists all image IDs.
    * **Note:** This command is powerful and removes all images unless they are actively used by a *running* container. If you have images you want to keep that are not currently tied to a running container, you might need a more selective approach.

### 5. Aliases for Convenience (Optional but Recommended)

To simplify frequently used commands, you can create aliases in your shell's configuration file (e.g., `~/.bashrc` for Bash).

* **Edit `~/.bashrc`:**
    ```bash
    vi ~/.bashrc # or nano ~/.bashrc
    ```
* **Add aliases (example):**
    ```bash
    alias crm='docker container rm -f $(docker container ls -aq)'
    alias irm='docker image rm -f $(docker image ls -aq)'
    alias cls='docker container ls -a'
    ```
* **Apply changes (after saving `~/.bashrc`):**
    ```bash
    source ~/.bashrc
    ```
    * This reloads the `.bashrc` file to make the new aliases effective in the current session.

