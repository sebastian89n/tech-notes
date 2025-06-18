>**Kubernetes DNS** is a built-in DNS service that automatically assigns DNS names to Kubernetes services and pods, making it easy to discover and communicate with them. It allows applications to use consistent names (like `my-service.my-namespace.svc.cluster.local`) instead of IP addresses. This simplifies service discovery and communication within the cluster.

![[Pasted image 20250618084809.png]]

![[Pasted image 20250618084919.png]]

Let's run a sample hello-world app provided by Google (Course Update: Changed to spurin/hello-app:1.0 to support amd64 and arm/apple silicon architectures), this app by default runs on port 8080 -

`$ kubectl run hello-world --image=spurin/hello-app:1.0 --port 8080`

We can query our pods -

`$ kubectl get pods -o wide`

Let's capture that IP address -

`$ HELLO_IP=$(kubectl get pods -o wide | grep hello-world | awk {'print $6'}); echo $HELLO_IP`

And we'll curl it, including port 8080 in the curl command -

`$ curl $HELLO_IP:8080`

This is a viewpoint from our cluster node but what is the view like, from within a pod, running in Kubernetes?

Let's run a curl container that we can use for this perspective -

`$ kubectl run curl --image=curlimages/curl --restart=Never -- sh -c "sleep infinity"`

And if we check, we can see that pod running -

`$ kubectl get pods`

We can if we desire, execute commands on this pod so for example, we could query example.com which is a useful html starter template example available on the internet -

`$ kubectl exec -it curl -- sh -c "curl example.com"`

That image, even though it’s for curl, also has nslookup so we could also use it to run -

`$ kubectl exec -it curl -- sh -c "nslookup example.com"`

Let's create friendly shell functions so this these are easier to use, for curl as kcurl -

`$ kcurl() { kubectl exec -it curl -- sh -c "curl $1"; }`

And for nslookup as knslookup -

`$ knslookup() { kubectl exec -it curl -- sh -c "nslookup $1"; }`

Now we can easily get a pod viewpoint of curl with -

`$ kcurl example.com`

And a pod viewpoint of nslookup with -

`$ knslookup example.com`

We can query the nginx service from a pod as follows -

`$ kcurl nginx.default.svc.cluster.local`

And we can also perform an nslookup from the pod against the service DNS name -

`$ knslookup nginx.default.svc.cluster.local`

We can see that the DNS resolution relates to our Service IP -

`$ kubectl get service`

For our pods, the DNS format is different, recall our IP address for the hello-world app -

`$ echo $HELLO_IP`

Let's change the dots to dashes -

`$ HELLO_IP_DASHES=$(echo $HELLO_IP | sed 's/\./-/g'); echo $HELLO_IP_DASHES`

And the format for a pod ip would be -

`$ echo $HELLO_IP_DASHES.default.pod.cluster.local`

Let's try curling that name via our pod with the addition of :8080 for the port -

`$ kcurl $HELLO_IP_DASHES.default.pod.cluster.local:8080`

And we can also do an nslookup using an alias -

`$ knslookup $HELLO_IP_DASHES.default.pod.cluster.local`

Which relates to the pod ip -

`$ kubectl get pods -o wide`

With this complete we can now clean up these components, remove the deployment -

`$ kubectl delete deployment/nginx`

Remove the service -

`$ kubectl delete service/nginx`

And the pods, we'll remove both the hello-world and curl pod at the same time -

`$ kubectl delete --grace-period=0 pod/hello-world pod/curl`