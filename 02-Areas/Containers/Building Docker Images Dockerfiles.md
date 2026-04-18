Creation date: Saturday, April 18th 2026, 4:27:52 am

**Goal**: Create custom container images using Dockerfiles.

### What is a Dockerfile?

A Dockerfile is a text file containing instructions to build a container image. Think of it as a recipe:

```
FROM ubuntu:24.04          # Start with Ubuntu
RUN apt-get update && apt-get install -y curl  # Install packages
COPY greet /app/           # Copy files in
CMD ["/app/greet"]         # Default command
```

Each instruction creates a **layer** in the image.

---
### First Dockerfile

Create a simple bash script:

```
mkdir myapp && cd myapp
```

Create `greet`:

```
#!/bin/bash
echo "Hello from my container!"
```

Make it executable:

```
chmod +x greet
```

Create `Dockerfile`:

```
FROM ubuntu:24.04

WORKDIR /app

COPY greet .

CMD ["./greet"]
```

Build the image:

```
docker build -t myapp:1.0.0 .
```

- `-t myapp:1.0.0`: Tag the image with name and version
    
- `.`: Use current directory as build context

Run it:

```
docker run myapp:1.0.0
```

```
Hello from my container!
```

> _“Dockerfile + source code = reproducible, portable application.”_

---
### Core Dockerfile Instructions

#### **FROM - Base Image

Every Dockerfile starts with FROM. It’s your starting point.

```
FROM ubuntu:24.04    # Full-featured Linux
FROM debian:bookworm-slim  # Smaller Debian
FROM alpine:3.20     # Minimal (~5MB)
FROM scratch         # Empty, for static binaries
```

**Always use a specific tag**, not `latest`.

#### **RUN - Execute Commands**

Run commands during the build process.

```
# Install packages
RUN apt-get update && apt-get install -y curl

# Create directories
RUN mkdir -p /app/data

# Download files
RUN curl -o /app/file.txt https://example.com/file.txt
```

Each RUN creates a new layer. Combine related commands:

```
# Bad: 3 layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get clean

# Good: 1 layer
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean
```

#### **Bundling Dependencies**

Containers must include everything the application needs.

Unlike your laptop, a container image starts minimal - if you need it, you must install it.

**Try it: See what happens when a dependency is missing**

```
# Install curl without CA certificates - try HTTPS
docker run --rm ubuntu:24.04 bash -c "
  apt-get update &&
  apt-get install -y --no-install-recommends curl &&
  curl https://example.com
"
```

This fails with a certificate error:

```
curl: (77) error setting certificate file: /etc/ssl/certs/ca-certificates.crt
```

Now add the missing dependency:

```
# Install curl WITH CA certificates - HTTPS works
docker run --rm ubuntu:24.04 bash -c "
  apt-get update &&
  apt-get install -y --no-install-recommends curl ca-certificates &&
  curl https://example.com
"
```

This succeeds. The lesson: containers bundle **all** dependencies explicitly. Nothing is assumed to exist.

#### **COPY - Copy Files**

Copy files from build context into the image.

```
# Copy single file
COPY start /app/

# Copy directory contents
COPY scripts/ /app/scripts/

# Copy multiple files
COPY config.json data.json /app/

# Copy with different name
COPY config.prod.json /app/config.json
```

#### **WORKDIR - Set Working Directory**

Sets the working directory for subsequent instructions.

```
WORKDIR /app

# Now these happen in /app
COPY . .
RUN chmod +x start
CMD ["./start"]
```

Better than `RUN cd /app` because it persists.

#### **ENV - Environment Variables**

Set environment variables available at runtime.

```
ENV APP_ENV=production
ENV PORT=8080
ENV LOG_LEVEL=info
```

#### **EXPOSE - Document Ports**

Documents which ports the application uses. **Doesn’t actually publish them.**

```
EXPOSE 8080
EXPOSE 443
```

You still need `-p` when running:

```
docker run -p 8080:8080 myapp
```

#### **CMD - Default Command**

The command to run when a container starts.

```
# Exec form (preferred)
CMD ["./start"]

# Shell form
CMD ./start
```

Can be overridden at runtime:

```
docker run myapp ./other-script
```

#### **ENTRYPOINT - Fixed Command**

Like CMD, but harder to override. Good for containers that should always run a specific executable.

**Example**: A backup tool container.

Create `backup`:

```
#!/bin/bash
case "$1" in
  create)  echo "Creating backup..." ;;
  restore) echo "Restoring from backup..." ;;
  list)    echo "Listing backups..." ;;
  *)       echo "Usage: backup {create|restore|list}" ;;
esac
```

