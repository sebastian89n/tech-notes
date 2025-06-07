**Container image:** 
- portable: self contained bundle of software and dependencies
- versatile: allowing us to run software consistently across computer environments
- OCI compliant container image

**Container image vs container:** container image is a bundle of software, whereas a container is an instance of a software that runs

**Tag:** a label to distinguish a version of container image. `latest` is a default tag used if tag is not specified.

**Commands follow following syntax:**
`docker <verb>`
`docker <noun> <verb>`

E.g.
`docker pull wernight/funbox:latest`
`docker pull docker.io/wernight/funbox:latest`
`docker image pull docker.io/wernight/funbox:latest`

---
Docker images are built in **layers**.
- Each instruction in a `Dockerfile` (e.g., `RUN`, `COPY`, `ADD`) creates a **new read-only layer**.
- Layers are **cached**, **reused**, and **stacked** to form the final image.
- Layers help with efficient builds and image size optimization.

Layers being pulled for the image:
```bash
sebastian89n@fedora:~/Dev/tech-notes$ docker pull wernight/funbox:latest
latest: Pulling from wernight/funbox
9d0a3fb611bd: Pull complete 
5e95dc2c9b1a: Pull complete 
b6b150b5319b: Pull complete 
ec9fc64f0a37: Pull complete 
fdf894e782a2: Pull complete 
Digest: sha256:ca5c7db6c15c7628cad57dd43de8674b9dfa75303eafcd498d0bd1e673e78921
Status: Downloaded newer image for wernight/funbox:latest
docker.io/wernight/funbox:latest
```
(5 layers)
E.g.
https://hub.docker.com/layers/wernight/funbox/latest/images/sha256-ca5c7db6c15c7628cad57dd43de8674b9dfa75303eafcd498d0bd1e673e78921

0 B are bundled together with non-zero command and are considered a layer.

**Container layer:** a stack of individual layers on top of each other

**Union file system:** merges layers into single view from an end user perspective. When we run a container instance from a container image, we are using these layers as a file system, with a thin, accessible, writeable layer on the top.

Images contains a calculated hash of the image content, it's layers and metadata using sha256.

`docker pull docker.io/wernight/funbox@sha256:xyz`
`docker images`
`docker images --digests` - displays also checksum
`docker save wernight/funbox -o funbox.tar` - if extracted, can demonstrate layers

`docker history` command shows the **layer history of a Docker image** — that is, a list of the image's layers along with details about how each was created

---
**Notes:**
- Docker uses `containerd` container engine.
- `-rm` removes the container when it exists. 
- `-it` runs the container in interactive mode with terminal
- You can override default command by adding command to the end of docker run command.

>Containers that aren't run with the `--rm` flag are kept after they stop, which can be useful for debugging, inspecting logs, or saving data from the container's filesystem. You might want to keep a container if you're developing or troubleshooting and need to restart it or explore its state after it exits. However, over time, stopped containers can accumulate and take up disk space, so it's good practice to clean them up when they're no longer needed. You can remove stopped containers manually using `docker rm <container-id>` or clean up all of them at once with `docker container prune`.

**Commands:**
`docker version`
`docker pull wernight/funbox`
`docker --help`
`docker run --help`
`docker run -it wernight/funbox:latest`
`docker run -it wernight/funbox:latest bash`
`docker run -it wernight/funbox:latest nyancat`
`docker ps`
`docker ps -a`

---
## 🐳 Docker Images vs Containers: Stateless or Stateful?

### 📦 Images Are Stateless

- A **Docker image** is a **read-only** template (like a blueprint).
- Every time you run `docker run <image>`, it creates a **new, fresh container**.
- The image itself does **not retain any state or changes** between runs.

### 🧱 Containers Are Stateful (By Default)

- When you start a container, it gets its own **writable layer**.
- Any changes (like installing software, creating files, modifying config) happen in this layer.
- If you **stop** the container and then **start it again** (with `docker start <container-id>`), your changes are **still there**.
- But if you **remove the container** (`docker rm`), those changes are lost.

### 🔄 Running a New Container Always Starts Fresh

- Running the same image again (with `docker run`) creates a **completely new container**, with **no access to previous changes**.
- To **persist data independently of containers**, use **volumes** or `docker commit` to save the state as a new image.

### ✅ Summary

| Concept       | Description                                                     |
| ------------- | --------------------------------------------------------------- |
| **Image**     | Stateless, read-only template                                   |
| **Container** | Stateful; has a writable layer that keeps changes until deleted |
| **New run**   | Creates a fresh container unless you reuse the old one          |

### 🧹 Tip: Clean Up When Needed

- Use `docker rm <container>` to remove containers you no longer need.
- Use `docker container prune` to remove **all stopped containers**.