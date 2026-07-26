---
publish: true
title: Docker Training
created: 2026-07-24
modified: 2026-07-26T19:01:04.929+05:30
tags:
  - docker
  - learning
---

# 24 July, 2026

## 1. Foundations

The standard hierarchy of an application environment, from top to bottom:

```
App Code
Third-Party Libraries
Runtime
Userland
Kernel
Hardware
```

**Containerization vs. Virtualization**

|           | Containerization                                                     | Virtualization                                                                  |
| --------- | -------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Mechanism | Packages an app as an isolated process sharing the **host's kernel** | Uses a **Hypervisor** to abstract hardware and run a full, independent guest OS |
| Overhead  | Lightweight                                                          | Heavier                                                                         |

### Hardware

- Most traditional servers and older PCs run on **`amd64`** (also known as x86\_64, built by Intel and AMD). In contrast, Apple's M-series chips (M1, M2, M3) and modern cloud processors (like AWS Graviton) run on **`arm64`**. These architectures process instructions entirely differently at the hardware level, meaning a binary compiled for one cannot natively run on the other.
- `eclipse-temurin:8-jre-alpine`: This specific image causes problems on Apple chips because it is an older version of Java (Java 8) running on a highly stripped-down Linux distribution (Alpine). The publishers of this image never created an `arm64` version of this specific combination.
- To bridge this hardware divide, modern containers are usually built as **multi-arch images**. When a publisher pushes a multi-arch image to Docker Hub, they bundle several different architecture builds under a single tag.
- When you type `docker pull`, your machine checks the image's manifest and automatically downloads the exact version built for your hardware. However, because compiling and testing for multiple architectures takes time and resources, many older, legacy, or community-maintained images are never updated to include an `arm64` build.

### Kernel-Level Isolation Primitives

Operating systems expose native features for isolating and managing resources:

- **Windows:** Job objects and Object namespaces
- **Linux:** Cgroups (Control Groups) and Namespaces

A **userland library** sits on top of these kernel primitives so higher-level tools don't have to call the kernel directly.

## 2. History: LxC, dotCloud & Docker

### LxC (Linux Containers)

- One of the first major container runtimes.
- Provided the functions needed to drive the kernel isolation primitives above.
- Aimed squarely at C programmers — a low-level, developer-facing library.
- Higher-level toolkits eventually emerged to make LxC-style isolation easier to use.

### dotCloud → Docker

- **dotCloud** was a cloud vendor offering a simpler alternative to full virtualization.
- Historically limited to Linux applications. _(Today Docker also supports native Windows containers, and uses lightweight utility VMs to run on macOS and Windows.)_
- dotCloud built an internal toolkit for packaging apps into isolated environments.
- That toolkit was open-sourced as **Docker**.
- dotCloud (the company) eventually shut down and rebranded entirely as Docker, Inc.

### Runtime Lineage

- Docker originally built on **LxC**.
- Docker later moved off LxC to build its own Go-based tool, **`libcontainer`**, for tighter integration.
- `libcontainer` evolved into **`runc`**.
- LxC is still actively developed today (e.g., LXD/Incus).

## 3. Images vs. Containers

**Create vs. Start:** Creating a container just defines it — a container is created _from_ an image; starting it launches the primary process.

### What's in an Image?

| Component    | Description                                                                                               |
| ------------ | --------------------------------------------------------------------------------------------------------- |
| **Metadata** | Instructions for the container runtime: isolation rules, memory limits, the primary process (`CMD`), etc. |
| **Data**     | The actual file system contents present in every container built from the image                           |

**The Primary Process (CMD)** starts and stops everything in the container. The lifetime of the primary process **is** the lifetime of the container.

Images are built with tooling and published to a **registry**.

## 4. Standards: OCI & CNCF

| Body                                         | Role                                                                                                                                        |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **OCI** (Open Container Initiative)          | Owned by the Linux Foundation. Defines the _standard_ for what a container is, what an image is, and how isolation works. Maintains `runc`. |
| **CNCF** (Cloud Native Computing Foundation) | Hosts tools and platforms built on top of OCI standards (e.g., Kubernetes, `containerd`).                                                   |

**Key components:**

- **`runc`** — the reference implementation of the OCI runtime spec; the actual engine that sandboxes and runs containers.
- **`containerd`** — a graduated CNCF project (donated by Docker). Downloads images from a registry, stores them locally, and invokes `runc` to execute them.

> **Note on the call chain:** Docker doesn't call `runc` directly — it calls `containerd`, which in turn calls `runc`. ("Docker → containerd → runc.")

## 5. Namespaces & Isolation

Containers isolate the following Linux namespaces:

- **Process (PID)**
- **File System (Mount)**
- **Network (Net)**
- **Hostname (UTS)**
- **Inter-Process Communication (IPC)**
- **User** — isolates UIDs/GIDs, letting a process be `root` inside the container while mapping to an unprivileged user on the host.

### Process Isolation in Practice

- Starting a container creates a new host process wrapped in a namespace.
- Ten containers = ten fully isolated environments; each only sees processes within its own namespace, and PIDs are aliased starting from 1 inside each container.
- A container is kept alive solely by its **PID 1**. If PID 1 dies, everything else in the container stops too.
- **`kill -9 1` fails inside a container.** Per Linux's PID-namespace semantics, a namespace's PID 1 ignores unhandled signals — including `SIGKILL` — _when the signal originates from within the same PID namespace_. (A `SIGKILL` sent from **outside** the namespace, e.g. `docker kill`, still works normally.)

