Creation date: Wednesday, April 15th 2026, 3:26:11 am

### Container States

A container can be in several states:

```
                    ┌──────────┐
         create     │ Created  │
        ─────────►  └────┬─────┘
                         │ start
                         ▼
┌─────────┐ stop    ┌──────────┐ pause   ┌──────────┐
│ Exited  │◄────────│ Running  │────────►│ Paused   │
└─────────┘         └──────────┘◄────────└──────────┘
     ▲                   │        unpause
     │                   │ (process exits)
     └───────────────────┘
```

### **Viewing Container States**

```
# Only running containers
docker ps

# All containers (any state)
docker ps -a
```

```
CONTAINER ID   IMAGE     STATUS                     NAMES
a1b2c3d4e5f6   nginx     Up 2 hours                 web
b2c3d4e5f6g7   ubuntu    Exited (0) 5 minutes ago   test
c3d4e5f6g7h8   redis     Created                    cache
```

The STATUS column shows the current state.

---

### Run vs Exec vs Attach

This is one of the most important distinctions to understand.

#### **docker run - Create NEW Container**

```
docker run -it ubuntu bash
```

- Creates a **new** container from an image
    
- Starts it with the specified command
    
- Every `run` creates a new container
    

#### **docker exec - Run in EXISTING Container**

```
# First, start a container
docker run -d --name mycontainer nginx

# Then exec into it
docker exec -it mycontainer bash
```

- Runs a command in an **already running** container
    
- Spawns a **new process** inside the container
    
- Container must be running (not stopped)

#### **docker attach - Connect to EXISTING Process**

```
docker attach mycontainer
```

- Connects to the **main process** (PID 1) of a running container
    
- Shares the same STDIN/STDOUT
    
- **Dangerous**: Ctrl+C may stop the container!


> _“Use exec to get a shell. Use attach only when you need to interact with the main process.”_

#### **Comparison**

| Command  | Creates Container? | Creates Process?          | Typical Use                |
| -------- | ------------------ | ------------------------- | -------------------------- |
| `run`    | Yes                | Yes (main process)        | Start something new        |
| `exec`   | No                 | Yes (additional)          | Debug running container    |
| `attach` | No                 | No (connects to existing) | Interact with main process |

#### **Practical Example**

```
# Start nginx in background
docker run -d --name web nginx

# This runs a NEW shell process inside the running container
docker exec -it web bash
root@a1b2c3:/# ps aux
USER       PID  COMMAND
root         1  nginx: master process
root        30  bash          # <-- our exec session
root@a1b2c3:/# exit           # Just exits bash, nginx keeps running

# This attaches to nginx's main process
docker attach web
# Now we see nginx output
# Ctrl+C would STOP nginx!
# Use Ctrl+P, Ctrl+Q to detach without stopping
```

---
### Common Exec Patterns

#### **Get a Shell**

```
# Bash (if available)
docker exec -it mycontainer bash

# Shell (almost always available)
docker exec -it mycontainer sh
```
#### **Run a Single Command**

```
# Check environment
docker exec mycontainer env

# List files
docker exec mycontainer ls -la /app

# Check processes
docker exec mycontainer ps aux

# Test network
docker exec mycontainer ping -c 3 google.com
```
#### **Run as Different User**

```
docker exec -u root mycontainer whoami
docker exec -u 1000 mycontainer id
```

#### **With Environment Variables**

```
docker exec -e DEBUG=true mycontainer ./myscript
```

---
### Starting and Stopping

#### **Stop (Graceful)**

```
docker stop mycontainer
```

1. Sends SIGTERM to main process
    
2. Waits 10 seconds (configurable with `-t`)
    
3. Sends SIGKILL if still running

#### **Kill (Immediate)**

```
docker kill mycontainer
```

Sends SIGKILL immediately. Use when container won’t stop gracefully.

#### **Start (Stopped Container)**

```
docker start mycontainer
```

Starts a previously stopped container. Data in the container layer persists.

#### **Restart**

```
docker restart mycontainer
```

Equivalent to `stop` then `start`.

#### **Difference Between Run and Start**

```
# Creates NEW container, runs it
docker run nginx

# Starts EXISTING stopped container
docker start mycontainer
```

---
### Restart Policies

What happens when a container crashes or the Docker daemon restarts?

#### **Setting Restart Policy**

```
docker run -d --restart=always nginx
```

#### **Options**

| Policy           | Behaviour                                 |
| ---------------- | ----------------------------------------- |
| `no`             | Never restart (default)                   |
| `on-failure`     | Restart only if exit code is non-zero     |
| `on-failure:3`   | Restart on failure, max 3 attempts        |
| `always`         | Always restart, including on daemon start |
| `unless-stopped` | Like always, but not if manually stopped  |

#### **Practical Usage**

```
# Database should always run
docker run -d --restart=always --name postgres postgres:16

# Web server should survive reboots
docker run -d --restart=unless-stopped --name web nginx

# One-off job, retry on failure
docker run -d --restart=on-failure:5 --name worker myworker
```

