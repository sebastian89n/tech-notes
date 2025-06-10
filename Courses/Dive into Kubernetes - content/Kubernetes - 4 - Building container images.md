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
LABEL org.opencontainers.image.description="Container image for https://github.com/abishekvashok/cmatrix"
```


`docker build . -t sebastian89n/cmatrix` -> builds `sebastian89n/cmatrix:latest` docker image