## 6. Image Storage: The Union File System

- An image is a stack of layers (data + metadata) combined in order.
- A private file system is created per-container by **unioning** these layers together.
- Images and layers are each uniquely identified by a **SHA256** hash and verified via checksum.
- **Deduplication:** each unique layer is stored once on disk; images sharing base layers don't re-download or re-store them.

### Two Golden Rules

**1. Images are immutable.**

- Once a layer is part of an image, it never changes.
- Container writes go to a new, container-specific **Container Storage Layer** — the only writable layer.
- **Copy-on-Write (CoW):** editing a file that exists in an image layer copies the whole file up to the Container Storage Layer first, then edits it there.
- The Union File System itself lives in memory — running _N_ containers from the same image costs zero extra disk space beyond each container's own storage layer.
- Container storage is created with the container and destroyed with it.

**2. Containers are disposable.**

- Only ephemeral data belongs in a container's storage layer — never store permanent data there.

## 7. Registry & Naming Conventions

```
registry_name/account_name/repo:tag
```

| Segment         | Default if omitted         |
| --------------- | -------------------------- |
| `registry_name` | `docker.io`                |
| `account_name`  | `library`                  |
| `repo`          | _(mandatory — no default)_ |
| `tag`           | `latest`                   |

## 8. Docker CLI Syntax

- **Format:** `docker <context> <command>` — context is always singular (`image`, not `images`).
- **Shortcut:** `docker <command>` works when the command name is unique across contexts.
- **Options:** single-hyphen for single letters (`-p`, `-i`, `-t`), double-hyphen for full words (`--publish`, `--interactive`). Single-letter flags can be chained (`-it`).
- **Values:** separate an option from its value with a space or `=`.
- **Parameters:** no hyphens, passed directly at the end of the command.
- **Order:** all options come before parameters.

## 9. Command Reference

### System & Image Info

```sh
docker version
docker image ls          # or: docker images
docker system df         # disk usage summary
```

Example `docker system df` output:

```
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          39        7         2.931GB   853.3MB (29%)
Containers      7         0         3.523MB   3.523MB (100%)
Local Volumes   8         4         72.4MB    28.67kB (0%)
Build Cache     309       0         14.48GB   12.8GB
```

### Managing Images

```sh
docker image pull alpine:latest
docker image history alpine:latest
docker image pull nginx:alpine
docker image rm nginx:alpine
docker image rm alpine:latest
docker pull alpine:3.23.5
docker pull nginx:alpine

# Inspect an image
docker image inspect alpine:3.24     # or: docker inspect alpine:3.24
```

Example `docker image history` output:

```
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
28bd5fe8b56d   5 weeks ago   CMD ["/bin/sh"]                                 0B        buildkit.dockerfile.v0
<missing>      5 weeks ago   ADD alpine-minirootfs-3.24.1-x86_64.tar.gz /…   9.07MB    buildkit.dockerfile.v0
```

Example `inspect` CMD snippet:

```json
"Cmd": [
    "nginx",
    "-g",
    "daemon off;"
]
```

### Container Lifecycle

```sh
docker container create <image_name>
docker container ls
docker container ls -a           # or: docker container ls --all
docker container start c1
```

Checking host-level processes:

```sh
ps -aef | grep nginx
# Output shows host PIDs corresponding to containerized nginx processes
```

### Executing Commands Inside a Container

Useful for debugging — commands can run interactively (with I/O) or non-interactively.

```sh
docker container exec c1 hostname
docker container exec -it c1 /bin/sh   # or: --interactive --tty
```

Inside the container shell:

```
/ # ps -aef
PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process nginx -g daemon off;
   30 nginx     0:00 nginx: worker process
```

### Stopping & Inspecting Containers

```sh
docker container stop <container_name>
```

**Stop protocol:** Docker sends `SIGTERM` first; if the process doesn't shut down gracefully, it follows up with `SIGKILL`.

```sh
docker container diff <container_name>
```

Shows what changed in the container's storage layer:

- `A` = Added
- `D` = Deleted
- `C` = Changed

```
C /var/cache
A /var/cache/nginx/client_temp
C /etc/nginx/conf.d/default.conf
```

> A large `diff` output is a red flag — it usually means data is being written to the container's temporary storage layer instead of a permanent mount.

### Storage Persistence Notes

- Container storage survives a stop, but is permanently wiped on delete.
- Logs (STDOUT/STDERR from PID 1) are stored separately from container storage and mounted storage, and persist across stops but not deletes.

## 10. Networking

