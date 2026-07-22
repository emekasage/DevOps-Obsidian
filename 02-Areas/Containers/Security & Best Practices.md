Creation date: Sunday, April 19th 2026, 4:23:19 am

**Goal**: Build secure, production-ready container images following industry best practices.

### Why Container Security Matters

Containers provide isolation, but they’re not magic:

- Containers share the host kernel
    
- A vulnerability in your image is a vulnerability in production
    
- Misconfigured containers can expose the host
    
- Running as root in a container is dangerous

> _“Just because it’s in a container doesn’t mean it’s secure. Defense in depth still applies.”_

Two areas to focus on:

1. **Image security**: What goes INTO your image
    
2. **Runtime security**: How containers are RUN

---
### Running as Non-Root

By default, containers run as root. This is a security risk.

#### **The Problem**

```
docker run nginx:1.28
# Inside the container, nginx master process runs as root
```

If an attacker escapes the container, they’re root on the host (in some configurations).

#### **The Solution: USER Instruction**

The official nginx image includes a `nginx` user. We can use it:

```
FROM nginxinc/nginx-unprivileged:1.28

# Already runs as non-root (UID 101)
```

For custom images, create a user:

```
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y nginx && \
    rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN useradd --create-home appuser

# Switch to non-root user
USER appuser

CMD ["nginx", "-g", "daemon off;"]
```

#### **Try It: Non-Root nginx**

The official `nginx` image runs as root by default. You can’t just add `--user nginx` because the nginx user lacks permissions for cache directories.

Instead, use the official **unprivileged** nginx image:

```
# Run default nginx (runs as root)
docker run -d --name root-nginx nginx:1.28
docker exec root-nginx whoami
# Output: root

docker rm -f root-nginx

# Run unprivileged nginx (runs as non-root by default)
docker run -d --name safe-nginx nginxinc/nginx-unprivileged:1.28
docker exec safe-nginx whoami
# Output: nginx

docker rm -f safe-nginx
```

The `nginxinc/nginx-unprivileged` image is pre-configured to run as UID 101 (nginx user) with proper permissions.

> _“Use unprivileged base images when available. Build your own non-root setup when needed.”_

---
### Multi-Stage Builds

Multi-stage builds produce smaller, more secure images by separating build-time dependencies from runtime.

#### **The Concept**

Use one stage to **build/prepare**, another stage to **run**. Only the final stage becomes your image.

```
# Stage 1: Build
FROM ubuntu:24.04 AS builder
# ... install build tools, generate files ...

# Stage 2: Runtime
FROM nginx:1.28
COPY --from=builder /build/output /usr/share/nginx/html
```

#### **Practical Example: Bash Generates HTML, nginx Serves It**

Let’s create a status page generator that runs at build time, then serve the result with nginx.

Create `generate-status`:

```
#!/bin/bash
# Generate a static HTML status page

cat << EOF
<!DOCTYPE html>
<html>
<head><title>System Status</title></head>
<body>
<h1>System Status</h1>
<p>Generated: $(date)</p>
<p>Hostname: $(hostname)</p>
</body>
</html>
EOF
```

Create `Dockerfile`:

```
# Stage 1: Generate HTML
FROM ubuntu:24.04 AS builder

WORKDIR /build
COPY generate-status .
RUN chmod +x generate-status && ./generate-status > index.html

# Stage 2: Serve with nginx (unprivileged)
FROM nginxinc/nginx-unprivileged:1.28

COPY --from=builder /build/index.html /usr/share/nginx/html/index.html

EXPOSE 8080
```

#### **Try It**

```
# Build the image
docker build -t status-page:1.0.0 .

# Run it (unprivileged nginx uses port 8080)
docker run -d -p 8080:8080 --name status status-page:1.0.0

# Test it
curl localhost:8080

# For viewing in the browser, you may need a port-forward
# if your machine is not running in the same network
ssh -L 8080:localhost:8080 cato

# Cleanup
docker rm -f status
```

The final image only contains nginx and the generated HTML - no bash, no build script.

#### **Why This Matters**

Many DevOps tools (kubectl, docker, terraform) use multi-stage builds with Go to produce tiny images. The same principle applies: separate what you need to **build** from what you need to **run**.

---
### Choosing Base Images

Your base image is the foundation. Choose wisely.

#### **nginx Variants**

The official nginx image comes in several variants:

```
# Standard (Debian-based)
docker pull nginx:1.28

# Alpine-based (smaller)
docker pull nginx:1.28-alpine
```

Use `nginx:1.28-alpine` when image size matters. Use `nginx:1.28` when you need Debian compatibility.

#### **Alpine Linux**

Very popular for small images:

```
FROM alpine:3.20
RUN apk add --no-cache bash curl
```

**Pros:**

- Tiny (~5-7MB base)
    
- Has package manager (apk)
    
- Good for many use cases

**Cons:**

- Uses musl libc (not glibc) - some compatibility issues
    
- Some packages unavailable or behave differently
    
- Debugging can be harder (different tools)

---
### Best Practices Checklist

#### **Image Security**

- **Use specific image tags** (not `latest`)
    
- **Run as non-root user**
    
- **Use multi-stage builds** to minimize image size
    
- **Choose minimal base images** (slim, alpine, distroless)
    
- **Don’t install unnecessary packages**
    
