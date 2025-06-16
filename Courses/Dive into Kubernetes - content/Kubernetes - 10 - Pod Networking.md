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

YAML is an important component that you'll use in your everyday Kubernetes usage, in the next video we'll explore how to quickly capture the YAML declaration from existing commands.