- Containers are isolated via **network interfaces**, each assigned its own IP.
- Each container gets a dedicated interface mapped to its own network namespace — it can't see other interfaces.
- Containers on the same Docker network can talk to each other directly.
- **Port forwarding** exposes a container port to the outside world, set at creation time: \`shdocker container create --name c3 --publish 8080:80 nginx:alpine
- **DNS** inside a container can be configured explicitly at creation, or defaults to whatever DNS server the host engine provides.

# 25 July, 2026

## 1. Persistent Storage

For durable storage, mount external storage into the container.

- **Shadowing:** mounting onto a directory that already has data hides (shadows) the existing container data.
- **OCI standard:** mounts can target either a directory or a single file.
- **Immutability of mounts:** all mounts for a container are defined at creation time (`docker create` / `docker run`) and **cannot** be added or removed while the container is running.
- **Bypassing the UFS:** writes to a mount path go straight to the external storage, bypassing the container's writable layer entirely.
- **Tracking bloat:** `docker container diff <name>` only shows changes to the container's own UFS layer — a large diff signals data is landing in temporary storage instead of a mount.

## 2. Bind Mounts

Maps a specific host directory or file to a directory or file inside the container.

- **Always use absolute paths** for the host side.
- **Windows note:** Docker Desktop mounts the Windows file system into its hidden Linux VM first, then binds from the VM into the container. This automatic translation does **not** happen on native Linux/server hosts.

### Syntax

Preferred (`--mount`):

```sh
docker container create --name v1 \
  --mount type=bind,source=/home/tanuv/projects/docker-training/my-dir-1,target=/data \
  nginx:alpine
```

Legacy (`-v`):

```sh
docker container create --name v1 \
  -v /home/tanuv/projects/docker-training/my-dir-1:/data \
  nginx:alpine
```

### Inspecting a Bind Mount

```json
"Mounts": [
  {
    "Type": "bind",
    "Source": "/home/tanuv/projects/docker-training/my-dir-1",
    "Target": "/data"
  }
]
```

### Aside: `exec -it` flags

```sh
docker container exec -it v1 /bin/sh
```

- `-t` allocates a pseudo-TTY.
- `-i` keeps stdin open (`OpenStdin: true`).
- Alpine is a common debugging image because its userland is guaranteed to include `sh`, even when the target image doesn't.

## 3. Volume Mounts

A **volume** is an isolated unit of external storage fully managed by Docker — could be a local disk directory, NFS, or cloud storage depending on the **volume driver** in use.

- **Default driver:** `local` (built in).
- **Protection:** Docker won't let you delete a volume that's still referenced by a container.
- **Concurrency:** one volume can be mounted into multiple containers at once — but Docker does **not** manage file locking. Preventing corruption is the application's responsibility.

### Shadowing Rule for Volumes

- Mount volumes onto _empty_ directories where possible.
- If you mount an **empty** volume onto a directory that already has files, Docker (by default) copies the existing container files into the volume on creation. **Don't rely on this** — other engines (e.g., Podman) may behave differently.

### Commands

```sh
docker volume ls
```

Create automatically at container creation (read-only example):

```sh
docker container create -v <volume_name>:<container_directory>:ro image:tag
```

Or via `--mount`:

```sh
docker container create \
  --mount type=volume,source=<volume_name>,target=<container_directory>,readonly \
  image:tag
```

Remove a volume:

```sh
docker volume rm volume-1
```

> Removing a volume still in use returns: `Error response from daemon: remove volume-1: volume is in use`

## 4. TMPFS (Temporary File System)

Mounts data directly into host **RAM** rather than disk. **Not persistent.**

- **Use cases:** image processing scratch space, temporary caches, sensitive secrets/keys that should never touch a physical disk.
- **Performance:** very fast, since it's memory-backed.
- **The `-v` flag cannot be used for tmpfs** — you must use `--mount` (or the tmpfs shorthand).

### Syntax

```sh
docker container create --mount type=tmpfs,target=<container_directory> image:tag
```

Shorthand: `--tmpfs <container_directory>`

> A read-only tmpfs mount (`--mount tmpfs-options=ro`) is technically possible but practically useless — since tmpfs always starts empty, a read-only mount means the app can never write to it _or_ meaningfully read from it.

## 5. Storage Best Practices

- The application/Dockerfile author should decide the target mount directory — not the end user.
- Images should **advertise** their expected persistent-data location via the Dockerfile's `VOLUME` instruction.
- Discover advertised mount points via: `docker image inspect <image_name>:<tag>`

## 6. Docker Hub & Image Tagging

- **Official Images** are maintained by the software's original creators, in collaboration with Docker, with quality guarantees from Docker.
- **Docker Hardened Images** are a separate, security-focused offering.
- Registry: [hub.docker.com](https://hub.docker.com/)

### Partial Version Tags

Multiple tags can point to the same image; partial versions point to the latest matching full version (e.g., `4.3` currently resolves to `4.3.4`, but could later move to `4.3.5`):

```
4.3.4, 4.3, 4, latest
4.2.9, 4.2
4.2.9-management, 4.2-management
4.2.9-alpine, 4.2-alpine
```

- Images can also be pulled by their content hash for full reproducibility.
- **Userland evolution:** many official images started on Ubuntu (feature-rich but bloated) → moved to Debian (lighter) → moved to Alpine (minimal) as the default/optional flavor. If a tag doesn't specify a flavor, it's usually **Debian** by default.

## 7. Container Creation Workflow (CSCC)

General workflow and concepts for spinning up _any_ container — not specific to Docker Hub, but this is where the notes originally introduced them.

### `run` vs. `create` + `start`

```sh
docker run -d --hostname my-rabbit --name some-rabbit rabbitmq:3
```

- `docker run` = `create` + `start` + `attach`.
- `-d` runs detached.
- Reattach later with: `docker container attach <container_name>`

### The "CSCC" Mental Checklist

When setting up a new container, decide in this order:

1. **C**ommand — which image/version, and any command override
2. **S**torage — where persistent data lives
3. **C**ommunication — port forwarding
4. **C**onfiguration — environment variables, etc.

### Environment Variables

- Managed by the OS, per-process — each process can have its own set.
- Set at process start, or modified while running.
- A child process can optionally **inherit** its parent's environment variables.
- Set at container creation with `-e <variable>=<value>` or `--env <variable>=<value>`.

## 8. RabbitMQ

_(Notes pending — revisit.)_

## 9. PostgreSQL

- `bookworm` and `trixie` are Debian release codenames used in Postgres image tags.

### Creating a Container

Basic one-shot run:

```sh
docker run --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres
```

Basic creation (no config, mainly for inspection):

```sh
docker container create postgres
```

Full example with a named volume:

```sh
docker container create --name pg1 \
  --mount type=volume,source=pg1vol,target=/var/lib/postgresql \
  -e POSTGRES_PASSWORD=something \
  -e POSTGRES_USER=admin \
  postgres:latest
