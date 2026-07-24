---
title: Docker Training
date: 2026-07-24
tags:
  - docker
  - learning
publish: true
---

# 25 July, 2026

## 1. The Flow of Everything

The standard hierarchy of an application environment is structured as follows:

- **App Code**
- **Third-Party Libraries**
- **Runtime**
- **Userland**
- **Kernel**
- **Hardware**

**Key Distinction:** Containerization is simply packaging an application to run as an isolated process that shares the host's kernel. Virtualization, on the other hand, involves abstracting hardware using a Hypervisor to run a full, independent guest operating system.

## 2. Kernel Functionalities in the Operating System

Operating systems provide native features to isolate and manage resources:

- **Windows:** Uses _Job objects_ and _Object namespaces_.
- **Linux:** Uses _Cgroups_ (Control Groups) and _Namespaces_.

A **Userland library** is created to interact with and utilize these underlying kernel functionalities.

## 3. The Origins: LxC & dotCloud

### LxC (Linux Containers)

- LxC was one of the first major container runtimes.
- It provided functions to perform the kernel services mentioned above.
- It was a library heavily focused on programmers, specifically C programmers.
- Eventually, container toolkits started popping up to streamline the usage of LxC-like concepts.

### dotCloud & the Birth of Docker

- **dotCloud** was a cloud vendor that claimed to offer simpler virtualization alternatives.
- Historically, they utilized containers exclusively for Linux applications _(Note: Today, Docker supports native Windows containers and uses lightweight utility VMs to run on Mac and Windows)_.
- To package apps to run in an isolated environment, they created an internal toolkit.
- This toolkit was eventually open-sourced and named **Docker**.
- The company dotCloud eventually shut down and rebranded fully as "Docker."

## 4. Containers vs. Images

### The "Create" vs. "Starting" Concept

Creating a container is simply defining it. You create a container _from_ an image.

### What is an Image?

An image contains everything a container needs to run, split into two main parts:

1. **Metadata:** Instructions for the container runtime (e.g., how to isolate the environment, memory limits, etc.).
    - **The Primary Process (CMD):** This is the command used to start the primary process. It starts and stops everything. _The lifetime of the primary process dictates the lifetime of the container._
2. **Data:** The actual file system and data that will be present in every container built from that image.

Tools are provided to create these images, which can then be published to a **Registry**.

## 5. Standards and Architecture (OCI & CNCF)

- **OCI (Open Container Initiative):** Owned by the Linux Foundation, the OCI sets the _standard/specifications_ for what a container is, what an image is, and how isolation is handled. They maintain `runc`.
- **CNCF (Cloud Native Computing Foundation):** Hosts the tools and platforms that build upon OCI standards (like Kubernetes and `containerd`).
- **runc:** The reference implementation for the OCI runtime standard. It is the actual engine sandboxing and running the containers. Docker internally calls `runc`.
- **containerd:** A graduated CNCF project (donated by Docker). Its job is to download images from the registry, store them locally, and call `runc` to execute them.
- _Historical Note:_ Docker originally used LxC but moved away from it to build their own Go-based tool called `libcontainer` for better integration. `libcontainer` eventually evolved into `runc`. LxC is still very much alive today (e.g., LXD/Incus).

## 6. Layers of Isolation (Namespaces)

Containers provide isolation across the following Linux namespaces:

- **Process (PID)**
- **File System (Mount)**
- **Network (Net)**
- **Hostname (UTS)**
- **Inter-Process Communication (IPC)**
- **User:** _(Isolates User IDs and Group IDs, allowing a user to be `root` inside the container but an unprivileged user on the host system)._

## 7. How Images and Storage Work

### The Union File System

- An image is a collection of layers stacked in a particular order (both data and metadata layers).
- A private file system is created for the container by _unioning_ these layers together.
- **Identification:** Both images and individual layers are uniquely identified by a SHA256 hash and verified using checksums.
- **Deduplication:** Each uniquely identified layer only needs to be stored once on your machine. If multiple images share the exact same base layers, you don't have to download or store those shared layers multiple times.

### The Two Golden Rules

1. **Images are Immutable:** Once a layer becomes part of an image, it cannot be changed.
    - When a container is created and needs to write to the file system, a new layer is created called the **Container Storage Layer**. This is the _only_ writable layer.
    - **Copy-on-Write (CoW):** If an existing file in an image layer needs to be edited, the entire file is copied up to the Container Storage Layer, where the edit is performed.
    - The underlying image takes up the space. The container's Union File System is built in memory, meaning running $N$ number of containers takes up zero extra disk space (beyond their specific storage layer).
    - Container storage is created when the container is created and is destroyed when the container is deleted.
2. **Containers are Disposable:** Containers should only hold disposable, ephemeral data. You should never store permanent data directly inside a container's storage layer.

## 8. Registry & Naming Conventions

Containers and images can be identified by name and pushed to registries. The standard format is:

`registry_name/account_name/repo:tag`

