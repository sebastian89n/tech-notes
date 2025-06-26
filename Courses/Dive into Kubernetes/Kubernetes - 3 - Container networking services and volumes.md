>**Networking** in Docker allows containers to communicate with each other and with the outside world through virtual networks, such as bridge, host, and overlay networks. **Services**, commonly used with Docker Swarm, enable the deployment and scaling of multi-container applications across a cluster. **Volumes** are used for persistent storage, allowing containers to store and share data independently of their lifecycle, ensuring data isn’t lost when a container stops or is removed.

#### Exposing ports:
`docker run -d --rm nginx`
`--rm` -> remove container upon exiting
`-d` -> detach

Runs on `80/tcp`

`docker stop PID`

In Docker, the `-P, --publish-all` flag in `docker run` tells Docker to **publish all exposed ports** in the image to **random ports on the host**. These ports are defined in the Dockerfile using the `EXPOSE` instruction.

`docker run -d --rm -P nginx`
`docker ps` ->`0.0.0.0:55000->80/tcp`

For more control, you can use `-p` instead, like `-p 8080:80`, which maps port 80 inside the container to port 8080 on the host.

`docker run -d --rm -p 12345:80 nginx` -> expose on port 12345
`docker exec -it 7e1d44498aee bash` - run terminal on the container

```bash
root@7e1d44498aee:/# find / -name "index.html" 2>/dev/null 
/usr/share/nginx/html/index.html
```

#### Volumes:
In Docker, the `-v` flag in `docker run` is used to **mount a volume**—a way to persist data outside the container's filesystem. Volumes let containers read/write files from the host or shared storage.

It can be used to:
- Persist data (e.g., databases)
- Share data between containers
- Inject config files

`docker run -v /home/sebastian89n/Dev/projects/diveintokubernetes/my_web_page:/usr/share/nginx/html -d --rm -p 12345:80 nginx` - expose my_web_page folder from local machine as volume in nginx directory on the image.