>**Container runtimes** are the software that run containers on Kubernetes nodes. Kubernetes interacts with them through the Container Runtime Interface (CRI). Common runtimes include containerd (default), CRI-O, and formerly Docker, which is now deprecated. These runtimes handle pulling images, starting containers, and managing their lifecycle.
## Docker:

**Docker Container Runtime:** for running and managing Containers (also referred to as the Docker Engine)

**Docker Images:** Name frequently used for Container Images. Everything you need to run software - Code, System Tools, Libraries etc.

**Docker Hub:** A Container Registry for hosting Container Images

**Docker Desktop:** A desktop offering from Docker

Docker can:
- run and manage container
- build container images

In the past Kubernetes would interact with Docker using "Dockershim".
Kubernetes <-> Dockershim <-> Docker

## Kubernetes history:

**So what is the dockershim, and why is it going away?**

>In the early days of Kubernetes, we only supported one container runtime. That runtime was Docker Engine. Back then, there weren’t really a lot of other options out there and Docker was the dominant tool for working with containers, so this was not a controversial choice. Eventually, we started adding more container runtimes, like rkt and hypernetes, and it became clear that Kubernetes users want a choice of runtimes working best for them. So Kubernetes needed a way to allow cluster operators the flexibility to use whatever runtime they choose.

`containerd`: a high-level container runtime that manages the entire lifecycle of containers, from image storage and distribution to container creation, start, stop and deletion (similar to docker)

`runc`:  a low-level container runtime engine used by containerd and other popular Container Runtimes. It is responsible for setting up a container's namespaces, cgroups and other low-level Linux primitives that are required for containers to run

`containerd` uses `runc`. Docker donated containerd to the `Cloud Native Computing Foundation` and `runc` was donated to `Open Container Initiative`.

In v1.24 Kubernetes stopped using Dockershim.
These days Kubernetes directly communicates with `containerd`(which uses `runc` underneath).
## Understanding Kubernetes, containerd, runc, and Docker

### 1. What Docker really is
Docker is a **complete platform** for developing, packaging, and running containers. It includes:

- **Docker CLI** – the command-line tool (`docker run`, `docker build`, etc.)
- **Docker Engine** – the daemon that handles container operations
- **containerd** – the core container runtime (used internally by Docker Engine)
- **runc** – the low-level tool that actually starts and stops containers

> So when you run `docker run`, the CLI talks to the Docker Engine, which uses **containerd**, which in turn uses **runc** to manage containers.

---

### 2. What Kubernetes needs
Kubernetes **doesn't need the full Docker platform**. It only needs a container runtime that can:

- Pull images
- Start containers
- Stop containers
- Manage their lifecycle

That’s exactly what **containerd** does — and it uses **runc** under the hood.

---

### 3. Why Docker was removed (Dockershim deprecation)
Originally, Kubernetes only supported Docker as the container runtime. However:
- Docker was **not** built to support Kubernetes directly.
- Kubernetes used an adapter called **Dockershim** to talk to Docker.

As **containerd** and **CRI-O** became mature and CRI-compliant, Kubernetes deprecated Dockershim in v1.20 and removed it in v1.24. This allowed Kubernetes to talk directly to modern runtimes without extra complexity.

---

### 4. What is `cri-containerd`?
`cri-containerd` was a **plugin project** that allowed containerd to support Kubernetes via the **Container Runtime Interface (CRI)**.

- It acted as a bridge between Kubernetes and containerd before containerd had native CRI support.
- Since **containerd v1.1**, CRI support was integrated directly into containerd, so `cri-containerd` was merged and is no longer a separate component.

> Today, containerd natively supports CRI, so you don't need `cri-containerd` as a separate layer.

---
### 5. Stack Comparison

**With Docker:**
Docker CLI → Docker Engine → containerd → runc

**With Kubernetes (today):**
Kubelet → containerd → runc


Kubernetes skips Docker and uses containerd directly, which is lighter and purpose-built.

---

### 6. Key Differences

| Feature                | Docker               | containerd + runc              |
| ---------------------- | -------------------- | ------------------------------ |
| Full platform          | Yes                  | No                             |
| Has CLI                | Yes (`docker`)       | No (can use `ctr` or `crictl`) |
| CRI support            | No (used Dockershim) | Yes (native support)           |
| Kubernetes recommended | No                   | Yes                            |

---

### Summary
- Docker is a full-featured container platform.
- containerd is a lightweight container runtime that Kubernetes uses directly.
- runc is the low-level tool used by both to actually create and run containers.
- Kubernetes deprecated Docker support in favor of containerd and CRI-O for simplicity and better integration.
---
## Popular Container Runtimes

- **containerd**  
  Lightweight runtime used by Kubernetes and Docker. Stable, fast, and extensible.

- **CRI-O**  
  Kubernetes-native runtime focused on simplicity and security. Smaller and more secure than Docker.

- **Docker Engine (dockerd)**  
  Full-featured runtime with built-in CLI and daemon. Easy for developers but heavyweight for production.

- **runc**  
  Low-level runtime that actually starts containers. Used by containerd and Docker; not meant for direct use.

- **gVisor**  
  User-space kernel for strong isolation. Great for security but slower than standard runtimes.

- **Kata Containers**  
  Lightweight VMs offering strong isolation like VMs with container speed. Slower startup and higher resource use.

- **wasmtime / runwasi**  
  WebAssembly-based runtimes. Emerging tech focused on portability and safety, but limited tooling.
