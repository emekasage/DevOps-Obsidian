Creation date: Saturday, April 11th 2026, 5:56:40 am

### What Are Containers?

**Goal**: Understand why containers exist and the problems they solve before using them.

### The Problem Containers Solve

Before containers, deploying software was painful:

**The Development Problem:**

- “It works on my machine” - the most dreaded phrase in software
    
- New team member joins: spend a day installing dependencies
    
- Different developers have different versions of everything
    
- Mac vs Linux vs Windows environments behave differently
    

**The Deployment Problem:**

- Build your app, then SSH into a server
    
- Install the right runtime version
    
- Install the right libraries
    
- Configure everything correctly
    
- Hope nothing conflicts with what’s already there
    
- Repeat for every server
    

> _“Containers took this whole complex sequence of events and turned it into a single command.”_

With containers:

- Development setup: `docker run` and you’re ready
    
- Deployment: same container runs everywhere
    
- “Works on my machine” becomes “works in my container” - which works everywhere
---
### The Evolution of Virtualization

Understanding how we got here helps understand why containers matter.

#### **Bare Metal (The Old Days)**

```
┌─────────────────────────────────┐
│         Application 1           │
├─────────────────────────────────┤
│         Application 2           │
├─────────────────────────────────┤
│     Binaries and Libraries      │
├─────────────────────────────────┤
│       Operating System          │
├─────────────────────────────────┤
│          Hardware               │
└─────────────────────────────────┘
```

Applications run directly on physical hardware.

**Problems:**

- **Dependency hell**: App 1 needs Python 3.8, App 2 needs Python 3.11 - conflict
    
- **Low utilization**: 64-core server running two small apps
    
- **Large blast radius**: One app crashes, takes down others
    
- **Slow provisioning**: Days or weeks to get new hardware
    
- **Slow startup**: Minutes to reboot a physical server
    

#### **Virtual Machines (The Improvement)**

```
┌──────────────┐ ┌──────────────┐
│     App 1    │ │     App 2    │
├──────────────┤ ├──────────────┤
│   Bins/Libs  │ │   Bins/Libs  │
├──────────────┤ ├──────────────┤
│   Guest OS   │ │   Guest OS   │
└──────────────┘ └──────────────┘
├─────────────────────────────────┤
│          Hypervisor             │
├─────────────────────────────────┤
│       Host OS (optional)        │
├─────────────────────────────────┤
│          Hardware               │
└─────────────────────────────────┘
```

A **hypervisor** creates virtual machines, each with its own OS.

**Improvements:**

- No more dependency conflicts - each VM isolated
    
- Better utilization - carve up big servers into smaller VMs
    
- Smaller blast radius - one VM crashing doesn’t affect others
    
- Faster provisioning - minutes, not days (think AWS EC2)
    

**Remaining Problems:**

- Each VM runs a complete OS - gigabytes of overhead
    
- Startup time still minutes (booting an OS)
    
- Heavy resource usage - each OS consumes CPU/memory
    

#### **Containers (Where We Are Now)**

```
┌──────────────┐ ┌──────────────┐
│ Container 1  │ │ Container 2  │
├──────────────┤ ├──────────────┤
│   Bins/Libs  │ │   Bins/Libs  │
└──────────────┘ └──────────────┘
├─────────────────────────────────┤
│       Container Runtime         │
├─────────────────────────────────┤
│       Operating System          │
├─────────────────────────────────┤
│    Hardware (or VM underneath)  │
└─────────────────────────────────┘
```

Containers share the host OS kernel. No guest OS per container.

**Improvements over VMs:**

- **Lightweight**: Megabytes, not gigabytes
    
- **Fast startup**: Seconds, not minutes
    
- **Better utilization**: Pack many more containers than VMs
    
- **Development friendly**: Light enough to run on your laptop
    

**Trade-off:**

- Less isolation than VMs (shared kernel)
    
- Linux containers only run on Linux (or a Linux VM on Mac/Windows)
---
### Containers vs Virtual Machines

> _“Containers are light enough that we actually use them during development. That’s the game changer.”_

In practice, containers often run inside VMs in the cloud. You get VM-level isolation between tenants, with container efficiency within your allocation.

---
### What Is a Container Image?

A **container image** is a standardized package containing everything needed to run an application:

```
┌─────────────────────────────────┐
│      Your Application Code      │
├─────────────────────────────────┤
│   Application Dependencies      │
│   (npm packages, pip modules)   │
├─────────────────────────────────┤
│      Runtime (Node, Python)     │
├─────────────────────────────────┤
│    Base OS Libraries (minimal)  │
└─────────────────────────────────┘
```

Think of it as a snapshot of a filesystem with everything your app needs.

#### **Image vs Container**

This distinction is important:

- **Image**: The blueprint (like a class in programming)
    
- **Container**: A running instance (like an object)
    

```
# One image, multiple containers
docker run nginx    # Container 1
docker run nginx    # Container 2
docker run nginx    # Container 3
```

Same image, three separate running containers.

---
### How Linux Enables Containers

Containers aren’t magic - they’re built on Linux kernel features.

Docker makes these features easy to use.

#### **Namespaces (Isolation)**

Namespaces make a process think it’s alone on the system.

A container’s process might be PID 1 inside the container but PID 4827 on the host. It can’t see other processes.

Let’s try it:

