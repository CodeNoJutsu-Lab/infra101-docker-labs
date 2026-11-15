# Docker Command — Nginx with a Bind Mount

## Command

```bash
docker run --name website \
  -v "$PWD/website:/usr/share/nginx/html" \
  -p 8080:80 \
  --rm \
  nginx
```

## Objective

Run an Nginx container that serves HTML files directly from a local directory, allowing website changes on the host machine to appear immediately inside the container.

## Command Breakdown

| Option                                    | Description                                       |
| ----------------------------------------- | ------------------------------------------------- |
| `docker run`                              | Creates and starts a container                    |
| `--name website`                          | Assigns the container the name `website`          |
| `-v "$PWD/website:/usr/share/nginx/html"` | Mounts the local website directory into Nginx     |
| `-p 8080:80`                              | Maps host port `8080` to container port `80`      |
| `--rm`                                    | Automatically removes the container when it stops |
| `nginx`                                   | Specifies the image used to create the container  |

## Understanding the Bind Mount

```bash
-v "$PWD/website:/usr/share/nginx/html"
```

The syntax is:

```text
-v HOST_PATH:CONTAINER_PATH
```

In this command:

- `$PWD` represents the current working directory.
- `$PWD/website` is the website directory on your computer.
- `/usr/share/nginx/html` is the directory where Nginx serves its default HTML content.

The bind mount makes the host directory available inside the container.

**Important:** The host directory overlays the container directory. Its contents are not merged with the original Nginx HTML files.

## Understanding Port Mapping

```bash
-p 8080:80
```

```text
Host machine                    Nginx container

localhost:8080  ──────────────►  port 80
```

- `8080`: Port exposed on your computer.
- `80`: Port used by Nginx inside the container.

Access the website at:

http://localhost:8080

## Understanding `--rm`

```bash
--rm
```

Automatically removes the container after it stops.

This is useful for temporary development containers because you don't need to manually remove them afterward.

The bind-mounted website files remain on your computer.

## Example Project Structure

```text
my-project/
├── website/
│   ├── index.html
│   └── style.css
```

Example `website/index.html`:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Docker Website</title>
  </head>
  <body>
    <h1>Hello from Docker!</h1>
  </body>
</html>
```

## Execute the Command

From the root of your project:

```bash
docker run --name website \
  -v "$PWD/website:/usr/share/nginx/html" \
  -p 8080:80 \
  --rm \
  nginx
```

Then open:

http://localhost:8080

## Development Workflow

1. Create or modify `website/index.html` on your computer.
2. Nginx reads the file through the bind mount.
3. Refresh your browser.
4. Changes appear without rebuilding the image or restarting the container.

## Key Takeaways

- `-v` provides a bind mount between the host and container.
- `-p` publishes a container port to the host.
- `--name` gives the container a predictable name.
- `--rm` automatically removes the stopped container.
- Bind mounts are ideal for local development because files can be edited directly on the host.
