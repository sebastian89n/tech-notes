>The **kube-apiserver** is the central management component of the Kubernetes control plane that exposes the Kubernetes API. It acts as the front end for all cluster interactions, handling requests from users, CLI tools, and internal components. All changes to the cluster state—such as creating pods or modifying configurations—go through the API server.

>The `.kube/config` file is a configuration file used by `kubectl` to connect to and manage Kubernetes clusters. It stores details such as cluster addresses, authentication credentials, and context settings that define which cluster and namespace `kubectl` should operate on by default. This file allows seamless switching between multiple clusters and user contexts without re-authenticating or reconfiguring each time.

![[Pasted image 20250624083647.png]]

Using our static pods manifests directory, let's move the kube-apiserver yaml file back to /etc/kubernetes/manifests, as we do so let's query the container runtime via nerdctl, press CTRL-C to exit -

`$ mv /etc/kubernetes/resources/kube-apiserver.yaml /etc/kubernetes/manifests; watch nerdctl -n k8s.io ps -a`

And with our kube-apiserver running, we can once again use kubectl as we did before -

`$ kubectl get nodes`

![[Pasted image 20250624083819.png]]