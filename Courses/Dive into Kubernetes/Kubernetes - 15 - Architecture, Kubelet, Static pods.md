>The **kubelet** is a Kubernetes agent that runs on each node in the cluster and ensures that containers are running as expected. It receives instructions from the control plane (like the API server) and manages the lifecycle of pods assigned to the node. The kubelet monitors the state of these pods and reports back to the Kubernetes control plane. It is also responsible for monitoring the `/etc/kubernetes/manifests` directory and stopping containers when the files are removed. It's a primary 'node agent' that runs each node. It is often referred to as the 'worker'.

>A **DaemonSet** is a Kubernetes resource that ensures a specific pod runs on **all (or selected) nodes** in the cluster. It is commonly used for deploying system-level services like log collectors, monitoring agents, or network plugins. When a new node joins the cluster, the DaemonSet automatically schedules the pod on that node as well.

>A **normal pod** in Kubernetes is managed by the control plane and scheduled by the kube-scheduler to run on one of the available nodes. It is defined through Kubernetes resources like Deployments, ReplicaSets, or directly using a Pod manifest applied via `kubectl`. The lifecycle of a normal pod is fully managed by Kubernetes.

>A **static pod** is directly managed by the **kubelet** on a specific node, not through the Kubernetes API server. It is defined by placing a pod manifest file in a specific directory on the node (e.g., `/etc/kubernetes/manifests`). Static pods are mostly used for running core Kubernetes components like the API server, scheduler, or controller-manager, especially in single-node or control plane setups.

>A **normal pod** is managed by the Kubernetes control plane via the API server, while a **static pod** is managed directly by the kubelet and defined by a manifest file on the node (e.g., in `/etc/kubernetes/manifests`).

![[Pasted image 20250623085313.png]]

Let's re-review the components that are visible from a nerdctl/containerd viewpoint -

`$ nerdctl -n k8s.io ps -a | grep -v 'k8s.io/pause' | grep Up`

And we can see this, from a Kubernetes perspective -

`$ kubectl get all -A`
(command is used to list all resources in all namespaces in Kubernetes)

We also have the kubelet, running as an independent process under systemctl, we can see this via ps -

`$ ps -ef | grep kubelet | grep -v kube-apiserver | grep -v grep`

And we can also see this via systemctl -

`$ systemctl status kubelet`

Recall, the list of components running in kubernetes -

`$ kubectl get all -A`

We're going to capture the configurations for each, and start dismantling the cluster. Starting with kube-proxy, capture the running declaration to a yaml file -

`$ kubectl -n kube-system get daemonset.apps/kube-proxy -o yaml > /etc/kubernetes/resources/kube-proxy.yaml`

And now delete kube-proxy -

`$ kubectl -n kube-system delete daemonset.apps/kube-proxy`

Capture the coredns declaration as a yaml file -

`$ kubectl -n kube-system get deployment.apps/coredns -o yaml > /etc/kubernetes/resources/coredns.yaml`

And remove coredns -

`$ kubectl -n kube-system delete deployment.apps/coredns`

Kube-dns is a service, let's capture this -

`$ kubectl -n kube-system get service/kube-dns -o yaml > /etc/kubernetes/resources/kube-dns.yaml`

And delete kube-dns -

`$ kubectl -n kube-system delete service/kube-dns`

If we check the remaining components, there will be 4 static pods, we know they are static pods as they have the nodename in the name -

`$ kubectl get all -A`

Move the contents of /etc/kubernetes/manifests to /etc/kubernetes/resources -

`$ mv /etc/kubernetes/manifests/* /etc/kubernetes/resources`

When we moved those files, the static pods will have stopped, if we try and query the nodes it will now error -

`$ kubectl get nodes`

Our final component, the kubelet is still running, we'll stop this via systemctl -

`$ systemctl stop kubelet`

To fully clean up, we'll create 2 aliases to stop and remove anything that is running within containerd (via nerdctl), execute -

`$ alias nerdctl_stopall="nerdctl -n k8s.io ps -a | grep -v CONTAINER | cut -d ' ' -f 1 | xargs -i sh -c 'nerdctl -n k8s.io stop {} || true'"`

And also -

`$ alias nerdctl_rmall="nerdctl -n k8s.io ps -a | grep -v CONTAINER | cut -d ' ' -f 1 | xargs -i sh -c 'nerdctl -n k8s.io rm {} || true'"`

Now we can run those aliases -

`$ nerdctl_stopall`

`$ nerdctl_rmall`

And if we check, this is looking clean -

`$ nerdctl -n k8s.io ps -a`

The only component that we now have running is containerd -

`$ systemctl status containerd`

---

Let's restart the kubelet and for this, we'll use systemctl -

`$ systemctl start kubelet`

And we can now see the kubelet running, notice that it makes reference to a configuration file of /var/lib/kubelet/config.yaml -

`$ ps -ef | grep kubelet`

Take a look at this file and note the reference to the /etc/kubernetes/manifests directory -

`$ cat /var/lib/kubelet/config.yaml`

To recap, at present nothing is running from a nerdctl/containerd k8s.io perspective -

`$ nerdctl -n k8s.io ps -a`

Recall, earlier on when exploring YAML, we created a YAML declaration for a simple nginx pod and saved it in /etc/kubernetes/resources, take a look as a reminder -

`$ cat /etc/kubernetes/resources/nginx_pod.yaml`

Move this file to /etc/kubernetes/manifests, after doing so we'll watch the k8s.io namespace with nerdctl and we'll see that the kublet, started our pod based on a declaration file, press CTRL-C to exit -

`$ mv /etc/kubernetes/resources/nginx_pod.yaml /etc/kubernetes/manifests; watch nerdctl -n k8s.io ps -a`

Show the running containers from nerdctl -

`$ nerdctl -n k8s.io ps -a`

Capture the container id to a variable and echo the variable -

`$ CONTAINER=$(nerdctl -n k8s.io ps -a | grep -v pause | grep nginx | awk {'print $1'}); echo $CONTAINER`

And inspect the container as you would with docker, but with nerdctl, where required press q to exit or space to page through -

`$ nerdctl -n k8s.io inspect $CONTAINER | more`

Capture the IP address from the inspect output and echo -

`$ IP=$(nerdctl -n k8s.io inspect $CONTAINER | grep IPAddress | grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" | uniq); echo $IP`

And curl the IP address -

`$ curl $IP`

What happens if we stop this container with it under the control of the kubelet, you can repeat this command multiple times and watch the status, press CTRL-C to exit -

`$ nerdctl_stopall; watch nerdctl -n k8s.io ps -a`

Before we move on, let's clean up -

`$ mv /etc/kubernetes/manifests/nginx_pod.yaml /etc/kubernetes/resources`

`$ nerdctl_stopall; nerdctl_rmall`