```

- To find where data is stored, check the image's advertised `-v`/`--mount` target.

### Overriding `CMD`

A container's `CMD` can be overridden by appending a command after the image name — useful for connecting a throwaway `psql` client to a running server on the same network:

```sh
docker run -it --rm --network some-network postgres psql -h some-postgres -U postgres
```

### Exit Codes & Logs

- **Exit codes:** `0` = clean exit; anything non-zero = crash (`127` = killed).
- **Logs:** `docker container logs <container_name>`
  - PID 1 is expected to write logs to STDOUT/STDERR; logs persist across stops but are deleted with the container. Log storage is distinct from container storage and mounted storage — managed separately by the container engine. _(Open question: how are logs handled with multiple concurrent processes inside one container?)_

### Inspecting the Container's IP

`docker container inspect pg1`:

```json
"Networks": {
    "bridge": {
        "IPAMConfig": null,
        "Links": null,
        "Aliases": null,
        "DriverOpts": null,
        "GwPriority": 0,
        "NetworkID": "efe83191686201734ebf3a632c26279c678b63a0d058c9f039900dfd716f994b",
        "EndpointID": "0ada15977653aa09eddfa111985d490e7ae85fece48cb8932914250969ae36f6",
        "Gateway": "172.17.0.1",
        "IPAddress": "172.17.0.2",
        "MacAddress": "72:44:c2:1a:69:9e",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "DNSNames": null
    }
}
```

## 10. MySQL

```sh
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```

- Data directory inside the container: `/var/lib/mysql`
- Default port: `3306` _(verify against current docs for the specific tag)_

## 11. Adminer

```sh
docker container create --name ad1 --publish 18080:8080 adminer
```

- Adminer connects to Postgres/MySQL internally over the Docker network.
- Port forwarding is only needed for **external → container** access; container-to-container traffic stays on the internal Docker network.

## 12. Docker Networks

Docker networks provide a software-defined way to connect containers and simulate a network experience, making them act as separate machines on a shared network.

To list all available networks, use:

```bash
docker network ls
```

Networking is handled by **Network Drivers**, which are built-in software components that use various techniques (like tunnel tapping) to ensure connectivity.

### Network Drivers

There are three default networks — `bridge`, `host`, and `none` — plus the MACVLAN/IPVLAN drivers for custom networks:

1. **Bridge Network (Default)**
   - **Behavior:** When a container is created without specifying a network, it automatically connects to the default bridge network (`docker0`).
   - **Accessibility:** Containers on the same bridge network can communicate internally. However, they cannot be accessed from the outside host directly; you must use **port forwarding** for external access.
   - **Verification:** You can verify a container's network details by running `docker container inspect <container_name>`.

2. **Host Network**
   - `host` means do not isolate this container's network — it can access all networks the host has.
   - Example: `docker container create --name h1 --network host nginx:alpine`

3. **None Network**
   - `none` means no networking at all is attached to the container.

4. **MAC VLAN & IP VLAN Methods**
   - These drivers allow containers to appear as physical devices on your network, meaning they can be accessed directly from outside the host without standard port forwarding.

## 13. IP Address Management (IPAM)

Network properties include an **IPAM** section, which defines the range of IP addresses allocated to the network.

- **IPv4 Addresses:** Composed of 32 bits (4 bytes), divided into four 8-bit sections.
- **Reserved Addresses:** Generally, `0` is reserved for the network address, and `255` is used for broadcast.
- **CIDR & Subnets:** IP addresses are accompanied by a subnet mask using CIDR notation (e.g., `/16` or `/24`), which dictates the network size.

## 14. Custom Networks and DNS

**Best Practice:** _Always create your own custom network._

When you create a custom network, Docker automatically runs an internal **DNS server**. This DNS server maps container names to their internal IP addresses, allowing containers to communicate using their names instead of IPs.

- **Isolating Environments:** You can place frontend containers on one network and backend containers on another. If they need to communicate, you can create a shared connecting network for both.
- **Dynamic Connectivity:** Unlike some settings, network metadata and connections can be changed dynamically while containers are running.

### Common Network Commands

**Create a Custom Network:**

```bash
docker network create --driver=bridge --subnet 192.168.125.0/24 pnet1
```

**Create a Container on a Custom Network:**

```bash
docker container create --name pn1 --network pnet1 nginx:alpine
```

**Connect a Running Container to a Network:**

```bash
docker network connect <network_name> <container_name>
```

### Network Aliases (Load Balancing)

```bash
docker container create --name pn4 --network pnet1 --network-alias webserver nginx:alpine
```

- `--network-alias` gives the container another name on the network.
- If multiple containers share the same network alias, and something hits that alias, Docker automatically load-balances requests across them.
- Use case: one way of achieving load balancing and redundancy is giving the same network alias to multiple containers.

## 15. Inspecting a Network (Example)

Running `docker network inspect bridge` provides detailed information about the network, including its IPAM configuration and connected containers.

```json
[
  {
    "Name": "bridge",
    "Id": "9025e0d8...",
    "Scope": "local",
    "Driver": "bridge",
    "IPAM": {
      "Config": [
        {
          "Subnet": "172.17.0.0/16",
          "Gateway": "172.17.0.1"
        }
      ]
    },
    "Containers": {
      "10a939...": {
        "Name": "ad1",
        "IPv4Address": "172.17.0.3/16"
      },
      "729d19...": {
        "Name": "n2",
        "IPv4Address": "172.17.0.5/16"
      },
      "8d6264...": {
        "Name": "n1",
        "IPv4Address": "172.17.0.4/16"
      },
      "dbcb8c...": {
        "Name": "pg1",
        "IPv4Address": "172.17.0.2/16"
      }
    }
  }
]
```

## 16. Docker Compose

**Evolution:** Previously, `docker-compose` was a separate, standalone tool. It has now been integrated directly into Docker as `docker compose`.

**How it Works:**

- Docker Compose reads **manifest files** (typically `docker-compose.yaml`).
- Based on the configuration in these files, it orchestrates the creation of multiple containers, volumes, and networks simultaneously.
- It provides a standardized solution for defining and managing multi-container applications, a feature commonly offered by all modern container engines.
- Docker Compose does **not** put any services on the default bridge network — it creates a new one for the project.

### YAML Format

- In YAML, what we are defining are documents or objects.
- We can define one document/object or multiple documents/objects.
- A YAML document consists of properties/fields and their values. Their syntax is `field_name: value`. Assume no space before `field_name` and no space after `field_name`. Must begin with a letter and end with a letter. In between we can have alphanumeric, dashes, etc.
- Three kinds of values:
  - `scalar`: A scalar value is written by writing one space after the colon, and then writing the value. E.g. `field_name: value`. No decorations needed for value. YAML will figure it out. Either single quotes, or double quotes. Single quotes takes the entire string as it is. Double quotes can allow special cases like escape tabs, etc. `yes` and `no` are valid boolean.

  - `lists`: Each value is called an element. All indentations should be spaces. Cannot handle tabs. And values should be aligned with the field. They can be written like:

    ```yml
    array_field_1:
      - value1
      - value2
      - value3
    ```

  - `map`: A field contains objects with properties. The sub properties are indented two spaces. Each level is two spaces of indentation.

    ```yml
    map_field_1:
      sub_property_1: value1
      sub_property_2: value2
      sub_property_3:
        sub_sub_property_1: value11
        sub_sub_property_2: value22
    ```

  - Nested array:

    ```yml
    array_field_1:
      - prop1: value1
        prop2: value2
        prop3: value3
      - prop1: value1
        prop2: value2
        prop3: value3
    ```

### Docker Compose File

A compose file has 3 top-level properties: `networks`, `volumes`, and `services` (containers).

**How the `volumes` property works** (on a service): lists `named-volume:mount-path` pairs, mounting each named volume declared in the top-level `volumes:` block into the container:

```yml
volumes:
  - dbvol:/var/lib/postgresql