Create `Dockerfile`:

```
FROM ubuntu:24.04

WORKDIR /app
COPY backup .
RUN chmod +x backup

ENTRYPOINT ["./backup"]
CMD ["list"]
```

```
docker build -t backup:1.0.0 .

docker run backup:1.0.0              # Runs: ./backup list
docker run backup:1.0.0 create       # Runs: ./backup create
docker run backup:1.0.0 restore      # Runs: ./backup restore
```

The container IS the backup command. Arguments you pass become arguments to the entrypoint.

To get a shell, you must override the entrypoint:

```
docker run backup:1.0.0 bash                      # Runs: ./backup bash (not what you want)
docker run --entrypoint bash backup:1.0.0         # Runs: bash (shell access)
docker run -it --entrypoint bash backup:1.0.0     # Interactive shell
```

**Rule of thumb**: Use CMD for most applications.

Use ENTRYPOINT when the container IS the command.

---
### Understanding Layers and Caching

Each instruction creates a layer. Docker caches layers to speed up builds.

#### **How Caching Works**

```
FROM ubuntu:24.04            # Layer 1 (cached if image exists)
RUN apt-get update           # Layer 2 (cached if layer 1 unchanged)
RUN apt-get install -y curl jq  # Layer 3 (cached if layer 2 unchanged)
WORKDIR /app                 # Layer 4 (cached if layer 3 unchanged)
COPY . .                     # Layer 5 (rebuilds if ANY file changed)
CMD ["./start"]              # Layer 6 (rebuilds if layer 5 changed)
```

#### **Cache Invalidation**

When a layer changes, all subsequent layers rebuild.

```
# Bad: Any code change reinstalls packages
COPY . .
RUN apt-get update && apt-get install -y curl jq

# Good: Packages install early, only reinstall when Dockerfile changes
RUN apt-get update && apt-get install -y curl jq
COPY . .
```

> _“Order your Dockerfile from least frequently changed to most frequently changed.”_

#### **Viewing Layers**

```
docker history myapp:1.0.0
```

```
IMAGE          CREATED        CREATED BY                                      SIZE
a1b2c3d4e5f6   2 minutes ago  CMD ["./start"]                                 0B
b2c3d4e5f6g7   2 minutes ago  COPY . .                                        1.5kB
c3d4e5f6g7h8   5 minutes ago  RUN apt-get install -y curl jq                  15MB
...
```

---
### The Build Context

When you run `docker build`, Docker sends the **build context** (the directory you specify) to the daemon.

```
docker build -t myapp .
#                     ^ build context is current directory

```

#### **.dockerignore**

Exclude files from the build context with `.dockerignore`:

```
# .dockerignore
.git/
*.log
.env
Dockerfile
.dockerignore
*.swp
*~
```

**Why it matters:**

- Faster builds (less data to send)
    
- Smaller images (excluded files can’t be COPYed)
    
- Security (don’t accidentally include secrets)
---
### Practical Example

#### **Bash Application with Dependencies**

```
FROM ubuntu:24.04

# Install dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl jq && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Copy scripts
COPY start process .
RUN chmod +x start process

CMD ["./start"]
```

This pattern works for most shell-based applications:

1. Start with Ubuntu (or another base)
    
2. Install system packages you need
    
3. Copy your scripts
    
4. Make them executable
    
5. Set the default command
---
### Building and Tagging

#### **Basic Build**

```
docker build -t myapp .
```

#### **With Specific Tag**

```
docker build -t myapp:1.0.0 .
docker build -t myapp:latest .
```

#### **Multiple Tags**

```
docker build -t myapp:1.0.0 -t myapp:latest .
```

#### **Specify Dockerfile Location**

```
docker build -f Dockerfile.prod -t myapp:prod .
```

#### **Build Arguments**

Pass variables at build time:

```
ARG UBUNTU_VERSION=24.04
FROM ubuntu:${UBUNTU_VERSION}

ARG APP_VERSION=1.0.0
ENV APP_VERSION=${APP_VERSION}
```

```
docker build --build-arg UBUNTU_VERSION=22.04 --build-arg APP_VERSION=2.0.0 -t myapp .
```

#### **No Cache**

Force rebuild all layers:

```
docker build --no-cache -t myapp .
```

---
### Tagging Best Practices

#### **Use Semantic Versioning**

