Creation date: Tuesday, April 14th 2026, 1:57:08 am

**Goal**: Get hands-on with Docker - pull images and run containers with confidence.

### Installing Docker

Using Docker Engine directly:

```
# Quick install script (Linux)
curl -fsSL https://get.docker.com | sh

# Add yourself to docker group (avoids sudo)
sudo usermod -aG docker $USER
# Log out and back in for group change to take effect
```

#### **Verify Installation**

```
docker version
```

You should see Client and Server versions. If you only see Client, Docker daemon isn’t running.

```
docker info
```

Shows detailed information about your Docker installation.

---
#### **First Container**

```
docker run hello-world
```

What just happened?

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
...
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

Docker:

1. Looked for `hello-world` image locally - didn’t find it
    
2. Pulled it from Docker Hub (the default registry)
    
3. Created a container from the image
    
4. Started the container
    
5. Container ran, printed message, exited
    

> _“One command did five things. That’s the Docker experience.”_

---
### Docker Hub and Images

Docker Hub ([hub.docker.com](https://hub.docker.com/ "https://hub.docker.com/")) is a public registry with thousands of images.

#### **Finding Images**

```
# Search from command line
docker search nginx
```

Or browse Docker Hub in your browser. Look for:

- **Official Images**: Maintained by Docker, verified, trustworthy
    
- **Verified Publisher**: From known companies
    
- **Stars and pulls**: Popularity indicators
    

#### **Pulling Images**

```
# Pull without running
docker pull nginx

# Pull specific version (tag)
docker pull nginx:1.28

# Pull specific variant
docker pull nginx:1.28-alpine
```

**Always specify a tag in production.** The `latest` tag changes over time.

#### **Listing Local Images**

```
docker images
```

```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         1.28      a6bd71f48f68   2 weeks ago    187MB
hello-world   latest    d2c94e258dcb   9 months ago   13.3kB
```

---
### Running Containers

#### **Basic Run**

```
docker run nginx
```

This runs nginx in the foreground. You’ll see logs but can’t use your terminal. Press `Ctrl+C` to stop.

#### **Detached Mode (-d)**

```
docker run -d nginx
```

Runs in the background. Returns a container ID and gives you your terminal back.

```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6
```

View the running container:

```
docker ps
```

#### **Naming Containers (–name)**

```
docker run -d --name webserver nginx
```

Without `--name`, Docker generates random names like `quirky_galileo`. Named containers are easier to manage.

#### **Viewing Running Containers**

```
docker ps
```

```
CONTAINER ID   IMAGE   COMMAND                  STATUS          PORTS     NAMES
a1b2c3d4e5f6   nginx   "/docker-entrypoint.…"   Up 2 minutes    80/tcp    webserver
```

#### **Viewing All Containers (including stopped)**

```
docker ps -a
```

Shows containers in any state: running, exited, created.

---
### Interactive Containers

Some containers are meant to be interactive - you want a shell inside.

#### **The -it Flags**

```
docker run -it ubuntu bash
```

- `-i` (interactive): Keep STDIN open
    
- `-t` (tty): Allocate a pseudo-terminal
    

Together, `-it` gives you an interactive shell session.

```
root@a1b2c3d4e5f6:/# ls
bin  boot  dev  etc  home  lib  ...
root@a1b2c3d4e5f6:/# cat /etc/os-release
PRETTY_NAME="Ubuntu 22.04.3 LTS"
root@a1b2c3d4e5f6:/# exit
```

When you exit, the container stops (the shell was the main process).

#### **Auto-Remove on Exit (–rm)**

```
docker run -it --rm ubuntu bash
```

The `--rm` flag removes the container when it exits. Useful for throwaway containers.

> _“Use --rm for quick tests. Otherwise you’ll accumulate stopped containers.”_

---
### Port Publishing (-p)

Containers have their own network. To access a containerized service from your host:

```
docker run -d -p 8080:80 nginx
```

Format: `-p HOST_PORT:CONTAINER_PORT`

- Traffic to `localhost:8080` on your machine
    
- Gets forwarded to port `80` inside the container
    

```
curl localhost:8080
```

You’ll see the nginx welcome page HTML.

#### **Multiple Ports**

```
docker run -d -p 8080:80 -p 8443:443 nginx
```

#### **Random Host Port**

```
docker run -d -p 80 nginx
docker ps   # Shows assigned port
```

Docker picks an available host port.

---
### Environment Variables (-e)

Many containers are configured through environment variables.

```
docker run -d \
  --name mydb \
  -e POSTGRES_PASSWORD=secretpassword \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_DB=myapp \
  postgres
```

The postgres image reads these variables to configure the database.

***To check the info on an env in a container***
```
docker exec container-name env
```
# Create .env file
echo "POSTGRES_PASSWORD=secret" > db.env
echo "POSTGRES_USER=myuser" >> db.env

# Use it
docker run -d --env-file db.env postgres
```

> _“Never put passwords in command history. Use --env-file or secrets management.”_

---
### **Viewing Logs**

```
# View logs
docker logs webserver

# Follow logs (like tail -f)
docker logs -f webserver

# Last 100 lines
docker logs --tail 100 webserver

# With timestamps
docker logs -t webserver
```

Press `Ctrl+C` to stop following logs.

---
### Cleaning Up

#### **Stop a Container**

```
docker stop webserver
```

Sends SIGTERM, waits, then SIGKILL. Container stops but remains on disk.

#### **Remove a Container**

```
docker rm webserver
```

Must be stopped first (or use `-f` to force).

#### **Stop and Remove in One**

```
docker rm -f webserver
```

#### **Remove All Stopped Containers**

```
docker container prune
```

#### **Remove an Image**

```
docker rmi nginx:1.28
```

Can’t remove if containers (even stopped) are using it.

#### **Remove Unused Images**

```
docker image prune
```

Removes “dangling” images (untagged, unused).

#### **Nuclear Option: Clean Everything**

```
docker system prune -a
```

Removes all stopped containers, unused networks, unused images. Use with caution.

---
### Commands Used

| Command                        | Description                       |
| ------------------------------ | --------------------------------- |
| `docker run IMAGE`             | Create and start a container      |
| `docker run -d IMAGE`          | Run in detached mode (background) |
| `docker run -it IMAGE CMD`     | Run interactively with terminal   |
| `docker run --name NAME IMAGE` | Run with specific name            |
| `docker run --rm IMAGE`        | Remove container on exit          |
| `docker run -p 8080:80 IMAGE`  | Publish port (host:container)     |
| `docker run -e VAR=val IMAGE`  | Set environment variable          |
| `docker run -v src:dst IMAGE`  | Mount volume                      |
| `docker ps`                    | List running containers           |
| `docker ps -a`                 | List all containers               |
| `docker logs CONTAINER`        | View container logs               |
| `docker logs -f CONTAINER`     | Follow logs                       |
| `docker stop CONTAINER`        | Stop a container                  |
| `docker rm CONTAINER`          | Remove a container                |
| `docker images`                | List images                       |
| `docker pull IMAGE`            | Download an image                 |
| `docker rmi IMAGE`             | Remove an image                   |
| `docker system prune -a`       | Remove all unused data            |

---
### Summary

- `docker run` pulls (if needed), creates, and starts containers
    
- `-d` runs detached (background), `-it` runs interactive
    
- `--name` gives containers meaningful names
    
- `-p HOST:CONTAINER` publishes ports to your machine
    
- `-e VAR=value` passes environment variables
    
- `docker ps` shows running containers, `-a` shows all
    
- `docker logs` shows container output
    
- `docker stop` and `docker rm` clean up containers
    
- Use `--rm` for throwaway containers to avoid clutter
---
### **Definitions**

**Detached Mode**: Running a container in the background (`-d`), freeing your terminal.

**Interactive Mode**: Running a container with STDIN attached (`-i`), typically with a TTY (`-t`) for shell access.

**Port Publishing**: Mapping a port on your host to a port in the container (`-p`), allowing external access.

**Docker Hub**: The default public registry for container images.

**Tag**: A version identifier for an image (e.g., `nginx:1.28`). `latest` is the default but should be avoided in production.

**Container ID**: Unique identifier for a container, used in commands. Can use first few characters.

**SIGTERM**: Graceful termination signal sent by `docker stop`. Applications should handle this to shut down cleanly.