>A **Dockerfile** is a text file that contains a set of instructions for building a Docker image. It defines the environment, dependencies, files, and commands needed to create a containerized application. Common instructions include `FROM` (base image), `COPY`, `RUN`, and `CMD`. When you run `docker build`, Docker reads the Dockerfile and produces an image that can be run as a container. This allows for consistent and repeatable application setups.

`FROM` – Sets the base image for building the new image.   
`RUN` – Executes a command during image build (e.g., installing software).
`CMD` – Sets the default command to run when the container starts.   
`LABEL` – Adds metadata to the image (e.g., author, version).
`EXPOSE` – Informs Docker which port the container will listen on.    
`ENV` – Sets environment variables in the image.    
`ADD` – Copies files/directories into the image, also supports URLs and auto-extracts archives.    
`COPY` – Copies files/directories from the build context into the image (simpler than `ADD`).
`ENTRYPOINT` – Configures a container to run as an executable.
`VOLUME` – Defines a mount point for external storage (a volume).
`WORKDIR` – Sets the working directory for following instructions.
`USER` – Specifies the user to run commands as inside the container.
`ARG` – Defines build-time variables (used only during build).
`ONBUILD` – Sets a trigger instruction to run when the image is used as a base for another build.
`STOPSIGNAL` – Sets the system call signal used to stop the container.
`HEALTHCHECK` – Defines how Docker should check if the container is still working.
`SHELL` – Changes the default shell used for `RUN` instructions.

---
`Dockerfile`
```bash
FROM alpine

#MAINTAINER Sebastian Nowak <>
LABEL org.opencontainers.image.authors="Sebastian Nowak"
LABEL org.opencontainers.image.description="Container image for https://github.com/abishekvashok/cmatrixe"

# Prepare apk for use and install git
RUN apk update
RUN apk add git

# Clone the repository, will succeed
RUN git clone https://github.com/spurin/cmatrix.git

WORKDIR /cmatrix

# Install autoconf
RUN apk add autoconf

# Install automake
RUN apk add automake

# Prepare compilation, will succeed, confirm with echo $?
RUN autoreconf -i

# Install compiler
RUN apk add alpine-sdk

# Install dependencies and create missing directories
RUN apk add ncurses-dev ncurses-static
RUN mkdir -p /usr/lib/kbd/consolefonts /usr/share/consolefonts

# Prepare configure, will succeed
RUN ./configure LDFLAGS="-static"

# Compile and view result
RUN make

# Run cmatrix
CMD ["./cmatrix"]
```

!! Notes:
Use `WORKDIR /path` to change directory _persistently_
`RUN cd dir` works only _for that line_, next command resets path.

`docker build . -t sebastian89n/cmatrix` -> builds `sebastian89n/cmatrix:latest` docker image

`docker run --rm -it sebastian89n/cmatrix`

```bash
sebastian89n@fedora:~/Dev/projects/diveintokubernetes/building_container/cmatrix$ docker images
REPOSITORY                                TAG                                                                           IMAGE ID       CREATED         SIZE
sebastian89n/cmatrix                      latest                                                                        0a35255e8f4d   3 minutes ago   454MB
```

---
#### **Improvements:**

- Minimalise amount of layers
- Minimalise size of the docker image using multi-stage docker build
- Run as a normal user instead of root
- Use `ENTRYPOINT` to allow to pass parameter to cmatrix command
```bash
# Build Container Image
FROM alpine as cmatrixbuilder

LABEL org.opencontainers.image.authors="Sebastian Nowak" \
      org.opencontainers.image.description="Container image for https://github.com/abishekvashok/cmatrixe"

WORKDIR cmatrix

RUN apk update --no-cache && \
 apk add git autoconf automake alpine-sdk ncurses-dev ncurses-static && \
 git clone https://github.com/spurin/cmatrix.git . && \
 mkdir -p /usr/lib/kbd/consolefonts /usr/share/consolefonts && \
 autoreconf -i && \
 ./configure LDFLAGS="-static" &&\
  make


# cmatrix container image
FROM alpine

LABEL org.opencontainers.image.authors="Sebastian Nowak" \
      org.opencontainers.image.description="Container image for https://github.com/abishekvashok/cmatrixe"

RUN apk update --no-cache && \ 
    apk add ncurses-terminfo-base && \
    adduser -g "Sebastian Nowak" -s /usr/sbin/nologin -D -H sebastian
COPY --from=cmatrixbuilder /cmatrix/cmatrix /cmatrix

USER sebastian
ENTRYPOINT ["./cmatrix"]
CMD ["-b"]
```

>Using `ENTRYPOINT ["./cmatrix"]` sets the main command to run, and `CMD ["-b"]` provides default arguments. This makes the container run `./cmatrix -b` by default, but you can override the `-b` with other options at runtime, like `docker run mycmatrix -s`. This way, `cmatrix` always runs, but you control its behavior with different flags.

```bash
sebastian89n@fedora:~/Dev/projects/diveintokubernetes/building_container/cmatrix$ docker images
REPOSITORY                                TAG                                                                           IMAGE ID       CREATED              SIZE
sebastian89n/cmatrix                      latest                                                                        848d6a41635b   About a minute ago   19.4MB
```

From 454MB to 19.4MB

`docker build . -t sebastian89n/cmatrix`
`docker images --digests` - check digests
`docker run --rm -it sebastian89n/cmatrix whoami`
`docker run --rm -it sebastian89n/cmatrix:latest`
`docker run --rm -it sebastian89n/cmatrix:latest --help`
`docker run --rm -it sebastian89n/cmatrix:latest -ab -u 2 -C magenta`
