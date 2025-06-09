### 🐳 Introduction to Docker Containers

##### What Are Docker Containers?

**Docker containers** are a way to package and run applications in isolated environments. A container includes everything needed to run the application: code, runtime, libraries, dependencies, and configuration files.

Containers make applications **portable**, **consistent across environments**, and **lightweight** compared to traditional virtual machines.

---

### Docker Containers vs. Virtual Machines (VMs)

| Feature            | Docker Containers                            | Virtual Machines                          |
|--------------------|-----------------------------------------------|--------------------------------------------|
| **Isolation**      | Process-level (via Linux kernel features)     | Full hardware-level (via hypervisor)       |
| **Startup Time**   | Fast (milliseconds)                           | Slow (minutes)                             |
| **Size**           | Lightweight (MBs)                             | Heavyweight (GBs)                          |
| **Performance**    | Near-native                                   | Some overhead from the guest OS            |
| **Guest OS**       | Shares host OS kernel                         | Includes full guest OS                     |

#### Example

- **VM**: Runs an entire operating system (e.g., Ubuntu), including its own kernel.
- **Container**: Runs an app with only its required libraries and dependencies, using the host's kernel.

---

### How Do Containers Work Under the Hood?

Containers rely on Linux kernel features that allow them to behave like lightweight, isolated virtual systems without the need for a full OS.

#### 1. **Namespaces** – Isolation

Namespaces provide isolation between containers and the host or other containers. Each container gets its own isolated view of system resources:

- **PID namespace** – Isolates process IDs.
- **NET namespace** – Isolates network interfaces, IP addresses, routing tables.
- **UTS namespace** – Isolates host and domain names.
- **MNT namespace** – Isolates file system mount points.
- **IPC namespace** – Isolates inter-process communication (semaphores, message queues).
- **USER namespace** – Isolates user and group IDs.

This makes each container appear like it is running on its own OS.

#### 2. **Control Groups (cgroups)** – Resource Management

**Cgroups** (Control Groups) allow the kernel to limit, prioritize, and isolate resource usage per container:

- **CPU** – Limit how much CPU time a container can use.
- **Memory** – Cap memory usage and trigger out-of-memory behavior.
- **Block I/O** – Limit disk I/O throughput.
- **Network** – (via additional tools) manage bandwidth.

---

### Summary

Docker containers are:
- **Lightweight** – no need for a full OS.
- **Fast** – start up in milliseconds.
- **Isolated** – each container is sandboxed using kernel features.
- **Portable** – works the same in dev, staging, and production.

These advantages make Docker a core tool in modern development and deployment workflows, especially in microservices, CI/CD, and cloud-native environments.

---
`docker run -it ubuntu bash`
`apt update`
`apt install htop`

You can configure Resources in Settings->Resources in Docker desktop.
You can also enable Kubernetes in the Settings.

`kubectl get nodes`