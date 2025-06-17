![[Pasted image 20250617083554.png]]

Check the running pods again but this time, hide the pause containers -

`$ nerdctl -n k8s.io ps -a | grep -v pause`

Let's focus on the nginx container that we started in Kubernetes and see what happens when we manipulate this directly in containerd. List the nginx containers -

`$ nerdctl -n k8s.io ps -a | grep nginx`

Exclude the pause container -

`$ nerdctl -n k8s.io ps -a | grep nginx | grep -v pause`

Capture the container ID and echo -

`$ CONTAINER=$(nerdctl -n k8s.io ps -a | grep nginx | grep -v pause | awk {'print $1'}); echo $CONTAINER`

Let's stop this container directly within containerd via nerdctl and watch how kubernetes responds, press CTRL-C to exit -

`$ nerdctl -n k8s.io stop $CONTAINER; watch kubectl get pods -o wide`

Let's see this again but this time, with the pause container for the nginx pod. List nginx and pause -

`$ nerdctl -n k8s.io ps -a | grep nginx`

Capture the container id of the pause container -

`$ CONTAINER=$(nerdctl -n k8s.io ps -a | grep nginx | grep pause | awk {'print $1'}); echo $CONTAINER`

And we'll stop the pause container and watch what happens with kubectl. Pay attention to the IP addresses, press CTRL-C to exit -

`$ nerdctl -n k8s.io stop $CONTAINER; watch kubectl get pods -o wide`

If we check containerd, we can see that kubernetes also recreated the nginx containers as the termination of the pause container would have also affected the nginx container -

`$ nerdctl -n k8s.io ps -a | grep nginx`

Before we go, we'll clean this up from the kubernetes side by deleting the pod -

`$ kubectl delete pod/nginx --grace-period=0`

---
**Challenge**: Customising the restart policy via CLI

Create pods with different restart policies via the CLI. Use the following examples to create pods with "Always," "OnFailure," and "Never" restart policies.

Task 1: Create a pod with the "Always" restart policy (default behaviour):

**Solution:**

`kubectl run nginx-always --image=nginx --restart=Always`

Task 2: Create a pod with the "OnFailure" restart policy:

**Solution:**

`kubectl run nginx-onfailure --image=nginx --restart=OnFailure`

Task 3: Create a pod with the "Never" restart policy:

**Solution:**

`kubectl run nginx-never --image=nginx --restart=Never`