#### **Updating Restart Policy**

```
docker update --restart=always mycontainer
```

> _“Use unless-stopped for services. It restarts on crash and reboot, but respects manual stops.”_

---
### Inspecting Containers

#### **Full Details**

```
docker inspect mycontainer
```

Returns JSON with everything: network, mounts, config, state.

#### **Specific Fields**

```
# Get IP address
docker inspect -f '{{.NetworkSettings.Networks.bridge.IPAddress}}' mycontainer

# Get status
docker inspect -f '{{.State.Status}}' mycontainer

# Get environment variables
docker inspect -f '{{.Config.Env}}' mycontainer
```

#### **Resource Usage**

```
# Live stats
docker stats

# One-time snapshot
docker stats --no-stream
```

```
CONTAINER ID   NAME   CPU %   MEM USAGE / LIMIT     MEM %
a1b2c3d4e5f6   web    0.00%   7.43MiB / 7.77GiB     0.09%
```

---
### Volumes for Persistent Data

Container filesystems are **ephemeral** - when the container is removed, data is lost.
#### **The Problem**

```
# Run a container, create a file
docker run -d --name mybox alpine sleep 3600
docker exec mybox sh -c "echo 'important data' > /tmp/myfile.txt"
docker exec mybox cat /tmp/myfile.txt   # Shows: important data

# Remove and recreate
docker rm -f mybox
docker run -d --name mybox alpine sleep 3600
docker exec mybox cat /tmp/myfile.txt   # Error: No such file
```

The file is gone. Container filesystems don’t survive removal.

#### **The Solution: Volumes**

```
# Create a named volume
docker volume create mydata

# Use it - mount volume to /data inside container
docker run -d \
  --name mybox \
  -v mydata:/data \
  alpine sleep 3600

# Create a file in the mounted volume
docker exec mybox sh -c "echo 'important data' > /data/myfile.txt"

# Remove and recreate with same volume
docker rm -f mybox
docker run -d \
  --name mybox \
  -v mydata:/data \
  alpine sleep 3600

# File survives!
docker exec mybox cat /data/myfile.txt   # Shows: important data
```

Data in volumes persists even when containers are removed.

#### **Volume Commands**

```
# List volumes
docker volume ls

# Inspect a volume
docker volume inspect mydata

# Remove a volume
docker volume rm mydata

# Remove unused volumes
docker volume prune
```

#### **Bind Mounts (Host Directory)**

```
# Mount current directory into container
docker run -it \
  -v $(pwd):/app \
  --name dev \
  ubuntu:24.04 bash
```

- Changes on host appear in container
    
- Changes in container appear on host
    
- Useful for development (edit on host, run in container)

#### **Volume vs Bind Mount**

| Feature    | Volume                   | Bind Mount          |
| ---------- | ------------------------ | ------------------- |
| Managed by | Docker                   | You                 |
| Location   | Docker’s data directory  | Anywhere on host    |
| Backup     | `docker volume` commands | Normal file tools   |
| Use case   | Persistent data (DBs)    | Development, config |

---
### Cleanup Commands

#### **Remove Stopped Containers**

```
# Single container
docker rm mycontainer

# All stopped containers
docker container prune
```

#### **Remove Images**

```
# Single image
docker rmi nginx:1.28

# Unused images
docker image prune

# All unused images (not just dangling)
docker image prune -a
```

#### **Remove Volumes**

```
# Single volume
docker volume rm pgdata

# Unused volumes
docker volume prune
```

#### **Remove Everything**

```
# All unused data (careful!)
docker system prune

# Including volumes and all images (very careful!)
docker system prune -a --volumes
```

#### **Disk Usage**

```
docker system df
```

```
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          5         2         2.15GB    1.2GB (55%)
Containers      3         1         62B       62B (100%)
Local Volumes   2         1         100MB     50MB (50%)
```

---
### **Summary**

- Containers have states: created, running, paused, exited
    
- `run` creates new containers, `exec` runs commands in existing ones
    
- `attach` connects to the main process - use carefully
    
- `stop` sends SIGTERM (graceful), `kill` sends SIGKILL (immediate)
    
- Restart policies (`always`, `unless-stopped`) keep services running
    
- Volumes persist data beyond container lifecycle
    
- Regular cleanup prevents disk space issues
    

---
### **Definitions**

**Container State**: The current condition of a container (created, running, paused, exited, dead).

**exec**: Run a new command/process inside an already running container.

**attach**: Connect your terminal to a container’s main process (PID 1).

**Restart Policy**: Rules for automatically restarting containers after exit or system reboot.

**Volume**: Docker-managed persistent storage that survives container removal.

**Bind Mount**: Mounting a host directory into a container, giving direct access to host files.

**SIGTERM**: Signal requesting graceful termination. Applications should clean up and exit.

**SIGKILL**: Signal forcing immediate termination. Cannot be caught or ignored.

**Ephemeral**: Temporary, not persistent. Container filesystems are ephemeral by default.
