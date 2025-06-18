>The **pause container** acts as the "parent" container for networking within a pod. It holds the shared network namespace and other containers join it, allowing them to communicate over `localhost`.
>
>A **pause container** is a minimal container used to hold shared Linux namespaces—like network, IPC, and PID—for a group of other containers. In Kubernetes, every pod includes a pause container that acts as the parent container for the pod's namespace, allowing all other containers in the pod to share the same environment. While it's commonly used in Kubernetes, the pause container concept can also be applied outside of Kubernetes to manually group containers into a shared
>
>IPC (Inter-Process Communication) is a mechanism that allows different processes to communicate and share data with each other. In Kubernetes, IPC can be shared between containers in the same pod using a shared IPC namespace. This enables containers to coordinate through shared memory or semaphores when needed.

![[Pasted image 20250616083753.png]]

**Run pause container:**
```bash
docker run --rm -d --ipc=shareable --name pause -p 8080:80 registry.k8s.io/pause:3.10
```

```bash
docker run --rm -d --name mysql -e MYSQL_DATABASE=exampledb -e MYSQL_USER=exampleuser -e MYSQL_PASSWORD=examplepass -e MYSQL_RANDOM_ROOT_PASSWORD=1 --net=container:pause --ipc=container:pause --pid=container:pause mysql:8
```

```bash
docker run --rm -d --name wordpress -e WORDPRESS_DB_HOST="127.0.0.1" -e WORDPRESS_DB_USER=exampleuser -e WORDPRESS_DB_PASSWORD=examplepass -e WORDPRESS_DB_NAME=exampledb --net=container:pause --ipc=container:pause --pid=container:pause wordpress:6.1.1
```

![[Pasted image 20250616090208.png]]

**Using shared memory:**
```bash
docker exec -it mysql sh
sh-5.1 echo hello > /dev/shm/test

docker exec -it wordpress sh
cat /dev/shm/test
hello
```

Running `ps -ef` on wordpress pod displays other processes as if they were running on the localhost:

```bash
# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
65535          1       0  0 06:41 ?        00:00:00 /pause
999          313       0  0 06:48 ?        00:00:03 mysqld
root         559       0  0 06:49 ?        00:00:00 apache2 -DFOREGROUND
www-data     615     559  0 06:49 ?        00:00:00 apache2 -DFOREGROUND
www-data     616     559  0 06:49 ?        00:00:00 apache2 -DFOREGROUND
www-data     619     559  0 06:49 ?        00:00:00 apache2 -DFOREGROUND
www-data     622     559  0 06:50 ?        00:00:00 apache2 -DFOREGROUND
www-data     624     559  0 06:50 ?        00:00:00 apache2 -DFOREGROUND
www-data     626     559  0 06:50 ?        00:00:00 apache2 -DFOREGROUND
www-data     627     559  0 06:50 ?        00:00:00 apache2 -DFOREGROUND
www-data     628     559  0 06:50 ?        00:00:00 apache2 -DFOREGROUND
www-data     629     559  0 06:50 ?        00:00:00 apache2 -DFOREGROUND
www-data     647     559  0 06:51 ?        00:00:00 apache2 -DFOREGROUND
root         656       0  0 06:53 pts/0    00:00:00 sh
root         665     656  0 06:57 pts/0    00:00:00 ps -ef
```

If we were to kill pause container process(1), it would kill all containers because they use shared PID namespace.