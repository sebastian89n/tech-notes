>A **Kubernetes Pod** is the smallest deployable unit in Kubernetes and represents one or more containers that share the same network namespace and storage. Containers in a pod can communicate with each other using `localhost` and share resources like volumes. Pods are typically used to run tightly coupled application components that need to work together.

Run an nginx pod with the nginx image, watch kubectl get pods with the wide option (to show IP addresses), press CTRL-C to exit -

`$ kubectl run nginx --image=nginx; watch kubectl get pods -o wide`

Let's check that IP address again -

`$ kubectl get pods -o wide | grep nginx`

And let's capture the IP address to a variable called IP (and echo it to verify) -

`$ IP=$(kubectl get pods -o wide | grep nginx | awk {'print $6'}); echo $IP`

With this information, we can use curl to query the webpage of the running container -

`$ curl $IP`

Let's see how this looks from the containerd side with nerdctl. Recall, Kubernetes is using containerd as it's container engine so these should be visible using nerdctl. Firstly lets check the running containers -

`$ nerdctl ps -a`

Let's check what namespaces are available within containerd -

`$ nerdctl namespace ls`

And let's query for running containers again but this time, specify the k8s.io namespace -

`$ nerdctl -n k8s.io ps -a`