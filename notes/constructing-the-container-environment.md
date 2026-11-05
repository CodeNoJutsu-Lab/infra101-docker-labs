# Docker — Constructing the Container Environment

> Source: [Docker Course Labs — Constructing the Container Environment](https://docker.courselabs.co/labs/env/)

## 1. Container Environment

Docker allows us to configure the environment in which a container runs.

A container's environment is configured when it is created. Its configuration is generally fixed for the lifetime of that container.

We can configure:

- Environment variables
- Filesystem mounts (volumes and bind mounts)
- CPU resources
- Memory resources

### Important Docker CLI options

| Option            | Description                                     |
| ----------------- | ----------------------------------------------- |
| `-e`              | Set a single environment variable               |
| `--env-file`      | Load multiple environment variables from a file |
| `-v`              | Mount a host directory or Docker volume         |
| `--cpus`          | Limit CPU resources                             |
| `--memory` / `-m` | Limit memory resources                          |

To see all available options:

```bash
docker run --help
```

---

## 2. Environment Variables

Environment variables are key-value pairs used by applications to read configuration from their operating system environment.

They are commonly used for:

- Database credentials
- Database connection URLs
- Application profiles
- API keys
- Application configuration

### 2.1 Display environment variables

Use `printenv` to display environment variables inside a Linux container.

```bash
docker run alpine printenv
```

You can also inspect variables in a Java container:

```bash
docker run openjdk:8-jre-alpine printenv
```

### 2.2 Define an environment variable

Use `-e` to define an environment variable.

```bash
docker run -e COURSELABS=env alpine printenv
```

Syntax:

```bash
docker run -e VARIABLE_NAME=value IMAGE
```

Multiple environment variables can be defined:

```bash
docker run \
  -e COURSELABS=env \
  -e LANG=C.UTF-16 \
  openjdk:8-jre-alpine printenv
```

**Important:**

- Environment variables can override defaults defined in the image.
- Variables are configured when the container is created.
- To change the container's configuration, recreate the container with the new values.
- Environment variables are not a secure secret-management mechanism by themselves.

### 2.3 Use an environment file

When an application requires multiple environment variables, use an `.env` file.

Example: `exercises.env`

```env
COURSELABS=env
LANG=C.UTF-16
APP_ENV=development
```

Read the file:

```bash
cat labs/env/exercises.env
```

Pass it to Docker:

```bash
docker run \
  --env-file labs/env/exercises.env \
  alpine \
  printenv
```

You can combine `--env-file` and `-e`:

```bash
docker run \
  --env-file labs/env/exercises.env \
  -e LANG=en_US.UTF-8 \
  alpine \
  printenv
```

An explicitly supplied `-e` value takes precedence over the corresponding value loaded from the environment file.

---

## 3. Container Filesystem

Every container has its own filesystem, initially provided by its Docker image.

For example, an Nginx image contains:

- Nginx web server
- Configuration files
- Default HTML pages
- Required operating system files and dependencies

### 3.1 Start a container

```bash
docker run -d --name nginx nginx:alpine
```

### 3.2 Access the container filesystem

Use `docker exec` to execute commands inside an existing running container.

```bash
docker exec -it nginx sh
```

Useful commands:

```bash
# List files
ls /usr/share/nginx/html/

# Read the default HTML page
cat /usr/share/nginx/html/index.html

# Exit the container shell
exit
```

**Remember:**

`docker exec` runs a command inside an already running container. It does not create a new container.

---

## 4. Bind Mounts — Host and Container Filesystems

A bind mount allows you to make a file or directory from your host machine accessible inside a container.

This is useful when you want to:

- Share local files with a container.
- Provide configuration files.
- Serve local HTML files through Nginx.
- Modify source code without rebuilding an image.

### 4.1 Bind mount syntax

```bash
-v HOST_PATH:CONTAINER_PATH
```

| Part             | Meaning                            |
| ---------------- | ---------------------------------- |
| `HOST_PATH`      | File or directory on your computer |
| `CONTAINER_PATH` | Destination inside the container   |

Example:

```bash
-v ./html:/usr/share/nginx/html
```

This mounts your local `html` directory into Nginx's HTML directory.

### 4.2 Use an absolute host path

On macOS/Linux:

```bash
htmlPath="${PWD}/labs/env/html"
```

On PowerShell:

```powershell
$htmlPath="${PWD}/labs/env/html"
```

Run the container:

```bash
docker run -d \
  -p 8081:80 \
  -v ${htmlPath}:/usr/share/nginx/html \
  --name nginx2 \
  nginx:alpine
```

Access the application:

http://localhost:8081

### 4.3 How bind mounts work

Suppose your host directory contains:

```text
HOST: ./html/
├── index.html
├── about.html
├── contact.html
└── style.css
```

The original Nginx container directory contains:

```text
CONTAINER: /usr/share/nginx/html/
├── index.html
└── 50x.html
```

After mounting:

```text
CONTAINER: /usr/share/nginx/html/
├── index.html
├── about.html
├── contact.html
└── style.css
```

**Important: Bind mounts do not merge directories.**

The host directory is mounted over the container directory.

Consequently:

- Host files become visible inside the container.
- Existing container files at the mount destination become hidden.
- Files existing only in the container directory are not deleted; they are simply inaccessible through that mount.
- An empty host directory can hide all the original files at the mount destination.

### 4.4 Live updates

When you edit a mounted file on your host:

```text
Host: index.html
       |
       | Bind mount
       v
Container: index.html
```

The container sees the updated content immediately.

You do not need to rebuild the Docker image.

This is particularly useful during development.

### 4.5 Bind Mount vs Named Volume

Both can be configured using `-v`, but they have different purposes.

| Feature          | Bind Mount                               | Named Volume             |
| ---------------- | ---------------------------------------- | ------------------------ |
| Storage location | Host path selected by you                | Managed by Docker        |
| Main purpose     | Share local files                        | Persist application data |
| Host access      | Directly accessible at the selected path | Managed through Docker   |
| Typical use      | Development files and configuration      | Database storage         |

**Bind mount:**

```bash
docker run -v ./html:/usr/share/nginx/html nginx
```

**Named volume:**

```bash
docker run \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:17
```

For Prodigo, a named volume is generally the preferred approach for PostgreSQL data persistence.

---

## 5. Compute Resources

Docker allows you to limit the resources available to a container.

By default, containers have no explicit CPU or memory limits imposed by Docker.

### 5.1 Memory limit

Use `--memory` or `-m` to limit memory.

```bash
docker run -d \
  --name my-app \
  --memory 200m \
  nginx
```

The container is limited to approximately 200 MB of memory.

### 5.2 CPU limit

Use `--cpus` to specify the maximum CPU capacity.

```bash
docker run -d \
  --name my-app \
  --cpus 0.25 \
  nginx
```

This limits the container to 25% of one CPU core's capacity.

### 5.3 Combine CPU and memory limits

```bash
docker run -d \
  --name pi \
  -p 8031:80 \
  --memory 200m \
  --cpus 0.25 \
  kiamol/ch05-pi -m web
```

The container now has:

- Maximum memory: 200 MB
- Maximum CPU capacity: 0.25 CPU

Resource limits can affect application performance.

### 5.4 Inspect resource configuration

```bash
docker inspect pi
```

To display selected configuration fields:

```bash
docker inspect \
  --format='Memory: {{.HostConfig.Memory}}, CPU: {{.HostConfig.NanoCpus}}' \
  pi
```

Useful fields:

| Field                 | Meaning                |
| --------------------- | ---------------------- |
| `HostConfig.Memory`   | Memory limit in bytes  |
| `HostConfig.NanoCpus` | CPU quota in nano-CPUs |

A value of `0` means no explicit limit is configured for that resource.

**Important:** Resource limits are configured when creating the container. To change them, recreate the container or use supported Docker resource-update commands.

---

## 6. Copy Files Between Host and Container

Sometimes files are generated inside a container but need to be retrieved on your computer.

For example:

- TLS certificates
- Logs
- Generated reports
- Exported data
- Configuration files

Docker provides `docker cp`.

### 6.1 Start a certificate generator

```bash
docker run -d \
  --name tls \
  kiamol/ch15-cert-generator
```

### 6.2 Copy files from container to host

Syntax:

```bash
docker cp CONTAINER:CONTAINER_PATH HOST_PATH
```

Examples:

```bash
# Copy certificate
docker cp tls:/certs/server-cert.pem ./server-cert.pem

# Copy private key
docker cp tls:/certs/server-key.pem ./server-key.pem
```

You can also copy an entire directory:

```bash
docker cp tls:/certs ./certs
```

### 6.3 Copy files from host to container

Syntax:

```bash
docker cp HOST_PATH CONTAINER:CONTAINER_PATH
```

Example:

```bash
docker cp ./application.yml nginx:/tmp/application.yml
```

**Remember:**

`docker cp` copies files. It does not establish a continuous connection between the host and container like a bind mount does.

---

## 7. Useful Commands — Quick Reference

| Command                                | Purpose                              |
| -------------------------------------- | ------------------------------------ |
| `docker run -e KEY=value image`        | Set environment variable             |
| `docker run --env-file .env image`     | Load environment variables from file |
| `docker exec -it container sh`         | Open container shell                 |
| `docker inspect container`             | Inspect container configuration      |
| `docker cp container:/path ./path`     | Copy container files to host         |
| `docker cp ./path container:/path`     | Copy host files into container       |
| `docker run -v ./data:/app/data image` | Create bind mount                    |
| `docker run -v mydata:/app/data image` | Create named volume                  |
| `docker run --memory 200m image`       | Limit memory                         |
| `docker run --cpus 0.5 image`          | Limit CPU capacity                   |

---

## 8. Cleanup

Remove a specific container:

```bash
docker rm -f nginx2
```

Remove all containers:

```bash
docker rm -f $(docker ps -aq)
```

**Warning:** The second command removes every container on your Docker host, including unrelated projects. Use it only in a dedicated learning environment.

Removing containers does not automatically remove named volumes.

---

## 9. Key Takeaways

1. **Environment variables:** Configure applications without hardcoding configuration into images.
2. **Environment files:** Convenient for passing multiple environment variables.
3. **Container filesystem:** Every container has its own filesystem based on its image.
4. **Bind mounts:** Share host files with containers without rebuilding images.
5. **Named volumes:** Persist data independently of container lifecycle.
6. **Resource limits:** Control the CPU and memory available to containers.
7. **Docker exec:** Execute commands inside running containers.
8. **Docker cp:** Copy files between the host and containers.

### What to remember for Prodigo

| Prodigo requirement               | Docker concept          |
| --------------------------------- | ----------------------- |
| Database connection configuration | Environment variables   |
| Local application configuration   | Bind mounts             |
| PostgreSQL persistence            | Named volumes           |
| Inspect backend logs              | `docker logs`           |
| Debug backend container           | `docker exec`           |
| Export generated files            | `docker cp`             |
| Limit database resources          | `--memory` and `--cpus` |

### Official documentation

- [Docker run](https://docs.docker.com/reference/cli/docker/container/run/)
- [Environment variables](https://docs.docker.com/engine/containers/run/#environment-variables)
- [Bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)
- [Volumes](https://docs.docker.com/engine/storage/volumes/)
- [Resource constraints](https://docs.docker.com/engine/containers/resource_constraints/)
- [Docker cp](https://docs.docker.com/reference/cli/docker/container/cp/)
- [Docker inspect](https://docs.docker.com/reference/cli/docker/inspect/)
