>A **Kubernetes Deployment** is a resource that manages the lifecycle of a set of Pods by defining how they should be created, updated, and scaled. It ensures that the desired number of pod replicas are running at all times and allows for seamless updates and rollbacks. Deployments provide declarative updates, making it easy to manage changes to your application over time.

>A **ReplicaSet** in Kubernetes ensures that a specified number of identical pod replicas are running at all times. If a pod fails or is deleted, the ReplicaSet automatically creates a new one to maintain the desired count. While it manages pod replication, it is typically controlled by a Deployment for easier updates and management.

Create an nginx deployment. We'll use the image of nginx and we'll specify a port of 80. Specifying a port at this point will assist later on when using other Kubernetes functionality and is a recommendation when creating deployments -

`$ kubectl create deployment nginx --image=nginx --port 80`

Query the running deployments -

`$ kubectl get deployment`

When a deployment is created, a replicaset is also created for the deployment, note the replicaset id appended to the replicaset name -

`$ kubectl get replicaset`

And if we take a look at our pods, notice that the pod created consists of the name of the deployment, the replicaset and a unique identifier -

`$ kubectl get pods -o wide`

Let's look behind the scenes at containerd with nerdctl -

`$ nerdctl -n k8s.io ps -a | grep nginx`

Deployments can be easily scaled, execute the following command to increase the replica count to 2 and watch for changes with kubectl get pods, press CTRL-C to exit -

`$ kubectl scale deployment/nginx --replicas=2; watch kubectl get pods -o wide`

We now have multiple pods running and can see this also via containerd/nerdctl -

`$ nerdctl -n k8s.io ps -a | grep nginx`

What would happen though, if we lost our pause containers for the deployment?

Let's capture the pause containers for this deployment -

`$ nerdctl -n k8s.io ps -a | grep nginx | grep pause`

We'll capture these values into a variable and echo -

`$ CONTAINERS=$(nerdctl -n k8s.io ps -a | grep nginx | grep pause | awk {'print $1'} | tr "\n" " "); echo $CONTAINERS`

And let's stop these containers and watch the pods, in particular the IP addresses -

`$ nerdctl -n k8s.io stop $CONTAINERS; watch kubectl get pods -o wide`

The IP addresses changed for both pods ... whilst it is great that Kubernetes is automatically handling and repairing failed instances, we cant use this deployment for nginx effectively, if the IP addresses keep changing. In the next video, we'll look at Services to provide consistency!