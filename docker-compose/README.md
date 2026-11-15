# Docker Compose — Introduction

## 1. What is Docker Compose?

Docker Compose is a tool used to **define, configure, and run multiple Docker containers as a single application**.

Instead of manually executing multiple `docker run` commands, you describe your application's services in a YAML configuration file, usually named `compose.yaml`.

Docker Compose is particularly useful for applications containing multiple components, such as:

- A Spring Boot backend
- A PostgreSQL database
- A Redis cache
- A frontend application

## 2. Why use Docker Compose?

Without Docker Compose, starting an application with multiple containers requires several commands and manually configuring their relationships.

Docker Compose allows you to:

- Define multiple containers in one file.
- Configure networks between services.
- Configure persistent volumes.
- Define environment variables.
- Configure ports and health checks.
- Start and stop the entire application with a single command.
- Reproduce the same development environment consistently.

## 3. Docker Compose architecture

Docker Compose organizes an application around three important concepts:

| Concept  | Description                             |
| -------- | --------------------------------------- |
| Services | Containers that make up the application |
| Networks | Communication channels between services |
| Volumes  | Persistent storage for container data   |

A Compose project typically creates a network automatically, allowing services to communicate using their service names.

## 4. The compose.yaml file

Example of a simple application containing PostgreSQL:

```yaml
services:
  db:
    image: postgres:17
    container_name: compose-db
    environment:
      POSTGRES_DB: prodigo
      POSTGRES_USER: prodigo
      POSTGRES_PASSWORD: devpassword
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Explanation

- `services`: declares the application's services.
- `db`: the service name.
- `image`: the Docker image used by the service.
- `container_name`: explicitly assigns a container name.
- `environment`: defines environment variables inside the container.
- `ports`: publishes a container port to the host.
- `volumes`: mounts persistent storage.
- `postgres_data`: a named Docker volume.
- The top-level `volumes` section declares the named volume.

## 5. Essential Docker Compose commands

### Start services

```bash
docker compose up
```

Start services in the foreground and display their logs.

```bash
docker compose up -d
```

Start services in the background.

### Stop services

```bash
docker compose stop
```

Stops containers without removing them.

### Stop and remove the application

```bash
docker compose down
```

Removes containers and the Compose network.

Named volumes are preserved.

### Remove everything, including volumes

```bash
docker compose down -v
```

⚠️ This removes declared named volumes and can permanently delete database data.

### Build images and start services

```bash
docker compose up --build
```

Builds or rebuilds images before starting the services.

### View running services

```bash
docker compose ps
```

### View service logs

```bash
docker compose logs
```

```bash
docker compose logs -f db
```

Follow the logs of a specific service.

### Execute commands inside a service

```bash
docker compose exec db psql -U prodigo
```

Execute a command inside the running database container.

## 6. Service-to-service communication

One of the most important Docker Compose concepts is that services can communicate using their **service names as hostnames**.

For example, if your Compose file contains:

```yaml
services:
  api: ...
  db: ...
```

Your Spring Boot application should connect to PostgreSQL using:

```text
jdbc:postgresql://db:5432/prodigo
```

Not:

```text
jdbc:postgresql://localhost:5432/prodigo
```

Inside the API container, `localhost` refers to the API container itself.

Docker Compose automatically creates a network where services can resolve one another by their service names.

## 7. Docker Compose vs Docker

| Docker                                        | Docker Compose                       |
| --------------------------------------------- | ------------------------------------ |
| Manages individual containers and images      | Manages multi-container applications |
| `docker run`                                  | `docker compose up`                  |
| `docker stop`                                 | `docker compose stop`                |
| `docker logs`                                 | `docker compose logs`                |
| `docker exec`                                 | `docker compose exec`                |
| Container configuration through CLI arguments | Container configuration through YAML |

Docker Compose does not replace Docker. It uses Docker to create and manage containers.

## 8. Practical example — PostgreSQL

Create a directory:

```bash
mkdir compose-intro
cd compose-intro
```

Create `compose.yaml`:

```yaml
services:
  db:
    image: postgres:17
    environment:
      POSTGRES_DB: demo
      POSTGRES_USER: demo
      POSTGRES_PASSWORD: devpassword
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

Start PostgreSQL:

```bash
docker compose up -d
```

Check the container:

```bash
docker compose ps
```

Connect to PostgreSQL:

```bash
docker compose exec db psql -U demo -d demo
```

Stop the application:

```bash
docker compose down
```

Start it again:

```bash
docker compose up -d
```

Your database data remains because it is stored in the named volume.

## 9. Important concepts to remember

- `compose.yaml` describes the application infrastructure.
- A service represents a component of the application.
- Compose automatically creates a network for the project.
- Services communicate using service names.
- Named volumes preserve data between container recreations.
- `docker compose down` does not normally delete named volumes.
- `docker compose down -v` deletes named volumes.
- `docker compose up -d` starts services in the background.
- `docker compose logs -f` helps debug running services.

## 10. Next learning topics

1. Docker Compose environment variables and `.env` files.
2. Build a Spring Boot API using Compose.
3. PostgreSQL health checks and `depends_on`.
4. Compose networking and service discovery.
5. Development vs production Compose configurations.
6. Docker Compose debugging and troubleshooting.