```

**How the `networks` property works** (on a service): lists which top-level networks, declared in the `networks:` block, the service should attach to:

```yml
networks:
  - db-net
```

**Sample compose file:**

```yml
networks:
  # DB Network
  db-net:

volumes:
  # DB Volume
  db-vol:

services:
  # Postgres Database Server
  db-server:
    image: postgres:18.4-alpine3.24
    volumes:
      - db-vol:/var/lib/postgresql
    networks:
      - db-net
    environment:
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=something

  # Frontend Server Using Adminer
  fe-server:
    image: adminer:5.5.0
    networks:
      - db-net
    ports:
      - 8080
    environment:
      - ADMINER_DEFAULT_SERVER=db-server
      - ADMINER_DESIGN=lucas-sandery
```

- **Service discovery:** refer to other services by their **service name** (e.g. `db-server`) instead of container name or IP — Compose automatically creates a network alias matching the service name.

### Compose Commands

- `docker compose config`
  - `-f` or `--file`: specify the manifest file name
  - `-p`: specify the project name
- `docker compose ps` — lists the project's containers.
- `docker compose up -d` — checks the desired state vs. the current state; creates what's missing, removes what shouldn't exist. If something in the compose file changed and you re-run it, only the changed resources (and their dependencies) are updated.
  - `--remove-orphans` — removes containers not defined in the compose file.
  - _Open question: if the hardcoded host `ports` value is removed/updated in the file, what happens on the next `up -d`? (Needs verification.)_
- `docker compose -p db-app-2 up -d` — run under a custom project name (same as the `-p` flag above).
- `docker compose ls` — shows how many projects are deployed.
- `docker compose down` — brings down containers and networks, but does **not** delete volumes by default. Use `-v` / `--volumes` to also remove volumes.
- `docker compose up` / `docker compose start` / `docker compose stop`
- `docker compose exec` — no need to add `-it`; use the **service name** instead of the container name.
- `prune` command — _(noted, not yet detailed)_.

# 26 July, 2026

## 17. Docker Images

### Indicators of Bad Images

- **Manual Intervention Required:** Images should provide a true one-shot setup. If you have to manually configure things after the container starts, the image is poorly designed.
- **Bloated Size:** Image size should be kept as minimal as possible to reduce pull times and attack surface.
- **Multiple Responsibilities:** Every app should run in a separate container following the single-responsibility principle.
- **Excessive Layers:** While layer planning is essential for caching, too many layers place unnecessary load on the Union File System (UFS).

### How to Plan and Create Your Own Image

- **Step 1:** Identify your PID 1 (the main process that keeps the container alive).
- **Step 2:** List all dependencies required to run **Step 1**.
- **Step 3:** Segregate the dependencies found in **Step 2** into two categories:
- **a) Things provided by you:** Source code, `.env` files, `package.json`, artifacts, etc.
- **b) Things NOT provided by you:** Runtime environments (Node.js, Java, Python), package managers (npm, pip), OS utilities.
- **Step 4:** Select a suitable base image based on the requirements in **Step 3b**.
- **Step 5:** Copy everything from **Step 3a** into the image's working directory.

### Building an OCI Image

Docker has transitioned to using `buildx` (BuildKit) as the default build engine, providing advanced features like multi-platform builds.

#### 1. The Build Context

To build an image, you need a **build context directory**. Everything required inside the image must reside within this directory. The entire context is sent to the Docker build engine at the start of the build process.

- **Tip:** Use a `.dockerignore` file to exclude unnecessary files (like local `.git` folders or `node_modules`) from the build context, speeding up the build process.

#### 2. The `Dockerfile`

Create a file named `Dockerfile` in the build context directory. (While you can name it anything, using `Dockerfile` eliminates the need for the `-f` flag during the build). Conventionally, if you have multiple, you might name them `AppName.Dockerfile`.

A Dockerfile consists of:

- **Comments:** Lines starting with `#` are ignored, as are blank lines.
- **Pragmas:** Directives at the top of the file (e.g., `# syntax=docker/dockerfile:1`) that change how the parser behaves.
- **Instructions:** Written in UPPERCASE, while their parameters are written in lowercase.