```
docker build -t myapp:1.0.0 .      # Points to latest 1.0.x
docker build -t myapp:1 .        # Points to latest 1.x.x
```

#### **Include Git Commit**

```
docker build -t myapp:$(git rev-parse --short HEAD) .
```

#### **Never Rely on Latest**

```
# Bad: What version is this?

docker pull myapp:latest

# Good: Explicit version
docker pull myapp:1.2.3
```

#### **Tag for Registries**

```
# For Docker Hub
docker build -t username/myapp:1.0.0 .

# For other registries
docker build -t ghcr.io/username/myapp:1.0.0 .
docker build -t registry.example.com/myapp:1.0.0 .
```

---
### Pushing to GitHub Container Registry

GitHub Container Registry ([ghcr.io](http://ghcr.io/ "http://ghcr.io/")) lets you store container images alongside your code.

#### **Create a Personal Access Token**

1. Go to GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
    
2. Click “Generate new token (classic)”
    
3. Select scopes:
    
    - `write:packages` (upload and download images)
        
    - `delete:packages` (optional, to remove images)
        
4. Copy the token

#### **Login to** [**ghcr.io**](http://ghcr.io/ "http://ghcr.io/")

```
# Store your token (don't commit this!)
export CR_PAT=ghp_xxxxxxxxxxxxxxxxxxxx

# Login
echo $CR_PAT | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
echo $CR_PAT | docker login ghcr.io -u emekasage --password-stdin
```

```
Login Succeeded
```

#### **Tag and Push**

Image format: `ghcr.io/OWNER/IMAGE_NAME:TAG`

```
# Tag your local image for ghcr.io
docker tag backup:1.0.0 ghcr.io/yourusername/backup:1.0.0

# Push to registry
docker push ghcr.io/yourusername/backup:1.0.0
```

#### **Build and Push**

```
docker build -t ghcr.io/yourusername/backup:1.0.0 .
docker push ghcr.io/yourusername/backup:1.0.0
```

#### **Verify Your Image**

```
# Pull your image on another machine
docker pull ghcr.io/yourusername/backup:1.0.0

# Run it
docker run ghcr.io/yourusername/backup:1.0.0 create
```

> _“Push to_ [_ghcr.io_](http://ghcr.io/ "http://ghcr.io/") _to share images with your team or deploy from CI/CD.”_

---
### Dockerfile Instructions Reference

| Instruction  | Purpose                           |
| ------------ | --------------------------------- |
| `FROM`       | Base image                        |
| `RUN`        | Execute command during build      |
| `COPY`       | Copy files from build context     |
| `ADD`        | Copy files (with URL/tar support) |
| `WORKDIR`    | Set working directory             |
| `ENV`        | Set environment variable          |
| `ARG`        | Build-time variable               |
| `EXPOSE`     | Document port                     |
| `CMD`        | Default command (overridable)     |
| `ENTRYPOINT` | Fixed command                     |
| `USER`       | Set user for subsequent commands  |
| `VOLUME`     | Create mount point                |
| `LABEL`      | Add metadata                      |

---
### **Summary**

- Dockerfiles are recipes for building images
    
- Each instruction creates a layer (cached for speed)
    
- Order matters: put frequently changing steps last
    
- Use `.dockerignore` to exclude unnecessary files
    
- Always tag with specific versions, not `latest`
    
- Use WORKDIR to set working directory
    
- Combine RUN commands to reduce layers
    
- Copy dependencies before code for better caching
    
- Push images to [ghcr.io](http://ghcr.io/ "http://ghcr.io/") to share with your team

---

### **Definitions**

**Dockerfile**: A text file containing instructions to build a container image.

**Build Context**: The directory sent to Docker when building. Files outside it can’t be COPYed.

**Layer**: A single instruction’s result in an image. Layers are cached and shared.

**Cache**: Docker stores built layers to speed up subsequent builds.

**Build Argument (ARG)**: Variable available during build, not at runtime.

**Environment Variable (ENV)**: Variable available at runtime inside the container.

**Exec Form**: Command written as JSON array `["cmd", "arg"]`. Preferred for CMD/ENTRYPOINT.

**Shell Form**: Command written as string `cmd arg`. Runs through /bin/sh.

**Container Registry**: A storage and distribution system for container images. Examples: Docker Hub, GitHub Container Registry ([ghcr.io](http://ghcr.io/ "http://ghcr.io/")).

**Personal Access Token (PAT)**: A token used to authenticate with services like [ghcr.io](http://ghcr.io/ "http://ghcr.io/") instead of a password.