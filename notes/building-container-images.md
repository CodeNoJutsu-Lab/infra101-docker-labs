# Docker — Building Container Images

## 1. What is a Docker Image?

A Docker image is a **package used to create and run containers**.

It contains:

- The filesystem and application files.
- Required dependencies and runtime.
- Configuration and metadata.
- The default command executed when a container starts.

**Relationship:**

- **Dockerfile:** Instructions used to build an image.
- **Image:** The packaged result of the build.
- **Container:** A running instance of an image.

---

## 2. Docker Image Commands

| Command                          | Description                      |
| -------------------------------- | -------------------------------- |
| `docker image ls`                | List local images                |
| `docker image ls 'courselabs/*'` | Filter images by repository name |
| `docker pull ubuntu`             | Download an image                |
| `docker pull ubuntu:22.04`       | Download a specific version      |
| `docker build -t name .`         | Build and tag an image           |
| `docker history image`           | Display image build layers       |
| `docker image inspect image`     | Display image configuration      |
| `docker image rm image`          | Remove an image                  |

### Image tags

A tag identifies a particular version of an image.

```bash
docker pull ubuntu
docker pull ubuntu:22.04
```

- `ubuntu` uses the `latest` tag by default.
- `ubuntu:22.04` explicitly selects version 22.04.
- Tags are references to images, not necessarily unique images.

### Build an image

```bash
docker build -t courselabs/base ./labs/images/base
```

- `docker build`: Builds an image.
- `-t`: Assigns a name and optional tag.
- `./labs/images/base`: Build context directory.

List the resulting images:

```bash
docker image ls
docker image ls 'courselabs/*'
```

---

## 3. Base Images and Image Hierarchy

Images are built on top of other images.

For example:

```text
Ubuntu
  └── Java Runtime
       └── Application Dependencies
            └── My Application
```

The `FROM` instruction defines the base image.

```dockerfile
FROM ubuntu:22.04
```

However, you don't always need to start from an operating system image.

Prefer official images that already contain the runtime you need:

- Java: Eclipse Temurin
- Node.js: Node
- Python: Python
- Web server: Nginx
- Database: PostgreSQL

Official images: https://hub.docker.com/search?type=image&image_filter=official

**Important:** Use an appropriate official runtime image instead of installing everything manually on top of a generic OS image.

---

## 4. Understanding Dockerfile Instructions

A Dockerfile contains instructions that describe how an image is constructed.

### FROM

Defines the base image.

```dockerfile
FROM ubuntu:22.04
```

### RUN

Executes a command during image construction.

```dockerfile
RUN apt-get update && apt-get install -y curl
```

The result becomes part of the image.

### CMD

Defines the default command executed when a container starts.

```dockerfile
CMD ["curl", "--head", "https://docker.courselabs.co"]
```

You can override the command when running the container.

```bash
docker run courselabs/curl
```

### ENTRYPOINT

Defines the executable that the container runs.

```dockerfile
ENTRYPOINT ["curl"]
```

Arguments supplied to `docker run` are passed to the entrypoint.

For example:

```bash
docker run courselabs/curl:v2 --head docker.courselabs.co
```

This effectively executes:

```bash
curl --head docker.courselabs.co
```

### CMD vs ENTRYPOINT

| Instruction  | Purpose                          |
| ------------ | -------------------------------- |
| `CMD`        | Default command or arguments     |
| `ENTRYPOINT` | Main executable of the container |

A common combination:

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
CMD ["--spring.profiles.active=prod"]
```

Here, the application is the fixed executable, while the default arguments can be overridden.

---

## 5. Building Images with a Custom Dockerfile

Build the first curl image:

```bash
docker build -t courselabs/curl ./labs/images/curl
```

Run it:

```bash
docker run courselabs/curl
docker run courselabs/curl docker.courselabs.co
docker run courselabs/curl curl --head docker.courselabs.co
```

The behavior depends on how `CMD` and `ENTRYPOINT` are configured.

### Specifying a custom Dockerfile

By default, Docker looks for a file named `Dockerfile`.

Use `-f` when the file has another name:

```bash
docker build \
  -t courselabs/curl:v2 \
  -f labs/images/curl/Dockerfile.v2 \
  labs/images/curl