_Legacy Build Behavior Note:_ Older Docker engines built images by creating a container for every single instruction, committing it as a layer, and repeating. Thus, the total layers equaled the base image layers plus the number of instructions. Modern BuildKit optimizes this significantly.

#### 3. Core Instructions

- `FROM`
  - Specifies the base image to start from. It must be the first instruction (preceded only by optional pragmas or ARGs).
- `LABEL`
  - Adds arbitrary metadata as key-value pairs. Docker doesn't use these functionally, but tools like Docker Compose do.
  - _Convention:_ `LABEL maintainer="Tanuv Nair <tanuvnair@gmail.com>"`
- `COPY <source_path> <destination_path>`
  - Copies files or directories from the build context into the image's file system.
  - **Source path** is _always_ relative to the build context directory.
  - **Destination path** can be absolute or relative to the `WORKDIR`.
  - **Trailing Slashes Matter:** The trailing `/` dictates whether Docker treats the target as a file or a directory.
  - **Combinations & Behaviors:**
    - `COPY ./file.jar /app/` $\rightarrow$ Copies `file.jar` _into_ the `/app` directory.
    - `COPY ./file.jar /app` $\rightarrow$ Copies `file.jar` and renames it to `app` (if `/app` doesn't already exist as a directory).
    - `COPY ./dir-1 /app/subdir/` $\rightarrow$ Copies the _contents_ of `dir-1` into `/app/subdir/`.
    - `COPY *.html /app/` $\rightarrow$ When using wildcards, the destination _must_ be a directory (ending in `/`).
- `WORKDIR`
  - Sets the working directory for any subsequent `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, and `ADD` instructions. When a process runs, it executes from this directory by default.
- `CMD`
  - Provides the default command to execute when running a container from the image.
  - This is strictly metadata added to the image; it is _not_ executed during image build time.
  - _Best Practice (Exec Form):_ Write it as a JSON array where each element is in double quotes: `CMD ["java", "-jar", "fitnesse-standalone.jar"]`.
- `EXPOSE`
  - Documents which ports the container will listen on (e.g., `EXPOSE 80 8080`). This is purely metadata and does not actually publish the port to the host.
- `VOLUME`
  - Adds metadata defining a mount point (e.g., `VOLUME /app/FitNesseRoot`).
  - _Docker-specific behavior:_ If a container is run from an image with a `VOLUME` instruction and no explicit volume is mapped by the user, Docker automatically creates an anonymous volume.
  - Anonymous volumes cannot be renamed and are typically orphaned unless removed specifically when the container is deleted.

#### 4. Executing the Build

The modern command utilizes BuildKit:

```bash
docker image build -t fitnesse:1.0-alpine ./fitnesse/
```

_(If your file is named differently, use `-f ./Custom.Dockerfile`)_

**Multi-Platform Builds:** BuildKit (`buildx`) allows you to build images for architectures other than your host machine (e.g., building for `linux/arm64` on an `amd64` machine).

1. Check available builders: `docker buildx ls`
2. Create and bootstrap a new builder:

```bash
docker buildx create --driver docker:container --platform linux/amd64,linux/arm64 --name builder-1 --bootstrap --use
```

1. Build across platforms using the `--platform` flag:

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .
```

**Image Metadata Example (from `docker image inspect`):**

```json
"ExposedPorts": {
	"80/tcp": {}
},
"Cmd": [
	"java",
	"-jar",
	"fitnesse-standalone.jar"
],
"WorkingDir": "/app",
"Architecture": "amd64",
"Os": "linux"
```

### fitnesse.org Practical Example

- **Step 1:** Identify PID 1 $\rightarrow$ `java -jar fitnesse-standalone.jar`
- **Step 2:** List dependencies $\rightarrow$ Java Runtime, the JAR file itself.
- **Step 3:** Segregate dependencies:
  - **a) Provided by you:** `fitnesse-standalone.jar`
  - **b) NOT provided by you:** Java Runtime Environment (JRE)