```
sudo unshare --fork --pid --mount-proc bash

root@cato:/home/user# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   4720  3712 pts/1    S    07:45   0:00 bash
root           9  0.0  0.0   8164  3840 pts/1    R+   07:45   0:00 ps aux

root@cato:/home/user# pstree
bash───pstree

root@cato:/home/user# uname -a
Linux cato 6.8.0-1042-raspi #46-Ubuntu SMP PREEMPT_DYNAMIC Tue Oct 14 18:59:27 UTC 2025 aarch64 aarch64 aarch64 GNU/Linux

root@cato:/home/user# ps -p 1
    PID TTY          TIME CMD
      1 pts/1    00:00:00 bash
```

#### **Cgroups (Resource Limits)**

Control groups limit what resources a container can use:

- CPU: “This container gets max 50% of one core”
    
- Memory: “This container gets max 512MB”
    
- Disk I/O: “This container gets max 100MB/s read”
    

Without cgroups, one container could starve others of resources.

#### **cgroups: resource limits**

Create a cgroup by making a directory:

```
sudo mkdir /sys/fs/cgroup/julius
```

Set a 10MB memory limit:

```
echo 10000000 | sudo tee /sys/fs/cgroup/julius/memory.max
```

Add current shell to the cgroup:

```
echo $$ | sudo tee /sys/fs/cgroup/julius/cgroup.procs
```

Now try to use 15MB of memory:

```
head -c 15M /dev/zero | tail
```

Boom! Killed. The cgroup enforced our 10MB limit.

This head command reads 15MB of zeros from /dev/zero, which is an infinite stream of null bytes.

Clean up:

```
sudo rmdir /sys/fs/cgroup/julius
```

#### **Union File Systems (Efficiency)**

Images are built in **layers**. Union file systems (like OverlayFS) stack these layers.

The best explanation:

[https://www.grant.pizza/blog/overlayfs/](https://www.grant.pizza/blog/overlayfs/ "https://www.grant.pizza/blog/overlayfs/")

```
┌─────────────────────────────────┐
│   Container Layer (read-write)  │  ← Your changes at runtime
├─────────────────────────────────┤
│   App Code Layer (read-only)    │
├─────────────────────────────────┤
│   Dependencies Layer (read-only)│
├─────────────────────────────────┤
│   Base Image Layer (read-only)  │
└─────────────────────────────────┘
```

**Why layers matter:**

- Shared layers are cached - download once, use many times
    
- 10 containers from the same image share base layers
    
- Only differences are stored per container
---
### What Is Docker?

Docker is a platform that makes containers easy to use. It wraps those Linux kernel features into simple commands.

#### **Docker Architecture**

```
┌─────────────────────────────────────────────────────────┐
│                      Your Machine                       │
│                                                         │
│  ┌─────────────┐         ┌─────────────────────────┐   │
│  │ Docker CLI  │ ──────► │    Docker Daemon        │   │
│  │ (docker)    │   API   │    (dockerd)            │   │
│  └─────────────┘         │                         │   │
│                          │  ┌─────┐ ┌─────┐        │   │
│                          │  │ C1  │ │ C2  │ ...    │   │
│                          │  └─────┘ └─────┘        │   │
│                          └─────────────────────────┘   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

- **Docker CLI** (`docker`): The command you type
    
- **Docker Daemon** (`dockerd`): Background service managing containers
    
- **Docker Desktop**: GUI + Linux VM (for Mac/Windows)
    

When you type `docker run`, the CLI sends a request to the daemon, which does the actual work.

#### **Docker Desktop vs Docker Engine**

**Docker Desktop** (Mac, Windows, Linux GUI):

- Includes everything: CLI, daemon, GUI, Kubernetes
    
- Runs Linux containers in a lightweight VM (Mac/Windows)
    
- Free for personal use, paid for large enterprises
    

**Docker Engine** (Linux only):

- Just the daemon and CLI
    
- Runs containers natively (no VM needed)
    
- Always free and open source

#### **Alternatives to Docker**

Podman

Rancher Desktop

---
### Summary

- Containers solve the “works on my machine” problem
    
- Evolution: Bare metal → VMs → Containers (each solving previous problems)
    
- Containers share the host kernel - lighter than VMs but less isolated
    
- Container images are standardized packages with everything an app needs
    
- Linux kernel features enable containers: namespaces (isolation), cgroups (limits), union FS (efficiency)
    
- Docker makes these features easy to use through simple commands
    
- OCI standards ensure portability - Docker images work everywhere
---
### Definitions

**Container**: A lightweight, isolated process running with its own filesystem, created from a container image. Shares the host kernel.

**Container Image**: A read-only package containing application code, dependencies, and a minimal OS filesystem. The blueprint for containers.

**Docker**: A platform for building, running, and managing containers. Makes Linux kernel features accessible through simple commands.

**Docker Daemon**: The background service (`dockerd`) that manages containers, images, and networks.

**Hypervisor**: Software that creates and manages virtual machines (VMware, Hyper-V, KVM).

**Namespace**: Linux kernel feature providing process isolation. Containers use namespaces to appear isolated.

**Cgroup (Control Group)**: Linux kernel feature that limits and monitors resource usage for processes.

**Union File System**: Filesystem that layers multiple directories into one view. Enables efficient image storage and sharing.

**OCI (Open Container Initiative)**: Industry standard for container image format and runtime, ensuring images work across different tools.

**Docker Desktop**: Docker’s application for Mac/Windows/Linux with GUI, CLI, and built-in Kubernetes.

**Docker Engine**: The core Docker daemon and CLI, available for Linux. Runs containers natively.