- **Scan images for vulnerabilities**

#### **Dockerfile Practices**

- **Pin versions** for base images and packages
    
- **Order instructions** from least to most frequently changed
    
- **Combine RUN commands** to reduce layers
    
- **Use .dockerignore** to exclude unnecessary files
    
- **Don’t store secrets** in images (use runtime injection)
    
- **Add HEALTHCHECK** for production images

#### **Never Include in Images**

- API keys or passwords
    
- Private SSH keys
    
- `.env` files with secrets
    
- AWS credentials
    
- Database connection strings with passwords

> _“Treat your image as public. If it leaked, would you be compromised?”_

---
### Health Checks

Health checks let Docker know if your container is healthy. Essential for production.

#### **Adding a Health Check to nginx**

```
FROM nginx:1.28

# Install curl for health checks
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1

EXPOSE 80
```

#### **Options**

- `--interval`: Time between checks (default 30s)
    
- `--timeout`: Max time for check to complete
    
- `--start-period`: Grace period for startup
    
- `--retries`: Failures before marking unhealthy

#### **Try It**

```
# Build
docker build -t healthy-nginx:1.0.0 .

# Run
docker run -d -p 8080:80 --name web healthy-nginx:1.0.0

# Note: This example runs as root for simplicity.
# Section 5.8 shows the complete production setup with non-root.

# Check health status (wait 30+ seconds)
docker ps
# STATUS column shows: healthy, unhealthy, or starting

# Detailed health info
docker inspect --format='{{.State.Health.Status}}' web

# Cleanup
docker rm -f web
```

---
### Putting It All Together

Combining everything into a production-ready image: bash generates HTML, nginx serves it, with non-root user and health check.

#### **Step 1: Create the Generator Script**

Create `generate-page`:

```
#!/bin/bash
# Generate a status page with system info

cat << EOF
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Server Status</title>
  <style>
    body { font-family: sans-serif; margin: 40px; }
    .status { color: green; }
  </style>
</head>
<body>
  <h1>Server Status</h1>
  <p class="status">Online</p>
  <p>Built: $(date)</p>
  <p>Build Host: $(hostname)</p>
</body>
</html>
EOF
```

#### **Step 2: Create the Dockerfile**

```
# Stage 1: Generate static HTML
FROM ubuntu:24.04 AS builder

WORKDIR /build
COPY generate-page .
RUN chmod +x generate-page && ./generate-page > index.html

# Stage 2: Production nginx (unprivileged - runs as non-root)
FROM nginxinc/nginx-unprivileged:1.28

# Labels
LABEL org.opencontainers.image.source="https://github.com/user/repo"
LABEL org.opencontainers.image.version="1.0.0"

# Copy generated HTML
COPY --from=builder /build/index.html /usr/share/nginx/html/index.html

# Document port (unprivileged nginx uses 8080)
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/ || exit 1
```

The `nginxinc/nginx-unprivileged` image:

- Runs as UID 101 (nginx) by default
    
- Listens on port 8080 (non-root can’t bind to ports below 1024)
    
- Has proper directory permissions pre-configured

#### **Step 3: Create .dockerignore**

```
.git
*.md
Dockerfile
.dockerignore
```

#### **Step 4: Build and Run**

```
# Build
docker build -t status-server:1.0.0 .

# Run (unprivileged nginx uses port 8080)
docker run -d -p 8080:8080 --name status status-server:1.0.0

# Test
curl localhost:8080

# Check health (wait 30+ seconds)
docker ps

# Verify non-root
docker exec status whoami
# Output: nginx

# Cleanup
docker rm -f status
```

```
ssh -L 8080:localhost:8080 caspian ($hostname)
```
forward the local port to access the server on your local machine (browser)

---
### **Summary**

- **Run as non-root**: Create a user with `useradd`, switch with `USER`
    
- **Multi-stage builds**: Separate build dependencies from runtime
    
- **Choose minimal base images**: slim, alpine, or distroless
    
- **Scan for vulnerabilities**: Use Docker Scout or Trivy
    
- **Never include secrets**: Inject at runtime, not build time
    
- **Add health checks**: Let Docker monitor container health
    
- **Pin versions**: Base images, packages, and dependencies
    
- **Use .dockerignore**: Exclude unnecessary files from build

---
### **Definitions**

**Multi-Stage Build**: Dockerfile technique using multiple FROM statements to separate build-time dependencies from runtime, producing smaller images.

**Distroless**: Google’s minimal container images containing only the application and its runtime dependencies - no shell, no package manager.

**Alpine Linux**: Minimal Linux distribution (~5MB) popular for containers. Uses musl libc instead of glibc.

**Attack Surface**: The sum of potential security vulnerabilities. Smaller images = smaller attack surface.

**CVE (Common Vulnerabilities and Exposures)**: Standardized identifier for security vulnerabilities.

**Health Check**: Command Docker runs periodically to verify a container is functioning correctly.

**Non-Root User**: A Linux user without root/administrator privileges. Running as non-root limits damage from container compromise.

**nginx-unprivileged**: Official nginx image (`nginxinc/nginx-unprivileged`) pre-configured to run as non-root (UID 101). Listens on port 8080 instead of 80.

**scratch**: Special empty Docker image used as a base for statically compiled binaries.