- **Step 4:** Select base image $\rightarrow$ `eclipse-temurin:11.0.31_11-jre-ubi9-minimal`
- **Step 5:** Write the Dockerfile.

**`fitnesse.org` Dockerfile:**

```dockerfile
FROM eclipse-temurin:11.0.31_11-jre-ubi9-minimal
LABEL maintainer="Tanuv Nair <tanuvnair@gmail.com>"

WORKDIR /app
COPY ./fitnesse-standalone.jar .

CMD ["java", "-jar", "fitnesse-standalone.jar"]
EXPOSE 80

```

**Testing and Execution:** To find out where data is being modified or stored inside a running container, use:

```bash
docker container diff <container_name>
```

To run the container and explicitly mount a volume to the designated data directory:

```bash
docker container create \
  --name f1 \
  --publish 10800:80 \
  --mount type=volume,source=f1vol,target=/app/FitNesseRoot/ \
  fitnesse:1.0.0-minimal
```

## 18. Docker Hub

### Publishing a Docker Image

The build command used locally can be repurposed for publishing. By default, `buildx` sends the built image to your local Docker engine. To instruct the build tool to push the image directly to a remote registry, append the `--push` flag.

- **Authenticate with the registry:**

```bash
docker login <registry_name>
```

- **Build and push across multiple platforms:**

```bash
docker image build --push --platform=linux/amd64,linux/arm64 \
-f ./fitnesse/Dockerfile \
-t docker.io/algorisystanuv/fitnesse:1.0.0-minimal \
-t docker.io/algorisystanuv/fitnesse:latest \
./fitnesse/
```

- _Note:_ Pushing happens iteratively on a layer-by-layer basis. In this command, two tags (`1.0.0-minimal` and `latest`) are built and pushed simultaneously.

## 19. Qubefini

### Core Dockerfile Concepts

- `FROM SCRATCH` **vs. Minimal OS:**
- Using `FROM SCRATCH` means you start with an entirely empty file system (no OS).
- Using a minimal userland (like `alpine:3.24`) provides essential OS utilities and files, which is necessary if your app relies on system-level configurations like timezones.
- `ENTRYPOINT` **vs.** `CMD`**:**
- **ENTRYPOINT:** Defines the primary executable of the container. It is harder to override at runtime (requires the `--entrypoint` flag, e.g., `--entrypoint /bin/sh`).
- **CMD:** Provides default arguments to the `ENTRYPOINT`. It is easily overridden by appending commands to the end of `docker run`.
- _Interaction:_ When both are used, Docker concatenates them (`ENTRYPOINT` + `CMD` = Final Command).
- _Best Practice:_ If your image runs exactly one app, use `ENTRYPOINT`. If your image houses multiple apps or utilities and you need to specify which one to run dynamically, rely on `CMD`.
- `ENV`**:**
- Sets environment variables inside the image.
- _Security Rule:_ Only use `ENV` for default settings. **Never** put sensitive variables (like API keys or database passwords) in a Dockerfile. Inject those via Docker Compose or `.env` files at runtime.
- **Go Architecture Variables:**
- `GOOS` and `GOARCH` are environment variables used by the Go compiler to target specific operating systems and CPU architectures during multi-platform builds.
- **Creating a Multi-Arch Builder:**

```bash
docker buildx create --name builder1 --platform=linux/amd64,linux/arm64 --driver=docker-container --bootstrap --use
```

### Qubefini Architecture Files

**Assumption:** The root directory context for these files is `qubefini-v2`.

#### 1. Initialization Script

**File:** `./init/init.sh`