- **Registry Name:** The DNS name (e.g., `docker.io`, `ghcr.io`). If omitted, it defaults to `docker.io`.
- **Account Name:** The namespace or user account. If omitted, it defaults to `library`.
- **Repo:** This is the _only_ mandatory part of the name. It describes the packaged application.
- **Tag:** Denotes the version or flavor of the image. If omitted, it defaults to `:latest`.

## 9. Docker Command Syntax Rules

- **Format:** `docker <context> <command>` (Context is always singular, e.g., `image`, not `images`).
- **Shortcut:** `docker <command>` (The command must be unique for this to work).
- **Options:** Written with single hyphens for single letters (`-p`, `-i`, `-t`) or double hyphens for full words (`--publish`, `--interactive`). Single letters can be chained together (e.g., `-it`).
- **Values:** If an option requires a value, you can separate it with a space or an equals sign (`=`).
- **Parameters:** Parameters do not have hyphens and are passed directly at the end.
- **Order:** All _options_ must come before _parameters_.

## 10. Common Commands & Outputs

### System & Image Commands

Bash

```sh
docker version
docker image ls          # Or 'docker images'
docker system df         # Shows disk usage
```

_Example Output for `docker system df`:_

Plaintext

```sh
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          39        7         2.931GB   853.3MB (29%)
Containers      7         0         3.523MB   3.523MB (100%)
Local Volumes   8         4         72.4MB    28.67kB (0%)
Build Cache     309       0         14.48GB   12.8GB
```

### Managing Images

Bash

```sh
docker image pull alpine:latest
docker image history alpine:latest
```

_Example Output for `docker image history`:_

Plaintext

```sh
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
28bd5fe8b56d   5 weeks ago   CMD ["/bin/sh"]                                 0B        buildkit.dockerfile.v0
<missing>      5 weeks ago   ADD alpine-minirootfs-3.24.1-x86_64.tar.gz /…   9.07MB    buildkit.dockerfile.v0
```

Bash

```sh
docker image pull nginx:alpine
docker image rm nginx:alpine
docker image rm alpine:latest
docker pull alpine:3.23.5
docker pull nginx:alpine

# Inspecting an image
docker image inspect alpine:3.24  # Or 'docker inspect alpine:3.24'
```

_Example Inspect CMD snippet:_

JSON

```sh
"Cmd": [
	"nginx",
	"-g",
	"daemon off;"
]
```

### Container Lifecycle Commands

Bash

```sh
docker container create <image_name>
docker container ls
docker container ls -a   # Or 'docker container ls --all'
docker container start c1
```

_Checking running processes on the host:_

Bash

```sh
ps -aef | grep nginx
```

_(Output shows host-level PIDs corresponding to containerized Nginx processes)._

### Executing Commands Inside a Container

Used heavily for debugging. You can run processes interactively (allowing I/O) or non-interactively.

Bash

```sh
docker container exec c1 hostname
docker container exec -it c1 /bin/sh  # Or '--interactive --tty'
```

_Inside the container shell (`/ # ps -aef`):_

Plaintext

```sh
PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process nginx -g daemon off;
   30 nginx     0:00 nginx: worker process
  ...
```

_Note:_ Running `kill -9 1` inside the container will fail. The Linux kernel hardcodes special protection for PID 1, ignoring the `SIGKILL` signal unless the process has registered a custom handler for it.

### Stopping & Inspecting Containers

Bash

```sh
docker container stop <container_name>
```

- **Stop Protocol:** Docker first sends a `SIGTERM`. If the process doesn't shut down gracefully, it follows up with a `SIGKILL` (`kill -9`).

Bash

```sh
docker container diff <container_name>
```

_Shows what has changed in the container's storage layer:_

- **A** = Added
- **D** = Deleted
- **C** = Changed

Plaintext

```sh
C /var/cache
A /var/cache/nginx/client_temp
C /etc/nginx/conf.d/default.conf
...
```

## 11. Core Operational Concepts

- **Process Isolation:** Starting a container creates a new process on the host operating system wrapped in a namespace. If you start 10 containers, they are all fully isolated. Each can only see the processes within its own namespace, and the PIDs are aliased (starting from 1 inside the container).
- **Container Lifecycle:** A container is kept alive solely by PID 1 inside that container. If PID 1 dies, the container stops. When a container stops, everything inside it stops.
- **Storage Persistence:** Container storage remains intact even when the container is stopped. It is only permanently wiped when the container is explicitly deleted.

## 12. Networking

- **Network Interfaces:** Containers are isolated via network interfaces. A network interface is assigned an IP address.
- Containers are not given access to every interface; a specific interface is created exclusively for that container and mapped to its network namespace.
- Processes inside the container can only see the interface mapped to their specific namespace.
- Containers on the same Docker network can communicate with one another.
- **Port Forwarding:** To expose a container's port to the outside world, you must map it during creation.
	- `docker container create --name c3 --publish 8080:80 nginx:alpine`
- **DNS Resolution:** DNS inside the container can either be explicitly set up during creation or will default to whatever DNS server the host engine provides.

%% ## Related

- [[linux-hacks]]
- [[ubuntu-terminal]]
 %%