```

- `-f`: Specifies the Dockerfile location.
- Final argument: Specifies the build context.
- `-t`: Specifies the image name and tag.

Compare image sizes:

```bash
docker image ls courselabs/curl
```

Smaller images generally mean:

- Less data to download.
- Less storage usage.
- Fewer unnecessary packages.
- A smaller attack surface.

---

## 6. Build Context

The **build context** is the directory Docker can access when building an image.

Example:

```text
web/
├── Dockerfile
└── index.html
```

Dockerfile:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
```

Build:

```bash
docker build -t courselabs/web ./labs/images/web
```

Docker sends the context to the builder. Files referenced by `COPY` and `ADD` must be available within that context.

**Important:** The build context is not necessarily the directory containing the Dockerfile. It is the final path argument of `docker build`.

---

## 7. Building and Running an Nginx Image

Build the image:

```bash
docker build -t courselabs/web ./labs/images/web
```

Run a container:

```bash
docker run -d -p 8090:80 courselabs/web
```

Test it:

```bash
curl localhost:8090
```

The request follows this path:

```text
Host machine
localhost:8090
      |
      | Port mapping
      v
Container
Nginx:80
      |
      v
/usr/share/nginx/html/index.html
```

The custom HTML page is served by Nginx inside the container.

---

## 8. Practical Lab — Build a Java Image

### Objective

Create an image containing a compiled Java application and run it inside a container.

The lab already provides:

```text
labs/images/java/
└── HelloWorld.class
```

The application must execute:

```bash
java HelloWorld
```

### Solution

Create a `Dockerfile` inside `labs/images/java`:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY HelloWorld.class .

CMD ["java", "HelloWorld"]
```

Build the image from the project root:

```bash
docker build -t courselabs/java ./labs/images/java
```

Run the container:

```bash
docker run --rm courselabs/java
```

### Explanation

| Instruction | Purpose                                                 |
| ----------- | ------------------------------------------------------- |
| `FROM`      | Uses an image containing Java 21 runtime                |
| `WORKDIR`   | Sets `/app` as the working directory                    |
| `COPY`      | Copies the compiled class into the image                |
| `CMD`       | Executes the Java application when the container starts |

`--rm` automatically removes the container after it exits.

**Note:** This works because `HelloWorld.class` is already compiled and does not require external dependencies. If the application uses packages or multiple classes, preserve the compiled directory structure when copying the files.

---

## 9. BuildKit

BuildKit is Docker's modern image-building engine.

The original lab disables it to display the individual Dockerfile instructions:

macOS / Linux:

```bash
export DOCKER_BUILDKIT=0
```

PowerShell:

```powershell
$env:DOCKER_BUILDKIT=0
```

BuildKit provides modern build capabilities, improved performance and better caching. Disabling it is only necessary for following this particular lab's output.

---

## 10. Cleanup

Remove all containers:

```bash
docker rm -f $(docker ps -aq)
```

**Warning:** This removes every container on your local Docker engine, including unrelated projects.

For a safer cleanup, remove only the lab containers:

```bash
docker rm -f $(docker ps -aq --filter ancestor=courselabs/web)
```

Remove lab images:

```bash
docker image rm courselabs/base courselabs/curl courselabs/curl:v2 courselabs/web courselabs/java
```

---

## 11. Important Concepts to Remember

| Concept       | Key takeaway                         |
| ------------- | ------------------------------------ |
| Image         | Package used to create containers    |
| Dockerfile    | Instructions for building an image   |
| `FROM`        | Selects the base image               |
| `RUN`         | Executes commands during the build   |
| `CMD`         | Defines the default startup command  |
| `ENTRYPOINT`  | Defines the main executable          |
| `COPY`        | Copies files into the image          |
| Tag           | Identifies an image version          |
| Build context | Files available to the image builder |
| `-f`          | Selects a custom Dockerfile          |
| `-t`          | Names and tags an image              |

### Practical recommendations

- Prefer official images and maintained runtime images.
- Use specific image tags instead of relying blindly on `latest`.
- Keep images small by avoiding unnecessary dependencies.
- Use `.dockerignore` to exclude files that aren't needed in the build context.
- Use multi-stage builds to separate compilation from runtime.
- Prefer `ENTRYPOINT` for the main application executable and `CMD` for default arguments when appropriate.

---

## 12. Official Documentation

- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [Docker image build](https://docs.docker.com/reference/cli/docker/image/build/)
- [Docker image commands](https://docs.docker.com/reference/cli/docker/image/)
- [BuildKit](https://docs.docker.com/build/buildkit/)
- [Docker Hub — Official Images](https://hub.docker.com/search?type=image&image_filter=official)