```bash
#!/bin/sh -e
echo "Running database initialization"
npx prisma migrate deploy
echo "Prisma migrations done"

echo "Running init-rbac"
./qubefini-seeder init-rbac
echo "Finished init-rbac"

echo "Running seed-features-permissions"
./qubefini-seeder seed-features-permissions
echo "Finished seed-features-permissions"
```

#### 2. Init Service Dockerfile

**File:** `./init/qubefini-init.Dockerfile`

```dockerfile
FROM golang:1.26.5-alpine3.24 AS go-builder
WORKDIR /app/init
COPY backend/go.* .
RUN go mod tidy -v
COPY backend .
RUN go build -o bin/qubefini-seeder ./cmd/seeder/

FROM node:24-alpine3.23 AS qubefini-init
LABEL org.opencontainers.image.authors="Tanuv Nair <tanuvnair@gmail.com>"
WORKDIR /app/init
RUN npm install -g prisma@6
COPY ./backend/prisma ./prisma
COPY --from=go-builder /app/init/bin/qubefini-seeder .
COPY ./init/init.sh .
RUN chmod +x ./init.sh
CMD ["./init.sh"]
```

#### 3. Backend API Dockerfile

**File:** `./backend/qubefini-be.Dockerfile`

```dockerfile
FROM golang:1.26.5-alpine3.24 AS go-builder
WORKDIR /app/backend
COPY backend .
RUN go build -o bin/qubefini-api ./cmd/api/

FROM alpine:3.24 AS qubefini-api-image
LABEL maintainer="Tanuv Nair <tanuvnair@gmail.com>"
WORKDIR /app
ENTRYPOINT ["/app/qubefini-api"]
CMD [""]

# Here we set the defaults for the environment variables
ENV PORT=8080 \
API_KEY_ALLOWED_IPS=127.0.0.1 \
SCHEDULER_SYNC_INTERVAL=30s \
TM1_CLIENT_TIMEOUT=300 \
ETL_DATA_ROOT_PATH="/app/qubefini/data" \
LOG_FILE_PATH="/tmp/" \
LOG_TARGET="console" \
LOG_SAMPLE_INTERVAL="100000" \
PASSWORD_RESET_TOKEN_EXPIRY_MINUTES=15 \
APP_NAME="qubefini" \
NOTIFY_EXPIRY_BEFORE_N_DAYS="1,2,3" \
DEFAULT_DEPENDENCY_DELAY_SECONDS="30" \
PAGINATION_GET_ALL_LIMIT="10000" \
PAGINATION_DEFAULT_LIMIT="10" \
PAGINATION_DEFAULT_PAGE="1" \
PAGINATION_MAX_LIMIT="100" \
MSSQL_VALIDATE_QUERY_TIMEOUT_SECONDS="30"

EXPOSE 8080
VOLUME /app/qubefini/data

COPY --from=go-builder /app/backend/bin/qubefini-api /app/qubefini-api
```

#### 4. Frontend Dockerfile

**File:** `./frontend/qubefini-fe.Dockerfile`

```dockerfile
FROM node:24-alpine3.23 AS node-builder
WORKDIR /app/frontend
COPY ./frontend/package*.json .
RUN npm ci
COPY ./frontend .
RUN npm run build

FROM node:24-alpine3.23 AS qubefini-frontend-image
LABEL org.opencontainers.image.authors="Tanuv Nair <tanuvnair@gmail.com>"
WORKDIR /app/frontend
COPY ./frontend/package*.json .
RUN npm ci
COPY --from=node-builder /app/frontend/build ./build
CMD ["npm", "run", "start"]
```

#### 5. Docker Compose Configuration

**File:** `./compose.yaml`

```yaml
networks:
  qf-net: null
volumes:
  redis-vol: null
  pg-vol: null
  api-scheduler-vol: null
services:
  postgres-db-server:
    image: 'postgres:18.4-alpine3.24'
    volumes:
      - 'pg-vol:/var/lib/postgresql'
    networks:
      - qf-net
    environment:
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=something
      - POSTGRES_DB=minietl
  redis-server:
    image: 'redis:8.8.1-alpine'
    command:
      - redis-server
      - /usr/local/etc/redis/redis.conf
    volumes:
      - >-
        /home/tanuv/projects/docker-training/qubefini/backend/redis-v2.conf:/usr/local/etc/redis/redis.conf
      - 'redis-vol:/data'
    networks:
      - qf-net
  qubefini-init:
    build:
      context: .
      dockerfile: ./init/qubefini-init.Dockerfile
    command:
      - /bin/sh
    tty: true
    stdin_open: true
    networks:
      - qf-net
    env_file:
      - .be-dev.env
  qubefini-fe:
    build:
      context: .
      dockerfile: ./frontend/qubefini-fe.Dockerfile
    networks:
      - qf-net
    ports:
      - 3000
    env_file:
      - .fe-dev.env
  qubefini-api:
    depends_on:
      - postgres-db-server
      - redis-server
    build:
      context: .
      dockerfile: ./backend/qubefini-be.Dockerfile
    volumes:
      - 'api-scheduler-vol:/app/qubefini/data'
    networks:
      - qf-net
    ports:
      - 8080
    env_file:
      - .be-dev.env
```
