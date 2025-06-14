Containerd is pre-installed in the lab environment, check this is running with systemctl -

`$ systemctl status containerd`

The installation of containerd also results in the installation of runc, check that this is available in a system path -

`$ which runc`

Containerd is bundled with an unsupported _debug administrative tool_ known as **ctr**, check that this is available in your path -

`$ which ctr`

We're going to see how you could run the equivalent docker command of **_docker run --rm -it ubuntu bash_** through ctr

Unlike docker which will automatically pull an image if it doesn't exist, ctr requires this to be actioned manually -

`$ ctr images pull docker.io/library/ubuntu:latest`

Firstly, we'll run a container: **-t** to create a tty, **-d** to detach, **docker.io/library/ubuntu:latest** for the container image and a name of **quirky** -

`$ ctr run -t -d docker.io/library/ubuntu:latest quirky`

We can verify that the container is running via ctr, note that runc is used the low level container runtime -

`$ ctr container list`

And we can see the task (process) that is running, this is the process on our running system for this container -

`$ ctr task list`

If we check our running processes we can see containerd running a container and a process of bash running within -

`$ ps -ef --forest`

And now we can attach, to the running container -

`$ ctr task attach quirky`

Whilst in the container, if we execute a **ps** we can see our running process of bash -

`$ ps`

This container has no external network connectivity, if you try an apt update, this will fail -

`$ apt update`

When you've finished using this container, you can exit -

`$ exit`

As we attached to the running container task which was our bash shell and instructed bash to exit, our task will no longer exist, lets check, note, we can use ls instead of list as a shortcut -

`$ ctr task ls`

Although no tasks are running, our container will still exist -

`$ ctr container list`

We will now delete the container -

`$ ctr container delete quirky`

Whilst ctr is useful, it's not friendly to use like the docker-cli, let's install nerdctl to provide a docker-cli like environment for containerd. Extract the nerdctl binary and place in /usr/local/bin -

`$ tar -xzvf /resources/nerdctl-1.1.0-linux.tar.gz --directory /usr/local/bin `nerdctl

Nerdctl requires CNI Plugins, the same ones used by Kubernetes CNI solutions. Unpack the plugins to /opt/cni/bin -

`$ tar -xzvf /resources/cni-plugins-linux-v1.2.0.tgz -C /opt/cni/bin`

Before we run nerdctl, verify that the standard CNI configuration directory of /etc/cni/net.d is empty -

`$ ls -altr /etc/cni/net.d`

When nerdctl runs for the first time it will bootstrap a CNI network for it's use. Let's try this out, we'll run a docker like command but we'll substitute **nerdctl** for **docker**, we'll use the following options -

>CNI (Container Network Interface) in Docker refers to a **standardized plugin system** used to manage networking for containers. It defines how container runtimes (like Docker or containerd) set up network interfaces, assign IPs, and connect containers to networks. While Docker traditionally used its own networking model, **CNI is more common in Kubernetes and containerd**, enabling flexible and pluggable networking setups via third-party CNI plugins (like Calico, Flannel, etc.).

- **`ctr`** is a low-level CLI for containerd and **does not support CNI networking out of the box** — containers started with `ctr` usually have no network unless explicitly configured with extra setup.

- **`nerdctl`** is a Docker-compatible CLI for containerd that **supports CNI networking by default** (when containerd is properly configured), so containers get network connectivity similar to Docker.

In short:  
 `ctr` = minimal, no default networking  
 `nerdctl` = user-friendly, supports CNI like Docker

| Feature         | Docker           | nerdctl       | ctr           |
| --------------- | ---------------- | ------------- | ------------- |
| CLI Level       | High-level       | Mid-level     | Low-level     |
| Networking      | Built-in         | CNI supported | Manual setup  |
| Volumes         | Supported        | Supported     | Not supported |
| Image Build     | Supported        | Supported     | Not supported |
| Compose         | Supported        | Supported     | Not supported |
| Uses containerd | Yes (internally) | Yes           | Yes           |

**run**: to run a container  
**--rm**: to cleanup up  
**-it**: for interactive  
**ubuntu**: the image we shall use  
**bash**: the command we will run

`$ nerdctl run --rm -it ubuntu bash`

Whilst in the container, we could run a command like we did when in the ctr container, for example -

`$ ps -ef`

Let's check our network connectivity by running an apt update -

`$ apt update`

We will install iproute2 to provide the ip command -

`$ apt install -y iproute2`

And we can also see that this container has an ip address -

`$ ip addr`

Let's exit the container -

`$ exit`

When we executed nerdctl, it boostraped a CNI configuration file that can be viewed here -

`$ cat /etc/cni/net.d/nerdctl-bridge.conflist`

We can see the nerdctl0 bridge interface that relates to this in ip addr, notice the 10.4.0.1/24 subnet that relates to the entry in the configuration file -

`$ ip addr`

As Kubernetes makes use of the same directory to search for a CNI configuration file, let's move that file out of the way -

`$ mv /etc/cni/net.d/nerdctl-bridge.conflist /resources`

In the next video we'll setup and install Kubernetes using kubeadm.

![[Pasted image 20250614105956.png]]