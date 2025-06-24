>The **kube-controller-manager** is a Kubernetes component responsible for running controllers that regulate the state of the cluster. Each controller watches the Kubernetes API for changes and works to move the current state toward the desired state—for example, ensuring the right number of pods are running for a ReplicaSet. It includes controllers like the node controller, replication controller, and endpoint controller, among others.

**controller-manager:**
- Daemon that embeds the core control loops shipped with Kubernetes
- A control loop is non-termnating loop that regulates the state of the system
- Monitors the state of the cluster through the kube-apiserver and makes changes according to change the state to the desired state

What happens if we attempt to deploy something more complex than a simple pod, run a deployment and watch the deployment status, press CTRL-C to exit -

`$ kubectl create deployment nginx --image=nginx --port=80; watch kubectl describe deployment/nginx`

It never deploys as deployments are more complex and require additional controllers, start the controller-manager as a static pod and watch the deployment status, be patient, this can take a moment to start, press CTRL-C to exit -

`$ mv /etc/kubernetes/resources/kube-controller-manager.yaml /etc/kubernetes/manifests; watch kubectl describe deployment/nginx`

![[Pasted image 20250624085311.png]]