>The **kube-scheduler** is a Kubernetes control plane component responsible for assigning unscheduled pods to suitable nodes based on resource availability and scheduling policies. It evaluates factors like CPU, memory, node taints, and affinity rules to determine the best placement. Once a decision is made, it updates the pod’s specification so the kubelet on the chosen node can start it.

**kube-scheduler**
- control plane process that assigns pods to nodes (kubelets)
- determines which nodes are valid according to constraints and available resources
- ranks each valid node and binds the prod to the most suitable node

In our current state, what happens if we run something ... Lets run a simple nginx pod and watch -

`$ kubectl run nginx --image=nginx; watch kubectl get pods -o wide`

It never scheduled as there is no scheduler running, let's start the kube-scheduler as a static pod -

`$ mv /etc/kubernetes/resources/kube-scheduler.yaml /etc/kubernetes/manifests`

And if watch the pod, this should be scheduled and running on our node, press CTRL-C to exit -

`$ watch kubectl get pods -o wide`

We'll cleanup the nginx pod -

`$ kubectl delete pod/nginx --grace-period=0`

![[Pasted image 20250624084526.png]]