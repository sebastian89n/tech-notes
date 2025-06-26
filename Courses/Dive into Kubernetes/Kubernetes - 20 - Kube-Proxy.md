>**kube-proxy** is a network component that runs on each node in a Kubernetes cluster. It manages network rules to direct traffic to the appropriate Pods, enabling communication between services and Pods. It supports different proxy modes (iptables, ipvs) to efficiently route traffic based on the service definition.
>
>Provides simple TCP, UDP and SCTP Stream Forwarding, round-robin TCP, UDP and SCTP Forwarding across a set of backends

Let's expose our deployment like we did previously -

`$ kubectl expose deployment/nginx --type=ClusterIP`

And if we check, we have our service visible -

`$ kubectl get service`

What happens though if we query this, firstly, capture the Service IP address and echo -

`$ SERVICE=$(kubectl get service | grep nginx | awk {'print $3'}) && echo $SERVICE`

And curl the service, at present, it wont be working as we would expect, press CTRL-C to cancel -

`$ curl $SERVICE`

Re-enable kube-proxy by applying the yaml declaration. Recall that this was a daemonset, not a static pod, we'll watch this start via nerdtl/containerd, when complete, press CTRL-C to exit -

`$ kubectl apply -f /etc/kubernetes/resources/kube-proxy.yaml; watch nerdctl -n k8s.io ps -a`

And we'll also check, via kubectl -

`$ kubectl get all -A`

Now with this running, we should now be able to access our service -

`$ curl $SERVICE`