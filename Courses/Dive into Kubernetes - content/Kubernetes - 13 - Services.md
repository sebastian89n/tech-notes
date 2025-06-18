>A **Kubernetes Service** is an abstraction that defines a stable way to access a set of pods, regardless of their individual IP addresses or restarts. It enables communication within the cluster (and optionally from outside) by providing a consistent DNS name and load balancing across the matching pods. Services decouple the client from the underlying dynamic pod infrastructure.

>A **NodePort** is a type of Kubernetes Service that exposes a pod to external traffic by opening a specific port on all nodes in the cluster. This allows users to access the service using `<NodeIP>:<NodePort>` from outside the cluster. It’s often used for simple external access in development or testing environments.

>A **LoadBalancer** is a type of Kubernetes Service that exposes applications to external traffic using a cloud provider’s external load balancer. It automatically provisions a publicly accessible IP address and routes traffic to the appropriate NodePort services behind the scenes. This is commonly used in production environments for handling internet-facing traffic.

>An **ExternalName** is a type of Kubernetes Service that maps a service name to an external DNS name. Instead of routing traffic to internal pods, it returns a CNAME record pointing to the external address. This is useful for integrating external services into the Kubernetes DNS system without using kube-proxy or creating actual endpoints.

Let's create a clusterIP for our deployment, as we specified a port when creating the deployment, we can use the expose command with kubectl to expose this as a service -

`$ kubectl expose deployment/nginx --type=ClusterIP`

Query the available services -

`$ kubectl get service`

Capture the IP address of our service and echo -

`$ SERVICE=$(kubectl get service | grep nginx | awk {'print $3'}); echo $SERVICE`

And let's curl that service -

`$ curl $SERVICE`

It now doesn't matter if the containers or it's pause components fails and restarts, the service will update accordingly, let's view the pause containers -

`$ nerdctl -n k8s.io ps -a | grep nginx | grep Up | grep pause`

Capture the pause container id's -

`$ CONTAINERS=$(nerdctl -n k8s.io ps -a | grep nginx | grep Up | grep pause | awk {'print $1'} | tr "\n" " "); echo $CONTAINERS`

And we'll stop them both so new IP addresses are assigned, we'll stop and watch in one command, press CTRL-C to exit -

`$ nerdctl -n k8s.io stop $CONTAINERS; watch kubectl get pods -o wide`

Even though the pods IP addresses have changed, the curl to the service IP still works as expected as Kubernetes updates automatically -

`$ curl $SERVICE`

![[Pasted image 20250618083115.png]]

![[Pasted image 20250618083241.png]]

![[Pasted image 20250618083356.png]]

![[Pasted image 20250618083524.png]]